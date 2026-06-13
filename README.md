# OrionVerse

**AI-Powered Aerospace Engineering & Multi-Planet Simulation Platform**

Browser-based SaaS / EdTech — no installation required. Design spacecraft, run CFD, simulate missions across five planetary bodies, model re-entry heat, and get AI engineering feedback — all in one environment.

---

## What It Does

| # | Module | Description |
|---|--------|-------------|
| 1 | 3D Spacecraft Builder | Drag-and-drop assembly from standard component library; real-time mass/TWR/delta-V calculator |
| 2 | CAD System (2D + 3D) | Parametric solid modelling via opencascade.js WASM; 2D schematics, cross-sections, trajectory maps; STEP/STL I/O |
| 3 | Browser-Based CFD | Educational (panel method) + engineering-accurate (Navier-Stokes FVM) — no install |
| 4 | Virtual Wind Tunnel | AoA sweep, Cl/Cd polar plot, surface Cp map, 3-variant split-screen comparison |
| 5 | Multi-Planet Simulation | Accurate atmosphere + gravity for Earth, Moon, Mars, Venus, Titan |
| 6 | Re-Entry Heat Simulation | Fay-Riddell heat flux, ablative shield modelling, full surface thermal gradient map |
| 7 | AI Engineering Mentor | Design critique, in-flight guidance, CFD interpretation, auto-optimisation, post-mission report |
| 8 | Risk Detection System | 60Hz real-time anomaly detection; Critical / Warning / Info severity classification |
| 9 | Communication System | Simulated telemetry, signal propagation delay, ground station interface, plasma blackout |
| 10 | Learning Hub | 5 structured course tracks linked to live simulation scenarios; progress tracking; certificates |

---

## Tech Stack

### Frontend
| Layer | Technology | Version |
|-------|-----------|---------|
| UI Framework | React | 18.x |
| 3D Rendering | Three.js | r165+ |
| CAD Kernel | opencascade.js (WASM) | 2.x |
| 2D Drawing | Konva.js | 9.x |
| State | Zustand | 4.x |
| Routing | React Router | 6.x |
| Styling | Tailwind CSS | 3.x |
| Animation | Framer Motion | 11.x |
| Build | Vite | 5.x |
| Types | TypeScript | 5.x |

### Computation (Web Workers + WASM)
- **Physics**: Custom RK4 integrator — 60Hz, off main thread
- **Educational CFD**: Vortex panel method — <200ms per solve
- **Engineering CFD**: FVM Navier-Stokes compiled C++ → WASM via Emscripten; k-omega SST turbulence
- **Thermal**: Fay-Riddell finite-difference solver — 10Hz
- **Mesh**: Custom Delaunay / OpenMesh WASM — unstructured tetrahedral
- **Landing physics**: Cannon-es WASM

### Backend
| Layer | Technology |
|-------|-----------|
| Runtime | Node.js 20 LTS |
| Framework | Express.js 4.x |
| WebSocket | Socket.IO 4.x |
| Auth | JWT + bcrypt (15min access / 7-day refresh) |
| File Storage | Cloudflare R2 |
| Task Queue | BullMQ + Redis |
| Server-side CFD | OpenFOAM in Docker (meshes >200k cells) |
| Email | Resend |

### Database
| Store | Technology | Purpose |
|-------|-----------|---------|
| Primary | MongoDB Atlas | Users, designs, missions, courses, progress |
| Cache/Sessions | Redis (Upstash) | JWT tokens, simulation state, CFD job status, AI cache |
| File refs | MongoDB (R2 URLs) | CAD files, VTK exports, replays, report PDFs |
| Search | MongoDB Atlas Search | Part gallery, course content, design library |

### AI
| Feature | Model |
|---------|-------|
| Design critique, auto-optimisation, CFD interpretation | GPT-4o / Gemini 1.5 Pro |
| In-flight hints, quiz generation | GPT-4o mini / Gemini Flash |
| Post-mission report | GPT-4o (async BullMQ job) |

### Infrastructure
- **Frontend hosting**: Vercel (edge CDN, preview URLs per PR)
- **Backend hosting**: Railway or Render (Docker, managed Redis)
- **CI/CD**: GitHub Actions (lint, tsc, unit tests, WASM build, deploy)
- **Error tracking**: Sentry
- **Secrets**: Doppler
- **CDN + DDoS**: Cloudflare

---

## Project Structure

```
orionverse/
├── apps/
│   ├── web/          # React + Vite frontend
│   └── api/          # Express.js backend
├── packages/
│   ├── physics/      # RK4 integrator Web Worker
│   ├── cfd/          # Panel method + FVM WASM solver
│   ├── thermal/      # Fay-Riddell solver Web Worker
│   └── cad/          # opencascade.js wrapper
```

Monorepo managed with **Turborepo**.

---

## Getting Started

```bash
npm install
npm run dev
```

Required environment variables:
```
MONGODB_URI=
REDIS_URL=
CLOUDFLARE_R2_BUCKET=
CLOUDFLARE_R2_ACCESS_KEY=
CLOUDFLARE_R2_SECRET_KEY=
OPENAI_API_KEY=
RESEND_API_KEY=
JWT_SECRET=
JWT_REFRESH_SECRET=
```

---

## Planetary Bodies Supported

| Body | Atmosphere Model | Gravity (m/s²) |
|------|-----------------|----------------|
| Earth | US Standard Atmosphere 1976 | 9.81 |
| Moon | Exosphere only (near-vacuum) | 1.62 |
| Mars | Mars GRAM 2010 (CO₂, dust) | 3.72 |
| Venus | VIRA model (96% CO₂, 90 bar) | 8.87 |
| Titan | Titan GRAM (dense N₂/CH₄) | 1.35 |

---

## Development Phases

| Phase | Timeline | Goal |
|-------|----------|------|
| 1 | Month 0–4 | 3D builder, basic CAD, educational CFD, Earth orbit mission |
| 2 | Month 4–9 | Full parametric CAD, engineering CFD, wind tunnel, multi-planet, AI mentor |
| 3 | Month 9–14 | Multiplayer, gamification, educator tools, NASA/ESA data, plugin SDK |
| 4 | Month 14–18+ | VR/AR, digital twins, institutional tier, public API |

---

## Performance Targets

| Metric | Target |
|--------|--------|
| 3D render loop | 60fps on GTX 1060 / Apple M1 |
| Educational CFD re-solve | <200ms |
| Engineering CFD (browser, <100k cells) | <60s to convergence |
| Thermal solver update | 10Hz |
| AI mentor hint latency | <3s end-to-end |
| CAD boolean operation | <2s |
| opencascade.js WASM load | <5s on broadband (cached by Service Worker) |

---

## Browser Requirements

Chrome 110+, Firefox 115+, Safari 16.4+, Edge 110+ — WebGL 2.0 and WASM required.

---

## Target Users

Students · Aerospace engineering learners · Space enthusiasts · Educators / Instructors
