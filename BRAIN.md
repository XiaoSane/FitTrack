# FitTrack Technical Brain & Architectural Source of Truth (Kundali)

---

## 0. DOCUMENT CONTROL

* **Project Name**: FitTrack
* **Project Description**: A dual-mode fitness workspace application featuring a dependency-free, client-side SPA (vanilla HTML5, CSS3, ES2022 JavaScript) backed by browser-local storage and crypto, paired with an optional production-ready Express 5 REST API and PostgreSQL 17 relational database backend, 16 local frame-loop exercise demonstration videos, and an interactive SVG/canvas muscle explorer.
* **Documentation Purpose**: Master technical architecture, engineering specification, reverse-engineered codebase map, security audit, and source-of-truth reference for onboarding, auditing, maintaining, and extending FitTrack.
* **Repository Root**: `c:\Users\patil\Downloads\openai`
* **Analysis Date**: 2026-10-04
* **Analysis Scope**: Complete repository inspection including client JavaScript (`js/`), server source (`server/`), database migrations and seeds (`server/migrations/`, `server/scripts/`), shell scripts (`scripts/`), automated test suites (`tests/`), stylesheet cascades (`css/`), static assets and media manifests (`assets/`), containerization configurations (`Dockerfile`, `docker-compose.yml`), academic coursework deliverables (`output/FitTrack_SE_Lab_Experiments_2026.docx`), and documentation (`docs/`, `README.md`).
* **Documentation Status**: Complete & Verified Against Live Codebase.
* **Confidence & Limitations**:
  * **Frontend & Local Datastore**: `Verified` — 100% inspected from source code.
  * **Backend API & PostgreSQL Schema**: `Verified` — 100% inspected from Express routes, Zod schemas, SQL migrations, and pg-mem/live tests.
  * **Static Deployment Configuration**: `Verified` — Configured via `.openai/hosting.json` pointing to `dist/`.
  * **Git History**: `Unknown` — Repository root is not initialized as a Git working tree (`.git` directory absent in inspected root).
  * **AI Provider API Integration**: `Verified Not Present / Mocked` — Coach uses local deterministic pattern-matching response templates (`coachAdapter`). No external LLM endpoint is called.
  * **Payment Gateway**: `Verified Not Present / Mocked` — Premium subscription workflow is a demonstration toggle; no payment processor SDK or webhook exists.
* **Last-Known Architecture State**: Dual-mode progressive hybrid (offline-first SPA falling back to `localStorage` + Web Crypto; progressive upgrade to PostgreSQL transactional synchronization when Express API at `/api/health` reports ready).
* **Maintenance Type**: Generated via comprehensive autonomous reverse-engineering audit; maintained as project technical source-of-truth.

---

## 1. EXECUTIVE PROJECT SUMMARY

### Architectural Essence
FitTrack is an engineering prototype and portfolio-grade fitness application developed with a dual-execution model. It was designed to satisfy rigorous academic Software Engineering (SE) laboratory requirements (UML Class, Object, and Use Case specifications; Scrum agile planning; SRS traceability; black-box and white-box test case suites) while matching a modern dark-mode athletic visual design system.

```
+----------------------------------------------------------------------------------------------------+
|                                    FITTRACK RUNTIME TOPOLOGY                                       |
+----------------------------------------------------------------------------------------------------+
|                                                                                                    |
|   +-----------------------+              +------------------------+                                |
|   |  STANDALONE STATIC    |              |  FULL-STACK CONTAINER  |                                |
|   |  (Port 5173 / CDN)    |              |  (Port 3000 / Docker)  |                                |
|   +-----------------------+              +------------------------+                                |
|               |                                       |                                            |
|               v                                       v                                            |
|   +-----------------------+              +------------------------+                                |
|   |   Static Web Host     |              |    Express 5 Server    |                                |
|   | (Node serve.cjs / CDN)|              |   (server/index.js)    |                                |
|   +-----------------------+              +------------------------+                                |
|               |                                     |          |                                   |
|               | (HTTP Static Assets)                |          | (REST API + Cookie Auth)          |
|               v                                     |          v                                   |
|   +-------------------------------------------------+     +--------------------------+             |
|   |                   BROWSER CLIENT RUNTIME        |     |  PostgreSQL 17 Database  |             |
|   |   - Vanilla ES2022 SPA (window.FitTrack)        |     |  (server/db.js Pool)     |             |
|   |   - Hash-based Router (#/dashboard, etc.)       |     +--------------------------+             |
|   |   - Dual Storage Adapter (backend-client.js)    |                  ^                   |
|   +-------------------------------------------------+                  |                   |
|            |                                |                          |                   |
|            | (If /api/health unavailable)   | (If /api/health online)  |                   |
|            v                                +--------------------------+                   |
|   +-----------------------+                   Debounced 350ms State Sync                   |
|   | Browser localStorage  |                   (PUT /api/state)                             |
|   | (Key: 'fittrack.v1')  |                                                                |
|   | + Web Crypto SHA-256  |                                                                |
|   +-----------------------+                                                                |
|                                                                                                    |
+----------------------------------------------------------------------------------------------------+
```

### Problem Statement & Domain
Fitness trackers often suffer from bloat, mandatory cloud lock-in, forced social feeds, or cumbersome navigation. FitTrack solves this by providing an ultra-responsive, zero-build-requirement interface that unifies goal setting, structured multi-day workout planning, live set-by-set logging with rest countdown timers, automated volume and streak computation, an interactive anatomical muscle explorer, and a deterministic fitness advice coach.

### User Roles
1. **Unauthenticated Public Visitor**: Browses marketing landing page, workout catalog, exercise directory, muscle explorer, and pricing tiers.
2. **Free Member (`free`)**: Accesses user workspace, restricted to logging up to 3 workouts per day, saving 1 custom workout plan, and exchanging up to 5 offline AI coach messages per month.
3. **Premium Member (`premium`)**: Unlocks unlimited workouts, unlimited saved plans, unlimited coach interactions, and extended analytics. In the demo, activated without payment.
4. **Administrator (`admin`)**: Accesses `/admin` operations console to manage users, exercise libraries, workout definitions, categories, review support submissions, and adjust internal configuration.

### Major Capabilities
* **Interactive Muscle Explorer**: Interactive front/back anatomical body map using SVG silhouettes and clip-paths over 3D mannequin imagery (`assets/muscle-body.png`), allowing users to click muscle groups (chest, deltoids, lats, glutes, quads, etc.) to immediately isolate matching exercises.
* **Exercise Media Engine**: 16 frame-based demonstration video loops (MP4) and companion JPEG poster frames generated from public domain movement archives (`free-exercise-db`).
* **Active Workout Execution Engine**: Live set logger tracking weight (kg), target reps, completed status, cumulative time under effort, automated calorie estimation, and rest countdown timers with audio-visual notifications.
* **Progressive State Synchronization**: Transparently mirrors local state changes (`goals`, `plans`, `workouts`, `sessions`, `sessionExercises`, `chatSessions`, `weightHistory`) to PostgreSQL when running alongside the Express backend, while gracefully falling back to browser `localStorage` when offline or hosted statically.

### Major Risks
* **State Replacement Race Condition**: `syncState` in `server/state.js` implements a bulk delete-and-reinsert transaction for an account's dynamic data. Concurrent writes across multiple browser tabs can cause state clobbering.
* **Client-Side Data Exposure in Demo Mode**: In static mode, all profiles and hashed passwords reside unencrypted in `localStorage` under `fittrack.v1`.
* **Zero Rate Limiting on Authentication Endpoints**: `/api/auth/login` and `/api/auth/register` lack brute-force throttling or IP-based rate limiting.

---

## 2. PROJECT IDENTITY & BUSINESS CONTEXT

### Domain Terminology
* **PlanWorkout**: The join entity connecting a 7-day calendar schedule (`dayNumber` 1–7) in a `WorkoutPlan` to an assigned `Workout` or designated rest day (`workoutId: null`).
* **WorkoutExercise**: The template association defining target exercise sets, target repetition string (e.g., `'8–12'`, `'30 sec'`), rest interval (seconds), and execution sequence (`orderIndex`) within a `Workout`.
* **SessionExercise**: The physical realization of a workout set during an active session, capturing actual weight lifted (`weight`), completed reps (`reps`), completion boolean (`completed`), and rest interval.
* **Volume**: Cumulative tonnage lifted, calculated as $\sum (\text{weight} \times \text{reps})$ across completed session exercises.
* **Streak**: Consecutive daily workout consistency count computed by walking backward through distinct completion dates.

### Academic Lineage & Coursework Alignment
* Verified from `output/FitTrack_SE_Lab_Experiments_2026.docx`:
  * Developed as an academic software engineering laboratory project spanning 11 structured experiments:
    1. Problem Analysis & Requirement Specification (SRS)
    2. UML Use Case Modeling & Actor Specifications
    3. Structural Modeling (Class Diagrams, Object Diagrams, Associations)
    4. Behavioral Modeling (Sequence Diagrams, Activity Diagrams, Statecharts)
    5. Architectural Design & Component Modeling
    6. Project Scheduling & Work Breakdown Structures (WBS)
    7. Agile Project Planning using Scrum (Product Backlogs, User Stories)
    8. Agile Project Management (Sprint Cycles, Kanban Tracking)
    9. Risk Analysis & Software Configuration Management (SCM)
    10. Software Testing & Test Case Design (TC-01 through TC-14, Boundary Value Analysis)
    11. Mini Software Project Deployment with Git, CI/CD, and Hosting
  * Seed data preserves the canonical UML Object Diagram entities: User **Rahul Mehta** (`userId: 101`), Goal **Build Muscle** (`goalId: 301`), Plan **Push Pull Legs** (`planId: 401`), Plan Workout (`planWorkoutId: 601`), Workout **Push Day** (`workoutId: 801`), Exercise **Bench Press** (`exerciseId: 1001`), Category **Chest** (`categoryId: 100`).

---

## 3. COMPLETE TECHNOLOGY STACK

| Layer | Technology | Version | Purpose | Evidence | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Language (Client)** | JavaScript (ES2022) | Native Browser | Client application logic, SPA router, UI renderers | `js/*.js`, `index.html:18-29` | Zero compiler/bundler required; runs natively |
| **Language (Server)** | Node.js | v22.x (Alpine) / CommonJS | Backend API runtime, utility scripts, tests | `package.json:9`, `Dockerfile:1` | Uses Node built-in test runner & crypto |
| **Frontend Architecture**| Vanilla SPA (Global Namespace) | N/A | Modular functional IIFEs mounted on `window.FitTrack` | `js/data.js:2`, `js/app.js:1` | Hash-based routing (`#/dashboard`) |
| **Styling** | Vanilla CSS3 | Modern CSS | Design system, responsive layouts, animations | `css/*.css`, `index.html:13-17` | CSS variables, CSS grid, dark color-scheme |
| **Typography** | Google Fonts | Web Font API | Brand typography: Barlow Condensed & DM Sans | `index.html:12`, `css/styles.css:1` | System font fallbacks configured |
| **Backend Framework** | Express | `^5.2.1` | REST API routing, middleware, static hosting | `package.json:21`, `server/app.js:1` | Uses next-gen Express 5 async error handling |
| **Database** | PostgreSQL | `17-alpine` | Relational persistent storage | `docker-compose.yml:3`, `server/migrations/` | ACID transactions, foreign keys, triggers |
| **Database Client** | node-postgres (`pg`) | `^8.23.0` | Connection pool, parameterized queries | `package.json:24`, `server/db.js:1` | Custom type parsers for int8 and numeric |
| **In-Memory Database**| `pg-mem` | `^3.0.14` | In-memory PostgreSQL engine for automated unit tests | `package.json:29`, `tests/backend.test.cjs:2` | Eliminates external DB dependency during testing |
| **Authentication (Server)**| JSON Web Tokens (`jsonwebtoken`)| `^9.0.3` | Cryptographic session tokens in HTTP-only cookies | `package.json:23`, `server/app.js:11` | 7-day expiration, HS256, issuer verification |
| **Password Hashing (Server)**| `bcryptjs` | `^3.0.3` | Salted cryptographic password hashing | `package.json:17`, `server/app.js:16` | Cost factor: 12 salt rounds |
| **Password Hashing (Client)**| Web Crypto API | Browser Native | Client-side password hashing for offline demo | `js/store.js:17` | SHA-256 with `crypto.randomUUID()` salt |
| **Validation** | Zod | `^4.5.4` | Server request schema validation & coercion | `package.json:25`, `server/validation.js:1` | Strict object/type validation |
| **Security Middleware**| Helmet | `^8.3.0` | HTTP response security headers | `package.json:22`, `server/app.js:9` | CSP disabled for inline SVG/data schemes |
| **CORS Middleware** | `cors` | `^2.8.6` | Cross-Origin Resource Sharing control | `package.json:19`, `server/app.js:9` | Strict origin whitelist from `CLIENT_ORIGINS` |
| **Cookie Parsing** | `cookie-parser` | `^1.4.7` | Parses session cookie `fittrack_session` | `package.json:18`, `server/app.js:9` | Standard Express middleware |
| **Media Processing** | `ffmpeg-static` | `^5.3.0` | Generates 24fps 720p MP4 loops from raw images | `package.json:28`, `scripts/generate-exercise-videos.cjs` | Build-time media generation tool |
| **Test Runner** | Node.js Test Runner | Native (`node --test`)| Unit, integration, schema, and UI regression tests | `package.json:11`, `tests/*.test.cjs` | Fast native execution without Jest/Mocha |
| **Static Dev Server** | Node.js HTTP Module | Native CommonJS | Zero-dependency static development server | `scripts/serve.cjs:1-13` | Serves port 5173 with MIME detection |
| **Containerization** | Docker & Docker Compose | Compose Spec v3.8 | Containerized Node server & PostgreSQL service | `Dockerfile`, `docker-compose.yml` | Healthchecks, named volumes, auto-restart |
| **Static Hosting Config**| OpenAI App Platform Hosting | N/A | Static deployment specification | `.openai/hosting.json:1` | Directs static root to `dist/` |

