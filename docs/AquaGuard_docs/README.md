# 🌊 AquaGuard — Flood Alert & Rescue Web Platform

> This README documents the existing AquaGuard platform. See the [repository overview](../../README.md) and [GeoRescue plan](../../georescue/README.md) for the combined project. Setup commands below are run from the repository root unless they explicitly change directories.

## Platform documentation

- [System architecture](04-system-architecture-and-tech-flow.md) ([HTML](04-system-architecture-and-tech-flow.html))
- [Roles and workflows](05-roles-and-workflows.html)
- [Data dictionary](AquaGuard%20Data%20Dictionary.md)
- [Technical guide](technical-guide/01-tong-quan-va-tinh-nang.md)
- [OpenAPI specification](technical-guide/05-api-swagger.yaml) and [Swagger UI](technical-guide/swagger-ui.html)
- [Project Q&A](aquaguard-qa.html)

<div align="center">

![Platform](https://img.shields.io/badge/Platform-Web-0077B6.svg?style=for-the-badge&logo=googlechrome&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB.svg?style=for-the-badge&logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-6-646CFF.svg?style=for-the-badge&logo=vite&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-4-38BDF8.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-20+-339933.svg?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express.js-4-000000.svg?style=for-the-badge&logo=express&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-12-FFCA28.svg?style=for-the-badge&logo=firebase&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-22C55E.svg?style=for-the-badge)

**AquaGuard Web** is a full-stack emergency flood management platform — the web companion to the [AquaGuard iOS App](https://github.com/shynnguyen1004/AquaGuard-iOS). It provides real-time flood mapping, community-driven rescue coordination, and AI-assisted emergency support across three dedicated role-based interfaces: Admin, Rescuer, and Citizen.

> **Status:** 🚀 Active Development — EPICS 8th Competition Build

[Live Demo](https://aquaguard.vn) · [iOS App](https://github.com/shynnguyen1004/AquaGuard-iOS) · [Report a Bug](https://github.com/shynnguyen1004/AquaGuard-WebApp/issues) · [Request a Feature](https://github.com/shynnguyen1004/AquaGuard-WebApp/issues)

</div>

---

## 📱 Screenshots

| Admin Dashboard | Live Flood Map | Rescue Operations | Citizen SOS | Safety Guides |
|:---:|:---:|:---:|:---:|:---:|
| ![Admin Dashboard](placeholder-admin-dashboard.png) | ![Live Map](placeholder-live-map.png) | ![Rescue Ops](placeholder-rescue-ops.png) | ![SOS Page](placeholder-sos-page.png) | ![Safety Guides](placeholder-safety-guides.png) |

---

## ✨ Key Features

### 🎭 Role-Based Dashboards (RBAC)
A full **Role-Based Access Control** system powers three distinct user experiences:

- **🛡️ Admin Dashboard:** Full-system overview with a live **Flood Map Editor**, user management panel, rescue team tracker, stats overview, and real-time SOS request monitoring. Admins can create, edit, and delete flood zones directly on the map.
- **⛑️ Rescuer Dashboard:** Mission-focused interface with an active rescue requests feed, mission acceptance workflow (`Pending → Assigned → In Progress → Resolved`), and personal mission history. Real-time updates via auto-refresh every 10 seconds.
- **🏘️ Citizen Dashboard:** Full-featured emergency management interface — detailed below in [Citizen Features](#-citizen-features).

### 🏘️ Citizen Features

#### 🏠 Dashboard (Home)
- **Personalization:** Displays the user's name and personal safety status (Safe / In Danger / Injured).
- **Active SOS Banner:** A prominent banner showing your active SOS request (if any), with real-time status (Pending → Assigned → In Progress), the name of the rescuer who accepted it, and the submission time. Tap it to jump straight to the SOS page.
- **Family Safety Board:** Shows your connected family members, their safety status (Safe ✅ / In Danger 🔴 / Injured 🟠), health notes, and last update time. Auto-refreshes every 30 seconds.
- **Pending Family Invites:** Family connection invite notifications — accept or decline with a single tap.
- **Quick Actions:** 3 quick-action buttons — Send SOS, Check Family, View Flood Map.
- **SOS Request History:** The 5 most recent completed/cancelled SOS requests, with location, urgency level, and rescuer info.

#### 🆘 SOS Page (Citizen SOS Page)
- **Submit a new SOS:** A full form with automatic GPS capture, reverse geocoding (Google Maps + Nominatim fallback), situation description, urgency level selection (Low / Medium / High / Critical), and scene photo upload (up to 5 images with drag-and-drop).
- **My requests list:** Shows all your SOS requests with real-time status, auto-refreshing every 10 seconds.
- **Status tracking:** Each request shows: status (Pending → Assigned → In Progress → Resolved), rescuer name, rescue team name (if accepted as a group), attached images, and a notice if it was returned by the rescuer.
- **🗺️ Rescue Tracking Map:** When a request is being handled (`in_progress`), the citizen can open a full-screen map to:
  - See their own location (red marker) and the rescuer's location (green marker) on a real-time map.
  - See the route from the rescuer to the citizen via the OSRM API.
  - See the distance (km) and estimated time.
  - Connect over a real-time WebSocket (Firebase Realtime DB) — location updates continuously.
  - Get an automatic notification when the mission is completed or cancelled.

#### 🗺️ Live Flood Map
- View all flood zones on the map with 4 color-coded severity levels (Purple / Red / Amber / Green).
- **Precipitation Radar Overlay:** Live rain radar from the RainViewer API, refreshing every 5 minutes.
- **Weather Layers:** Wind, temperature, cloud, and pressure layers from OpenWeatherMap.
- **VNDMS Tiles:** Vietnam government map tiles — including Hoàng Sa and Trường Sa.
- **Family Map:** View your family members' locations on the map (if connected).
- **Map Legend:** Severity legend and map controls.
- **Quick Actions Panel:** Quick SOS, Find Shelter, Check Family — right on the map.

#### 🛡️ Safety Protocols
- **Emergency Contacts:** Direct-dial Police (113), Fire (114), and Ambulance (115) — one tap.
- **Safety Guides:** 5 detailed guides with expandable content:
  - 📦 Before the flood (Preparation)
  - 🏠 During the flood (Emergency response)
  - 📝 After the flood (Recovery)
  - 🏃 Evacuation guide
  - 🏥 Emergency medical care
- **4G SOS SMS:** Register emergency data packages from Viettel, Vinaphone, and Mobifone — works when there's no internet.

#### 📰 News & Alerts
- **Flood News Feed:** Aggregated flood news with images, sources, category labels (Danger / Warning / Info), and content summaries.
- **Report Issue Form:** An issue report form (Bug / Feature / Data / Other) with file attachments and drag-and-drop.

#### 👨‍👩‍👧‍👦 Family Safety
- **Find family members:** Search by phone number and send a connection invite with a relationship (Father, Mother, ...).
- **Manage connections:** Accept/decline invites, remove connections.
- **Safety status:** Update your personal status (Safe / In Danger / Injured / Unknown) + health notes.
- **Family monitoring:** View each member's safety status, health notes, address, and last update time.

#### 🤖 AI ChatBot
- A built-in AI assistant (powered by Groq / Llama 3.3 70B) — provides instant help on flood safety, app usage, and general questions.
- Built-in quick replies ("What is AquaGuard?", "What to do during a flood?", "How to report an emergency?", ...).
- Smart fallback when the API connection is lost.
- Bilingual support (Vietnamese–English).

#### ⚙️ Settings
- **Profile:** Update name, email, emergency contact, gender, date of birth (auto-calculates age), and address (auto GPS detection + reverse geocoding).
- **Family:** Manage family members (see above).
- **Appearance:** Choose Light / Dark / System theme.
- **Language:** Switch between Vietnamese ↔ English.

### 🗺️ Interactive Live Flood Map
- **Real-time Flood Zones:** Flood severity visualized with 4 distinct color-coded map pins for immediate situational awareness:
    - 🟣 **Purple:** Critical (Emergency)
    - 🔴 **Red:** Severe (High Risk)
    - 🟡 **Amber:** Moderate (Caution)
    - 🟢 **Green:** Safe (Low Risk)
- **Precipitation Radar Overlay:** Live rain radar powered by the free **RainViewer API**, refreshing every 5 minutes.
- **Weather Layers:** Optional wind, temperature, cloud, and pressure overlays via OpenWeatherMap.
- **Sovereign Map Tiles:** Uses **VNDMS Vietnam government tiles** — officially includes Hoàng Sa and Trường Sa island coverage.
- **Admin Map Editor:** Admins can add, edit, and remove flood zone markers directly on the map in real time.
- **Map Legend:** Built-in severity legend and map controls for quick reference.

### 🆘 Emergency Action Center
- **One-Click SOS:** Citizens submit rescue requests with auto GPS capture, reverse geocoding, description, urgency level (4 tiers), and up to 5 scene photos — tracked in real time.
- **Rescue Tracking Map:** Full-screen real-time map showing citizen + rescuer positions, driving route via OSRM, distance/ETA, and WebSocket-powered live location updates.
- **Quick Emergency Contacts:** Direct-dial shortcuts for Police (113), Fire Station (114), and Ambulance (115).
- **Active Alerts Feed:** Real-time community alert stream with severity badges and location data.
- **4G SOS SMS:** One-tap registration for emergency data packages from Viettel, Vinaphone, and Mobifone — works without internet.

### 📰 Community & Support
- **Flood News Feed:** Aggregated flood news and active community reports for situational awareness.
- **Safety Protocol Guides:** Offline-accessible survival instructions for pre-, during-, and post-flood scenarios (5 guide categories).
- **AI ChatBot:** Integrated conversational assistant powered by Groq/Llama 3.3 70B for immediate guidance on flood safety and app usage. Supports bilingual (Vietnamese + English).
- **Report Issue Form:** Citizens can submit bug reports, feature requests, and data issues with file attachments.

### 🔐 Authentication & Sessions
- **Phone + Password Auth:** Primary login is Vietnamese phone number (`+84...`) + password, hashed with bcrypt and issued a role-encoded **JWT** (7-day expiry) by the backend.
- **Optional Google Sign-In:** Firebase Google sign-in is available as a secondary option.
- **Role at Registration:** Users pick their role (Citizen, Rescuer, or Admin) during sign-up; Admin/Rescuer roles require a shared role password.
- **OTP Password Reset:** "Forgot password" sends an SMS OTP via **Twilio Verify**, then issues a short-lived reset session.
- **Transactional Email:** Welcome email on registration and family-invite/accept notifications are sent via **Resend** (fire-and-forget, never blocks the request).
- **Persistent Sessions:** JWT + user are stored client-side and restored on reload; falls back to Firebase user data if the backend is unreachable.

---

## 🛠 Tech Stack

### Frontend (`/frontend`)

| Technology | Version | Purpose |
|---|---|---|
| **React** | 19 | Component-based UI framework |
| **Vite** | 6 | Build tool & dev server |
| **Tailwind CSS** | 4 | Utility-first CSS styling (`@tailwindcss/vite`) |
| **React Router DOM** | 7 | Client-side routing |
| **Firebase SDK** | 12 | Optional Google sign-in |
| **Leaflet + React Leaflet** | 1.9 / 5.0 | Interactive map rendering |
| **Recharts** | 3 | Charts (analytics) |
| **react-joyride** | 3 | Guided onboarding tours |

### Backend (`/backend`)

| Technology | Version | Purpose |
|---|---|---|
| **Node.js** | 20+ | JavaScript runtime (CommonJS) |
| **Express.js** | 4 | REST API framework |
| **PostgreSQL** (`pg`) | — | Primary relational database — raw SQL, no ORM |
| **ws** | 8 | Native WebSocket server for live rescue tracking |
| **JWT** + **bcrypt** | — | Role-encoded access tokens + password hashing |
| **Resend** | — | Transactional email |
| **Twilio Verify** | — | SMS OTP for password reset |
| **Multer + Cloudinary** | — | Image upload and CDN storage |

### Architecture

```
AquaGuard Web
├── frontend/               # React 19 + Vite SPA  (ESM)
│   └── src/
│       ├── config/             # Firebase init, RBAC config
│       ├── contexts/           # AuthContext, LanguageContext
│       ├── hooks/              # WebSocket rescue-tracking hook
│       ├── services/           # API service layer (VITE_API_BASE_URL)
│       ├── translations/       # i18n (vi.js, en.js)
│       ├── utils/              # Auth storage, phone utils
│       ├── pages/              # Page components by role (admin/ rescuer/ citizen/)
│       └── components/         # Reusable UI (map, layout, auth, chat, dashboard,
│                               #   rescue, reports, safety, alerts, onboarding, ...)
├── backend/                # Node.js + Express REST API + WebSocket  (CommonJS)
│   ├── index.js                # Entry point: Express, CORS, rate limits, WS server
│   ├── db.js                   # Shared pg Pool (auto-SSL for cloud DBs)
│   ├── routes/                 # auth.js, sos.js, family.js, analytics.js (logic inline)
│   ├── middleware/             # auth.js (JWT + role guards), rateLimit.js
│   ├── utils/                  # email.js (Resend), upload.js (Cloudinary)
│   └── migrations/             # Raw .sql migrations (applied manually)
└── infrastructure/         # docker-compose.yml, Makefile, database/init_db.sql
```

---

## 🎭 Role-Based Access Control (RBAC)

| Page | Citizen | Rescuer | Admin | Description |
|---|:---:|:---:|:---:|---|
| Main Dashboard | ✅ | — | — | Family Safety Board, Active SOS Banner, Quick Actions, SOS History |
| Live Flood Map | ✅ | ✅ | — | Interactive map with flood zones, radar, weather layers |
| SOS Request | ✅ | — | — | GPS-based SOS form + personal request tracker + tracking map |
| Rescue Queue | — | ✅ | — | All rescue requests with filter/sort, accept missions |
| Rescuer Missions | — | ✅ | — | Personal mission dashboard with tracking |
| Rescuer Team | — | ✅ | — | Create/join rescue groups |
| Safety Protocols | ✅ | — | — | Emergency contacts + 5 safety guides |
| News & Alerts | ✅\* | ✅\* | — | Flood news feed, report issue form |
| About Us | ✅\* | ✅\* | — | About the AquaGuard team |
| Admin Dashboard | — | — | ✅ | System overview, flood map editor, user management |
| Admin SOS Requests | — | — | ✅ | All SOS requests management |
| Admin Sensors | — | — | ✅ | Flood sensor monitoring |
| Admin Analytics | — | — | ✅ | System-wide analytics |
| Settings | ✅ | ✅ | ✅ | Profile, Family, Appearance, Language |
| AI ChatBot | ✅ | ✅ | ✅ | Global floating chatbot (all roles) |

> \* Pages marked with ✅\* are accessible but not shown in the primary sidebar navigation.

---

## 🚀 Getting Started

### 🐳 Quick Start with Docker (Recommended)

The fastest way to run the project — you only need **Docker Desktop** installed.

```bash
# 1. Clone repo
git clone https://github.com/shynnguyen1004/AquaGuard-WebApp.git
cd AquaGuard-WebApp

# 2. Copy env templates
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# 3. Fill in API keys in backend/.env and frontend/.env (ask your team lead)

# 4. Start the whole stack (the compose file lives in infrastructure/)
docker compose -f infrastructure/docker-compose.yml up --build
```

Once it's up and running:
- 🌐 **Frontend:** http://localhost:5173
- 🔌 **Backend API:** http://localhost:5001/api/health
- 🐘 **PostgreSQL:** localhost:5433 (user: `aquaguard`, pass: `aquaguard_pass`, db: `aquaguard_db`)

> 📌 The database is automatically initialized with the schema and seed data from `infrastructure/database/init_db.sql`.

Useful Docker commands (or use `make -f infrastructure/Makefile <target>`):
```bash
DC="docker compose -f infrastructure/docker-compose.yml"

$DC down                       # Stop all services (keeps data)
$DC logs -f backend            # View backend logs
$DC down -v && $DC up --build  # Reset database (wipes data, re-runs init_db.sql)
$DC up postgres backend        # Run only backend + database

# Add an npm package to the backend → must rebuild & renew the anonymous node_modules volume:
$DC up -d --build --renew-anon-volumes backend
```

> ⚠️ The backend runs `node --watch`, so editing `.js` files auto-reloads. But changing `backend/.env` requires `up -d` to recreate the container (a plain `restart` does not reload env).

---

### 🖥️ Manual Setup (Without Docker)

#### Prerequisites
- **Node.js** >= 20.x, **npm** >= 10.x
- A **PostgreSQL** instance (local, Docker, or a cloud provider like Neon)

#### 1. Backend (`/backend`)

```bash
cd backend
npm install
cp .env.example .env          # then fill in real values
npm run dev                   # node --watch index.js → http://localhost:5001
```

Create the schema by loading `infrastructure/database/init_db.sql` into your database
(e.g. `psql "$DATABASE_URL" -f infrastructure/database/init_db.sql`).
There is no migration runner — the files in `backend/migrations/` are SQL applied manually.

#### 2. Frontend (`/frontend`)

```bash
cd frontend
npm install
cp .env.example .env          # then fill in real values
npm run dev                   # vite → http://localhost:5173
npm run build                 # production build
```

---

## 🔑 Environment Variables Summary

Env files are per-package (both git-ignored): `backend/.env` and `frontend/.env`.

| Variable | Location | Required | Description |
|---|---|:---:|---|
| `DATABASE_URL` | `backend/.env` | ✅ | PostgreSQL connection string (auto-SSL for cloud DBs) |
| `JWT_SECRET` | `backend/.env` | ✅ | JWT signing secret |
| `PORT` | `backend/.env` | — | Backend port (defaults to 5001) |
| `CLOUDINARY_URL` | `backend/.env` | — | Image upload (Cloudinary) |
| `TWILIO_ACCOUNT_SID` / `TWILIO_AUTH_TOKEN` / `TWILIO_VERIFY_SERVICE_SID` | `backend/.env` | — | SMS OTP (Twilio Verify) |
| `RESEND_API_KEY` / `EMAIL_FROM` | `backend/.env` | — | Transactional email (Resend) |
| `VITE_API_BASE_URL` / `VITE_WS_URL` | `frontend/.env` | ✅ | Backend REST + WebSocket URLs |
| `VITE_FIREBASE_*` | `frontend/.env` | ✅ | Firebase client SDK (Google sign-in) |
| `VITE_GROQ_API_KEY` | `frontend/.env` | — | AI ChatBot (Groq/Llama) |
| `VITE_OWM_API_KEY` / `VITE_WINDY_API_KEY` / `VITE_GOOGLE_MAPS_API_KEY` | `frontend/.env` | — | Weather overlays, forecast, geocoding |

> ⚠️ Every `VITE_` variable is embedded in the bundle and exposed to the browser — never put secrets there.

---

## 📡 API Overview

Base URL: `<backend>/api`. All protected routes use a `Bearer <JWT>` header. Representative endpoints (see `backend/routes/` for the full list):

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/auth/register` | — | Register with phone + password + role |
| `POST` | `/api/auth/login` | — | Login, returns JWT |
| `POST` | `/api/auth/forgot-password` · `/verify-otp` · `/reset-password` | — | SMS OTP password reset (Twilio) |
| `GET` · `PUT` | `/api/auth/profile` | 🔒 JWT | Get / update current user profile |
| `POST` | `/api/auth/rescue-groups` · `/:id/invite` | 🔒 Rescuer | Create rescue group / invite members |
| `POST` | `/api/sos` | 🔒 Citizen | Create an SOS request (with images) |
| `GET` | `/api/sos/my` · `/all` · `/team` | 🔒 JWT | List own / all / team SOS requests |
| `PUT` | `/api/sos/:id/assign` · `/accept` · `/complete` · `/cancel` | 🔒 Rescuer | Mission lifecycle transitions |
| `GET` | `/api/family/search` | 🔒 JWT | Find a user by phone number |
| `POST` · `PUT` | `/api/family/request` · `/requests/:id/accept` | 🔒 JWT | Send / accept family connection (triggers email) |
| `GET` | `/api/analytics/overview` · `/users` · `/rescue` | 🔒 Admin | System-wide analytics |
| `GET` | `/api/health` | — | Health check (used by uptime monitor) |

---

## 🗺️ Deployment

| Part | Hosted on | Notes |
|---|---|---|
| **Frontend** | **Vercel** — `aquaguard.vn` (+ `www`) and the `*.vercel.app` subdomain | Auto-deploys on push to `main`. |
| **Backend API + WebSocket** | **Render** (`aquaguard-api.onrender.com`) | Free tier sleeps after ~15 min idle; an UptimeRobot monitor pings `/api/health` to keep it warm. |
| **PostgreSQL** | **Neon** (serverless Postgres) | Connected via `DATABASE_URL`; SSL auto-enabled. |
| **Email** | **Resend** | Sends from the DNS-verified domain `aquaguard.vn`. |

Production secrets are configured as environment variables on each platform (Render / Vercel), not committed to the repo.

---

## 🔮 Roadmap

**Frontend**
- [x] Role-Based Access Control (Citizen / Rescuer / Admin)
- [x] Interactive Flood Map with Leaflet + VNDMS tiles
- [x] Live Rain Radar Overlay (RainViewer — free)
- [x] Admin Flood Map Editor
- [x] Rescuer Mission Dashboard with auto-refresh
- [x] Citizen SOS Request with GPS, images, and 4-tier urgency
- [x] Rescue Tracking Map (real-time citizen-rescuer positioning via WebSocket)
- [x] OSRM-powered driving route with distance/ETA display
- [x] Family Safety Board with real-time safety status tracking
- [x] Family connection system (search by phone, invite, accept/reject)
- [x] Active SOS Banner on Dashboard with push-to-SOS
- [x] SOS Request History widget
- [x] Phone + Password Auth (bcrypt + JWT) with optional Google sign-in
- [x] OTP password reset via Twilio Verify
- [x] Transactional email via Resend (welcome + family invite/accept notifications)
- [x] Role selection at registration
- [x] Mobile-responsive layout with Bottom Navigation
- [x] AI ChatBot integration (Groq / Llama 3.3 70B)
- [x] Bilingual support (Vietnamese + English)
- [x] Dark/Light/System theme
- [x] Profile management with GPS location detection
- [x] Rescuer Team/Group system (individual + group missions)
- [ ] **PWA + Service Worker** (offline capability)
- [ ] **IndexedDB offline queue** for SOS/reports without internet
- [ ] **Push Notifications** (FCM) when entering flood zones
- [ ] **Heatmap density layer** (leaflet.heat) for rescue request clusters
- [ ] **Commune-level (xã) boundaries** overlay for granular risk display
- [ ] **Forecast layer** integration (Google Flood Hub / GloFAS API)

**Backend**
- [x] JWT auth middleware with role-based guards (`requireAdmin`, `requireRoles`)
- [x] Phone + password auth with bcrypt; in-memory rate limiting on auth routes
- [x] SOS CRUD API with image upload (Cloudinary) + mission lifecycle
- [x] Family connection API (search, request, accept, reject, status) with email notifications
- [x] Profile API (CRUD with GPS coordinates)
- [x] Rescue Group API (create, join, manage)
- [x] Real-time rescue tracking via native WebSocket (`ws`) server
- [x] Admin analytics API
- [ ] **Geospatial queries** for nearby shelters/resources
- [ ] **Trust Score system** for community report verification

---

## 🤝 Contributing

Contributions are always welcome! Please follow these steps:

1. Fork the project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'feat: add AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

Please follow the [Conventional Commits](https://www.conventionalcommits.org/) standard for commit messages.

---

## 🔗 Related Projects

| Project | Description | Link |
|---|---|---|
| **AquaGuard iOS** | Native SwiftUI companion app | [GitHub](https://github.com/shynnguyen1004/AquaGuard-iOS) |
| **AquaGuard Web** | This repository | — |

---

## 📖 Citizen User Guide

> This section explains in detail how to use AquaGuard Web for users with the **Citizen** role.

### 1. Sign In & Choose a Role

1. Go to [AquaGuard Web](https://aquaguard.vn).
2. **Register** with your **phone number + password** (or sign in with **Google**). During registration you can also enter an email to receive notifications.
3. During registration: choose the **Citizen** role (Rescuer/Admin roles require a role password).
4. Your role is saved with your account — no need to choose again on future logins.

### 2. Dashboard — Home

| Area | Description |
|---|---|
| **Header** | Shows your name + 3 status buttons (Safe / In Danger / Injured) — tap to update what your family sees |
| **Active SOS Banner** | If you have an unresolved SOS request, the banner appears here — tap to view details |
| **Family Safety Board** | List of family members and their safety status — auto-updates every 30 seconds |
| **Pending Invites** | Unhandled family connection invites — tap ✅ to accept or ✕ to decline |
| **Quick Actions** | 3 buttons: Send SOS (red), Check Family, View Flood Map |
| **SOS History** | The 5 most recent completed/cancelled SOS requests |

### 3. Send an Emergency SOS Request

1. Go to the **SOS** page from the sidebar or tap **Quick Action** ► Send SOS.
2. Tap the **"Send SOS"** button (red) in the top-right corner.
3. Fill in the form:
   - **Location:** GPS is captured automatically + reverse-geocoded into an address. Editable.
   - **Description:** Describe the dangerous situation.
   - **Urgency level:** Low / Medium / High / Critical.
   - **Scene photos:** Up to 5 images, drag-and-drop or select files.
4. Tap **"Submit SOS Request"**.
5. The request will appear in the list with status **"Pending"**.

### 4. Track SOS Status

After submitting an SOS, follow the workflow:

```
🟡 Pending → 🔵 Assigned → 🔵 In Progress → 🟢 Resolved
```

- The status **auto-updates every 10 seconds** — no need to refresh.
- When the status changes to **"In Progress"**, a **"View Map"** button appears.
- Tap **"View Map"** to open the **Rescue Tracking Map** — a full-screen map showing:
  - 📍 **Red marker:** Your location
  - 📍 **Green marker:** The rescuer's location
  - 🛣️ **Route:** The route from the rescuer to you
  - 📏 **Distance + estimated time**
  - ⚡ **Real-time updates** via WebSocket

### 5. View the Flood Map

1. Go to the **Live Flood Map** page from the sidebar.
2. View flood zones with color codes:
   - 🟣 Purple = Critical
   - 🔴 Red = High risk
   - 🟡 Amber = Caution
   - 🟢 Green = Safe
3. Toggle layers:
   - **Rain radar** (RainViewer)
   - **Wind / temperature / cloud layers**
4. Use the **Quick Actions** on the right:
   - **SOS:** Send an emergency request
   - **Find Shelter:** Find the nearest shelter
   - **Family Check:** Check on your family

### 6. Connect with Family

1. Go to **Settings** ► the **Family** tab (or tap Quick Action ► Check Family).
2. Tap **"Add Family Member"**.
3. Enter a phone number → tap 🔍 to search.
4. If found: enter the relationship (Father, Mother, ...) → tap **"Send Invite"**.
5. The other person will see the invite on their Dashboard — they accept to connect.
6. Once connected: update your safety status so your family can follow along.

### 7. Safety Guides

1. Go to the **Safety Protocols** page from the sidebar.
2. See the 3 emergency numbers — tap to call directly.
3. Read the 5 safety guides — tap to expand the detailed content.
4. Scroll down to see how to register a **4G SOS data package** when there's no internet.

### 8. AI ChatBot

1. Tap the 💬 button in the bottom-right corner (desktop) or from the Mobile Header.
2. Ask any question — the chatbot supports both Vietnamese and English.
3. Built-in quick replies for fast questions.
4. If the API connection is lost → the fallback automatically provides emergency information.

### 9. Personal Settings

| Tab | Content |
|---|---|
| **Profile** | Update name, email, emergency contact, gender, date of birth, and address (auto GPS detection) |
| **Family** | Manage family members, update safety status + health notes |
| **Appearance** | Theme: Light / Dark / System |
| **Language** | Language: Vietnamese / English |

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.

---

## 🏆 Recognition

AquaGuard was awarded **1st Prize** at the OPC Competition and is currently being upgraded for the **EPICS 8th (Engineering Projects in Community Service)** international competition.

---

<div align="center">

**Built with ❤️ for a safer community.**

*Protecting lives during Vietnam's flood and typhoon season.*

</div>
