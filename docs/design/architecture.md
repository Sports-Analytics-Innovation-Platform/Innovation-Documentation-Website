# Architecture Overview

## System shape

```
┌─────────────────────┐        HTTPS/fetch         ┌──────────────────────┐
│   apps/web           │ ───── /api/v1/... ───────▶ │   apps/api            │
│   React + Vite       │ ◀──── JSON, cookies ─────  │   NestJS              │
│   (Tailwind, Recharts│                             │   (BetterAuth, Prisma)│
│   React Router)      │                             │                       │
└─────────────────────┘                             └──────────┬────────────┘
                                                                  │
                                                                  ▼
                                                        ┌──────────────────┐
                                                        │   PostgreSQL       │
                                                        └──────────────────┘
                                                                  ▲
                                                                  │ writes predictions/lineups
                                                        ┌──────────────────┐
                                                        │  apps/optimizer    │
                                                        │  (Python, PuLP/CBC)│
                                                        └──────────────────┘

                                              ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
                                              │  nba_api (Python) ingestion  │
                                              │  — not yet started; the      │
                                              │  optimizer above establishes │
                                              │  the intended pattern for it │
                                              │  (see Tech Stack)            │
                                              └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

This satisfies the brief's non-monolithic requirement (§2.1): `apps/web` and `apps/api` are separate, independently buildable applications that only communicate over HTTP — confirmed directly from the code (`apiClient.ts` calls `fetch` against a base URL, nothing shares in-process state).

## Frontend (`apps/web`)

- **React + Vite**, **Tailwind CSS v4** (via `@theme` custom properties in `index.css`, not the older `tailwind.config.js` token approach).
- **React Router** — client-side routing, three routes currently: `/`, `/players`, `/players/:playerId`.
- **Recharts** — `RadarChart` (player traits) and `LineChart` (points trend), both themed against the same CSS variables as the rest of the UI.
- All backend calls go through `lib/apiClient.ts`, a single `fetchJson<T>` wrapper — one place controls the base URL and request options, which is a solid pattern for API changes later.
- `credentials: "include"` on every request implies cookie-based auth is expected once auth-gated routes exist, even though the three current endpoints appear to be public reads.
- **TanStack Query** and **shadcn/ui** are confirmed added (Teams pages, filters/sort/pagination on Players) — see [Tech Stack](../tech-stack.md).

## Backend (`apps/api`)

- **NestJS** on top of **Prisma** and **PostgreSQL** — see [ADR-001](../decisions/adr-001-database.md).
- **BetterAuth** (Google OAuth) for auth — see [ADR-002](../decisions/adr-002-auth.md), including an open compliance question this introduces.
- Routes are versioned under `/v1/` (`/v1/players`, `/v1/teams`, `/v1/games`, `/v1/optimizer/lineup`, plus `/health` and `/auth/*`), served under an `/api` base path the frontend proxies to — see [API Design](api-design.md) for the full picture of what's confirmed vs. open.

## Optimizer (`apps/optimizer`)

A separate Python service (see [Tech Stack](../tech-stack.md#optimizer-appsoptimizer)) that writes predictions and lineups directly into the same Postgres database. NestJS only reads from the tables it writes — it doesn't invoke the Python scripts itself.

## Local development

**Docker Compose** runs Postgres locally; both `apps/api` and `apps/web` run with their own dev server (`npm run dev`) against it — see [Getting Started](../getting-started.md).

## Open questions

- **`nba_api` (Python) integration path** — not yet started (also flagged in [Tech Stack](../tech-stack.md)). `apps/optimizer` establishes the intended pattern (a separate Python process writing straight into Postgres, NestJS only reading), but the actual `nba_api` ingestion pipeline pulling real league data doesn't exist yet — everything currently in the database is mock-seeded.
- **Production hosting topology** — not decided (see the not-yet-written [ADR-003](../decisions/adr-003-hosting-topology.md)).
- **Whether any endpoint is actually auth-gated yet** — no route currently requires a logged-in user; `SessionAuthGuard`/`RolesGuard` exist and are unit-tested but aren't applied to any route yet.

## Prediction / optimisation component

Previously the biggest open box in this diagram — a **basic-tier version now exists**, in `apps/optimizer` (see above). See [Feature Tiers](feature-tiers.md) and [Roadmap](roadmap.md) for what's still ahead.

```
PostgreSQL (historical game/player data)
        │
        ▼
┌───────────────────┐
│  Prediction model   │   Built (basic tier): apps/optimizer/predict.py,
│  (basic tier built) │   recency-weighted average of each player's own
└─────────┬──────────┘   game log — not yet a trained regression model
          │
          ▼
┌───────────────────┐
│  Recommendation     │   Built: apps/optimizer/optimize.py — MILP
│  layer (basic)      │   (PuLP/CBC) picks a 5-player lineup under a
└───────────────────┘   salary cap, maximising predicted points
```

The question of *where this runs* is answered by what's already built: a separate Python service (`apps/optimizer`) writing into Postgres, which NestJS only reads — the same pattern the still-open `nba_api` ingestion question below will likely reuse. What's still open is model **quality**, not architecture: `predict.py`'s recency-weighted average is a placeholder for a real regression/ML model, which the client indicated is realistically a Sprint 3 target (see [client meeting notes](../meetings/client/index.md)).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