---

## 4. COMPLETE REPOSITORY STRUCTURE

```text
c:\Users\patil\Downloads\openai\
├── .dockerignore                            # Excludes node_modules, .git, .env, dist from Docker builds
├── .env.example                             # Environment variable template with default localhost values
├── .gitignore                               # Ignores dist/, node_modules/, test-results/, .env files
├── .openai/
│   └── hosting.json                         # Static hosting specification (project_id, target: dist)
├── Dockerfile                               # Multi-stage production container definition (node:22-alpine)
├── docker-compose.yml                       # Multi-container orchestration (PostgreSQL 17 service)
├── index.html                               # SPA entry document, CSS/script manifests, accessibility roots
├── package.json                             # Project metadata, NPM scripts, runtime/dev dependencies
├── package-lock.json                        # Deterministic dependency lockfile
├── README.md                                # Developer setup, feature summary, operational instructions
├── assets/                                  # Photographic assets, media loops, vector icons
│   ├── anatomy-back.png                     # Unused reference asset (posterior muscular anatomy)
│   ├── anatomy-reference.png                # Unused reference asset (anterior muscular anatomy)
│   ├── bench.jpg                            # Photography: Bench press training
│   ├── coach.jpg                            # Photography: Bench press with spotter
│   ├── deadlift.jpg                         # Photography: Barbell deadlift training
│   ├── favicon.svg                          # FitTrack branded orange geometric favicon
│   ├── hero.jpg                             # Photography: Dark gym athlete holding dumbbell
│   ├── muscle-body.png                      # Production mannequin asset for Muscle Explorer
│   ├── running.jpg                          # Photography: City distance runner
│   ├── squat.jpg                            # Photography: Barbell back squat
│   ├── strength.jpg                         # Photography: Overhead barbell press
│   ├── woman.jpg                            # Photography: Barbell lifting female athlete
│   ├── exercise-media/                      # 16 MP4 loops and poster images for movements
│   │   ├── 1001-bench-press.mp4 (and .jpg)  # Movement demo: Flat barbell bench press
│   │   ├── ...                              # [1002 - 1015 videos and posters]
│   │   ├── 1016-goblet-squat.mp4 (and .jpg) # Movement demo: Kettlebell goblet squat
│   │   ├── manifest.json                    # Mapping of exerciseId, slug, source, video, and poster
│   │   └── UNLICENSE.md                     # Public domain license text for free-exercise-db assets
│   └── exercise-source/
│       └── exercises.json                   # Upstream raw exercise database dump (~1MB)
├── css/                                     # Modulated stylesheet layers
│   ├── styles.css                           # Global design system, typography, landing, footer, cards
│   ├── workspace.css                        # App shell, sidebar, dashboard, tracker, sessions, admin
│   ├── management.css                       # Workout builder, plan editor, modal forms, records, PRs
│   ├── exercises.css                        # Exercise video player, responsive video frames, status tags
│   └── muscle-explorer.css                  # Mannequin container, SVG muscle highlights, responsive map
├── dist/                                    # Static build output generated by scripts/build.cjs
│   ├── index.html                           # Copied production SPA shell
│   ├── assets/                              # Mirrored static assets and media
│   ├── css/                                 # Mirrored CSS files
│   └── js/                                  # Mirrored client JavaScript files
├── docs/                                    # Supplementary technical documentation
│   ├── ARCHITECTURE.md                      # Diagram-to-code mapping, domain model relationships
│   ├── ASSETS.md                            # Unsplash photography credits and source URLs
│   ├── BACKEND.md                           # Express API routes, PostgreSQL setup, Docker usage
│   └── MUSCLE-EXPLORER.md                   # Artwork generation prompts, SVG clipping implementation
├── js/                                      # Browser-side modular client source code
│   ├── data.js                              # Initial seed dataset, image map, exercise records
│   ├── store.js                             # LocalStorage persistence, Web Crypto hashing, session lifecycle
│   ├── api.js                               # Local domain validation, coachAdapter, business logic limits
│   ├── components.js                        # Reusable HTML string renderers, SVG icons, cards, form inputs
│   ├── pages.js                             # Public pages (home, workouts, exercises, pricing, auth, about)
│   ├── workspace.js                         # Authenticated workspace views (dashboard, plans, goals, active)
│   ├── management.js                        # Builders (workout, plan), modals, photo upload, event wiring
│   ├── admin.js                             # 9-tab operations console (users, catalog, logs, settings)
│   ├── finalize.js                          # Cross-cutting accessibility, history detail, JSON data export
│   ├── backend-client.js                    # Auto-detects Express API, bridges state via debounced sync
│   ├── muscle-explorer.js                   # Interactive SVG body map logic, front/back mannequin switching
│   └── app.js                               # Hash router, toast manager, timer interval, WebMCP tool
├── output/
│   └── FitTrack_SE_Lab_Experiments_2026.docx# Academic Software Engineering Lab Manual portfolio document
├── scripts/                                 # Operational automation and build scripts
│   ├── build.cjs                            # Static distribution packager (copies assets to dist/)
│   ├── generate-exercise-videos.cjs         # Downloads frames and compiles MP4 loops with ffmpeg
│   └── serve.cjs                            # Zero-dependency Node HTTP server for local testing (5173)
├── server/                                  # Express & PostgreSQL backend implementation
│   ├── app.js                               # Express app factory, CORS, Helmet, auth, API routes
│   ├── db.js                                # PG connection pool factory, type parsers, transactions
│   ├── index.js                             # Production server entrypoint, graceful shutdown handlers
│   ├── state.js                             # Full-database bootstrap retrieval and transactional state sync
│   ├── validation.js                        # Zod schemas for user registration, login, and full state
│   ├── migrations/
│   │   └── 001_initial.sql                  # Complete DDL schema: 17 tables, foreign keys, indexes
│   └── scripts/
│       ├── migrate.js                       # Schema migration runner (executes 001_initial.sql)
│       └── seed.js                          # Database seed runner (populates demo accounts and catalog)
└── tests/                                   # Automated test suites
    ├── backend.test.cjs                     # Tests PostgreSQL DDL, Express routes, auth, isolation, media
    ├── core.test.cjs                        # Tests local store, seed graph, crypto, session lifecycle, UI
    └── muscle-explorer.test.cjs             # Tests muscle group filtering, anatomical SVG markup, routes
```

---

## 5. FILE-BY-FILE SYSTEM INVENTORY

| File | Type | Responsibility | Imports / Dependencies | Used By | Key Logic | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `index.html` | HTML5 Shell | Master entry document, script loading order, accessibility dialogs | Google Fonts, CSS files, JS files | Browser HTTP clients | Loads scripts in sequence, defines `#app`, `#modal`, `#toast` containers | `Verified` |
| `package.json` | Manifest | Package metadata, dependencies, scripts | NPM | Developer, CI/CD, Docker | Declares dependencies and scripts (`start`, `server`, `test`, `db:seed`) | `Verified` |
| `Dockerfile` | Container | Production container configuration | `node:22-alpine` | Docker / Cloud Deploy | Installs production dependencies, copies repo, exposes 3000, runs server | `Verified` |
| `docker-compose.yml` | Orchestration| PostgreSQL service orchestration | `postgres:17-alpine` | Docker Compose | Sets up DB container, port 5432, persistent volume, healthcheck | `Verified` |
| `.env.example` | Config Template| Defines required runtime configuration variables | N/A | DevOps, Local Dev | Sets default `DATABASE_URL`, `JWT_SECRET`, `CLIENT_ORIGINS`, `PORT` | `Verified` |
| `.openai/hosting.json` | Platform Config| Static hosting deployment configuration | N/A | Static CDN Hosting | Points static site host to `dist/` | `Verified` |
| `scripts/serve.cjs` | Node Script | Standalone HTTP server for static demo | `node:http`, `node:fs`, `node:path` | `npm start`, `npm run dev` | Serves port 5173, handles MIME types, path traversal security check | `Verified` |
| `scripts/build.cjs` | Node Script | Assembles production static build | `node:fs`, `node:path` | `npm run build` | Recursively copies `index.html`, `css`, `js`, `assets` into `dist/` | `Verified` |
| `scripts/generate-exercise-videos.cjs`| Node Script | Generates exercise demonstration MP4 loops | `ffmpeg-static`, `node:child_process`, `fetch`| `npm run media:generate` | Fetches frames from GitHub, uses ffmpeg concat filter, outputs MP4s | `Verified` |
| `server/index.js` | Server Entry | Node process entrypoint and lifecycle | `dotenv`, `./db`, `./app` | `npm run server`, Docker | Creates pool, starts Express on port 3000, handles SIGINT/SIGTERM | `Verified` |
| `server/db.js` | Database Layer| Manages PostgreSQL pool and transactions | `pg` (`Pool`, `types`) | `server/index.js`, `state.js`, scripts | Configures type parsers (int8, numeric -> Number), connection pooling | `Verified` |
| `server/validation.js` | Validation | Zod schemas for request validation | `zod` | `server/app.js` | Schemas: `register`, `login`, `state` (strict bounds on arrays/strings) | `Verified` |
| `server/app.js` | HTTP Controller| Express application factory and API routing | `express`, `helmet`, `cors`, `cookie-parser`, `jsonwebtoken`, `bcryptjs` | `server/index.js`, `tests/backend.test.cjs` | Implements `/api/auth/*`, `/api/state`, `/api/bootstrap`, `/api/admin/*` | `Verified` |
| `server/state.js` | Service Layer | DB queries for bootstrap and state sync | `./db` (`transaction`) | `server/app.js` | `bootstrap`: fetches user rows; `syncState`: transactional bulk update | `Verified` |
| `server/migrations/001_initial.sql` | DDL Migration | Relational schema definitions | PostgreSQL | `server/scripts/migrate.js`, `pg-mem` | Creates 17 tables, foreign keys, CHECK constraints, composite indexes | `Verified` |
| `server/scripts/migrate.js` | Runner | Executes initial database migration | `dotenv`, `node:fs`, `../db` | `npm run db:migrate` | Reads `001_initial.sql` and executes against PostgreSQL pool | `Verified` |
| `server/scripts/seed.js` | Runner | Seeds initial users, catalog, and workouts | `dotenv`, `bcryptjs`, `../db` | `npm run db:seed` | Populates Rahul Mehta (101), Admin (1), 16 exercises, 6 workouts, PPL plan | `Verified` |
| `js/data.js` | Seed State | Base seed model and image reference mapping | Global `window.FitTrack` | Loaded 1st in `index.html` | Defines `F.images`, `exerciseRows`, `F.seed()` returning canonical seed graph | `Verified` |
| `js/store.js` | Persistence | Local storage manager and crypto auth | `localStorage`, `crypto.subtle` | Loaded 2nd in `index.html` | Manages `fittrack.v1`, `F.register`, `F.login`, `F.startWorkout`, `F.stats` | `Verified` |
| `js/api.js` | Domain Logic | Business logic, limits, offline coach adapter| `F.db`, `F.user` | Loaded 3rd in `index.html` | Enforces plan limits, workout builder limits, `coachAdapter.reply()` | `Verified` |
| `js/components.js` | View Helpers | Reusable HTML string rendering components | N/A | Loaded 4th in `index.html` | Renders `header`, `footer`, `workoutCard`, `exerciseCard`, icons, form inputs | `Verified` |
| `js/pages.js` | Public Views | Renders marketing and public views | `F.components` | Loaded 5th in `index.html` | Renders `home`, `workouts`, `exercises`, `pricing`, `auth`, `about`, `contact` | `Verified` |
| `js/workspace.js` | User Views | Renders member dashboard and tracker | `F.components`, `F.user` | Loaded 6th in `index.html` | Renders `dashboard`, `plans`, `goals`, `history`, `progress`, `activeSession` | `Verified` |
| `js/management.js`| Modals/Builders| Advanced builders, workout editing, coach UI | `F.pages`, `F.db` | Loaded 7th in `index.html` | Injects workout builder, plan editor, profile upload, photo Base64 reader | `Verified` |
| `js/admin.js` | Admin View | Operations console and record management | `F.pages.admin`, `F.db` | Loaded 8th in `index.html` | Injects 9-tab admin dashboard, user table, category/exercise editors | `Verified` |
| `js/finalize.js` | Enhancements | Accessibility, export, and history details | `F.db`, `F.exportData` | Loaded 9th in `index.html` | Extends history with set details, adds JSON export, sets ARIA labels | `Verified` |
| `js/backend-client.js`| Sync Client | Progressive API client & video adapter | `fetch`, `IntersectionObserver` | Loaded 10th in `index.html`| Health probe, debounced auto-sync (350ms), replaces video elements | `Verified` |
| `js/muscle-explorer.js`| Interactive Map| Anatomical muscle selector and body map | SVG, `assets/muscle-body.png` | Loaded 11th in `index.html`| Renders anterior/posterior clipped mannequins, filters exercises by muscle | `Verified` |
| `js/app.js` | Global App | Router, clock ticks, toasts, WebMCP tool | `window.location`, DOM events | Loaded 12th in `index.html`| Hash routing (`#/route`), modal handling, clock updates, WebMCP registration | `Verified` |
| `tests/core.test.cjs` | Test Suite | Core data model and client logic unit tests | `node:test`, `node:assert`, `node:vm` | `npm test` | Verifies seed graph, auth hashing, workout completion, plan edits, pages | `Verified` |
| `tests/backend.test.cjs`| Test Suite | Backend Express API and PostgreSQL tests | `node:test`, `pg-mem`, `server/app` | `npm test` | Verifies schema execution, auth, persistence, isolation, media integrity | `Verified` |
| `tests/muscle-explorer.test.cjs`| Test Suite | Muscle Explorer unit tests | `node:test`, `node:assert`, `node:vm` | `npm test` | Verifies muscle-to-exercise mapping, compound exercises, route accessibility | `Verified` |

