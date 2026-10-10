# 4. System Architecture and Tech Flow

AquaGuard is a flood-response coordination platform that connects three actors —
**citizens** who raise SOS requests, **rescuers** who accept and carry out missions
in teams, and **admins** who oversee and dispatch. This chapter describes how the
system is put together: the overall topology, the backend and its data model, the
real-time coordination engine that streams live GPS between citizens and rescuers,
the web and mobile clients, and the security posture that protects it all.

---

## 4.1 Overall architecture

AquaGuard follows a classic **three-tier architecture** (clients → stateless API →
data stores), extended with a **dedicated real-time channel** for live rescue
tracking. The system is deliberately split across managed, independently
scalable services so that no single component is a bottleneck or a single point of
failure.

```
                         ┌──────────────────────────────┐
        Web (Vercel) ────┤                              │
   aquaguard.vn / SPA    │   Backend API + WebSocket    │
                         │   Express 4 + ws (Render)     │
   Mobile (native) ──────┤   aquaguard-api.onrender.com  │
   iOS / Flutter client  │                              │
                         └───────┬──────────────┬───────┘
                                 │              │
                    ┌────────────▼───┐   ┌──────▼──────────┐
                    │  PostgreSQL     │   │  Redis (Upstash) │
                    │  (Neon)         │   │  live GPS hot    │
                    │  system of      │   │  store (60s TTL) │
                    │  record         │   └─────────────────┘
                    └─────────────────┘
                             │
            ┌────────────────┼────────────────┐
     Resend (email)   Cloudinary (images)   Twilio (SMS OTP)
```

**Deployment topology.** Each tier is hosted on a purpose-built managed service:

| Tier | Component | Hosting | Notes |
| --- | --- | --- | --- |
| Presentation | React SPA | **Vercel** (`aquaguard.vn`) | Auto-deploys on push to `main`. |
| Presentation | Native mobile app | App distribution | Consumes the same REST + WebSocket API. |
| Application | Express REST API + `ws` WebSocket server | **Render** | Single Node process; HTTP and WS share one port. |
| Data (record) | PostgreSQL | **Neon** (serverless) | System of record; SSL auto-enabled for cloud hosts. |
| Data (hot) | Redis | **Upstash** (serverless) | Live-location hot store; optional, degrades to Postgres. |
| Integrations | Email / Images / SMS | Resend / Cloudinary / Twilio | All called from the backend only. |

**Design principles that shape the architecture:**

- **Stateless application tier.** The API keeps no session state in memory — every
  request carries a self-contained JWT. This lets Render restart or scale the
  process freely. The *only* in-process state is the set of live WebSocket
  connections and tracking rooms, which are intentionally ephemeral (see §4.3.4).
- **Separation of hot and durable data.** Continuously changing GPS fixes are
  written to a Redis hot store with a short TTL, while PostgreSQL keeps only the
  durable milestones. This keeps the write-heavy tracking path off the relational
  database (see §4.3.2).
- **Graceful degradation.** Redis is optional: if `REDIS_URL` is unset or the
  connection drops, `isRedisReady()` returns `false` and live tracking transparently
  falls back to the PostgreSQL `user_locations` table. Similarly, email is always
  fire-and-forget so a mail-provider outage can never break registration or a rescue
  action.
- **Secrets live in the environment, not the repo.** Every credential
  (`DATABASE_URL`, `JWT_SECRET`, `RESEND_API_KEY`, `CLOUDINARY_URL`, `TWILIO_*`,
  `REDIS_URL`) is injected as a platform environment variable on Render/Vercel and
  read via `process.env` / `import.meta.env`.

---

## 4.2 Backend and API layer

The backend is a **Node.js 20 + Express 4** application (CommonJS) with a single
entry point, [`backend/index.js`](../../backend/index.js). It wires up CORS, JSON
parsing, per-route rate limiters, the REST routers, a health-check endpoint, and —
on the *same* HTTP server — the native WebSocket server.

