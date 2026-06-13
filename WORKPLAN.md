# OrionVerse — 4-Person Phase-Wise Work Plan

## Team Roles

| Person | Role | Primary Domain |
|--------|------|----------------|
| **Dev A** | Frontend / 3D | React UI, Three.js rendering, CAD workspace, Wind Tunnel UI |
| **Dev B** | Physics / CFD / WASM | Web Workers, RK4 integrator, CFD solvers, thermal engine, mesh |
| **Dev C** | Backend / Infra | Express API, Socket.IO, BullMQ, auth, MongoDB, Cloudflare R2 |
| **Dev D** | AI / Learning / UX | AI mentor integration, Learning Hub, Risk Detection UI, UX polish |

---

## Phase 1 — Foundation Core (Month 0–4)

**Goal:** Working 3D builder, basic CAD, educational CFD, Earth orbit mission, auth, deploy.

### Dev A — Frontend / 3D
- [ ] Turborepo monorepo scaffold (`apps/web`, `apps/api`, `packages/*`)
- [ ] Vite + React + TypeScript + Tailwind + Framer Motion base setup
- [ ] Three.js scene scaffold — renderer, orbit controls, lighting, camera rig
- [ ] 3D Spacecraft Builder — drag-and-drop part placement, snap-to-grid, free mode
- [ ] Standard component library UI — 8 parts (engines, tanks, fairings, heat shields, legs, reaction wheels, solar panels, payload bays)
- [ ] Symmetry tools — bilateral and radial (N-fold)
- [ ] Real-time sidebar calculator — total mass, CoM, TWR, delta-V display
- [ ] Design snapshot UI — save named checkpoints, diff view between versions
- [ ] Basic CAD workspace shell — opencascade.js lazy-load, loading progress bar
- [ ] CAD 3D primitives UI — box/cylinder/sphere parameter inputs, extrude tool
- [ ] Mission HUD shell — altitude, velocity, Mach, G-force, fuel % panels
- [ ] Mission replay scrubber UI — frame slider, event bookmarks

**Milestone:** User can open the app, assemble a rocket from parts, and see a 3D scene.

---

### Dev B — Physics / CFD / WASM
- [ ] Validate opencascade.js WASM in Vite — boolean ops + STEP export on Day 1
- [ ] Vortex panel method CFD solver in Web Worker — prove <200ms re-solve
- [ ] Panel method: animated streamline data + Cd/Cl output
- [ ] CFD pressure contour data → postMessage to main thread (Three.js vertex colour)
- [ ] Subsonic / supersonic / hypersonic flight regime presets
- [ ] RK4 Physics Web Worker — Earth gravity, US Standard Atmosphere 1976 lookup table
- [ ] Orbital trajectory integration — launch, coast, LEO orbit insertion
- [ ] State vector postMessage at 60Hz: `{position, velocity, mass, attitude}`
- [ ] Basic re-entry: Fay-Riddell stagnation point heat flux computation
- [ ] Mission state recording at 10Hz — typed array ring buffer for replay

**Milestone:** Educational CFD solves in <200ms. Physics Worker runs LEO orbit at 60Hz.

---

### Dev C — Backend / Infra
- [ ] Express.js API setup — CORS, helmet, rate-limit, morgan logging
- [ ] MongoDB Atlas connection — Mongoose ODM, schema definitions
- [ ] Redis (Upstash) connection — ioredis
- [ ] JWT auth — register, login, refresh rotation (15min access / 7-day refresh httpOnly cookie)
- [ ] User CRUD + role middleware (`student | instructor | admin`)
- [ ] Spacecraft design CRUD API — save/load assembly JSON
- [ ] CAD project CRUD — save/load BREP refs to R2
- [ ] Cloudflare R2 integration — upload helper, presigned URL generation
- [ ] File upload endpoint — MIME validation, 50MB limit, virus scan stub
- [ ] Mission save endpoint — store outcome, telemetry URL, risk events
- [ ] GitHub Actions CI/CD — lint, tsc, test, deploy to Vercel + Railway
- [ ] Docker Compose local dev — `api`, `worker`, `cfd-server` containers

