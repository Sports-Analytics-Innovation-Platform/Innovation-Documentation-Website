# Tech Stack

Every component below is listed with why it was chosen, not just what it is — this doc exists specifically to score the brief's "Tech Stack" criterion (motivated stack, not just named).

## Backend

| Choice | Why |
|---|---|
| **NestJS** (TypeScript) | Started as a plain Express + Prisma API, migrated to NestJS on 2026-08-06 for its structured module/controller/service pattern, built-in dependency injection, guards for auth, and a global exception filter — this gives a consistent shape to the hand-written API the brief requires, rather than routes accumulating ad hoc across an Express app as the team grows to six people working in parallel. |
| **Prisma** (ORM) | Type-safe query builder and migration tool over Postgres. Keeps the schema in one place (`schema.prisma`), generates TypeScript types automatically, and makes migrations reviewable in PRs rather than hand-written SQL drifting from the actual DB state. Confirmed as the team's choice — a real `schema.prisma` with 12 models and multiple migrations exist in the repo. |
| **PostgreSQL** | Relational DB, matches the naturally relational shape of NBA data (players, teams, games, events, predictions) and is well-supported by Prisma and Supabase's managed hosting. |
| **BetterAuth** (Google OAuth) | Established authentication library, satisfying the brief's ban on hand-rolled auth (§2.1). Replaces the initial scaffold's Passport.js local-strategy. BetterAuth mounts its own route set at `/api/auth/*` and manages sessions via cookies. See [ADR-002](decisions/adr-002-auth.md) for the full history. |

## Data source

| Choice | Why |
|---|---|
| **`nba_api`** (Python client for stats.nba.com) | Free, MIT-licensed, actively maintained, and covers historical seasons, the current season, and a live in-progress-game endpoint — broader and more current than the StatsBomb EPL data the team originally considered (limited to two dated seasons). See [NBA vs EPL pitch](decisions/index.md) for the full comparison. |

`nba_api` is a Python package, and the backend is NestJS/TypeScript. This is resolved by running `nba_api` as a separate Python ingestion service (`apps/ingestion`) that writes directly to Postgres — NestJS then reads from the same database. The two languages coexist without needing an inter-process bridge because Postgres is the shared data layer. See [Architecture](design/architecture.md) for the full diagram.

Per the brief's requirement that statistics be derived from individual event records rather than stored totals, `nba_api`'s precomputed advanced stats (offensive rating, PIE, usage%) are used only for **verification** — the team calculates these itself from event-level data.

## Jobs / queue, cache, validation, storage

These four are confirmed decisions (team's stack list). Their implementation status:

| Choice | Why | Status |
|---|---|---|
| **Redis + BullMQ** | Batch submission/ingestion processing and incremental recomputation jobs. | Planned for Sprint 2 if ingestion pipeline needs batch/scheduled processing |
| **Redis** (cache / rate limiting) | Doubles as the cache layer and a rate-limit store — relevant given `nba_api` itself is rate-limit-sensitive (see [Security](security.md)). | Planned for Sprint 2 |
| **Zod** | Schema validation for incoming data (e.g. event/stat submissions), with descriptive rejection errors. | Planned for Sprint 2 |
| **S3-compatible storage** (MinIO for self-hosted/course use) | For any exported/versioned data artifacts. | Planned if needed |

## Frontend

| Choice | Why |
|---|---|
| **React** | Team familiarity, large ecosystem, straightforward to keep the frontend fully decoupled from the backend (non-monolithic requirement) since it only talks to the API over HTTP. |
| **Vite** | Fast dev server and build tool, minimal config compared to older bundlers. |
| **Tailwind CSS** (v4, `@theme` custom properties) | Utility-first styling — quick to build consistent, responsive layouts without hand-writing a separate stylesheet per component, which matters given six people are touching the frontend. |
| **React Router** | Client-side routing with eight routes: `/`, `/players`, `/players/:playerId`, `/teams`, `/teams/:teamId`, `/predictions`, `/optimizer`, `/games/:gameId`. |
| **Recharts** | Charting library used for the player-profile radar and points-trend charts — themed against the same CSS custom properties as the rest of the UI rather than hardcoded colours. |
| **TanStack Query** | Data-fetching/caching against the API. Confirmed built — added as part of the frontend foundations work. |
| **shadcn/ui** | Component library, paired with Tailwind. Confirmed built — used for Teams pages, general UI components, and the court view. |

## Infrastructure

| Choice | Why |
|---|---|
| **Docker Compose** | Runs Postgres locally with one command, so every team member's dev environment matches without a manual install. |
| **Gitea** | Version control, hosted per the university's own requirement to use university-provided infrastructure — see [Git Methodology](git-methodology.md). |
| **Gitea Actions** (CI) | Pipeline defined in `.gitea/workflows/ci.yml`, running lint → typecheck → test as three parallel jobs (`api`, `web`, `coverage`) on every push and PR, pinned to Node 24. Confirmed built. Chosen simply because it's built into the Gitea instance already hosting the code — no second platform to register runners with. See [CI/CD Pipeline](ci-cd.md). |
| **ESLint 10** (API) / **oxlint** (web) | Two linters rather than one, because the apps already shipped with different toolchains and unifying them would be churn without benefit — oxlint is dramatically faster on the frontend, and the API's flat config composes `@eslint/js` + `typescript-eslint`. Both are enforced in CI. |
| **Vitest** | Unit testing on both apps — Vitest + Supertest for backend integration tests (against a real disposable Postgres DB), Vitest + React Testing Library on the frontend. Test files exist in both `apps/api` and `apps/web` — the CI `coverage` job runs both suites against a real Postgres with no `--passWithNoTests` tolerance. |
| **MkDocs Material + GitHub Pages** | Documentation site. MkDocs Material was chosen over Docusaurus/mdBook for a lower setup cost with strong out-of-the-box search, admonitions, and theming; GitHub Pages is used specifically for the docs site's static hosting (separate from Gitea, which hosts the actual codebase). |
| **Cloudflare Pages** | Static CDN hosting for the frontend SPA (`apps/web` build output). Global edge network, managed TLS, auto-deploy from GitHub mirror. Per [ADR-003](decisions/adr-003-hosting-topology.md). |
| **Render** | NestJS API hosting (Node.js web service, free tier). Auto-deploy from GitHub mirror. ~30s cold start after 15 min inactivity accepted by the team. Per [ADR-003](decisions/adr-003-hosting-topology.md). |
| **Supabase** | Managed PostgreSQL hosting with built-in connection pooling and a free tier. Supabase's auto REST/Auth/Storage APIs are deliberately unused — the NestJS API stays the only HTTP path to the data. Per [ADR-003](decisions/adr-003-hosting-topology.md). |

!!! success "CI/CD host confirmed: Gitea + GitHub mirror"
    CI runs on Gitea Actions, matching everywhere else the repo is described. CD is now live: the Gitea repo is mirrored to GitHub, which triggers auto-deploys to Cloudflare Pages (frontend) and Render (API). The docs site deploys separately via GitHub Pages from the docs repo.

## Confirmed: local dev proxy

`vite.config.ts` proxies `/api/*` to `http://localhost:4000` (with the `/api` prefix stripped before forwarding), so `apiClient.ts`'s relative `/api/v1/...` calls do reach the NestJS backend in dev. In production, `VITE_API_BASE_URL` points at the Render API URL directly.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