**Architectural style.** The API is deliberately lightweight and has **no ORM and
no service/model layer**. Business logic lives inline in the route handlers under
[`backend/routes/`](../../backend/routes/), and all persistence is **raw parameterised
SQL** against a single shared `pg` connection pool exported by
[`backend/db.js`](../../backend/db.js). Two shared singletons are imported everywhere
and never re-instantiated: the Postgres pool (`db.js`) and the Redis client
(`redisClient.js`).

**REST surface.** Routers are mounted by domain:

| Mount point | Router | Responsibility |
| --- | --- | --- |
| `/api/auth` | `auth.js` | Registration, login, profile, team/group management, roles. |
| `/api/sos` | `sos.js` | SOS request lifecycle (create → assign → accept → resolve). |
| `/api/family` | `family.js` | Family connections and safety-status checks. |
| `/api/locations` | `locations.js` | Live-location reads (`/live`, `/live/:userId`, `/nearby`). |
| `/api/analytics` | `analytics.js` | Aggregate dashboards for admins. |
| `/api/notifications` | `notifications.js` | In-app notifications and admin broadcasts. |
| `/api/health` | — | Liveness probe (kept awake by an UptimeRobot ping). |

**Cross-cutting concerns** are handled by small middleware:

- **CORS** — a hardcoded allowlist of frontend origins in `index.js`, plus a
  regex that permits LAN IPs in development (for mobile devices on the same
  network) and requests with no `Origin` header (native mobile apps, curl,
  Postman).
- **Rate limiting** — an in-memory, per-IP limiter
  ([`middleware/rateLimit.js`](../../backend/middleware/rateLimit.js)) applied
  selectively to sensitive endpoints: login (10 / 15 min), registration
  (5 / 15 min), forgot-password (5 / 15 min) and admin broadcasts (20 / min).
- **A consistent response envelope** — handlers return
  `{ success: boolean, data | message }`, so every client can branch on a single
  field.

### 4.2.1 Data model

The canonical schema lives in
[`infrastructure/database/init_db.sql`](../../infrastructure/database/init_db.sql),
which the Postgres container auto-runs on first boot; incremental changes are raw
SQL files under `backend/migrations/`. PostgreSQL is the **system of record** for
everything except live GPS.

The core entities and their relationships:

```
users ──1:N── rescue_requests ──1:N── rescue_request_logs   (status audit trail)
  │                  │
  │                  └── assigned_to ──► users (rescuer)
  │                  └── assigned_group_id ──► rescue_groups
  │
  ├──1:N── rescue_group_members ──N:1── rescue_groups ──1:N── rescue_group_invites
  ├──1:N── family_connections (requester ↔ receiver)
  ├──1:1── user_locations         (durable last-known GPS)
  └──1:N── notifications
```

**`users`** — one row per account, keyed by a unique `phone_number` (phone is the
primary identity; `email` is nullable and defaults to `''`). Holds the
`password_hash`, the `role` (`citizen` | `rescuer` | `admin`, enforced by a `CHECK`
constraint), profile fields, an `address`, and a `safety_status`
(`unknown` | `safe` | `danger` | `injured`) used by the family-check feature.
Live GPS is deliberately **not** stored here.

**`rescue_requests`** — the heart of the system. Each SOS row records who raised it
(`user_id`), the free-text `location` and `description`, `latitude`/`longitude`, an
array of Cloudinary image URLs, an `urgency`
(`low` | `medium` | `high` | `critical`), and the `status` that drives the state
machine (§4.3.3). Assignment fields (`assigned_to`, `assigned_group_id`,
`accepted_mode`), the rescuer's last GPS fix, cancellation metadata, and timeline
milestones (`assigned_at`, `resolved_at`) complete the row.

**`rescue_request_logs`** — an append-only audit trail: every status transition
writes `(request_id, changed_by, old_status, new_status, note)`, giving a complete
history of who moved a request and when.

**`rescue_groups` / `rescue_group_members` / `rescue_group_invites`** — rescuers
operate as **teams**. A group has members with a `member_role`
(`leader` | `co_leader` | `member`) and a `join_status`. Only leaders and
co-leaders may accept missions, and an admin dispatches to a *group* (resolving to
its leader for tracking compatibility).

**`family_connections`** — a symmetric requester ↔ receiver relationship with a
`status` (`pending` | `accepted` | `rejected`); accepted connections receive
"your relative is safe" notifications when a request resolves.