---

## 6. ARCHITECTURE

FitTrack adopts a **Layered, Progressive Hybrid Architecture**. It operates autonomously in the browser as a client-managed Single Page Application, yet elevates itself to a traditional N-tier client-server architecture when a PostgreSQL-backed Express server is detected at runtime.

### High-Level System Architecture

```mermaid
flowchart TD
    subgraph Browser ["Client Runtime (Browser)"]
        UI["DOM / HTML5 SPA Shell (index.html)"]
        Router["Hash Router (js/app.js)"]
        Views["Page & Workspace Views (js/pages.js, js/workspace.js)"]
        Builders["Builders & Editors (js/management.js, js/admin.js)"]
        MuscleExp["Interactive Muscle Explorer (js/muscle-explorer.js)"]
        Media["Exercise Video Player (js/backend-client.js)"]
        Domain["Domain Logic & Limits (js/api.js)"]
        Store["Local State & Web Crypto (js/store.js)"]
        BackendClient["Progressive Sync Client (js/backend-client.js)"]
        LocalStorage[("Browser LocalStorage\nKey: fittrack.v1")]
    end

    subgraph Server ["Server Runtime (Node.js / Express 5)"]
        Entry["Server Entry (server/index.js)"]
        App["Express App & Middleware (server/app.js)"]
        AuthMiddleware["JWT Cookie Auth Middleware"]
        Validation["Zod Schemas (server/validation.js)"]
        StateService["State Sync Service (server/state.js)"]
        DBPool["node-postgres Pool (server/db.js)"]
    end

    subgraph Storage ["Database Layer"]
        Postgres[("PostgreSQL 17 Database\n17 Relational Tables")]
    end

    UI --> Router
    Router --> Views
    Router --> Builders
    Router --> MuscleExp
    Views --> Media
    Views --> Domain
    Builders --> Domain
    MuscleExp --> Domain
    Domain --> Store
    Store --> LocalStorage
    Store --> BackendClient

    BackendClient -- "HTTP GET /api/health (Probe)" --> App
    BackendClient -- "HTTP GET /api/bootstrap" --> App
    BackendClient -- "HTTP PUT /api/state (Debounced 350ms)" --> App
    BackendClient -- "HTTP POST /api/auth/*" --> App

    Entry --> App
    App --> AuthMiddleware
    App --> Validation
    App --> StateService
    StateService --> DBPool
    DBPool --> Postgres
```

### Request & Data Synchronization Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as Athlete / User
    participant UI as DOM / Set Tracker
    participant Store as Local Store (store.js)
    participant Client as Backend Client (backend-client.js)
    participant API as Express API (app.js)
    participant DB as PostgreSQL 17 (state.js)

    User->>UI: Clicks "Complete Set" (Logs weight & reps)
    UI->>Store: F.logSet(setId, weight, reps)
    Note over Store: Validates numeric ranges & sets rest countdown
    Store->>Store: Saves to localStorage ('fittrack.v1')
    Store->>UI: F.render() updates UI & starts rest timer
    Store->>Client: Triggers F.save() wrapper
    
    alt If Backend is Offline
        Client-->>Client: No HTTP request dispatched; state remains local
    else If Backend is Online & Authenticated
        Client->>Client: Clears previous syncTimer, sets 350ms debounce
        Note over Client: User pauses typing / logging
        Client->>API: PUT /api/state (Payload: entire user graph)
        API->>API: Authenticates JWT via HTTP-only cookie
        API->>API: Zod schema validation (server/validation.js)
        API->>DB: BEGIN Transaction
        Note over DB: Verifies foreign keys & user ownership
        DB->>DB: DELETE existing user dynamic rows
        DB->>DB: INSERT updated goals, plans, sessions, sets, chats
        DB->>API: COMMIT Transaction
        API-->>Client: 200 OK { syncedAt: "2026-10-04T..." }
        Client-->>UI: Displays green status tag "POSTGRESQL CONNECTED"
    end
```

### Component Relationship Diagram

```mermaid
flowchart LR
    subgraph Data ["Data & Seed Layer"]
        D["js/data.js\n(F.images, F.seed)"]
        S["js/store.js\n(F.db, F.register, F.login, F.startWorkout)"]
        A["js/api.js\n(F.coachAdapter, F.savePlan, F.saveWorkout)"]
    end

    subgraph UI_Components ["UI Rendering Components"]
        C["js/components.js\n(F.header, F.workoutCard, F.icon)"]
        P["js/pages.js\n(Public Views: Home, Catalog, Auth)"]
        W["js/workspace.js\n(Dashboard, Plans, Tracker, History)"]
    end

    subgraph Management ["Feature Layer"]
        M["js/management.js\n(Workout Builder, Plan Editor, Coach Chat)"]
        AD["js/admin.js\n(9-Tab Operations Console)"]
        FIN["js/finalize.js\n(Set Detail Expansion, JSON Export)"]
        ME["js/muscle-explorer.js\n(SVG Mannequin Body Map)"]
        BC["js/backend-client.js\n(Express/PG Sync, Video Loops)"]
        APP["js/app.js\n(Hash Router, Timers, WebMCP Tool)"]
    end

    D --> S
    S --> A
    A --> C
    C --> P
    C --> W
    P --> M
    W --> M
    M --> AD
    AD --> FIN
    FIN --> BC
    BC --> ME
    ME --> APP
```

---

## 7. APPLICATION STARTUP FLOW

When the browser loads FitTrack, scripts execute in strict synchronous deferred order as declared in `index.html`:

```text
[1] Document Parsing (index.html:1-38)
    ↓
[2] Stylesheets Loaded (css/styles.css -> workspace.css -> management.css -> exercises.css -> muscle-explorer.css)
    ↓
[3] Script Execution Order:
    1. js/data.js            -> Initializes window.FitTrack = {}; populates F.images and F.seed()
    2. js/store.js           -> Loads localStorage('fittrack.v1') or seeds F.db; initializes Web Crypto & user auth helpers
    3. js/api.js             -> Mounts domain rules, plan limits, coachAdapter, workout limits
    4. js/components.js      -> Exposes reusable UI markup generators (F.header, F.workoutCard, etc.)
    5. js/pages.js           -> Mounts public page renderers (F.pages.home, F.pages.workouts, etc.)
    6. js/workspace.js       -> Mounts authenticated workspace views (F.pages.dashboard, activeSession, etc.)
    7. js/management.js      -> Wraps pages with workout builder, plan editor, modals, photo upload
    8. js/admin.js           -> Overrides F.pages.admin with full 9-tab operations console
    9. js/finalize.js        -> Wraps history with set breakdown details; attaches JSON data export
   10. js/backend-client.js  -> Probes GET /api/health; if online, tests GET /api/auth/me; mounts video players
   11. js/muscle-explorer.js -> Mounts F.pages.muscleExplorer and injects promo banners
   12. js/app.js             -> Binds hashchange event, registers WebMCP tool, invokes F.render()
    ↓
[4] Initial Hash Route Resolution (js/app.js:7)
    ↓ Reads window.location.hash (defaults to '#/' if empty)
[5] Page Rendering
    ↓ Resolves route via F.extraRoute(route) or routes regex table
    ↓ Target markup written to document.querySelector('#app').innerHTML
[6] Post-Render Lifecycle (F.afterRender pipeline)
    ↓ Binds filter listeners (workout/exercise searches)
    ↓ Starts live workout timers (F.clockTimer, rest countdown)
    ↓ Injects backend status badge ("LOCAL DEMO" or "POSTGRESQL CONNECTED")
    ↓ Attaches IntersectionObserver for exercise video autoplay
```

---

## 8. ROUTING & NAVIGATION

The application uses hash-based client routing (`window.location.hash`), enabling compatibility with static file systems (`file://`) and static web hosts without server rewrite rules.

