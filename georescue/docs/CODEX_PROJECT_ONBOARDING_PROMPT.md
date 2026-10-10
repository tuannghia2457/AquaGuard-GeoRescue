# Codex prompt — AquaGuard GeoRescue project onboarding

You are joining my project as a senior computer vision / robotics software engineer. I want you to **audit and understand the existing repository before implementing anything**.

## Project intent

I am building **AquaGuard GeoRescue**, a personal portfolio prototype for drone-assisted flood rescue. The core idea is to (a) segment stable landmark structures such as roofs and tree canopies in a georeferenced reference map and UAV image, (b) localize the UAV in GNSS-denied conditions using Generalized Voronoi / integral descriptors, (c) detect people in flood imagery using a fine-tuned YOLO detector, and (d) estimate and display the detected people's geographic positions, along with uncertainty. I may later integrate the outputs with the AquaGuard rescue map. This is not a request to write a research paper or build autonomous flight-control software.

I plan to fine-tune **SegFormer** for landmark semantic segmentation and **YOLO** for aerial/flood-person detection. For the first MVP, keep the **handcrafted Feature 4 Voronoi localization** as the baseline; do not introduce a Siamese network or new learned localization model unless we later decide evidence justifies it.

Relevant documents may include:
- [GeoRescue README](../README.md) (initial concept and scope)
- `references/Feature4_leaf_tree_non_GPS-2026100615h.pdf` (Generalized Voronoi–Integral candidate generation / matching; add when available and permitted)
- `references/Feature_1b__nonGPS_sampling_characteristics.pdf` (integral-descriptor stability / sampling theory; add when available and permitted)
- [AquaGuard platform README](../../docs/AquaGuard_docs/README.md) and [system architecture](../../docs/AquaGuard_docs/04-system-architecture-and-tech-flow.md).
- Any existing `README.md`, architecture docs, source files, tests and build configuration in the repository.

If these documents do not exist in your current workspace, **tell me which are missing**; do not pretend to have read them. You may extract text from PDFs with tools available in your environment. Treat research code, figures and datasets as separately licensed: do not copy or publish them without permission.

## Your first task: READ-ONLY repository audit

1. Identify the repository's actual purpose, platforms, languages, build system, active branches and main entrypoints.
2. Produce a directory tree showing important modules (skip generated files, dependencies and large datasets).
3. Trace the **current implemented data flow** end-to-end, not just the intended flow described in docs.
4. Inspect the segmentation, polygons/convex-hull, distance transforms, Voronoi candidate generation, integral descriptors, descriptor matching, reference database and query-image localization modules **if present**.
5. Find any existing orientation / scale handling, Top-K matching, coordinate transformations, georeferencing, visualizations, benchmarks and tests.
6. Separate **implemented**, **partially implemented**, **documented but unimplemented**, and **missing** functionality. Never assume a module is implemented based only on a folder name.
7. Determine whether available code produces a true map-coordinate position or only an image-relative query-match position. Check whether yaw, altitude and camera calibration are available — 2D localization alone does not yield a reliable victim position.
8. Review build/run/test instructions and evidence for reproducibility. Do not install dependencies, download model weights, run destructive commands or modify files during the audit.
9. Identify code/data ownership, license and third-party dependencies; flag anything requiring permission before public reuse in a personal repo.
10. Identify the **smallest achievable MVP**, accounting for the current codebase and data availability.

## Expected audit report

Please respond in this structure:

### A. Repository snapshot
- Repo type / current branch / primary language(s) / build system
- Short summary of what the current codebase actually does

### B. Architecture and source map
- Important paths with 1–2 line purpose each
- Current end-to-end processing flow (ASCII or Mermaid diagram)

### C. Feature inventory
Use a table with `Feature | Status | Evidence (file paths/functions) | Gaps`. Cover segmentation, geospatial reference-map preparation, Voronoi construction, descriptors, matching, pose recovery, person detection, victim geolocation, map integration and testing.

### D. Reuse plan
- What can be reused without modification?
- What needs refactoring/adaptation for flood scenes?
- What needs to be implemented from scratch?
- Any permission/license blockers?

### E. Technical risks / unknowns
Address reference map georeferencing, view/scale/rotation differences, domain shift, pose observability, surface height and safety/uncertainty, plus actual repository-specific concerns.

### F. Proposed 3–5 milestone implementation plan
Give task order, concrete files/modules likely involved, dependencies, acceptance criteria and validation datasets/metrics. Prioritize baseline localization and evaluation before adding fine-tuning.

### G. Questions for me
Ask **only the few critical questions** required to begin safely, especially about dataset availability, deployment target, camera metadata and permission to reuse upstream code.

## Guardrails

- **Do not edit, generate, delete, commit, switch branches or push anything in this first pass.**
- **Do not claim test results unless tests were actually executed.**
- Reference exact file paths and functions for factual claims.
- Prefer a small local prototype that can replay UAV footage over speculative full autonomy.
- Keep offline processing, GNSS-denied localization and network-denied communications as separate concerns.
- Distinguish author-provided algorithms from our new engineering work.
- If this repository is actually the older Feature 4 research project, plan a clean separation of general reusable geometry code from new AquaGuard-specific DL and integration modules; do not overwrite the research project.

**Stop after presenting the audit and proposed implementation plan. Wait for my approval before touching the code.**
