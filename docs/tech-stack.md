# Tech Stack

Every component below is listed with why it was chosen, not just what it is — this doc exists specifically to score the brief's "Tech Stack" criterion (motivated stack, not just named).

## Backend

| Choice | Why |
|---|---|
| **NestJS** (TypeScript) | Started as a plain Express + Prisma API, migrated to NestJS on 2026-08-06 for its structured module/controller/service pattern, built-in dependency injection, guards for auth, and a global exception filter — this gives a consistent shape to the hand-written API the brief requires, rather than routes accumulating ad hoc across an Express app as the team grows to six people working in parallel. |
| **Prisma** (ORM) | Type-safe query builder and migration tool over Postgres. Keeps the schema in one place (`schema.prisma`), generates TypeScript types automatically, and makes migrations reviewable in PRs rather than hand-written SQL drifting from the actual DB state. |
| **PostgreSQL** | Relational DB, matches the naturally relational shape of NBA data (players, teams, games, events) and is well-supported by Prisma and every hosting option we're likely to use. |
| **[BetterAuth](https://better-auth.com)** | Established authentication library rather than hand-rolled auth, satisfying the brief's requirement to rely on established practices for auth (§2.1). Replaced an earlier Passport.js local-strategy prototype on 2026-08-06 — its Prisma adapter fits this stack directly, and Google OAuth means the app never stores or handles a password. Handles sign-in, session cookies, and the OAuth redirect flow; see [Security](security.md) for the account-deletion gap this still leaves open. |

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

## Frontend

| Choice | Why |
|---|---|
| **React** | Team familiarity, large ecosystem, straightforward to keep the frontend fully decoupled from the backend (non-monolithic requirement) since it only talks to the API over HTTP. |
| **Vite** | Fast dev server and build tool, minimal config compared to older bundlers. |
| **Tailwind CSS** (v4, `@theme` custom properties) | Utility-first styling — quick to build consistent, responsive layouts without hand-writing a separate stylesheet per component, which matters given six people are touching the frontend. |
| **React Router** | Client-side routing (`/`, `/players`, `/players/:playerId` confirmed in `App.tsx`). |
| **Recharts** | Charting library used for the player-profile radar and points-trend charts — themed against the same CSS custom properties as the rest of the UI rather than hardcoded colours. |

## Infrastructure

| Choice | Why |
|---|---|
| **Docker Compose** | Runs Postgres (and any other services) locally with one command, so every team member's dev environment matches without a manual install. |
| **GitLab** (`sdp.ms.wits.ac.za`) | Version control and CI/CD (GitLab CI), hosted per the university's own requirement to use university-provided infrastructure — see [Git Methodology](git-methodology.md). |
| **MkDocs Material + GitHub Pages** | Documentation site. MkDocs Material was chosen over Docusaurus/mdBook for a lower setup cost with strong out-of-the-box search, admonitions, and theming; GitHub Pages is used specifically for the docs site's static hosting (separate from GitLab, which hosts the actual codebase). |

## Not yet decided

- **Production hosting** for the API and frontend — no equivalent to the FPL plan's Hetzner VPS decision has been made yet for the NBA project (see [ADR-003](decisions/adr-003-hosting-topology.md)).

## Confirmed

- **Testing framework**: Vitest across both `apps/api` (+ Supertest for e2e) and `apps/web` (+ React Testing Library) — see [Definition of Done](definition-of-done.md) and `PROJECT_OVERVIEW.md`'s Testing section.
- **Local dev end-to-end**: `vite.config.ts` has a `server.proxy` entry forwarding `/api/*` to `http://localhost:4000`, so `apiClient.ts`'s relative fetches do reach the NestJS API. Confirmed working — see [Getting Started](getting-started.md).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