| Route | Page / Component | Access Level | Purpose | Data Sources | Key Logic |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `#/` | `F.pages.home` | Public | Marketing landing page, hero, ticker, features | `F.images`, `F.db.workouts` | High-impact visual landing, "Explore demo" button |
| `#/workouts` | `F.pages.workouts` | Public | Workout catalog with search and filters | `F.visibleWorkouts()` | Filters by goal, difficulty, duration, and query |
| `#/workouts/:id` | `F.pages.workoutDetail`| Public | Workout overview, exercise breakdown | `F.db.workouts`, `workoutExercises`| Start workout button; edit/delete tools for creator |
| `#/workouts/create` | `F.pages.workoutBuilder`| User / Admin | Custom workout builder | `F.db.exercises`, `categories` | Add/reorder/remove exercises, configure sets/reps/rest |
| `#/workouts/:id/edit`| `F.pages.workoutBuilder`| Owner / Admin | Custom workout editor | `F.db.workouts`, `workoutExercises`| Pre-populates existing workout and exercises |
| `#/workouts/:id/start`| Route Action | Authenticated | Quick workout start shortcut | `F.startWorkout(id)` | Creates active session and redirects to `#/session/:id` |
| `#/exercises` | `F.pages.exercises` | Public | Movement library with filters | `F.db.exercises` | Search by name, muscle group, equipment, difficulty |
| `#/exercises/:id`| `F.pages.exerciseDetail`| Public | Movement technique & looped MP4 demo | `F.db.exercises`, `exercise-media`| Autoplaying/looping MP4 video, technique cues |
| `#/muscle-explorer` | `F.pages.muscleExplorer`| Public | Interactive anatomical body map | `groups`, `F.muscleExercises` | Front/back SVG mannequin, highlights exercises by muscle |
| `#/ai-coach` | `F.pages.aiCoach` | Public | AI coach promotional landing page | `coach.jpg`, starter prompts | Directs to coach chat or triggers demo mode |
| `#/ai-coach/chat` | `F.pages.coachChat` | Authenticated | Interactive coach conversation UI | `chatSessions`, `chatMessages` | Local deterministic replies via `F.coachAdapter` |
| `#/ai-coach/history`| `F.pages.chatHistory` | Authenticated | Archive of previous coach conversations | `F.own('chatSessions')` | View and re-open prior chat threads |
| `#/pricing` | `F.pages.pricing` | Public | Tier comparison (Free vs Premium) | Hardcoded features list | "Try Premium demo" button toggles role without payment |
| `#/about` | `F.pages.about` | Public | Project story & SE design background | Static text, `strength.jpg` | Explains diagram mappings and project motivation |
| `#/contact` | `F.pages.contact` | Public | Demo support form and FAQ accordion | `F.db.contactMessages` | Saves message to browser storage; FAQ disclosures |
| `#/login` | `F.pages.auth('login')`| Public | User sign-in interface | `F.login(email, password)` | Authenticates local demo or PostgreSQL account |
| `#/register` | `F.pages.auth('register')`| Public | User registration interface | `F.register(values)` | Validates email uniqueness, salts/hashes password |
| `#/forgot-password`| `F.pages.forgotPassword`| Public | Account recovery guidance | Static text | Informs user that prototype has no external email server |
| `#/privacy` | `F.pages.privacy('privacy')`| Public | Privacy policy documentation | Static text | Discloses local storage and absence of remote tracking |
| `#/terms` | `F.pages.privacy('terms')`| Public | Terms of service documentation | Static text | Discloses prototype status and disclaims medical advice |
| `#/dashboard` | `F.pages.dashboard` | Authenticated | Main user workspace dashboard | `F.stats()`, `todayWorkout()`, `goals`| Streaks, weekly activity chart, goal ring, today's workout |
| `#/my-plan` | `F.pages.myPlan` | Authenticated | Active 7-day schedule view | `user.activePlanId`, `planWorkouts`| Redirects to active plan detail or plans listing |
| `#/plans` | `F.pages.plans` | Authenticated | List of saved plans and templates | `F.own('plans')`, template list | Manage plans, activate plan, select PPL/Starter templates |
| `#/plans/create` | `F.pages.planCreate` | Authenticated | 7-day calendar schedule builder | `F.visibleWorkouts()`, `goals` | Configure workout or rest for Monday through Sunday |
| `#/plans/:id` | `F.pages.planDetail` | Authenticated | Plan schedule breakdown | `planWorkouts`, `workouts` | Displays 7-day cards, allows starting day's workout |
| `#/plans/:id/edit` | `F.pages.planEditor` | Authenticated | Plan schedule editor | `planWorkouts`, `workouts` | Edits title, description, linked goal, and daily workouts |
| `#/goals` | `F.pages.goals` | Authenticated | Goal tracking dashboard | `F.own('goals')` | Displays radial progress rings, start/current/target metrics |
| `#/history` | `F.pages.history` | Authenticated | Completed workout session log | `F.completed()`, `sessionExercises`| Expandable set logs, CSV history download |
| `#/progress` | `F.pages.progress` | Authenticated | Analytical training volume and metrics | `weightHistory`, `sessionExercises`| Volume tally, weekly frequency bars, weight sparkline, PRs |
| `#/profile` | `F.pages.profile` | Authenticated | Profile overview and details editor | `F.user()`, `profileImage` | Photo upload (Base64 <1MB), height/weight update |
| `#/settings` | `F.pages.settings` | Authenticated | Prototype configuration and data tools | `F.db.settings` | Reduced-motion toggle, JSON export, prototype reset |
| `#/session/:id` | `F.pages.activeSession`| Authenticated | Live workout execution tracker | `sessions`, `sessionExercises` | Stopwatches, set logging, rest timer, MP4 video cues |
| `#/admin` | `F.pages.admin` | Admin / Demo | Operations & management console | Entire `F.db` datastore | 9 tabs: Overview, Users, Workouts, Exercises, Categories, etc.|

---

## 9. FRONTEND ARCHITECTURE

### Design Tokens & Visual Hierarchy
Configured in `css/styles.css:1`:
* Background: `#0b0807` (Deep Obsidian Black)
* Surface Panels: `#15110f` (Dark Charcoal Brown)
* Elevated Surfaces: `#211a16` (Warm Dark Brown)
* Primary Accent: `#ff5a18` (High-Energy Blaze Orange)
* Contrast Foreground: `#f5f2ed` (Warm Cream / Off-White)
* Muted Text: `#a9a29d` (Warm Slate Gray)
* Borders: `rgba(255, 255, 255, 0.11)`
* Headings Font: `'Barlow Condensed', 'Arial Narrow', Impact, sans-serif`
* Body Font: `'DM Sans', Arial, sans-serif`

### Reusable Component System (`js/components.js`)
* `F.escape(v)`: Escapes `&`, `<`, `>`, `"`, `'` to prevent DOM XSS vulnerabilities during HTML string concatenation.
* `F.icon(name, size)`: Generates inline SVG vector icons with stroke width 1.65, zero runtime overhead.
* `F.link(href, text, cls)`: Renders hash-anchored hyperlinks (`href="#/..."`).
* `F.button(text, action, cls, attrs)`: Renders buttons with standard `data-action` attributes for centralized event delegation.
* `F.workoutCard(w)`: Renders responsive workout cards with difficulty badges, equipment list, and duration.
* `F.exerciseCard(e)`: Renders exercise cards with movement demo badges and muscle group tags.
* `F.image(key, alt, cls, eager)`: Resolves local asset paths from `F.images` map with explicit dimensions and lazy-loading attributes.
* `F.empty(title, copy, link, label)`: Standardized empty-state banner with call-to-action buttons.

### Active Workout Session Tracker (`js/workspace.js:17`, `js/management.js:32`)
The session tracker provides a complete live workout experience:
1. **Clock Interval**: Updates elapsed session duration every second (`js/app.js:11`).
2. **Rest Timer**: When a set is logged (`F.logSet`), sets `s.restUntil = Date.now() + restTime * 1000`. The rest timer displays a countdown with a "Skip" option.
3. **Movement Guidance**: Displays the frame-based looped MP4 video for the current movement alongside instructions.
4. **Dynamic Sets**: Athletes can input weight and reps for each set, check them off, or use "Add set" to add extra volume.
5. **Completion Modal**: Calculates total session duration and estimated energy expenditure ($6 \text{ kcal/min}$), prompts for optional session notes, and commits the session to history.

---

## 10. BACKEND ARCHITECTURE

The server is implemented using Express 5 and PostgreSQL 17.

```
                    +------------------------------------------+
                    |          server/index.js (3000)          |
                    +------------------------------------------+
                                         |
                                         v
                    +------------------------------------------+
                    |           server/app.js (createApp)      |
                    +------------------------------------------+
                                         |
         +-------------------------------+-------------------------------+
         |                               |                               |
         v                               v                               v
+-------------------+           +-------------------+           +-------------------+
|  Security Headers |           |    CORS Policy    |           | Origin Guard      |
|  Helmet (No CSP)  |           | CLIENT_ORIGINS    |           | (CSRF Protection) |
+-------------------+           +-------------------+           +-------------------+
                                         |
                                         v
         +---------------------------------------------------------------+
         |                      API ROUTE CONTROLLERS                    |
         +---------------------------------------------------------------+
         |  - /api/health           (Database readiness probe)           |
         |  - /api/auth/register    (Zod validate -> bcrypt -> JWT)      |
         |  - /api/auth/login       (Zod validate -> bcrypt verify -> JWT|
         |  - /api/auth/demo        (Development fast-login)             |
         |  - /api/auth/me          (Current session account profile)    |
         |  - /api/auth/password    (Current pass verify -> new hash)    |
         |  - /api/bootstrap        (Full user relational graph load)    |
         |  - /api/state            (Transactional full state replace)   |
         |  - /api/exercises        (Public catalog search & filter)     |
         |  - /api/workouts         (Public workouts with exercise counts|
         |  - /api/account/premium  (Demo role upgrade/downgrade)        |
         |  - /api/admin/*          (RBAC protected admin endpoints)     |
         +---------------------------------------------------------------+
                                         |
                                         v
                    +------------------------------------------+
                    |          server/state.js                 |
                    |  - bootstrap() : Parallel SELECT queries |
                    |  - syncState() : Atomic ACID Transaction |
                    +------------------------------------------+
                                         |
                                         v
                    +------------------------------------------+
                    |          server/db.js (pg.Pool)          |
                    +------------------------------------------+
```

### Server Entry Point (`server/index.js`)
Initializes the database pool, creates the Express application, binds to `process.env.PORT || 3000`, and attaches `SIGINT` and `SIGTERM` handlers to cleanly close open HTTP connections and drain the PostgreSQL pool.

### Database Pool & Transaction Manager (`server/db.js`)
* Configures `pg.types.setTypeParser` for type 20 (`int8`) and type 1700 (`numeric`) to automatically coerce PostgreSQL numeric and bigint values into JavaScript `Number` instances rather than strings.
* Exposes `transaction(db, work)`: Acquires a client from the pool, runs `BEGIN`, invokes the async `work(client)` callback, runs `COMMIT`, handles `ROLLBACK` on errors, and guarantees client release in a `finally` block.

### Transactional State Synchronization (`server/state.js`)
The `syncState` function implements full-graph state synchronization:
1. **Relational Integrity Verification**: Validates that all goal IDs, plan IDs, workout IDs, session IDs, and chat IDs referenced by sub-records strictly belong to the authenticated user.
2. **Atomic Replacement**:
   ```sql
   UPDATE users SET name=$2, email=$3, ... WHERE user_id=$1;
   DELETE FROM chat_sessions WHERE user_id=$1;
   DELETE FROM workout_sessions WHERE user_id=$1;
   DELETE FROM workout_plans WHERE user_id=$1;
   DELETE FROM fitness_goals WHERE user_id=$1;
   DELETE FROM workouts WHERE user_id=$1 AND is_public=false;
   DELETE FROM weight_history WHERE user_id=$1;
   -- Followed by sequential re-insertion of updated child records
   ```
3. Guarantees transactional consistency, preventing orphaned foreign key rows.

---

## 11. API DOCUMENTATION

All API endpoints are prefixed with `/api`.

| Method | Endpoint | Authentication | Request Body / Query Params | Response Schema | Handler File & Lines | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `GET` | `/api/health` | None | None | `{ status: "ready", database: "postgresql" }` | `server/app.js:15` | Health check & DB readiness probe |
| `POST` | `/api/auth/register` | None | `{ name, email, password, gender?, dateOfBirth?, height, weight }` | `201 Created`: `{ user: PublicUser }` + Cookie | `server/app.js:16` | Registers account, creates FreeUser record, sets JWT |
| `POST` | `/api/auth/login` | None | `{ email, password }` | `200 OK`: `{ user: PublicUser }` + Cookie | `server/app.js:17` | Validates credentials via bcrypt, sets JWT cookie |
| `POST` | `/api/auth/demo` | None / Dev | `{ role: "user" \| "admin" }` | `200 OK`: `{ user: PublicUser }` + Cookie | `server/app.js:18` | Fast login as demo user 101 or admin 1 |
| `POST` | `/api/auth/logout` | None | None | `204 No Content` (Clears `fittrack_session` cookie) | `server/app.js:19` | Invalidates client session cookie |
| `GET` | `/api/auth/me` | JWT Cookie / Bearer | None | `200 OK`: `{ user: PublicUser }` | `server/app.js:20` | Returns current user's profile |
| `POST` | `/api/auth/password`| JWT Cookie / Bearer | `{ currentPassword, newPassword }` | `200 OK`: `{ updated: true }` | `server/app.js:21` | Verifies current password, hashes new password |
| `GET` | `/api/bootstrap` | JWT Cookie / Bearer | None | `200 OK`: Complete relational graph snapshot | `server/app.js:22` | Hydrates client datastore with user and public rows |
| `PUT` | `/api/state` | JWT Cookie / Bearer | `{ user, goals, plans, planWorkouts, workouts, ... }` | `200 OK`: `{ syncedAt: ISOString }` | `server/app.js:23` | Transactionally syncs user's client state to DB |
| `POST` | `/api/account/premium-demo`| JWT Cookie / Bearer | None | `200 OK`: `{ role: "premium" }` | `server/app.js:24` | Upgrades role to premium, adds `premium_users` row |
| `DELETE`| `/api/account/premium-demo`| JWT Cookie / Bearer| None | `200 OK`: `{ role: "free" }` | `server/app.js:25` | Downgrades role to free, removes `premium_users` row|
| `GET` | `/api/exercises` | None | `?muscle=...&equipment=...&difficulty=...&q=...` | `200 OK`: `Array<Exercise>` | `server/app.js:26` | Searches public exercises with SQL ILIKE |
| `GET` | `/api/workouts` | None | None | `200 OK`: `Array<Workout & {exercise_count}>` | `server/app.js:27` | Returns public workout templates with exercise counts |
| `GET` | `/api/admin/stats` | Admin JWT Cookie | None | `200 OK`: `{ users, sessions, exercises, messages }` | `server/app.js:28` | Aggregated system metrics |
| `GET` | `/api/admin/users` | Admin JWT Cookie | None | `200 OK`: `Array<{ user_id, name, email, role, ... }>`| `server/app.js:29` | User listing up to 500 rows |
| `PATCH`| `/api/admin/users/:id`| Admin JWT Cookie| `{ role: "free" \| "premium" \| "admin" }` | `200 OK`: `{ user_id, name, email, role }` | `server/app.js:30` | Updates role of target user account |

---

## 12. DATABASE / DATA MODEL

The database schema is defined in `server/migrations/001_initial.sql`.