**`user_locations`** — exactly one row per user (`UNIQUE user_id`) holding the
**durable** last-known position. It is written only at the **start and end** of a
tracking session; the fast-moving updates in between live in Redis (see §4.3.2).

### 4.2.2 Authentication and authorization

**Authentication** is phone + password with **stateless JWTs**
([`backend/routes/auth.js`](../../backend/routes/auth.js)):

1. On **registration**, the password is hashed with `bcrypt` (salted) and stored as
   `password_hash`; the plaintext is never persisted. Registering as `rescuer` or
   `admin` additionally requires a shared **role password**, preventing arbitrary
   users from self-elevating.
2. On **login**, the submitted password is verified with `bcrypt.compare` against
   the stored hash.
3. On success, the server signs a JWT with `jsonwebtoken` containing exactly
   `{ id, phone_number, role }` and a **7-day expiry**. The client stores it and
   sends it as `Authorization: Bearer <token>` on every request.

An optional **Google sign-in** path exists on the frontend (via Firebase) but
phone + password against this backend is the primary mechanism. **Twilio** provides
SMS OTP for phone verification / password reset, itself protected by an in-process
per-number rate limit.

**Authorization** is role-based, enforced by middleware in
[`backend/middleware/auth.js`](../../backend/middleware/auth.js):

- `authMiddleware` verifies the Bearer token and sets `req.user` to **only**
  `{ id, phone_number, role }` — deliberately minimal; a handler that needs the
  name or email queries the `users` table. An invalid/missing token yields `401`.
- `requireAdmin` gates admin-only endpoints (`403` otherwise).
- `requireRoles([...])` restricts an endpoint to a set of roles — e.g. creating an
  SOS is `citizen`-only, while accepting one is `rescuer`-only.

A **second layer of authorization** lives inside the SQL itself: mission-mutating
queries include ownership predicates in their `WHERE` clause
(e.g. `... AND assigned_to = $1`), so even a valid rescuer token cannot complete or
cancel a request that is not their own. This makes the check atomic with the write
and immune to time-of-check/time-of-use races.

---

## 4.3 Real-time coordination engine

Rescue is a live activity: a citizen in the water and a rescuer en route both need
to see each other move in real time. AquaGuard delivers this with a native
WebSocket server co-located with the REST API, backed by a Redis hot store.

### 4.3.1 WebSocket transport

The real-time layer is a native [`ws`](../../backend/index.js) `WebSocketServer`
attached to the **same HTTP server** as Express (`http.createServer(app)`), so REST
and WebSocket share a single port and a single TLS termination — no separate
gateway to operate.

**Connection handshake.** A client connects with the JWT as a query parameter
(`wss://host?token=…`). On `connection` the server verifies the token with the same
`JWT_SECRET` used for REST; a missing token closes the socket with code `4001`, an
invalid one with `4002`. On success it stamps the socket with `ws.userId` and
`ws.userRole` and initialises per-connection tracking state.

**Two location flows** run over this one transport:

- **Continuous presence (always-on).** Whenever a user is logged in, the frontend
  `LiveLocationProvider` keeps a dedicated socket open and streams
  `presence_location` messages. These are written to the **Redis hot store only** —
  pure presence *never* touches PostgreSQL. This powers the live flood map and lets
  the SOS endpoints overlay a requester's current position onto their request.
- **Per-request tracking rooms.** When a citizen and their assigned rescuer open the
  tracking view, each joins a **room** keyed by request ID
  (`join_tracking`). Fixes are broadcast between the two members of the room in
  real time, and *only* these tracking sessions write to PostgreSQL — the first fix
  (start) and the last fix on disconnect (end).

**Server-initiated events.** REST handlers can push into a room via a shared
`trackingRooms` map (exposed with `app.set("trackingRooms", …)`): accepting a
mission broadcasts `tracking_started`, cancelling broadcasts `tracking_cancelled`,
and completing broadcasts `tracking_ended` — so the citizen's map reacts instantly
to actions taken through the REST API.

### 4.3.2 GPS streaming at 2-second cadence

