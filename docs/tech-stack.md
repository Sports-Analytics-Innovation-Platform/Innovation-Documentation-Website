# Tech Stack

Every component below is listed with why it was chosen, not just what it is — this doc exists specifically to score the brief's "Tech Stack" criterion (motivated stack, not just named).

## Backend

| Choice | Why |
|---|---|
| **NestJS** (TypeScript) | Started as a plain Express + Prisma API, migrated to NestJS on 2026-08-06 for its structured module/controller/service pattern, built-in dependency injection, guards for auth, and a global exception filter — this gives a consistent shape to the hand-written API the brief requires, rather than routes accumulating ad hoc across an Express app as the team grows to six people working in parallel. |
| **Prisma** (ORM) | Type-safe query builder and migration tool over Postgres. Keeps the schema in one place (`schema.prisma`), generates TypeScript types automatically, and makes migrations reviewable in PRs rather than hand-written SQL drifting from the actual DB state. **Confirmed in use** — a real `schema.prisma` and migration (`20260806085439_init`) exist in the repo. |
| **PostgreSQL** | Relational DB, matches the naturally relational shape of NBA data (players, teams, games, events) and is well-supported by Prisma and every hosting option we're likely to use. |
| **[BetterAuth](https://better-auth.com)** (Google OAuth) | Established authentication library rather than hand-rolled auth, satisfying the brief's ban on hand-rolled auth (§2.1). Replaces the initial scaffold's Passport.js local-strategy — the team's confirmed target stack always specified BetterAuth + Google OAuth; the scaffold shipped with Passport first and was migrated on top of it on 2026-08-06. See [ADR-002](decisions/adr-002-auth.md) for the full history, including a real ⚠️ compliance risk this migration introduces, and [Security](security.md) for the account-deletion gap it leaves open. |

!!! warning "Open question: Prisma vs. Drizzle"
    The team's own stack list still poses this as `Prisma (or Drizzle)`. What's actually built and merged is Prisma — but the team has told Adrian this isn't formally decided, just what's in place so far. Treat Prisma as "current default," not "final decision," until someone says otherwise.

## Optimizer (`apps/optimizer`)

A third app alongside `apps/api` and `apps/web` — a small Python service, not exposed directly to users, that NestJS's `GET /v1/optimizer/lineup` reads from.

| Choice | Why |
|---|---|
| **Python** | Needed for **PuLP**, a mature MILP (mixed-integer linear programming) library with no equivalent TypeScript package of comparable maturity — the lineup-optimization problem (pick 5 players under a salary cap and position constraints, maximizing predicted points) is a textbook MILP. |
| **PuLP + CBC solver** | PuLP is the modeling layer; CBC is the free, open-source solver it calls. No paid solver license needed. |
| Recency-weighted average (in `predict.py`) | Predicts each player's next-game fantasy points from their own game log. A full regression model was judged not worth it yet given the current mock-data volume — see the module docstring in `apps/optimizer/predict.py`. |

Both scripts write directly into the same Postgres database Prisma/NestJS manages (`PlayerPrediction`, `Lineup`, `LineupSlot` tables); NestJS only ever reads them. This mirrors the architecture already planned for real `nba_api` ingestion below — realized here first, for the optimizer's own mock data.

## Data source

| Choice | Why |
|---|---|
| **`nba_api`** (Python client for stats.nba.com) | Free, MIT-licensed, actively maintained, and covers historical seasons, the current season, and a live in-progress-game endpoint — broader and more current than the StatsBomb EPL data the team originally considered (limited to two dated seasons). See [NBA vs EPL pitch](decisions/index.md) for the full comparison. |

!!! warning "Not yet started: `nba_api` ingestion"
    `nba_api` is a Python package, but the backend is NestJS/TypeScript. The `apps/optimizer` service above establishes the intended pattern — a separate Python process writing straight into the same Postgres database, which NestJS only reads — but the actual `nba_api` ingestion pipeline pulling real league data hasn't been built yet; all current data is mock-seeded (`apps/api/prisma/seed.ts`). See [Architecture](design/architecture.md) and [Roadmap](design/roadmap.md).

Per the brief's requirement that statistics be derived from individual event records rather than stored totals, `nba_api`'s precomputed advanced stats (offensive rating, PIE, usage%) are intended to be used only for **verification** once ingestion exists — the team calculates these itself from event-level data.

## Jobs / queue, cache, validation, storage

These four are confirmed decisions (team's stack list) but **not yet built** as of the last code snapshot I have — noting that distinction so this table isn't read as "already working":

| Choice | Why | Status |
|---|---|---|
| **Redis + BullMQ** | Batch submission/ingestion processing and incremental recomputation jobs. | Decided, not yet implemented |
| **Redis** (cache / rate limiting) | Doubles as the cache layer and a rate-limit store — relevant given `nba_api` itself is rate-limit-sensitive (see [Security](security.md)). | Decided, not yet implemented |
| **Zod** | Schema validation for incoming data (e.g. event/stat submissions), with descriptive rejection errors. | Decided, not yet implemented |
| **S3-compatible storage** (MinIO for self-hosted/course use) | For any exported/versioned data artifacts. | Decided, not yet implemented |

## Frontend

| Choice | Why |
|---|---|
| **React** | Team familiarity, large ecosystem, straightforward to keep the frontend fully decoupled from the backend (non-monolithic requirement) since it only talks to the API over HTTP. |
| **Vite** | Fast dev server and build tool, minimal config compared to older bundlers. |
| **Tailwind CSS** (v4, `@theme` custom properties) | Utility-first styling — quick to build consistent, responsive layouts without hand-writing a separate stylesheet per component, which matters given six people are touching the frontend. |
| **React Router** | Client-side routing (`/`, `/players`, `/players/:playerId` confirmed in `App.tsx`). |
| **Recharts** | Charting library used for the player-profile radar and points-trend charts — themed against the same CSS custom properties as the rest of the UI rather than hardcoded colours. |
| **TanStack Query** | Data-fetching/caching against the API. **Confirmed built** — added as part of the "frontend foundations" work alongside the BetterAuth migration. |
| **shadcn/ui** | Component library, paired with Tailwind. **Confirmed built** — used for the new Teams pages and general UI (hand-set-up after hitting a CLI bug, per the team's own dev log). |

## Infrastructure

| Choice | Why |
|---|---|
| **Docker Compose** | Runs Postgres (and any other services) locally with one command, so every team member's dev environment matches without a manual install. |
| **Gitea** (`sdp.ms.wits.ac.za`) | Version control and CI/CD (Gitea Actions, via a self-hosted Docker runner registered against the instance — see `docker-compose.yml`'s `runner` service), hosted per the university's own requirement to use university-provided infrastructure — see [Git Methodology](git-methodology.md). |
| **Vitest** | Unit testing on both apps — **confirmed built**: Vitest + Supertest for backend integration tests (against a real disposable Postgres DB), Vitest + React Testing Library on the frontend. |
| **MkDocs Material + GitHub Pages** | Documentation site. MkDocs Material was chosen over Docusaurus/mdBook for a lower setup cost with strong out-of-the-box search, admonitions, and theming; GitHub Pages is used specifically for the docs site's static hosting (separate from Gitea, which hosts the actual codebase). |

!!! success "CI/CD host confirmed: Gitea"
    Settled by checking the actual `sdp.ms.wits.ac.za` UI directly (Gitea-branded: Issues/Pull Requests/Milestones nav, Gitea's activity-feed style) — it's **Gitea**, running via `.gitea/workflows/ci.yml` on `main` with a self-hosted `gitea/act_runner` Docker container. A `.gitlab-ci.yml` file exists on some branches (e.g. `lineup-optimizer`) — that's a **dead leftover from an earlier, abandoned attempt** at a GitLab-style pipeline, superseded once the team settled on Gitea Actions on `main`, but never cleaned up on branches that forked before that happened. Don't read its presence as evidence of a second CI host; it isn't wired to anything.

## Not yet decided

- **Production hosting** for the API and frontend — no equivalent to the FPL plan's Hetzner VPS decision has been made yet for the NBA project (see [ADR-003](decisions/adr-003-hosting-topology.md), still a stub).

## Confirmed

- **Testing framework**: Vitest across both `apps/api` (+ Supertest for e2e) and `apps/web` (+ React Testing Library) — see [Definition of Done](definition-of-done.md) and `PROJECT_OVERVIEW.md`'s Testing section.
- **Local dev proxy**: `vite.config.ts` has a `server.proxy` entry forwarding `/api/*` to `http://localhost:4000` (with the `/api` prefix stripped before forwarding), so `apiClient.ts`'s relative `/api/v1/...` fetches do reach the NestJS API. This was an open question in an earlier pass — confirmed working directly, see [Getting Started](getting-started.md).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
