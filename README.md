# Nirikshan — Making the invisible visible

> **Nirikshan** (निरीक्षण — "observation") is a civic-tech platform for child safety in India. Ordinary citizens can safely report the child-safety concerns they observe — no investigation, no confrontation, no guesswork — and a verified, human-led response network of trained responders, coordinators, professionals and partner NGOs carries each report through to resolution.

**Made with 💗 by Team Vanguard26**

---

## Table of contents

- [Overview](#overview)
- [The response network](#the-response-network)
- [Features](#features)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
- [Project structure](#project-structure)
- [Routes](#routes)
- [Role switching (developer preview)](#role-switching-developer-preview)
- [Progressive Web App](#progressive-web-app)
- [Deployment](#deployment)
- [Data & backend readiness](#data--backend-readiness)
- [Disclaimer](#disclaimer)

---

## Overview

Nirikshan tackles a hard problem: risks to children are often only visible to people who happen to be in the right place at the right time — and those people are rarely equipped to classify or investigate what they saw. The platform lowers the barrier to that first, crucial step:

- **Citizens observe, report, and step back.** A report flow asks only for what was observed — location, description, context — never for legal or clinical judgments. Reporters stay protected.
- **Trained humans decide.** AI assists with organization, but only verified human responders and professionals verify, route and intervene.
- **Communities learn from aggregates.** Analysis surfaces patterns in a non-identifying way, so no individual child is ever exposed.

The current build is a mobile-first PWA, focused on Maharashtra, and currently runs on curated mock data while the backend is brought online.

## The response network

A report travels through the **Progressive Community Response Network (PCRN)**:

| Tier | Who | Role |
| --- | --- | --- |
| **Citizen** | Anyone | Observe and report. Identity is never exposed to responders. |
| **Level 1** · Community Responder | Verified citizens | Safe identification, reporting and initial on-ground assistance in their area. |
| **Level 2** · Regional Coordinator | Certified responders | Triage emergency requests; coordinate handoffs between field responders, professionals and NGOs. |
| **Level 3** · Professional | Child-welfare professionals, medical experts, legal advisors | Authorized professional intervention; appointed only through registered NGO invitations. |
| **NGO** · Verified Partner | Partner organizations | Review curated referrals, decide whether to accept, and connect children to statutory care (e.g. district Child Welfare Committees). |

Portal status: the **Citizen** portal is live in the preview; **Level 1, Level 2 and NGO** portals are under active development; **Level 3** is onboarded via NGO invitation.

## Features

- **Structured confidential reporting** — location → observation → child → supporting context → assistance → emergency → review, ending with a traceable case ID.
- **Human-first verification** — explicit "AI assists. Humans decide." boundary; no automated labelling as trafficking, abuse or exploitation.
- **Role-based workspaces** — separate portals for Citizens, PCRN Levels 1–3 and NGO partners, each with its own dashboard, cases, safety toolkit and profile.
- **Aggregated, privacy-safe analysis** — community patterns and impact metrics without identifying children.
- **Full PWA** — installable, offline-capable, theme-aware (light/dark), designed mobile-first with a desktop sidebar shell.
- **Emergency path** — reporters can flag reports for immediate attention and are directed to local emergency services.

## Tech stack

| Layer | Choice |
| --- | --- |
| UI | React **19**, React Router **7**, Tailwind CSS **3.4** |
| Components | shadcn/ui primitives built on **Radix UI**, **lucide-react** icons |
| Charts | **Recharts** |
| Toasts | **sonner** |
| Data layer | **@tanstack/react-query** (provider is wired; all current data is curated mock files — see below) |
| Build | **CRACO** (react-scripts 5) with alias `@/ → src/` and optional integrated health-check plugin behind `ENABLE_HEALTH_CHECK=true` |
| Tooling | npm with `legacy-peer-deps=true` (`.npmrc`) for React 19 peer resolutions |

Facts you should know from the current build:

- **No backend is wired yet.** All screens render from `frontend/src/lib/mockData.js`, `roleData.js` and `l3NgoData.js`, which intentionally mirror the future API shape. The service worker already reserves a network-first `/api/*` route for when the backend lands.
- `framer-motion`, `swr`, `zod` and `react-hook-form` are declared in `package.json` but not yet imported in `src/` — reserved for upcoming work (animations, data fetching, and form validation on the report flow).

## Getting started

Requirements: **Node 18+** and npm.

```bash
# 1. Install dependencies (from repo root)
npm install --prefix frontend

# 2. Start the dev server (http://localhost:3000)
npm start --prefix frontend

# 3. Run tests
npm test --prefix frontend

# 4. Production build
npm run build --prefix frontend
```

Or work directly from the `frontend/` directory:

```bash
cd frontend
npm install
npm start
```

The dev server runs on **http://localhost:3000** and hot-reloads changes.

## Project structure

```
frontend/
├─ public/                 # Static shell, PWA manifest + icons, service worker (sw.js)
├─ src/
│  ├─ components/          # App components (Layout, CaseCard, StatCard, PrivacyNote, …)
│  │  └─ ui/               # shadcn/ui primitives (button, card, dialog, select, …)
│  ├─ constants/           # Shared UI constants
│  ├─ hooks/               # Custom hooks (use-toast)
│  ├─ lib/                 # Mock data, role/theme providers, cn() util
│  ├─ pages/               # Citizen pages …
│  │  ├─ l1/ l2/ l3/ ngo/  # … and role-specific portals
│  ├─ App.js               # Route table
│  ├─ index.js             # Entry, providers, PWA service worker registration
│  └─ index.css            # Design tokens + global styles
├─ craco.config.js         # CRACO setup, @/ alias, optional health-check plugin
├─ vercel.json             # SPA fallback rewrite for static hosting
└─ package.json
```

## Routes

All top-level routes render inside the shared app shell (desktop sidebar / mobile top bar + bottom nav):

| Path | Page | Notes |
| --- | --- | --- |
| `/`, `/home` | Citizen Home | `/home` redirects to `/` |
| `/report` | Report | Multi-step observer flow |
| `/cases`, `/cases/:caseId` | My Cases / Case details | |
| `/analysis` | Analysis | Aggregated community insights |
| `/profile`, `/notifications`, `/safety`, `/login` | Citizen profile, notifications, safety, login | |

Role-specific portals:

| Role | Path prefix | Pages |
| --- | --- | --- |
| Level 1 | `/l1` | home, requests, cases, safety, profile |
| Level 2 | `/l2` | home, assistance, cases, safety, profile |
| Level 3 | `/l3` | home, cases, mycases, safety, profile, login |
| NGO | `/ngo` | home, professionals, cases, assigned, impact, profile |

## Role switching (developer preview)

Each portal can be reached without authentication via the **Developer Preview** switcher:

- **Desktop:** bottom of the sidebar.
- **Mobile:** floating button above the bottom nav.

The selected role is persisted in `localStorage` under the key `nirikshan.devRole`. This is **UI-only** — it is intentionally not wired to any auth or backend permission system and must be removed or replaced before a public production launch.

## Progressive Web App

- `frontend/public/manifest.json` — app name, icons (light/dark themed), `standalone` display, theme color.
- `frontend/public/sw.js` — hand-written service worker: cache-first for the static shell, network-first for `/api/*`, offline shell pre-cache, and stale-cache cleanup. Stub push handlers are included for future notifications.
- The service worker registers **only in production builds** (`NODE_ENV === "production"`); the dev server does not register it.

## Deployment

The app builds to a static bundle and deploys anywhere that serves static files (Vercel, Netlify, nginx, …). Since routing is client-side, every path must fall back to `index.html`; `frontend/vercel.json` ships the SPA rewrite for Vercel.

```bash
npm run build --prefix frontend
```

The service worker is hand-tuned for the build output, so regenerate `build/` with every deploy to keep the offline shell in sync.

## Data & backend readiness

Today, everything renders from hand-curated mock data under `frontend/src/lib/`. When the backend integration begins, the data layer is already shaped for it:

- REST-style mock modules mirror the future API resources.
- `@tanstack/react-query` is installed and provided at the root.
- The service worker already handles network-first `/api/*`.

This repository intentionally contains **no secrets**; do not commit tokens or keys.

## Disclaimer

This is an early civic-tech preview. It is not a substitute for contacting local emergency services or statutory bodies in an emergency, and it provides no legal or clinical advice. Production deployment requires closing the gap between the UI and a compliant backend, and removing or locking down the developer role switcher.