### Entity-Relationship Diagram

```mermaid
erDiagram
    users ||--o| free_users : "specializes as"
    users ||--o| premium_users : "specializes as"
    users ||--o| admins : "specializes as"
    users ||--o{ fitness_goals : "owns"
    users ||--o{ workout_plans : "creates"
    users ||--o{ workouts : "creates custom"
    users ||--o{ workout_sessions : "performs"
    users ||--o{ chat_sessions : "conducts"
    users ||--o{ weight_history : "logs"

    workout_plans ||--o{ plan_workouts : "contains daily"
    fitness_goals ||--o{ workout_plans : "linked to"

    workouts ||--o{ plan_workouts : "assigned in"
    workout_categories ||--o{ workouts : "categorizes"
    workout_categories ||--o{ exercises : "groups"

    workouts ||--o{ workout_exercises : "specifies"
    exercises ||--o{ workout_exercises : "prescribed in"

    workouts ||--o{ workout_sessions : "executed in"
    workout_plans ||--o{ workout_sessions : "guided by"

    workout_sessions ||--o{ session_exercises : "records"
    exercises ||--o{ session_exercises : "performed in"

    chat_sessions ||--o{ chat_messages : "contains"

    users {
        BIGSERIAL user_id PK
        VARCHAR name
        VARCHAR email UK
        TEXT password_hash
        VARCHAR role
        VARCHAR gender
        DATE date_of_birth
        NUMERIC height
        NUMERIC weight
        TEXT profile_image
        BIGINT active_plan_id FK
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    fitness_goals {
        BIGINT goal_id PK
        BIGINT user_id FK
        VARCHAR goal_type
        NUMERIC start_value
        NUMERIC current_value
        NUMERIC target_value
        VARCHAR unit
        DATE target_date
        TEXT description
        TIMESTAMPTZ created_at
    }

    workout_plans {
        BIGINT plan_id PK
        BIGINT user_id FK
        VARCHAR title
        TEXT description
        BIGINT goal_id FK
        DATE start_date
        DATE end_date
        BOOLEAN is_custom
        TIMESTAMPTZ created_at
    }

    plan_workouts {
        BIGINT plan_workout_id PK
        BIGINT plan_id FK
        BIGINT workout_id FK
        SMALLINT day_number
        VARCHAR notes
    }

    workouts {
        BIGINT workout_id PK
        BIGINT user_id FK
        VARCHAR title
        TEXT description
        SMALLINT duration
        VARCHAR difficulty
        TEXT equipment
        BIGINT category_id FK
        BOOLEAN is_public
        VARCHAR image_key
        VARCHAR goal
        TIMESTAMPTZ created_at
    }

    exercises {
        BIGINT exercise_id PK
        VARCHAR name
        TEXT description
        VARCHAR muscle_group
        VARCHAR equipment
        VARCHAR difficulty
        BIGINT category_id FK
        TEXT instruction
        VARCHAR image_key
        TEXT video_url
        TEXT poster_url
        TEXT video_source
        VARCHAR video_license
        TIMESTAMPTZ created_at
    }

    workout_exercises {
        BIGINT workout_ex_id PK
        BIGINT workout_id FK
        BIGINT exercise_id FK
        SMALLINT sets
        VARCHAR reps
        SMALLINT rest_time
        SMALLINT order_index
    }

    workout_sessions {
        BIGINT session_id PK
        BIGINT user_id FK
        BIGINT plan_id FK
        BIGINT workout_id FK
        VARCHAR title
        TIMESTAMPTZ started_on
        TIMESTAMPTZ completed_on
        SMALLINT duration
        NUMERIC calories_burned
        TEXT notes
        VARCHAR status
        SMALLINT exercise_index
        BIGINT rest_until
    }

    session_exercises {
        BIGINT set_id PK
        BIGINT session_id FK
        BIGINT exercise_id FK
        SMALLINT set_number
        VARCHAR reps
        NUMERIC weight
        BOOLEAN completed
        SMALLINT rest_time
    }

    chat_sessions {
        BIGINT session_id PK
        BIGINT user_id FK
        TIMESTAMPTZ started_on
        VARCHAR title
    }

    chat_messages {
        BIGINT message_id PK
        BIGINT session_id FK
        VARCHAR sender
        TEXT message
        TIMESTAMPTZ sent_on
    }

    weight_history {
        BIGSERIAL weight_entry_id PK
        BIGINT user_id FK
        NUMERIC value
        TIMESTAMPTZ recorded_at
    }
```

---

## 13. AUTHENTICATION & AUTHORIZATION

FitTrack features dual authentication implementations:

### 1. Browser-Local Offline Authentication (`js/store.js:17-19`)
* Non-demo accounts store salted hashes in `localStorage`:
  $$\text{hash} = \text{SHA-256}(\text{salt} \parallel \text{password})$$
* Computed using native browser `window.crypto.subtle.digest('SHA-256', ...)`.
* Passwords are never stored in plaintext. Exports (`F.exportData`) omit `passwordHash` and `salt`.
* Demo accounts (`Rahul Mehta` and `FitTrack Admin`) bypass password validation for easy evaluation.

### 2. Production PostgreSQL + Express Authentication (`server/app.js:11-21`)
* Passwords hashed using `bcryptjs` with cost factor 12.
* Authentication generates a signed JSON Web Token:
  ```javascript
  jwt.sign({ sub: String(userId) }, secret, { expiresIn: '7d', issuer: 'fittrack' });
  ```
* Token dispatched in an HTTP-Only, SameSite=Strict cookie named `fittrack_session`.
* Alternatively accepts `Authorization: Bearer <token>` for programmatic API requests.
* Protected endpoints enforce role-based access:
  * `auth` middleware verifies token validity and checks user existence in PostgreSQL.
  * `admin` middleware enforces `req.user.role === 'admin'`.

### Authentication Flow Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Client as Browser Client
    participant API as Express API (/api/auth)
    participant Bcrypt as bcryptjs
    participant JWT as jsonwebtoken
    participant DB as PostgreSQL (users table)

    Client->>API: POST /api/auth/login { email, password }
    API->>DB: SELECT * FROM users WHERE email = $1
    DB-->>API: User Record (user_id, password_hash, role)
    API->>Bcrypt: bcrypt.compare(password, password_hash)
    alt Password Invalid
        Bcrypt-->>API: false
        API-->>Client: 401 Unauthorized { error: "Email or password is incorrect." }
    else Password Valid
        Bcrypt-->>API: true
        API->>JWT: jwt.sign({ sub: userId }, JWT_SECRET, { expiresIn: '7d' })
        JWT-->>API: Signed JWT Token
        API-->>Client: 200 OK { user: PublicUser } + Set-Cookie: fittrack_session=JWT; HttpOnly; SameSite=Strict
    end
```

---

## 14. SECURITY ARCHITECTURE

| Mechanism | Status | Implementation Details | Evidence |
| :--- | :--- | :--- | :--- |
| **Authentication** | `Implemented` | Dual: Client Web Crypto SHA-256 (demo) & Server bcrypt (12 rounds) + JWT | `js/store.js:17`, `server/app.js:11,16` |
| **Authorization (RBAC)** | `Implemented` | `free`, `premium`, `admin` roles checked at middleware and service layers | `server/app.js:14,28-30`, `js/api.js:9,14` |
| **Input Validation** | `Implemented` | Server-side Zod validation with strict bounds; client HTML5 + regex validation | `server/validation.js:1-7`, `js/api.js:14-17` |
| **Output Encoding (XSS)** | `Implemented` | String escaping utility `F.escape()` sanitizes `&`, `<`, `>`, `"`, `'` | `js/components.js:2` |
| **CSRF Protection** | `Implemented` | Custom origin-check middleware rejects non-GET requests with disallowed origin; SameSite=Strict cookies | `server/app.js:10,12` |
| **SQL Injection** | `Implemented` | 100% parameterized queries (`$1, $2, ...`) via `node-postgres` | `server/app.js`, `server/state.js` |
| **Path Traversal** | `Implemented` | Static server verifies path resolution remains inside project root | `scripts/serve.cjs:10-11` |
| **CORS Policy** | `Implemented` | Whitelists origins defined in `CLIENT_ORIGINS` environment variable | `server/app.js:8-9` |
| **Content Security Policy** | `Missing` | Explicitly disabled in Helmet (`contentSecurityPolicy: false`) | `server/app.js:9` |
| **Security Headers** | `Implemented` | Helmet sets X-Content-Type-Options, Strict-Transport-Security, etc. | `server/app.js:9` |
| **Secrets Management** | `Partial` | Uses `.env` with fallback; server refuses default secret in production | `server/app.js:7` |
| **Rate Limiting** | `Missing` | No rate limiting middleware on auth routes or coach endpoints | Inspected `server/app.js` |
| **Error Exposure** | `Implemented` | Production hides stack traces and returns generic message | `server/app.js:33` |
| **File Upload Security** | `Partial` | Profile images restricted to JPG/PNG/WebP, validated client-side (<1MB) and server-side (<1.5MB) | `js/management.js:64`, `server/validation.js:5` |

---

## 15. ENVIRONMENT VARIABLES & CONFIGURATION

| Variable | Purpose | Required | Used In | Example / Default | Secret? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `DATABASE_URL` | PostgreSQL connection connection string | Yes (for backend) | `server/db.js:5` | `postgresql://fittrack:fittrack@127.0.0.1:5432/fittrack` | Yes |
| `DATABASE_SSL` | Enables SSL/TLS for PostgreSQL connections | No | `server/db.js:7` | `false` or `true` | No |
| `DATABASE_SSL_REJECT_UNAUTHORIZED`| Enforces strict CA certificate validation | No | `server/db.js:7` | `true` or `false` | No |
| `DB_POOL_SIZE` | Maximum connections in `pg.Pool` | No | `server/db.js:7` | `10` | No |
| `JWT_SECRET` | Secret key used to sign and verify session JWTs | Yes (in prod) | `server/app.js:6` | `<REDACTED>` (Min 32 characters) | Yes |
| `CLIENT_ORIGINS` | Comma-delimited list of allowed CORS origins | No | `server/app.js:8` | `http://127.0.0.1:3000,http://localhost:3000,http://127.0.0.1:5173` | No |
| `PORT` | TCP port for Express HTTP server | No | `server/index.js:2`| `3000` | No |
| `NODE_ENV` | Runtime environment mode | No | `server/app.js:6` | `development` or `production` | No |
| `ENABLE_DEMO_ACCOUNTS` | Enables passwordless demo sign-in in production | No | `server/app.js:18`| `false` or `true` | No |

---

## 16. THIRD-PARTY SERVICES & INTEGRATIONS

| Service | Purpose | Integration Files | Authentication | Data Sent | Data Received | Failure Handling |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Google Fonts** | Typography (Barlow Condensed & DM Sans) | `index.html:10-12` | None | Font request HTTP headers | WOFF2 font binaries | System font fallback (`Arial Narrow`, `Impact`, `Arial`) |
| **free-exercise-db** | Exercise movement demonstration frames | `scripts/generate-exercise-videos.cjs`| None (Public GitHub) | HTTP GET requests | JPEG image frames & UNLICENSE.md | Throws error during manual script execution; offline files used at runtime |
| **Unsplash** | Editorial gym photography assets | `docs/ASSETS.md`, `assets/*.jpg` | None (Local files) | None | Offline JPEG images | All photos stored locally in `assets/`; no runtime network call |
| **WebMCP / ModelContext**| Browser AI agent tooling | `js/app.js:42` | Feature Detection | Tool definition (`start_fittrack_workout`) | Tool execution call | Feature detected via `document.modelContext?.registerTool`; silently skipped if absent |
| **PostgreSQL Database** | Relational data persistence | `server/db.js` | Connection string credentials | SQL queries & data parameters | Relational row records | Express error handling logs failure; client reverts to `localStorage` |

---

## 17. BUSINESS LOGIC

### 1. Free Tier Quotas & Constraints (`js/store.js:21`, `js/api.js:12,14`)
* **Workout Tracking Quota**: Free accounts are limited to completing up to 3 workouts in a calendar day. Starting a 4th workout throws:
  `"You have reached the Free limit of 3 workouts today. Your next session is available tomorrow."`
* **Plan Quota**: Free accounts may save only 1 custom workout plan. Saving a 2nd plan throws:
  `"Free includes one saved plan. Edit your existing plan or try Premium demo."`
* **Coach Quota**: Free accounts are limited to 5 AI coach queries per calendar month. Exceeding throws:
  `"You have used the 5 Free coach messages for this month. Try Premium demo for more."`

### 2. Workout Execution & Metrics Calculation (`js/store.js:16,25`, `js/api.js:17`)
* **Session Duration**: $\max\left(1, \text{round}\left(\frac{\text{Date.now}() - \text{startedOn}}{60000}\right)\right)$ minutes.
* **Calorie Estimation**: Modeled as $6 \text{ kcal}$ burned per minute of active training:
  $$\text{Calories Burned} = \text{duration} \times 6$$
