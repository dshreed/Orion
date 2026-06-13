# AGENTS.md — OrionVerse

AI agent and LLM coding assistant context for the OrionVerse codebase.

---

## What This Project Is

OrionVerse is a browser-based aerospace engineering simulation platform. It is **not** a standard web app — it runs physics, CFD, and thermal solvers in Web Workers using WebAssembly, renders 3D scenes via Three.js WebGL, and does parametric CAD via opencascade.js compiled to WASM. Treat it like an engineering workbench that happens to run in a browser.

---

## Monorepo Layout

```
apps/web/         React + Vite frontend
apps/api/         Express.js backend (Node.js 20 LTS)
packages/physics/ RK4 physics Web Worker
packages/cfd/     Vortex panel + FVM Navier-Stokes WASM solver
packages/thermal/ Fay-Riddell finite-difference thermal worker
packages/cad/     opencascade.js WASM wrapper
```

Turborepo manages the monorepo. Run `npm run dev` from root.

---

## Architecture Rules

### Web Workers
Physics, CFD, and thermal computation run in **separate Web Workers**, not on the main thread. Never move heavy computation to the main thread. Workers communicate via `postMessage` with Transferable objects (ArrayBuffers) for zero-copy transfers.

- Physics Worker: 60Hz RK4 tick
- CFD Worker: panel method or FVM WASM solve
- Thermal Worker: 10Hz Fay-Riddell solve

### WASM Modules
Each Worker loads its own WASM module. WASM is **never** loaded on the main thread except opencascade.js (CAD only). opencascade.js (~30MB) is lazy-loaded only when the user opens the CAD workspace, then cached by the Service Worker.

### State Management
Zustand is used for high-frequency state (physics ticks, CFD residuals, telemetry). **Never** use React state (`useState`) for data that updates faster than ~10Hz — it causes render storms. Physics/CFD/thermal output goes into Zustand stores; React components subscribe selectively.

### Three.js
Three.js renders the spacecraft, CFD pressure maps (vertex colour shaders), thermal surface gradients, streamlines, and wind tunnel particles. All custom visual effects use GLSL shaders, not CSS or inline styles. Never use `style={{}}` for 3D canvas elements.

### 2D Canvas
Konva.js handles 2D schematics, cross-section views, and trajectory maps. It runs on a separate `<canvas>` element below or beside the Three.js canvas. Do not use SVG for interactive 2D — use Konva.

---

## Backend Rules

### Auth
JWT with bcrypt. Access token: 15 minutes. Refresh token: 7 days, stored in `httpOnly` cookie. Roles: `student | instructor | admin`. Always check `req.user.role` before returning sensitive data.

### WebSocket
Socket.IO 4.x. Used for:
- CFD job progress (large mesh server-side solves)
- Real-time telemetry during simulation
- Risk Detection alerts
- AI mentor response streaming
- Multiplayer mission rooms (Phase 3)

Use Socket.IO rooms for session isolation. Never broadcast to all connected clients.

### Job Queue
BullMQ + Redis for async jobs:
- Server-side OpenFOAM CFD solves (meshes >200k cells)
- AI post-mission report generation
- Mesh generation preprocessing
- Email delivery (Resend)

Never block an Express route handler waiting for a long-running job. Queue it, return a job ID, stream progress via Socket.IO.

### File Storage
All binary files (CAD models, VTK results, mission replays, report PDFs) go to **Cloudflare R2**. MongoDB stores the R2 URLs only, not file contents.

---

## Database

MongoDB Atlas. No SQL, no Drizzle, no Prisma — Mongoose ODM.

Key collections:
- `users` — role, tier (free/pro/institutional)
- `spacecraft_designs` — parts[], assembly_tree, version_history[]
- `cad_projects` — brep_url, step_url (R2), snapshots[]
- `cfd_results` — Cd, Cl, Cm, pressure_field_url (R2)
- `missions` — telemetry_url (R2), risk_events[], ai_report
- `part_library` — standard + user-created parts
- `courses` / `user_progress` — learning hub

Redis (Upstash) for: JWT refresh tokens, simulation state snapshots, CFD job status, AI response cache (keyed by content hash of spacecraft JSON), rate-limit counters.

---

## AI Integration

### Model Routing
- **GPT-4o / Gemini 1.5 Pro**: design critique, auto-optimisation (function calling), CFD interpretation, post-mission report
- **GPT-4o mini / Gemini Flash**: in-flight hints (triggered by Risk Detection), quiz generation — ~10x cheaper

### Cost Controls
- Daily token budget per tier (free/pro/institutional)
- Redis cache for design critiques: key = SHA256 of spacecraft JSON — never re-evaluate identical designs
- Post-mission reports are async BullMQ jobs — never block the UI