**Milestone:** Auth works. Designs and missions persist to MongoDB. R2 file upload live.

---

### Dev D — AI / Learning / UX
- [ ] Risk Detection system — monitors Physics Worker output for structural overload, fuel critical, trajectory divergence
- [ ] Risk Detection alert panel in HUD — non-intrusive notification, expand-on-click
- [ ] Alert severity classification UI — Critical (red) / Warning (amber) / Info (blue)
- [ ] Alert event log — timestamped list in mission record view
- [ ] Basic AI mentor integration — GPT-4o mini in-flight hints triggered by Risk Detection
- [ ] AI hint display component — card overlay in HUD, dismiss/expand
- [ ] Landing page + onboarding flow — feature introduction, "Start Building" CTA
- [ ] Auth pages — login, register, forgot password forms (Tailwind, accessible)
- [ ] User dashboard — saved designs list, mission history, account settings
- [ ] Responsive layout shell — sidebar navigation, top bar, panel system
- [ ] Sentry integration — frontend error boundary + source maps

**Milestone:** Risk alerts fire and display. AI hints appear on Critical alerts. Auth UI complete.

---

### Phase 1 Integration Checklist (all 4 together — Week 16)
- [ ] CFD pressure contour renders on spacecraft in Three.js scene
- [ ] Physics Worker drives spacecraft position in 3D scene during mission
- [ ] Thermal HUD panel shows stagnation temperature from Physics Worker
- [ ] Risk Detection fires from Physics Worker events, AI hint displays in HUD
- [ ] Mission replay plays back from stored state vectors
- [ ] Full E2E: register → build rocket → run CFD → launch to LEO → receive alert → replay

---

## Phase 2 — Full Engineering Platform (Month 4–9)

**Goal:** Full parametric CAD, engineering CFD, wind tunnel, multi-planet, full thermal, AI mentor.

### Dev A — Frontend / 3D
- [ ] Full CAD workspace — boolean ops UI (union/subtract/intersect), loft, revolve, sweep
- [ ] CAD parametric constraint panel — dimensional inputs that propagate on change
- [ ] CAD assembly system UI — joint definitions (rigid, revolute, slider, ball)
- [ ] CAD file I/O — STL/OBJ/STEP import, STL/OBJ/STEP export buttons
- [ ] 2D Konva.js schematic builder — lines, arcs, circles, splines, hatching, text, leader lines, title block
- [ ] 2D auto cross-section view — section plane selector → Konva 2D output
- [ ] 2D trajectory planner — top-down orbital map view linked to mission destination
- [ ] Assembly export to CAD — push 3D builder assembly into parametric CAD workspace
- [ ] Virtual Wind Tunnel scene — configurable tunnel geometry, free-stream display
- [ ] Wind Tunnel UI — AoA sweep controls, Cl/Cd polar curve chart, surface Cp colour map
- [ ] Wind Tunnel smoke trail particle system — Three.js particle emitters
- [ ] Wind Tunnel 3-variant split-screen — side-by-side Three.js viewports
- [ ] Multi-planet mission selector UI — destination picker with body info card
- [ ] Full thermal surface gradient map — Three.js vertex colour shader from Thermal Worker
- [ ] TPS material selector UI — dropdown + material spec card + Tmax indicator
- [ ] TPS comparison mode — split-screen two material configs on same trajectory
- [ ] CFD convergence monitor panel — residual plot chart, iteration counter
- [ ] Engineering CFD results panel — Cd/Cl/Cm readout, CSV/VTK download buttons
- [ ] Communication System HUD — propagation delay counter, ground station panel, link budget gauge, blackout indicator

**Milestone:** Full CAD works. Wind tunnel renders and sweeps. Multi-planet mission UI complete.

---