* **Volume Tonnage**: Sum of $(\text{weight} \times \text{reps})$ across all completed sets in completed sessions.
* **Personal Records (PRs)**: Maximum weight logged for any completed set of each exercise ID.
* **Streak Calculation**: Counts consecutive daily workouts by iterating backward from today (or yesterday if no session logged today).

### 3. Safety & Edit Integrity Locks (`js/finalize.js:8`, `js/api.js:16`)
* Workouts with an ongoing active session (`status: 'active'`) cannot be modified or deleted.

---

## 18. STATE MANAGEMENT

FitTrack employs a **Hierarchical Single Store Architecture** on the client:

```
[window.FitTrack.db] (Single Source of Truth)
 ├── users: Array<User>
 ├── freeUsers / premiumUsers / admins
 ├── goals: Array<FitnessGoal>
 ├── plans: Array<WorkoutPlan>
 ├── planWorkouts: Array<PlanWorkout>
 ├── workouts: Array<Workout>
 ├── workoutExercises: Array<WorkoutExercise>
 ├── categories: Array<WorkoutCategory>
 ├── sessions: Array<WorkoutSession>
 ├── sessionExercises: Array<SessionExercise>
 ├── coaches: Array<AICoach>
 ├── chatSessions / chatMessages
 ├── weightHistory: Array<{userId, value, date}>
 ├── settings: { reducedMotion: boolean }
 ├── contactMessages: Array<SupportMessage>
 ├── activeSessionId: number | null
 └── currentUserId: number | null
```

* **Local Persistence**: Any mutation invokes `F.save()`, which serializes the store to `localStorage` under `fittrack.v1`.
* **State Bridge Synchronization**: `js/backend-client.js` wraps `F.save()`. When `B.online` and `B.authenticated` are true, it initiates a 350ms debounced synchronization request (`PUT /api/state`). If a sync is already inflight, a `pending` flag queues an immediate follow-up sync.

---

## 19. DATA FLOW TRACES

### Trace A: User Registration & Initial Plan Activation
```text
User Submits Registration Form (js/pages.js)
  ↓
F.register(values) invoked (js/store.js / js/backend-client.js)
  ↓
[If Online] POST /api/auth/register -> bcrypt -> INSERT users, free_users -> JWT Set-Cookie
[If Offline] Web Crypto SHA-256(salt + password) -> Push to F.db.users -> localStorage
  ↓
F.db.currentUserId updated -> F.save()
  ↓
Router navigates to '#/dashboard'
  ↓
F.pages.dashboard renders welcome state & daily workout
```

### Trace B: Live Workout Execution & State Sync
```text
User Clicks "Start Workout" on Push Day (801)
  ↓
F.startWorkout(801) creates WorkoutSession (status: 'active') and clones WorkoutExercises into SessionExercises
  ↓
F.db.activeSessionId set -> Redirect to '#/session/:id'
  ↓
Athlete Enters Weight (60kg) & Reps (10) -> Clicks "Complete Set"
  ↓
F.logSet(setId, 60, 10) sets completed: true, restUntil: Date.now() + 60s
  ↓
F.save() writes to localStorage -> starts 350ms debounce timer
  ↓
Athlete Finishes Workout -> F.completeSession() computes duration & calories -> marks 'completed'
  ↓
Client dispatches PUT /api/state with full user graph
  ↓
PostgreSQL transaction deletes existing user records and commits updated sessions & sets
```

---

## 20. ERROR HANDLING

* **Client Form Validation**: Forms catch errors and display inline error messages in `.form-error` elements.
* **Toast Notification Fallback**: Unexpected exceptions trigger floating toasts via `F.toast(message)`.
* **Server Status Codes**:
  * `400 Bad Request`: Zod validation errors, schema violations, password length mismatch.
  * `401 Unauthorized`: Missing session cookie, expired JWT, invalid login credentials.
  * `403 Forbidden`: Admin role required, origin not allowed, demo accounts disabled.
  * `404 Not Found`: Account or target resource not found.
  * `409 Conflict`: Duplicate email registration (Postgres error 23505), foreign key constraint violations.
  * `500 Internal Server Error`: Unhandled server exceptions (masked in production).

---

## 21. PERFORMANCE ARCHITECTURE

* **Zero Framework Overhead**: The client loads zero JavaScript libraries (no React, Vue, jQuery, or Lodash). Full initial JS bundle is under 150 KB uncompressed.
* **Fast Time to Interactive (TTI)**: Since HTML is pre-rendered via template literals directly to the DOM, time-to-first-paint is near-instantaneous.
* **Lazy Loading & Video Autoplay Optimization**:
  * Images use native `loading="lazy"` (except the hero image which uses `fetchpriority="high"`).
  * Video elements use `IntersectionObserver` in `js/backend-client.js:36`: videos automatically pause when scrolled out of viewport and resume when visible.
* **Database Connection Pooling**: Configured with `pg.Pool` (default 10 connections, 30s idle timeout). Bigint and numeric fields are parsed directly in the C++ layer.

---

## 22. TESTING AUDIT

Automated testing is executed via Node's native test runner (`node --test`).

```bash
npm test
```

### Verified Test Suites
1. **`tests/core.test.cjs`**:
   * Seed relationship integrity (Rahul Mehta, PPL plan, workout assignments).
   * Password hashing, registration, login, and account isolation in VM context.
   * Session completion constraints, volume accumulation, cancel exclusion.
   * Plan schedule atomic replacement and Free tier limits.
   * Workout builder reordering, foreign key detachment on deletion.
   * Bodyweight progress tracking and week boundary edge cases.
   * Full HTML rendering assertions for all 18 primary views.
2. **`tests/backend.test.cjs`**:
   * PostgreSQL schema execution, tables, foreign keys using `pg-mem`.
   * Authentication workflow, cookie dispatch, profile bootstrap.
   * Transactional state synchronization (`PUT /api/state`).
   * Foreign key violation rejections and cross-account isolation.
   * Exercise media integrity (verifies all 16 MP4s and poster JPEGs exist and exceed 1 KB).
3. **`tests/muscle-explorer.test.cjs`**:
   * Muscle group exercise queries (primary and compound mappings).
   * Graceful handling of empty muscle groups.
   * Accessibility markup verification (ARIA labels, roles, button keyboard tags).

### Critical Untested Areas
* Multi-user concurrent race conditions during `PUT /api/state` bulk synchronization.
* Browser storage exhaustion (`localStorage` quota exceedance).
* Mobile touch gesture handling and viewport rotation under active workout tracking.

---

## 23. BUILD, RUN & DEPLOYMENT

### Commands Discovered from Repository

```bash
# 1. Install dependencies
npm install

# 2. Start static development server (Port 5173 - Zero DB required)
npm start
# or
npm run dev

# 3. Start PostgreSQL container via Docker Compose
docker compose up -d postgres

# 4. Run database migrations
npm run db:migrate

# 5. Seed database with initial users and exercises
npm run db:seed

# 6. Start full Express + PostgreSQL backend server (Port 3000)
npm run server

# 7. Run automated test suites
npm test

# 8. Assemble production static bundle into dist/
npm run build

# 9. Regenerate exercise demonstration MP4 loops using ffmpeg
npm run media:generate
```

### Production Deployment
* **Static SPA Hosting**: Configured in `.openai/hosting.json` pointing to `dist/`. Suitable for Cloudflare Pages, Vercel, Netlify, or AWS S3/CloudFront.
* **Full-Stack Container Deployment**: `Dockerfile` packages the Node 22 runtime, installs production dependencies (`npm ci --omit=dev`), and executes `server/index.js`. Connects to managed PostgreSQL (AWS RDS, Cloud SQL, Neon, Supabase) via `DATABASE_URL`.

---

## 24. CI/CD & DEVOPS

* **Container Infrastructure**: Production `Dockerfile` uses lightweight `node:22-alpine` base image.
* **Local Multi-Container Environment**: `docker-compose.yml` provisions PostgreSQL 17 with healthcheck (`pg_isready`) and persistent named volume `fittrack-postgres`.
* **CI Pipelines**: `Missing` — No GitHub Actions (`.github/workflows`) or GitLab CI configurations exist in the repository root.

---

## 25. OBSERVABILITY

* **Application Logging**: Basic stdout logging for server boot and unhandled errors via `console.error(error)`.
* **Health Check**: `GET /api/health` probes database connectivity (`SELECT 1`).
* **Client Diagnostics**: Visual badge injected into navigation and auth panels (`POSTGRESQL CONNECTED` vs `LOCAL DEMO`).
* **Missing Observability**: No structured JSON logger (e.g. Pino/Winston), OpenTelemetry tracing, Prometheus metrics, or Sentry error capture.

---

## 26. SEO, ACCESSIBILITY & WEB QUALITY

### SEO Implementation
* Comprehensive meta tags in `index.html`: `viewport`, `theme-color: #0b0807`, `description`.
* Descriptive page `<title>`: `FitTrack — Find your stronger.`.
* Semantic HTML5 landmark structure: `<header>`, `<main id="main">`, `<nav>`, `<section>`, `<article>`, `<footer>`.

### Accessibility (a11y)
* **Skip Link**: `<a class="skip-link" href="#main">Skip to content</a>` implemented in `index.html:32` and styled in `css/styles.css:2`.
* **Keyboard Navigation**: Muscle Explorer body map paths support `tabindex="0"`, `role="button"`, and listen for `Enter` and `Space` key presses (`js/muscle-explorer.js:18`).
* **Screen Reader Attributes**: Modals use `<dialog>` with `aria-labelledby="modal-title"`. Notifications use `role="status"` and `aria-live="polite"`. Buttons without visible text include explicit `aria-label` tags.
* **Reduced Motion**: Full support for CSS `@media (prefers-reduced-motion: reduce)` and in-app settings toggle disabling CSS ticker animations and transitions (`css/styles.css:11`, `js/app.js:26`).

---

## 27. CODE QUALITY AUDIT

### Code Quality Assessment: `Acceptable` to `Good`
* **Strengths**: Exceptionally lean implementation, zero dependency bloat, clean domain modeling, robust parameterized SQL, 100% passing test coverage.
* **Weaknesses**:
  * **Monkey-Patching Cascade**: Scripts dynamically mutate and wrap functions declared by earlier scripts (e.g., `finalize.js` monkey-patches `F.pages.history` and `F.saveWorkout`; `management.js` wraps `F.pages.dashboard`).
  * **Event Listener Shadowing**: `management.js` calls `e.stopImmediatePropagation()` on `chat-form` and `builder` events, causing shadowed implementations in `app.js` to become dead code.
  * **Minified / Dense Formatting**: Several server files (`server/state.js`, `server/app.js`) are written in compressed multi-statement single lines, hampering readability.

---

## 28. DEAD CODE & UNUSED ASSETS

### Confirmed Unused Assets
1. `assets/anatomy-back.png` (629 KB): Muscular anatomy diagram; unreferenced in code.
2. `assets/anatomy-reference.png` (754 KB): Muscular anatomy diagram; unreferenced in code.
3. `assets/exercise-source/exercises.json` (1 MB): Upstream raw exercise database dump; unreferenced in application runtime (code uses `assets/exercise-media/*`).

### Confirmed Shadowed / Dead Code
1. `coachReply` function in `js/app.js:12-13`: Shadowed by `F.coachAdapter.reply` in `js/api.js:13`.
2. `sendChat` function in `js/app.js:13`: Shadowed by `F.sendCoach` in `js/management.js:35` due to `stopImmediatePropagation()` on form submission.
3. Chat submission handler in `js/app.js:21`: Never reached because `management.js:56` intercepts `chat-form`.

---

## 29. DEPENDENCY AUDIT

### Runtime Dependencies
* `express` (`^5.2.1`): Core web framework.
* `pg` (`^8.23.0`): PostgreSQL client driver.
* `zod` (`^4.5.4`): Runtime schema validation.
* `bcryptjs` (`^3.0.3`): Pure JavaScript password hashing.
* `jsonwebtoken` (`^9.0.3`): JWT issuance and verification.
* `helmet` (`^8.3.0`): Security response headers.
* `cors` (`^2.8.6`): CORS origin control.
* `cookie-parser` (`^1.4.7`): Cookie parsing.
* `dotenv` (`^17.4.2`): Environment variable loader.

### Development Dependencies
* `ffmpeg-static` (`^5.3.0`): Static FFmpeg binary for exercise media loop compilation.
* `pg-mem` (`^3.0.14`): In-memory PostgreSQL instance for zero-dependency test execution.

---

## 30. TECHNICAL DEBT