### AI Auto-Optimisation
Returns a JSON delta of parametric constraint changes. Frontend applies to CAD model in preview mode before user commits. Never apply geometry changes directly without user confirmation.

### Context Seeding
Always seed AI prompts with actual spacecraft JSON + live mission state. Generic prompts are not acceptable — the mentor must reference the user's specific design.

---

## CFD Architecture: Browser vs Server

| Condition | Path |
|-----------|------|
| Mesh < 200k cells | FVM WASM in CFD Web Worker (browser) |
| Mesh > 200k cells | POST to `/api/cfd/submit` → BullMQ → OpenFOAM Docker container → result via Socket.IO + R2 |

The client handles both paths transparently. Never expose the routing decision to the user — the UI shows a single "solving" state.

---

## Risk Detection System

Runs as a monitor inside the Physics and Thermal Workers:
- Physics Worker monitors at 60Hz: structural load, fuel, trajectory deviation, attitude stability
- Thermal Worker monitors at 10Hz: surface temperature vs. TPS material Tmax

Severity levels: `Critical | Warning | Info`

On Critical/Warning: post alert to main thread → HUD alert panel → Socket.IO emit to backend → AI mentor triggered automatically.

Never throttle Critical alerts. Warning alerts can debounce at 1s. Info events batch-log only.

---

## Planetary Atmosphere Models

| Body | Model | Notes |
|------|-------|-------|
| Earth | US Standard Atmosphere 1976 | Lookup table in Physics Worker |
| Moon | Exosphere (near-vacuum) | Effectively zero drag |
| Mars | Mars GRAM 2010 | CO₂ atmosphere; dust storm hazard events |
| Venus | VIRA | 96% CO₂, 90 bar surface; sulphuric acid cloud layer |
| Titan | Titan GRAM | Dense N₂/CH₄ haze; parachute descent scenarios |

Atmosphere tables are precomputed per body and loaded into the Physics Worker as typed arrays. Never fetch atmosphere data from the server during simulation — latency kills physics stability.

---

## TPS Material Library (Re-Entry)

| Material | Use Case |
|----------|----------|
| PICA | Dragon capsule; high-heating Earth return |
| SLA-561V | Mars Science Laboratory; moderate heating |
| RCC | Space Shuttle leading edges; extreme heating |
| Aluminium honeycomb | Low-heating trajectories only |

Thermal Worker computes ablation recession rate, char layer thickness, and surface temperature map. Trigger a `Critical` Risk Detection alert when surface temp exceeds material `Tmax`.

---

## Key Invariants

1. **No physics computation on the main thread.** All RK4, CFD, thermal, and mesh work runs in Web Workers.
2. **No blocking Express handlers.** Long jobs → BullMQ queue.
3. **No AI calls that block the UI.** Hints are async fire-and-forget; reports are BullMQ jobs.
4. **No file contents in MongoDB.** Binary files go to R2; MongoDB stores URLs.
5. **No Zustand for UI-only state.** Use `useState` for component-local state (modal open/close, etc.).
6. **No inline `style={{}}` on 3D/canvas elements.** Use Tailwind or GLSL.
7. **JWT auth on every API route.** Public routes are explicitly whitelisted.

---

## Phase 1 Priorities (Current)

Focus on getting these working before anything else:
1. Turborepo monorepo setup: `apps/web`, `apps/api`, `packages/physics`, `packages/cfd`, `packages/thermal`, `packages/cad`
2. opencascade.js WASM — validate boolean ops and STEP export work in Vite on Day 1
3. Vortex panel method CFD in a Web Worker — prove <200ms performance before building UI
4. Three.js scene scaffold with orbit controls and basic part placement
5. MongoDB Atlas + Redis (Upstash) + Cloudflare R2 + GitHub Actions CI/CD wired up before feature work

Phase 1 ships when: user can register, assemble a rocket, run educational CFD (see pressure map), launch to Earth orbit, see telemetry including stagnation temperature, receive a Risk Detection alert, and replay the mission.

---

## Non-Functional Targets Agents Must Respect

| Metric | Target |
|--------|--------|
| 3D render | 60fps on GTX 1060 / Apple M1 |
| Educational CFD re-solve | <200ms |
| Engineering CFD browser solve | <60s (meshes <100k cells) |
| Thermal update | 10Hz |
| AI hint latency | <3s |
| CAD boolean op | <2s |
| opencascade.js initial load | <5s (broadband, cached) |

Do not accept implementations that violate these without explicit discussion.

---

## Security Requirements

- MIME type validation + virus scan + 50MB size limit on all file uploads before storing to R2
- WASM processing of uploaded models runs sandboxed before any publish action
- JWT `httpOnly` cookie for refresh token — never expose to JavaScript
- Cloudflare WAF + rate limiting on all API endpoints
- WCAG 2.1 AA on all non-3D UI (menus, HUD panels, forms, learning hub)