### Dev B — Physics / CFD / WASM
- [ ] Engineering CFD — FVM Navier-Stokes C++ → WASM via Emscripten compile pipeline
- [ ] Automatic unstructured tetrahedral mesh generator (OpenMesh WASM) — targets <100k cells
- [ ] k-omega SST turbulence model in FVM solver
- [ ] CFD convergence: residual array postMessage every 10 solver steps
- [ ] CFD results: pressure field, velocity field, Cd, Cl, Cm postMessage on convergence
- [ ] Large-mesh detection — if mesh >200k cells: emit `mesh_too_large` event to main thread
- [ ] Surface pressure coefficient (Cp) array output for Wind Tunnel Cp map
- [ ] AoA sweep automation — step AoA -20° to +20°, collect Cl/Cd per step, post polar curve data
- [ ] Force balance continuous output — axial force, normal force, pitching moment during Wind Tunnel solve
- [ ] Moon physics — exosphere (near-vacuum), 1.62 m/s² gravity, ballistic trajectory
- [ ] Mars physics — Mars GRAM 2010 CO₂ atmosphere, 3.72 m/s², dust storm hazard event triggers
- [ ] Venus physics — VIRA model 90 bar, 8.87 m/s², sulphuric acid cloud layer events, probe survival timer
- [ ] Titan physics — Titan GRAM dense N₂/CH₄, 1.35 m/s², parachute deployment, methane lake landing
- [ ] Full re-entry thermal — ablative shield model: recession rate, char layer thickness, cumulative mass loss
- [ ] Full surface thermal gradient map — 2D surface temperature array at 10Hz for Three.js shader
- [ ] Tmax breach detection — emit `tps_failure` Critical alert when surface temp exceeds material limit
- [ ] Cannon-es WASM — landing gear contact physics, surface impact, structural failure events
- [ ] Communication propagation delay — one-way light-time based on spacecraft-to-Earth distance
- [ ] Communication window calculation — ground station coverage arc from orbit geometry
- [ ] Plasma blackout detection during re-entry — emit `comm_blackout` event

**Milestone:** Engineering CFD converges <60s. All 5 planets simulate. Full ablation thermal model runs.

---

### Dev C — Backend / Infra
- [ ] BullMQ worker service — separate container; queues: `cfd-server`, `ai-report`, `mesh`, `email`
- [ ] Server-side CFD endpoint — `POST /api/cfd/submit` receives BREP, queues OpenFOAM job
- [ ] OpenFOAM Docker container — job runner, result → R2, completion via Socket.IO emit
- [ ] Socket.IO rooms — per-session isolation; CFD job progress events, telemetry stream, AI events
- [ ] CFD result storage — mesh URL, VTK URL, Cd/Cl/Cm to MongoDB `cfd_results`
- [ ] Mission telemetry storage — compressed state vector recording to R2, URL to `missions`
- [ ] Risk Detection event logging — timestamped alert log in `missions.risk_events[]`
- [ ] Part library API — CRUD, stress test endpoint, community sharing (is_public flag)
- [ ] Custom part upload — GLB/STEP server-side validation, polygon count limit, sandboxed WASM preview
- [ ] Community gallery API — public designs, like/fork endpoints
- [ ] Instructor mode API — custom Risk Detection threshold config per assignment
- [ ] Resend email integration — post-mission report delivery, certificate delivery, account verification
- [ ] Winston + Logtail structured logging — CFD job failures, AI API errors, auth events
- [ ] Doppler secrets management — production env vars synced to Railway

**Milestone:** Server-side CFD pipeline works end-to-end. BullMQ processes all job types. Socket.IO rooms stable.

---

