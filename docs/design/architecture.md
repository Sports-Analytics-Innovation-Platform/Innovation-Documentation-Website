# Architecture Overview

## System shape

```
┌─────────────────────┐        HTTPS/fetch         ┌──────────────────────┐
│   apps/web           │ ───── /api/v1/... ───────▶ │   apps/api            │
│   React + Vite       │ ◀──── JSON, cookies ─────  │   NestJS              │
│   (Tailwind, Recharts│                             │   (BetterAuth, Prisma)│
│   React Router)      │                             │                       │
│   Cloudflare Pages   │                             │   Render              │
└─────────────────────┘                             └──────────┬────────────┘
                                                                  │
                                                                  ▼
                                                        ┌──────────────────┐
                                                        │   Supabase        │
                                                        │   PostgreSQL      │
                                                        └──────────────────┘
                                                                  ▲
                                                                  │
                                              ┌───────────────────┼───────────────────┐
                                              │                   │                   │
                                    ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
                                    │  apps/ingestion   │ │  apps/predictor   │ │  apps/optimizer   │
                                    │  (Python)         │ │  (Python)         │ │  (Python)         │
                                    │  nba_api → PG     │ │  Elo + Four Fac.  │ │  MILP lineup      │
                                    └──────────────────┘ └──────────────────┘ └──────────────────┘
```

This satisfies the brief's non-monolithic requirement (§2.1): `apps/web` and `apps/api` are separate, independently deployed applications that only communicate over HTTP — confirmed directly from the code (`apiClient.ts` calls `fetch` against `VITE_API_BASE_URL`, nothing shares in-process state). The three Python services (`ingestion`, `predictor`, `optimizer`) write directly to Postgres as separate processes, never through the API.

## Frontend (`apps/web`)

- **React + Vite**, **Tailwind CSS v4** (via `@theme` custom properties in `index.css`, not the older `tailwind.config.js` token approach).
- **React Router** — client-side routing with eight routes: `/` (Home), `/players`, `/players/:playerId`, `/teams`, `/teams/:teamId`, `/predictions` (auth-gated), `/optimizer` (auth-gated), `/games/:gameId` (auth-gated).
- **Recharts** — `RadarChart` (player traits) and `LineChart` (points trend), both themed against the same CSS variables as the rest of the UI.
- **TanStack Query** for data-fetching/caching against the API.
- **shadcn/ui** for component library, paired with Tailwind.
- All backend calls go through `lib/apiClient.ts`, a single `fetchJson<T>` wrapper — one place controls the base URL and request options.
- `credentials: "include"` on every request for cookie-based auth.
- **Top navbar** with five nav links (Home, Players, Teams, Optimizer, Predictions), a recent-result widget, and an auth status button.
- **Court view** visualisation for predicted top scorers by position on a basketball court.
- Deployed on **Cloudflare Pages** (global CDN, managed TLS, auto-deploy from GitHub mirror).

## Backend (`apps/api`)

- **NestJS** on top of **Prisma** and **PostgreSQL (Supabase)** — see [ADR-001](../decisions/adr-001-database.md).
- **BetterAuth** (Google OAuth) for auth — see [ADR-002](../decisions/adr-002-auth.md). BetterAuth mounts its own route set at `/api/auth/*`.
- Routes are versioned under `/v1/` — see [API Design](api-design.md) for the full endpoint table.
- **Auth-gated endpoints**: Games, predictions, and optimizer endpoints require an authenticated session (`SessionAuthGuard`). Player and team browsing is public.
- **Health check** at `/health` for Render liveness probes.
- Deployed on **Render** (Node.js web service, free tier with ~30s cold start after 15 min inactivity).

## Python services

Three Python services run alongside the TypeScript apps, writing directly to Postgres:

- **`apps/ingestion`** — `nba_api` client that fetches teams, rosters, games, and box scores into Postgres. Orchestrated by `ingest.py`. Built during Sprint 1 (week of 18 Aug).
- **`apps/predictor`** — computes Elo-based home win probability and Four Factors-based predicted score margin for each game. Writes to the `GamePrediction` table.
- **`apps/optimizer`** — predicts per-player fantasy points and solves a 5-player lineup under a salary cap via MILP (PuLP/CBC). Writes to `PlayerPrediction`, `Lineup`, and `LineupSlot` tables.

These are planned to run as Render Cron Jobs or Background Workers in production.

## Local development

**Docker Compose** runs Postgres locally; both apps run with their own dev server (`npm run start:dev` / `npm run dev`) against it — see [Getting Started](../getting-started.md).

## Deployment topology

Production hosting per [ADR-003](../decisions/adr-003-hosting-topology.md):

| Component | Host | Deploy method |
|---|---|---|
| Frontend (`apps/web`) | Cloudflare Pages | Auto-deploy from GitHub mirror |
| API (`apps/api`) | Render | Auto-deploy from GitHub mirror |
| Database | Supabase (managed Postgres) | Direct connection from API and Python services |
| Python services | Render (Cron/Worker, planned) | Manual or scheduled |
| CI | Gitea Actions | Lint, typecheck, test on every push/PR |
| Docs site | GitHub Pages | Auto-deploy on push to `main` |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