| Issue | Severity | Impact | Location | Why It Matters | Suggested Direction |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Bulk Replace State Sync** | `High` | Concurrent writes clobber data | `server/state.js:10-31` | Deleting and re-inserting entire user datasets causes multi-tab race conditions | Refactor to delta/patch updates (`PATCH /api/sessions`, etc.) |
| **Script Chaining via Globals** | `Medium` | Fragile load order | `js/*.js`, `index.html` | Relies on exact script tag sequence and monkey-patching on `window.FitTrack` | Migrate to standard ES Modules (`import`/`export`) with Vite bundler |
| **Base64 Photo Uploads** | `Medium` | Memory and DB payload bloat | `js/management.js:64`, `server/validation.js:5` | Images stored as 1.5MB Base64 strings inside PostgreSQL text column | Move to dedicated multipart S3/Cloud Storage object upload |
| **Missing Auth Rate Limiting** | `High` | Vulnerable to brute force | `server/app.js:16-17` | Attackers can automate credential guessing attacks against `/api/auth/login` | Introduce `express-rate-limit` middleware |
| **Disabled CSP** | `Medium` | Increased XSS impact | `server/app.js:9` | Helmet's Content Security Policy is disabled | Define strict CSP whitelist for Google Fonts, media, and styles |

---

## 31. KNOWN BUGS, RISKS & WEAKNESSES

1. **State Sync Race Condition**:
   * **Location**: `server/state.js:18-28`
   * **Problem**: When a user operates in multiple tabs, debounced calls to `PUT /api/state` execute `DELETE FROM workout_sessions WHERE user_id=$1` and rewrite from client memory.
   * **Impact**: Workout history or sets logged in Tab A can be wiped out by an older state snapshot sent from Tab B.
   * **Remedy**: Introduce entity-level REST endpoints or optimistic concurrency versioning (`updated_at` / `version` check).

2. **Unused File Footprint in Distribution**:
   * **Location**: `assets/anatomy-*.png`, `assets/exercise-source/exercises.json`
   * **Problem**: Unused image and JSON dumps totaling >2.3 MB are copied into `dist/` by `scripts/build.cjs`.
   * **Impact**: Unnecessary build artifact bloat.
   * **Remedy**: Exclude unreferenced assets in `scripts/build.cjs`.

---

## 32. FEATURES INVENTORY

| Feature | Implemented? | Location | Dependencies | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Interactive Muscle Explorer** | `Fully Implemented` | `js/muscle-explorer.js` | SVG, `assets/muscle-body.png` | Front/back anatomical body map filtering 16 exercises |
| **16 Exercise Video Loops** | `Fully Implemented` | `assets/exercise-media/` | HTML5 `<video>`, MP4 loops | Looped demonstrations with IntersectionObserver autoplay |
| **Live Workout Tracker** | `Fully Implemented` | `js/workspace.js`, `js/app.js`| Clocks, timers, state store | Stopwatch, set logging, rest countdown timer |
| **Custom Workout Builder** | `Fully Implemented` | `js/management.js` | Form handlers, draft state | Reorderable exercises, custom sets, reps, rest intervals |
| **7-Day Plan Scheduler** | `Fully Implemented` | `js/workspace.js`, `js/management.js`| `F.savePlan` | Assigns workouts to Monday-Sunday with rest days |
| **Goal Tracking & Radial Rings**| `Fully Implemented` | `js/workspace.js` | Conic gradients | Visual progress percentage, metric targets, unit labels |
| **Volume & Streak Analytics** | `Fully Implemented` | `js/workspace.js`, `js/store.js`| `F.stats()` | Tonnage computation, consecutive days streak, charts |
| **Offline AI Coach** | `Fully Implemented (Mocked)`| `js/api.js:13` | Deterministic template matcher | Local rule-based coaching without external LLM API |
| **Data Export (JSON & CSV)** | `Fully Implemented` | `js/finalize.js`, `js/app.js` | Blob URL generation | Safe user export omitting password hashes |
| **Admin Operations Console** | `Fully Implemented` | `js/admin.js`, `server/app.js` | RBAC admin checks | 9 tabs managing users, catalog, categories, reports |
| **WebMCP Agent Tooling** | `Fully Implemented` | `js/app.js:42` | `document.modelContext` | Allows browser AI agents to trigger workouts |

---

## 33. UI / UX INVENTORY

* **Hero Section**: High-contrast typography (`TRAIN SMARTER. TRACK BETTER. GET STRONGER.`), gradient shadows, social proof avatar stack.
* **Dual Tickers**: Animated diagonal text ribbons running in opposite directions with reduced-motion support.
* **Radial Progress Rings**: Dynamic CSS conic gradients (`conic-gradient(var(--orange) calc(var(--progress)*1%), #2b211b 0)`).
* **Weekly Activity Bar Charts**: CSS flexbox column charts displaying relative daily training frequencies.
* **Live Rest Countdown**: Visual countdown display with progress indicator and instant skip button.
* **Mannequin Muscle Selector**: Custom charcoal bodysuit mannequins with anatomical SVG hitboxes and hover glow highlights.

---

## 34. IMPORTANT ALGORITHMS & COMPLEX LOGIC

### 1. Streak Computation Algorithm (`js/store.js:16`)
```javascript
const days = new Set(sessions.map(s => new Date(s.startedOn).toLocaleDateString('en-CA')));
let streak = 0, d = new Date(now);
if (!days.has(d.toLocaleDateString('en-CA'))) d.setDate(d.getDate() - 1);
while (days.has(d.toLocaleDateString('en-CA'))) {
  streak++;
  d.setDate(d.getDate() - 1);
}
```
* **Complexity**: $O(N + S)$ where $N$ is total sessions and $S$ is the streak length in days.
* **Logic**: Formats dates into ISO `YYYY-MM-DD` (`en-CA`), checks if today was logged; if not, allows yesterday to maintain streak continuity; iterates backward until a day is missed.

### 2. Anatomical SVG Clip-Path Mannequin Isolation (`js/muscle-explorer.js:8`)
```html
<clipPath id="body-cutout-front">
  <path d="M448 10 C417 10 410 34 414 76 ... Z"/>
</clipPath>
<image href="assets/muscle-body.png" width="1536" height="1024" clip-path="url(#body-cutout-front)"/>
```
* **Purpose**: Isolates the front and back mannequin figures from an RGB image asset with an opaque background, eliminating the halo effect without requiring browser canvas pixel processing.

---

## 35. IMPORTANT CODE SNIPPETS

### 1. Server Database Pool Initialization (`server/db.js:1-8`)
```javascript
const {Pool, types} = require('pg');
types.setTypeParser(20, value => Number(value));    // bigint to Number
types.setTypeParser(1700, value => Number(value));  // numeric to Number

function createPool() {
  const url = process.env.DATABASE_URL;
  if (!url) throw new Error('DATABASE_URL is required. Copy .env.example to .env and set the PostgreSQL connection string.');
  return new Pool({
    connectionString: url,
    ssl: process.env.DATABASE_SSL === 'true' ? { rejectUnauthorized: process.env.DATABASE_SSL_REJECT_UNAUTHORIZED !== 'false' } : false,
    max: Number(process.env.DB_POOL_SIZE || 10),
    idleTimeoutMillis: 30000
  });
}
```

### 2. Debounced State Synchronization Bridge (`js/backend-client.js:27-28`)
```javascript
async function sync() {
  if (B.syncing) { B.pending = true; return; }
  const body = statePayload();
  if (!B.online || !B.authenticated || !body) return;
  B.syncing = true;
  try {
    await request('/api/state', { method: 'PUT', body: JSON.stringify(body) });
    B.lastError = '';
  } catch (e) {
    if (!B.lastError) F.toast?.('Saved in this browser. Database sync failed: ' + e.message);
    B.lastError = e.message;
    clearTimeout(B.syncTimer);
    B.syncTimer = setTimeout(sync, 15000);
  } finally {
    B.syncing = false;
    if (B.pending) { B.pending = false; sync(); }
  }
}
F.save = () => {
  const ok = local.save();
  if (!B.suppress && B.online && B.authenticated) {
    clearTimeout(B.syncTimer);
    B.syncTimer = setTimeout(sync, 350);
  }
  return ok;
};
```

---

## 36. ARCHITECTURAL DECISION LOG

* **Decision: Vanilla JavaScript over React/Vue/Svelte**
  * *Status*: `Explicit Decision Found` (`README.md:1-5`, `docs/ARCHITECTURE.md:3`)
  * *Rationale*: Guarantees instant startup with zero build toolchain (`node_modules` or bundler not required for client demo).
* **Decision: Dual-Mode Storage (LocalStorage + PostgreSQL)**
  * *Status*: `Explicit Decision Found` (`docs/BACKEND.md:3`)
  * *Rationale*: Allows offline static evaluation for coursework presentations while providing production multi-user database persistence when hosted with Docker.
* **Decision: Hash-Based Routing (`#/route`)**
  * *Status*: `Explicit Decision Found` (`README.md:80`)
  * *Rationale*: Avoids HTTP 404 errors on static hosts lacking URL rewrite engines.
* **Decision: Deterministic Offline AI Coach Templates**
  * *Status*: `Explicit Decision Found` (`docs/ARCHITECTURE.md:44`, `docs/BACKEND.md:32`)
  * *Rationale*: Protects student privacy, eliminates external API costs, and enables fully offline demo operation.

---

## 37. PROJECT DEPENDENCY GRAPH

```
[Core Platform]
   ├── Client Layer (Browser)
   │     ├── Stylesheet Cascade (css/*.css)
   │     ├── Asset Catalog (assets/exercise-media, photography)
   │     ├── Modular Script Pipeline (js/data.js -> js/app.js)
   │     └── Local Datastore (localStorage: 'fittrack.v1')
   │
   ├── Backend Layer (Node.js)
   │     ├── Express 5 Framework (server/app.js)
   │     ├── Security Middleware (helmet, cors, cookie-parser)
   │     ├── Auth Engine (bcryptjs, jsonwebtoken)
   │     └── Schema Validation (zod)
   │
   ├── Database Layer (PostgreSQL)
   │     ├── Connection Pool (server/db.js)
   │     ├── DDL Migrations (server/migrations/001_initial.sql)
   │     └── Seed Harness (server/scripts/seed.js)
   │
   └── Tooling & Operations
         ├── Test Harness (node --test, pg-mem)
         ├── Build Automation (scripts/build.cjs)
         ├── Media Generation (scripts/generate-exercise-videos.cjs, ffmpeg-static)
         └── Containerization (Dockerfile, docker-compose.yml)
```

---

## 38. CRITICAL PATHS

1. **Client Bootstrap**: If `js/data.js` or `js/store.js` fails, the client datastore fails to mount and the application crashes into a blank screen.
2. **Database Connectivity**: If `DATABASE_URL` is unreachable, `server/index.js` fails health checks, causing `backend-client.js` to operate strictly in local mode.
3. **Session Logging & Active Workout Integrity**: If `F.startWorkout` or `F.logSet` throws an error, the athlete cannot record training sets, breaking streak and volume progression.

---

## 39. SINGLE POINTS OF FAILURE (SPOF)

* **PostgreSQL Database Instance**: In backend mode, there is no read-replica or multi-region failover configured.
* **JWT Secret Key (`JWT_SECRET`)**: If compromised, attackers can forge valid session tokens for any `user_id`.
* **LocalStorage Quota**: If an athlete logs hundreds of workouts with base64 profile pictures, browser `localStorage` may exceed its 5MB limit, throwing `QuotaExceededError`.

---

## 40. BACKUP, RECOVERY & RESILIENCE

* **Client Data Export**: Built-in JSON data export (`#/settings` -> "Export your data") downloads the complete user record (`fittrack-data.json`), scrubbed of password hashes.
* **History CSV Export**: History view provides direct CSV export for spreadsheet backups.
* **Offline Resilience**: If the Express API goes down during a workout session, `backend-client.js` catches the error, maintains data in `localStorage`, and retries syncing every 15 seconds.

---

## 41. PRIVACY & DATA HANDLING

* **Personal Data Handled**: Name, email, gender, date of birth, height (cm), weight (kg), workout history, and personal coach conversations.
* **Data Transmission**: In backend mode, transmitted over HTTPS/HTTP via JSON payloads.
* **Third-Party Telemetry**: Zero external tracking scripts, Google Analytics, or third-party tracking pixels exist in the codebase.
* **Retention & Deletion**: Users can purge all local data via "Reset prototype" in Settings. Admins can delete users, cascade-deleting related records in PostgreSQL.

---

## 42. CHANGE IMPACT MAP

* **Modifying `server/migrations/001_initial.sql`**:
  * Directly impacts `server/state.js` (queries will fail if columns change).
  * Directly impacts `server/scripts/seed.js` and `tests/backend.test.cjs`.
* **Modifying `js/store.js` or Store Key Version**:
  * Bumping `KEY` from `'fittrack.v1'` resets client storage for returning users unless migration logic is added.