### Dev D — AI / Learning / UX
- [ ] AI Design Critique — GPT-4o with spacecraft JSON + stress test in system prompt; structured JSON output (issue[], severity, fix, resource_link)
- [ ] Pre-launch engineering scorecard UI — 8-dimension pass/warning/fail card
- [ ] AI CFD Interpretation — send Cd/Cl/Cm + flow summary after every CFD run; plain-English result panel
- [ ] AI Auto-Optimisation — GPT-4o function calling → geometry delta JSON; preview mode UI; optimisation history panel
- [ ] AI Post-Mission Report — async BullMQ job; full debrief (trajectory accuracy, fuel efficiency, structural margins, thermal performance, comm budget); narrative summary; stored + emailed
- [ ] Voice narration mode — Web Speech API reads AI hints aloud
- [ ] AI quiz mode — contextual questions from active simulation state; answer evaluation
- [ ] Risk Detection — full threshold coverage: structural, thermal, fuel, trajectory, attitude, communication loss
- [ ] Custom Risk Detection thresholds UI — instructor mode threshold editor
- [ ] Learning Hub — all 5 course tracks (Orbital Mechanics, Propulsion, Aerodynamics & CFD, Thermal Protection, CAD Fundamentals)
- [ ] Learning Hub module content — each module links to pre-configured live simulation scenario
- [ ] Progress tracking — completion %, module scores, time spent per track
- [ ] Completion certificates — generated on finishing track, stored in user profile
- [ ] Aerospace glossary — 200+ terms, in-context pop-up triggered during simulation
- [ ] Instructor assignment UI — assign modules + scenarios to student accounts
- [ ] AI token budget enforcement — daily limit per tier (free/pro/institutional), Redis counter
- [ ] AI response caching — Redis key = SHA256(spacecraft JSON), skip re-evaluation on cache hit

**Milestone:** AI critique and auto-optimisation work. All 5 Learning Hub tracks navigable. Certificates generate.

---

### Phase 2 Integration Checklist (all 4 together — Month 9)
- [ ] Engineering CFD browser path + server-side path both work seamlessly
- [ ] Wind Tunnel AoA sweep produces Cl/Cd polar curve from CFD Worker
- [ ] All 5 planets simulate with correct atmospheric behaviour + hazard events
- [ ] Full ablation thermal map renders on vehicle during Mars/Venus/Titan entry
- [ ] AI critique fires before launch; auto-optimisation previews geometry change in CAD
- [ ] Full Learning Hub with progress tracking and certificates functional
- [ ] Post-mission report generated, stored, emailed

---

## Phase 3 — Collaboration & Advanced Features (Month 9–14)

**Goal:** Multiplayer, gamification, educator tools, real-world data, plugin SDK.

### Dev A — Frontend / 3D
- [ ] Multiplayer mission UI — co-op role assignment panel (pilot / engineer / communications)
- [ ] Shared CAD session — cursor presence display, design comment annotations
- [ ] Yjs CRDT integration — shared parametric CAD editing (beta)
- [ ] Mission crisis event UI — scripted failure overlays, avoidance mini-game interfaces
- [ ] Scenario editor UI — no-code mission builder: destination, objectives, constraints, thresholds
- [ ] NASA Horizons planetary position overlay — real positions on trajectory map
- [ ] ISS TLE orbital data overlay — ISS track on 3D scene
- [ ] Gamification UI — XP bar, mission achievement badges, leaderboard tables (Cd, fuel efficiency, landing precision, thermal margin, trajectory accuracy)
- [ ] Mobile companion app — React Native: telemetry viewer, mission status, community gallery, part browser
- [ ] WebXR scaffold — Three.js XR manager, VR controller stub (Phase 4 prep)

**Milestone:** Multiplayer mission UI works. Scenario editor ships. Gamification badges display.

---

### Dev B — Physics / CFD / WASM
- [ ] Multiplayer physics — server-authoritative physics for shared missions; client-side interpolation
- [ ] Physics state resync on reconnect — snapshot + replay from last authoritative tick
- [ ] Mission crisis event engine — scripted and random failure injection (engine failure, TPS partial breach, debris field, comm loss)
- [ ] NASA Horizons API adapter — real planetary positions fed to trajectory planner
- [ ] Real atmospheric measurement data feed adapter — update lookup tables from live sources
- [ ] Plugin SDK — physics extension API: custom atmosphere model, custom mission scenario, custom CFD preset
- [ ] CFD HPC prep — OpenFOAM job schema for future Kubernetes scaling (Phase 4 prep)
- [ ] Time-series telemetry — high-frequency stream format for InfluxDB/TimescaleDB ingestion (Phase 4 prep)