Both location flows sample the device GPS at a **2-second cadence**. On the client,
`navigator.geolocation.watchPosition` is paired with a `setInterval(…, 2000)`
fallback poll — because `watchPosition` fires irregularly on some devices, the 2 s
timer guarantees a steady stream of fixes. Positions are requested with
`enableHighAccuracy: true` and a low `maximumAge` so each fix is fresh.

On the server, `handleLocation()` processes every incoming fix through a
**three-step pipeline**:

1. **Broadcast** the fix to the other member(s) of the active tracking room
   immediately — every update, so the on-screen marker moves smoothly.
2. **Durable START** — on the *first* fix of a tracking session, persist the
   position to PostgreSQL `user_locations` (upsert on `user_id`).
3. **Hot store** — write the fix to Redis, but **throttled**: only when the device
   has moved a meaningful distance (≈ 5.5 m, i.e. ≥ 0.00005° in lat or lng) **or**
   at least 2 s have elapsed since the last Redis write. This respects the
   serverless Redis request quota while keeping presence fresh.

Redis stores each position as a hash `live:loc:{userId}` with a **60-second TTL** —
so a key's mere existence means "online and fresh" — plus a per-role GEO set
(`live:geo:{role}`) that enables `GEOSEARCH`-based "nearest rescuer" queries. On
disconnect, the server flushes the **last** fix to PostgreSQL (durable END) and
removes the Redis presence.

This **hot/durable split** is the key design decision: high-frequency, disposable
updates never hit the relational database, while PostgreSQL always retains a
"last known location" that survives a Redis eviction or a server restart.

### 4.3.3 SOS state machine implementation

Every rescue request moves through a well-defined **state machine**, enforced by
the `status` column's `CHECK` constraint and by guarded SQL updates in
[`backend/routes/sos.js`](../../backend/routes/sos.js):

```
   (citizen creates)
         │
         ▼
     ┌────────┐   admin assigns to group   ┌──────────┐
     │pending ├───────────────────────────►│ assigned │
     └───┬────┘                            └────┬─────┘
         │  rescuer accepts (leader/co_leader)  │ rescuer accepts
         │◄─────────────────────────────────────┘
         │
         ▼
   ┌─────────────┐   rescuer cancels    ┌────────┐
   │ in_progress ├─────────────────────►│pending │  (released back)
   └──────┬──────┘                      └────────┘
          │ rescuer / admin completes
          ▼
     ┌──────────┐
     │ resolved │
     └──────────┘
```

**Transitions and their guards:**

- **create** → `pending`. A citizen `POST`s the SOS with GPS and up to five images
  (uploaded to Cloudinary); the row is inserted and the creation logged.
- `pending` → **`assigned`**. An admin dispatches the request to a rescue *group*;
  the group's leader becomes `assigned_to` for tracking compatibility. The update is
  guarded by `WHERE id = $ AND status = 'pending'`.
- `pending` / `assigned` → **`in_progress`**. A rescuer accepts. The handler first
  verifies the caller is a `leader` or `co_leader` of an active group, then performs
  a single guarded `UPDATE` whose `WHERE` clause allows the transition only if the
  request is still `pending` (and unassigned or assigned to this rescuer) or
  `assigned` to this rescuer's group — closing the "two rescuers accept at once"
  race. Acceptance broadcasts `tracking_started` and notifies the citizen (in-app +
  email).
- `in_progress` → **`pending`**. A rescuer can release a case; the update is guarded
  by `assigned_to = $ AND status = 'in_progress'`, so only the owning rescuer can
  cancel, and it broadcasts `tracking_cancelled`.
- `in_progress` → **`resolved`**. The assigned rescuer (or an admin) completes the
  mission; broadcasts `tracking_ended` and notifies both the citizen and their
  accepted family members that they are safe.

Every transition is written to `rescue_request_logs` for a full audit trail, and
every state-changing SQL statement embeds its precondition in the `WHERE` clause so
that **validity is enforced atomically at the database**, not just in application
code.

### 4.3.4 Concurrent connection management

Because many citizens and rescuers may be connected at once, the server manages
concurrency carefully:

