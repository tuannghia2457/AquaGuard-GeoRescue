# AquaGuard / GeoRescue

This repository contains the AquaGuard flood-alert and rescue web platform and the planned GeoRescue computer-vision prototype.

| Part | Current status | Start here |
|---|---|---|
| AquaGuard | Existing web, API, dispatch, live-location, and water-sensor code | [Platform README](docs/AquaGuard_docs/README.md) |
| GeoRescue | Concept and implementation plan; no vision pipeline implemented yet | [GeoRescue README](georescue/README.md) |

## Repository layout

```text
AquaGuard-GeoRescue/
├── README.md
├── frontend/                    # AquaGuard React / Vite application
├── backend/                     # AquaGuard Express API and WebSocket server
├── hardware/                    # ESP32 water-level firmware and serial bridge
├── infrastructure/              # Docker Compose, PostgreSQL schema, nginx
├── docs/
│   └── AquaGuard_docs/
│       ├── README.md            # Platform features and setup instructions
│       ├── technical-guide/     # Vietnamese guides and OpenAPI / Swagger UI
│       └── ...                  # Architecture, workflows, dictionary, and Q&A
└── georescue/
    ├── README.md                # Vision, scope, evaluation, and milestones
    └── docs/
        └── CODEX_PROJECT_ONBOARDING_PROMPT.md
```

GeoRescue source, configurations, scripts, and tests will be added under `georescue/` as implementation progresses. The planned layout is described in its README.

## AquaGuard

Follow the [platform setup instructions](docs/AquaGuard_docs/README.md#-getting-started). Run commands from the repository root unless a command explicitly changes directories.

- [System architecture](docs/AquaGuard_docs/04-system-architecture-and-tech-flow.md)
- [Architecture HTML](docs/AquaGuard_docs/04-system-architecture-and-tech-flow.html)
- [Roles and workflows](docs/AquaGuard_docs/05-roles-and-workflows.html)
- [Data dictionary](docs/AquaGuard_docs/AquaGuard%20Data%20Dictionary.md)
- [Technical guide](docs/AquaGuard_docs/technical-guide/01-tong-quan-va-tinh-nang.md)
- [OpenAPI specification](docs/AquaGuard_docs/technical-guide/05-api-swagger.yaml)
- [Swagger UI](docs/AquaGuard_docs/technical-guide/swagger-ui.html)
- [Project Q&A](docs/AquaGuard_docs/aquaguard-qa.html)

## GeoRescue

Start with the [GeoRescue project plan](georescue/README.md) and [onboarding prompt](georescue/docs/CODEX_PROJECT_ONBOARDING_PROMPT.md).

The initial target is offline replay of recorded UAV imagery over one georeferenced test region. Establish the handcrafted Feature 4 localization baseline before model fine-tuning. Victim-coordinate projection requires validated camera, pose, and surface information.

The reference manuscripts, research implementation, datasets, and model weights are not included. Record their provenance and reuse permissions before adding them. Future local datasets and generated results belong in `georescue/data/` and `georescue/outputs/`, which are ignored by Git.

The existing platform README declares MIT licensing, but this repository currently has no `LICENSE` file. Research materials, models, datasets, and media assets require separate ownership and license verification before public reuse.