**Milestone:** Multiplayer physics stable under 4 simultaneous users. Mission crisis events fire correctly.

---

### Dev C — Backend / Infra
- [ ] Socket.IO multiplayer rooms — co-op mission session management, role assignment, graceful reconnect
- [ ] Server-authoritative physics endpoint — receives client inputs, returns authoritative state
- [ ] Educator dashboard API — class management, assignment CRUD, student progress analytics endpoints
- [ ] Class-level leaderboard API — ranked by mission performance metrics
- [ ] Scenario editor API — save/load/share public mission scenarios
- [ ] NASA Horizons API integration — scheduled fetch job (BullMQ cron) for planetary positions
- [ ] Plugin SDK API — versioned REST endpoints for third-party part types, atmosphere models, CFD presets
- [ ] Plugin registry — MongoDB collection, review/approval workflow
- [ ] AI embeddings pipeline — `text-embedding-3-small` for Learning Hub semantic search
- [ ] Semantic search API — similarity search over course content and design library
- [ ] Public API v1 — versioned REST, API key management, rate limiting, usage analytics dashboard

**Milestone:** Educator dashboard functional. Plugin SDK documented and testable. Public API v1 live.

---

### Dev D — AI / Learning / UX
- [ ] Educator dashboard UI — teacher view: student progress analytics, assignment management, class leaderboard
- [ ] Gamification engine — XP calculation rules, achievement badge trigger conditions, leaderboard ranking logic
- [ ] AI semantic search UI — Learning Hub "find similar" and semantic keyword search
- [ ] AI similar design recommendations — "missions like this one" surface in community gallery
- [ ] Quiz generation upgrade — questions adapt to student weakness patterns from progress data
- [ ] Scenario editor UX — no-code builder with live preview of mission parameters
- [ ] Plugin SDK documentation site — developer guide, API reference, example plugins
- [ ] Accessibility audit — WCAG 2.1 AA pass on all non-3D UI (forms, HUD panels, learning hub, reports)
- [ ] Performance profiling — identify and fix any Zustand re-render storms or Three.js frame drops

**Milestone:** Educator dashboard ships. Gamification live. AI semantic search functional.

---

### Phase 3 Integration Checklist (all 4 together — Month 14)
- [ ] 4-user multiplayer mission with no physics desync
- [ ] Instructor assigns Titan descent mission to 30 student accounts; progress shows in dashboard
- [ ] External plugin published using SDK (even a simple custom part type)
- [ ] Leaderboards update after every mission completion
- [ ] AI semantic search returns relevant courses from natural language query

---

## Phase 4 — Scale, Immersion & Ecosystem (Month 14–18+)

**Goal:** VR/AR, digital twins, institutional deployment, open API, HPC CFD.

### Dev A — Frontend / 3D
- [ ] WebXR integration — Three.js XR manager, VR controller for CAD part placement
- [ ] VR wind tunnel walk-through — immersive tunnel environment at 72fps (Quest 3 target)
- [ ] AR mission overlay — real-world camera + spacecraft trajectory AR layer
- [ ] CAD real-time collaboration — full multi-user parametric CAD with conflict resolution, version branch and merge, design review workflow
- [ ] Digital twin UI — live satellite TLE feeds displayed alongside user designs
- [ ] ISS proximity scenario UI — rendezvous visualisation in 3D scene
- [ ] Procedural planet viewer — AI-generated terrain + atmosphere rendering

**Milestone:** WebXR wind tunnel runs at 72fps on Meta Quest 3.

---