* **Modifying `js/components.js` Component Markup**:
  * Impacts CSS class selectors across `css/styles.css` and `css/workspace.css`.
  * May break query selectors in `js/management.js` and `js/app.js`.

---

## 43. DEVELOPER ONBOARDING GUIDE

### Step-by-Step Setup
1. **Prerequisites**: Install Node.js (v20+ recommended) and Docker Desktop.
2. **Environment Configuration**:
   ```bash
   cp .env.example .env
   ```
3. **Start Database**:
   ```bash
   docker compose up -d postgres
   ```
4. **Initialize Schema & Seed**:
   ```bash
   npm run db:migrate
   npm run db:seed
   ```
5. **Start Dev Server**:
   ```bash
   npm run server
   ```
6. **Access Application**:
   * Open `http://127.0.0.1:3000` for full PostgreSQL backend.
   * Open `http://127.0.0.1:5173` via `npm start` for standalone static demo.
7. **Default Credentials**:
   * Athlete: `rahul@fittrack.demo` / `Demo123!`
   * Administrator: `admin@fittrack.demo` / `Demo123!`

---

## 44. "WHERE SHOULD I LOOK?" INDEX

| I Need to Modify... | Primary File Path | Secondary / Related Files |
| :--- | :--- | :--- |
| **Authentication & Sessions** | `server/app.js:11-21` | `js/store.js:17-19`, `js/backend-client.js:29-32` |
| **Database Schema & Tables** | `server/migrations/001_initial.sql` | `server/state.js`, `server/scripts/seed.js` |
| **Exercise Library & Videos** | `server/scripts/seed.js:4-8` | `assets/exercise-media/`, `js/data.js:5-22` |
| **Muscle Explorer Body Map** | `js/muscle-explorer.js` | `css/muscle-explorer.css`, `assets/muscle-body.png`|
| **Workout Execution Tracker**| `js/workspace.js:17` | `js/management.js:32`, `js/app.js:11,35` |
| **Workout Builder Form** | `js/management.js:18-19` | `js/api.js:15`, `css/management.css:1` |
| **Navigation & Shell Layout** | `js/workspace.js:3` | `js/components.js:8`, `css/workspace.css:1` |
| **Color Scheme & Fonts** | `css/styles.css:1-2` | `index.html:10-12` |
| **AI Coach Responses** | `js/api.js:13` | `js/workspace.js:14`, `js/management.js:35` |
| **Admin Operations Panel** | `js/admin.js` | `server/app.js:28-30` |
| **Automated Tests** | `tests/core.test.cjs` | `tests/backend.test.cjs`, `tests/muscle-explorer.test.cjs` |

---

## 45. MASTER ARCHITECTURE DIAGRAM

```mermaid
graph TD
    subgraph Users ["User Personas"]
        Visitor["Public Visitor"]
        FreeUser["Free Member (Rahul Mehta)"]
        PremiumUser["Premium Member"]
        AdminUser["System Administrator"]
    end

    subgraph Client ["Client Browser Runtime (index.html)"]
        UI["DOM Shell & CSS Theme (css/styles.css)"]
        Router["Hash Router (js/app.js)"]
        Views["Public & Workspace Views (js/pages.js, js/workspace.js)"]
        MuscleMap["Muscle Explorer SVG Mannequin (js/muscle-explorer.js)"]
        MediaDemo["MP4 Exercise Video Player (assets/exercise-media)"]
        Builder["Workout & Plan Builders (js/management.js)"]
        AdminConsole["Admin Operations Console (js/admin.js)"]
        Store["Single State Store & Web Crypto (js/store.js)"]
        SyncClient["Sync Client Adapter (js/backend-client.js)"]
        LocalDB[("Browser LocalStorage\nKey: fittrack.v1")]
    end

    subgraph API_Gateway ["Express 5 Application (server/app.js)"]
        CORS["CORS & Origin Guard"]
        Helmet["Helmet Security Headers"]
        AuthFilter["JWT Cookie Authentication Middleware"]
        RBAC["Admin Role Guard Middleware"]
        ZodVal["Zod Schema Validator (server/validation.js)"]
        StateService["State Sync Service (server/state.js)"]
    end

    subgraph Persistence ["PostgreSQL 17 Database"]
        UserTables[("Users & Subtypes\n(users, free_users, premium_users, admins)")]
        CatalogTables[("Exercise Catalog\n(workout_categories, exercises, workouts, workout_exercises)")]
        PlanTables[("Plans & Goals\n(fitness_goals, workout_plans, plan_workouts)")]
        SessionTables[("Sessions & History\n(workout_sessions, session_exercises, weight_history)")]
        ChatTables[("Coach Conversations\n(ai_coaches, chat_sessions, chat_messages)")]
    end

    Users --> UI
    UI --> Router
    Router --> Views
    Views --> MuscleMap
    Views --> MediaDemo
    Views --> Builder
    Views --> AdminConsole
    Views --> Store
    Builder --> Store
    AdminConsole --> Store
    Store --> LocalDB
    Store --> SyncClient

    SyncClient -- "JSON / HTTPS" --> CORS
    CORS --> Helmet
    Helmet --> AuthFilter
    AuthFilter --> ZodVal
    ZodVal --> StateService
    AuthFilter --> RBAC
    RBAC --> StateService

    StateService --> UserTables
    StateService --> CatalogTables
    StateService --> PlanTables
    StateService --> SessionTables
    StateService --> ChatTables
```

---

## 46. MASTER PROJECT STATUS DASHBOARD

| Functional Area | Status | Confidence | Architectural Notes |
| :--- | :--- | :--- | :--- |
| **Architecture** | `Healthy` | `Verified` | Dual-mode topology works seamlessly across offline static and PostgreSQL modes. |
| **Frontend** | `Healthy` | `Verified` | Vanilla JS SPA renders fast, zero bundle dependencies, clean DOM management. |
| **Backend** | `Healthy` | `Verified` | Express 5 with async error handling, Helmet, parameterized queries, Zod validation. |
| **Database** | `Healthy` | `Verified` | Comprehensive 17-table schema with foreign keys, CHECK constraints, composite indexes. |
| **Authentication**| `Healthy` | `Verified` | Production bcrypt (12 rounds) + HTTP-only SameSite=Strict JWT cookie auth. |
| **Security** | `Acceptable` | `Verified` | Parameterized SQL, CSRF origin checks, input validation; needs rate limiting & CSP. |
| **Testing** | `Healthy` | `Verified` | 100% test pass rate across 12 automated test suites in Node test runner. |
| **Deployment** | `Acceptable` | `Verified` | Dockerfile & Docker Compose present; static hosting config defined; CI pipeline missing. |
| **Documentation** | `Healthy` | `Verified` | Extensive documentation in `README.md`, `docs/`, and SE Lab experiments report. |
| **Technical Debt**| `Acceptable` | `Verified` | Functional monkey-patching in client scripts; bulk replace pattern in `syncState`. |

---

## 47. FINAL "BRAIN" SUMMARY

### Project in One Paragraph
FitTrack is an open-source, dual-mode fitness platform and academic software engineering capstone engineered with zero client-side dependencies (vanilla HTML5, modern CSS3, and modular ES2022 JavaScript). It delivers an offline-first single page application running from browser `localStorage` and Web Crypto SHA-256 hashing, while progressively synchronizing with an optional Express 5 REST API and PostgreSQL 17 database. The application features live set-by-set workout tracking with automated calorie ($6 \text{ kcal/min}$) and volume metrics, rest interval timers, a 7-day plan scheduler, a custom workout builder, an interactive SVG/canvas mannequin muscle explorer, 16 local frame-loop MP4 exercise demonstrations, deterministic AI coaching, and an administrative operations console.

### Project in 10 Lines
1. **Core Purpose**: Comprehensive fitness workspace integrating goal setting, workout planning, active session tracking, and exercise exploration.
2. **Dual Execution Mode**: Operates as a 100% standalone static SPA or as a containerized client-server web app backed by PostgreSQL.
3. **Frontend Stack**: Zero-framework vanilla ES2022 JavaScript, semantic HTML5, and bespoke CSS3 design tokens with dark-mode palette.
4. **Backend Stack**: Node.js, Express 5, `pg` connection pool with custom bigint/numeric type parsers, and Zod schema validation.
5. **Database Architecture**: PostgreSQL 17 relational database featuring 17 normalized tables, cascade constraints, and indexed queries.
6. **Authentication Architecture**: Dual auth — client-side Web Crypto SHA-256 for local demo; server-side bcrypt (12 rounds) + HTTP-only JWT cookies.
7. **Muscle Explorer**: Interactive front and back anatomical mannequin body maps using SVG clip-paths and path hitboxes.
8. **Exercise Media**: 16 frame-loop MP4 videos and JPEG posters compiled via FFmpeg from the public domain `free-exercise-db`.
9. **Academic Lineage**: Built to fulfill 11 software engineering lab experiments covering UML diagrams, Scrum planning, and test cases.
10. **Automated Verification**: 12 comprehensive unit and integration tests passing in Node's native test runner using `pg-mem`.

### Top 10 Critical Files
1. `server/app.js`: Express 5 application factory containing route definitions, security middleware, and authentication filters.
2. `server/state.js`: Database transaction service handling full bootstrap queries and atomic state synchronization.
3. `server/migrations/001_initial.sql`: Master PostgreSQL DDL migration creating all 17 relational tables, constraints, and indexes.
4. `js/backend-client.js`: Progressive enhancement bridge that probes backend health and debounces state sync to PostgreSQL.
5. `js/store.js`: Client-side datastore manager, Web Crypto hashing engine, and active workout session state tracker.
6. `js/muscle-explorer.js`: Interactive anatomical body map implementation using SVG path clipping and muscle group filtering.
7. `js/management.js`: Custom workout builder, 7-day schedule editor, modal controllers, and photo upload handlers.
8. `js/app.js`: Hash-based client router, live stopwatch intervals, toast notifications, and WebMCP tool registration.
9. `server/db.js`: PostgreSQL connection pool factory with custom bigint/numeric parser overrides and transaction wrappers.
10. `index.html`: Master SPA entry point defining the stylesheet cascade, container elements, and deterministic script load order.

### Top 10 Risks
1. **Bulk Delete-and-Insert Synchronization Race Condition**: Multi-tab operations can overwrite recent workout data.
2. **Missing Rate Limiting**: Authentication endpoints (`/api/auth/login`) are vulnerable to automated credential stuffing.
3. **Disabled Content Security Policy**: Helmet's CSP is disabled, increasing exposure if user input escaping fails.
4. **Base64 Photo Storage**: Uploaded avatars stored directly as Base64 strings in PostgreSQL risk database bloat.
5. **Client Script Monkey-Patching Fragility**: Global namespace function wrapping can break if script tags in `index.html` are reordered.
6. **Local Storage Quota Exceedance**: In static mode, intensive long-term workout logging can hit browser 5MB `localStorage` limits.
7. **Hardcoded Demo Accounts in Production**: If `ENABLE_DEMO_ACCOUNTS=true` is inadvertently left enabled in production, unauthorized access is possible.
8. **Single Point of Failure on Database**: Backend mode has no multi-AZ or read-replica database failover.
9. **Absence of CI/CD Automation**: No automated GitHub Actions or deployment pipelines exist to run tests before merging.
10. **Dead Code & Asset Bloat**: ~2.3 MB of unused anatomy images and JSON dumps are bundled into distribution artifacts.

### Top 10 Recommended Improvements
1. **Refactor Sync to Granular REST/Patch Endpoints**: Replace bulk `PUT /api/state` with discrete resource updates (`POST /api/sessions`, etc.).
2. **Implement Rate Limiting**: Add `express-rate-limit` on `/api/auth/login`, `/api/auth/register`, and `/api/auth/password`.
3. **Configure Strict Content Security Policy**: Enable Helmet CSP with whitelisted origins for Google Fonts and video media.
4. **Transition to ES Modules & Vite**: Replace global namespace script concatenation with modern ES module imports and bundling.
5. **Offload Photo Storage to Object Storage**: Use presigned S3/GCS URLs for avatar uploads rather than Base64 database strings.
6. **Implement Real AI LLM Adapter**: Connect `F.coachAdapter` to an external AI API (e.g. OpenAI / Gemini) using secure server-side proxying.
7. **Setup GitHub Actions CI Pipeline**: Create `.github/workflows/ci.yml` running `npm test` and `npm run build` on push and PR.
8. **Prune Unused Distribution Assets**: Exclude `anatomy-*.png` and `exercises.json` in `scripts/build.cjs` to reduce bundle size.
9. **Add Structured Logging & APM**: Integrate Pino or Winston alongside Prometheus or Sentry for production observability.
10. **Implement Optimistic Concurrency Control**: Add a `version` or `updated_at` column check on records to detect and prevent sync conflicts.