- **Room registry.** Tracking rooms are held in a `Map<requestId, Map<userId, ws>>`.
  Joining adds `(userId → socket)`; on socket `close` the user is removed, and when
  a room empties it is deleted — so the structure never leaks memory as missions
  complete.
- **Liveness / dead-connection reaping.** A 30-second **heartbeat** pings every
  client and marks it `isAlive = false`; a socket that fails to answer with a
  `pong` before the next tick is `terminate()`d. This detects half-open connections
  (phones that lose signal in a flood) and frees their resources.
- **Presence freshness via TTL.** Rather than trusting sockets to close cleanly,
  online status is derived from the 60-second Redis TTL on each `live:loc:{userId}`
  key — a user whose fixes stop is automatically considered offline once the key
  expires, independent of socket state.
- **Backpressure-aware sends.** Before writing to a peer, the server checks
  `readyState === 1` (OPEN), so it never enqueues into a closing or closed socket.
- **Write throttling.** The per-connection Redis throttle (§4.3.2) bounds how much
  each client can write to the shared hot store regardless of how fast it emits
  fixes, protecting the serverless Redis quota under many concurrent trackers.

The one caveat of holding rooms in process memory is that they are **ephemeral**:
if the single Render instance restarts, live rooms are lost and clients reconnect.
This is an accepted trade-off — durable state (the request status and start/end
positions) is always safe in PostgreSQL, so a reconnect fully restores the session.

---

## 4.4 Frontend: web platform

The web client is a **React 19 + Vite 6 single-page application**
([`frontend/`](../../frontend/), ESM), styled with **Tailwind CSS v4** and routed with
**React Router DOM 7**.

**Application shell.** [`App.jsx`](../../frontend/src/App.jsx) composes a tree of
context providers whose nesting order matters: `AuthProvider` wraps
`LiveLocationProvider` (which needs the token to open its socket), inside
`ToastProvider` and `NotificationProvider`, all under an `ErrorBoundary`. A single
protected `/` route renders a role-aware `Dashboard`; `/login`,
`/forgot-password`, and `/unauthorized` are public.

**State and data flow:**

- **`AuthContext`** is the central auth store — `registerWithPhone`,
  `loginWithPhonePassword`, `loginWithGoogle` — and persists/restores the session,
  falling back to Firebase-cached data if the backend is briefly unreachable.
- **`LiveLocationProvider`** keeps the always-on presence socket open and streams
  `presence_location` fixes at the 2 s cadence described in §4.3.2.
- **`services/`** is a thin REST layer with the base URL from
  `VITE_API_BASE_URL`; **`hooks/useRescueTracking`** encapsulates the per-request
  tracking socket used by the tracking-map modal.

**Role-scoped UI.** Pages live under
[`pages/{admin,rescuer,citizen}/`](../../frontend/src/pages/) and are gated by
`components/auth/ProtectedRoute.jsx` (RBAC on the client, mirroring the server's
role checks). Citizens raise and track SOS requests; rescuers see team missions and
accept/complete them; admins get dispatch, analytics, and broadcast tools.

**Maps, charts, and onboarding.** **Leaflet 1.9 + React-Leaflet 5** render the live
flood map and rescue-tracking view (polling `/api/locations/live` and consuming
socket updates), **Recharts 3** draws the analytics dashboards, and
**react-joyride 3** provides guided onboarding tours.

**Internationalisation.** All copy is bilingual (Vietnamese / English) via
`useLanguage().t("section.key")`, with every user-facing string defined under the
same key in both `translations/vi.js` and `translations/en.js` — no text is ever
hardcoded.

---

## 4.5 Mobile: iOS application

Alongside the web platform, AquaGuard ships a **native mobile client** aimed at the
people most likely to need it in the field — citizens raising an SOS from a phone in
a flood, and rescuers navigating to them. The mobile app is a **separate client
codebase** that consumes the **same backend API** as the web app, so the two share
one source of truth and one security model.

**Integration with the backend.** The mobile app is a first-class citizen of the
API contract:

- **Same authentication.** It authenticates with the identical phone + password /
  JWT flow (§4.2.2) and sends the same `Authorization: Bearer <token>` header, so no
  separate identity system is required.