### Dev B — Physics / CFD / WASM
- [ ] Digital twin physics — live satellite TLE data ingestion, simulate real active missions
- [ ] Procedural planet generator — AI-generated atmosphere, gravity, terrain profiles for beyond-Solar-System scenarios
- [ ] CFD HPC backend — Kubernetes-managed OpenFOAM cluster, parallel large-scale solves, job priority queue
- [ ] MATLAB/Simulink export format — mission trajectory and telemetry data export
- [ ] ANSYS Workbench compatible export — CAD geometry and CFD results export
- [ ] InfluxDB/TimescaleDB integration — high-frequency telemetry + CFD convergence time-series storage and analytics

**Milestone:** Server-side CFD scales to 1,000+ concurrent users via Kubernetes cluster.

---

### Dev C — Backend / Infra
- [ ] SSO SAML 2.0 — institutional identity provider integration
- [ ] Bulk user provisioning API — CSV/API import for institutional onboarding
- [ ] LMS integration — Canvas, Moodle, Blackboard LTI connector
- [ ] Dedicated tenancy — per-institution database isolation option
- [ ] Kubernetes cluster setup — OpenFOAM HPC, auto-scale, job priority queue
- [ ] Public API v2 — expanded endpoints, SDK generation, developer portal
- [ ] API key management dashboard — usage analytics, rate-limit tiers, billing hooks
- [ ] Platform load test — validate 1,000+ concurrent simulation users without physics interference

**Milestone:** One institutional pilot (200+ students) live on SSO + LMS integration.

---

### Dev D — AI / Learning / UX
- [ ] Procedural planet AI — generative atmosphere and terrain parameter models
- [ ] AI-driven course personalisation — adaptive module sequencing based on student performance
- [ ] Industry export UX — MATLAB/ANSYS export wizard with format guide
- [ ] VR/AR onboarding UX — first-time WebXR user flow, controller calibration, help overlays
- [ ] Institutional admin dashboard — SSO management, bulk provisioning UI, tenant analytics
- [ ] Full accessibility re-audit post-VR — WCAG review of all new Phase 4 UI

**Milestone:** Institutional admin dashboard ships. AI course personalisation active.

---

### Phase 4 Integration Checklist (all 4 together — Month 18+)
- [ ] Platform sustains 1,000+ concurrent users
- [ ] WebXR wind tunnel at 72fps on Meta Quest 3
- [ ] Institutional pilot with 200+ students live
- [ ] Third-party developer plugin published via public API
- [ ] MATLAB/ANSYS export tested end-to-end with real engineering coursework

---

## Cross-Cutting Responsibilities (All Phases)

| Responsibility | Owner |
|----------------|-------|
| PR review gate — no merge without review from 1 other dev | Rotating |
| TypeScript strict mode — zero `any`, all AI response shapes typed | Dev D |
| Web Worker / WASM interface contracts — typed postMessage schemas | Dev B |
| API contracts — OpenAPI 3.0 spec kept in sync | Dev C |
| Three.js shader performance — frame budget maintained at 60fps | Dev A |
| AI prompt versioning — prompts stored in code, not ad-hoc strings | Dev D |
| Security — JWT, file upload validation, rate limiting, WCAG | Dev C + Dev D |
| Sentry triage — weekly review of top errors | Rotating |

---

## Shared Setup (Week 0 — All 4 Together)

These must be done before anyone starts feature work:

1. **Dev C**: MongoDB Atlas + Redis (Upstash) + Cloudflare R2 + Doppler secrets provisioned
2. **Dev A**: Turborepo monorepo scaffold committed to `main`
3. **Dev B**: opencascade.js WASM boolean op + STEP export validated in Vite (go/no-go gate)
4. **Dev C**: GitHub Actions CI/CD pipeline — lint, tsc, test, deploy on merge
5. **Dev D**: Sentry projects created (web + api), DSNs in Doppler
6. **All**: Local Docker Compose runs `api + worker + cfd-server` successfully