- **Same REST + WebSocket endpoints.** It calls the `/api/*` REST surface and opens
  the same `wss://host?token=…` socket for presence and per-request tracking — the
  backend treats web and mobile connections identically.
- **CORS accommodates native clients.** The server explicitly allows requests with
  no `Origin` header (native mobile apps, unlike browsers, send none) and, in
  development, permits LAN IP origins so the app can talk to a locally-running
  backend from a physical device on the same Wi-Fi.

**Why mobile matters here.** A phone provides exactly what rescue coordination
needs and a laptop cannot: it is **carried on the person**, it has a **high-accuracy
GPS**, and it can stay connected while moving. The mobile app therefore streams the
continuous `presence_location` fixes and participates in tracking rooms just like
the web client, giving rescuers a live, moving marker to follow. Because the
real-time engine (§4.3) is transport- and client-agnostic, adding the mobile client
required no change to the coordination logic — only a new consumer of the existing
socket protocol.

> **Note.** The native mobile application is maintained in its own repository; this
> document (and this monorepo) covers the backend and web platform. The sections
> above describe how the mobile client integrates rather than its internal
> implementation.

---

## 4.6 Security and data protection

Security is layered across authentication, authorization, transport, input
handling, and secret management.

**Credentials and identity.**

- Passwords are hashed with **bcrypt** (salted, per-user) and never stored or logged
  in plaintext; login verifies via constant-time `bcrypt.compare`.
- Sessions are **stateless JWTs** signed with a server-side `JWT_SECRET`, carrying
  only `{ id, phone_number, role }` and expiring after **7 days**. The same secret
  guards both REST and WebSocket handshakes, so an unauthenticated socket is closed
  immediately (codes `4001`/`4002`).
- Privileged registration (`rescuer` / `admin`) requires an additional shared
  **role password**, preventing self-elevation.

**Authorization in depth.** Every protected endpoint runs `authMiddleware`, and
sensitive ones add `requireAdmin` / `requireRoles([...])`. Critically, authorization
is **also enforced in SQL**: mutating queries embed ownership predicates in their
`WHERE` clause (e.g. `assigned_to = $1`, `status = 'in_progress'`), so a valid token
still cannot act on a resource the user doesn't own, and the check is atomic with the
write.

**Transport and origin control.**

- All production traffic is **HTTPS/WSS** (TLS terminated by Vercel and Render);
  `db.js` auto-enables SSL to the Neon database and Upstash uses `rediss://`.
- A **hardcoded CORS allowlist** restricts browser origins to the known frontends;
  the development-only LAN regex is gated behind `NODE_ENV !== "production"`.

**Abuse resistance.** An in-memory **rate limiter** throttles the endpoints most
attractive to attackers — login, registration, forgot-password, and admin
broadcasts — and SMS OTP requests are rate-limited per phone number to blunt
brute-force and toll-fraud attempts.

**Input handling and injection safety.** All database access uses **parameterised
queries** (`pg` placeholders `$1, $2, …`) — never string concatenation — so the data
layer is structurally resistant to SQL injection. Image uploads go through `multer`
into memory and are handed to **Cloudinary** rather than written to the app server's
disk, and the API validates required fields before persisting.

**Data minimisation and privacy.**

- **Live GPS is transient by design.** Continuous fixes live only in Redis with a
  **60-second TTL** and are deleted on disconnect; PostgreSQL retains just the
  first/last position of a tracking session. The platform therefore does not
  accumulate a fine-grained movement history of its users.
- The JWT and `req.user` carry the **minimum** identity needed; richer profile data
  is fetched from `users` only when a handler actually requires it.
- `email` is optional, and all transactional mail (Resend) is **fire-and-forget** and
  no-ops silently when a recipient has no address — a mail failure can never block or
  leak a rescue action.

**Secret management.** No credential is committed to the repository. Every secret is
an environment variable on the hosting platform (Render/Vercel), read at runtime via
`process.env`; frontend variables are `VITE_`-prefixed and contain **no** secrets,
since anything shipped to the browser is public.

**Auditability.** Security- and safety-relevant actions are recorded: every SOS
status transition is appended to `rescue_request_logs` with the actor and timestamp,
giving a tamper-evident trail of who did what during an incident.
