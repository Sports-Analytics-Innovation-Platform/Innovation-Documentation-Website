# Tech Stack

Every component below is listed with why it was chosen, not just what it is — this doc exists specifically to score the brief's "Tech Stack" criterion (motivated stack, not just named). Every runtime dependency in `apps/api/package.json`, `apps/web/package.json`, and the three Python services' `requirements.txt` is accounted for here.

## Backend (apps/api)

### Framework

| Choice | Why |
|---|---|
| **NestJS 10** (TypeScript) | Started as a plain Express + Prisma API, migrated to NestJS on 2026-08-06 for its structured module/controller/service pattern, built-in dependency injection, guards for auth, and a global exception filter — this gives a consistent shape to the hand-written API the brief requires, rather than routes accumulating ad hoc across an Express app as the team grows to six people working in parallel. |
| **Express 5** | Underlying HTTP framework that NestJS runs on via `@nestjs/platform-express`. The API uses Express 5 directly (not Nest's bundled Express 4) for the `ExpressAdapter` in `main.ts` — this matters specifically because the `NotFoundController` catch-all route uses Express 5's `*splat` wildcard syntax, which silently fails under Express 4. |
| **TypeScript 5.9** | Type-safe JavaScript — every backend source file is `.ts`, compiled to `.js` via `tsc`. Required for Prisma's generated client types and NestJS's decorator metadata. |
| **reflect-metadata** | Required by NestJS for decorator-based dependency injection and route metadata. Peer dependency of the NestJS decorator system. |
| **RxJS 7** | Required by NestJS for its reactive internals (interceptors, guards, pipes return `Observable`). Not used directly in application code beyond what Nest wraps. |

### Database

| Choice | Why |
|---|---|
| **Prisma 5.22** (ORM) | Type-safe query builder and migration tool over Postgres. Keeps the schema in one place (`schema.prisma`), generates TypeScript types automatically, and makes migrations reviewable in PRs rather than hand-written SQL drifting from the actual DB state. Confirmed as the team's choice — a real `schema.prisma` with 15 models and multiple migrations exist in the repo. |
| **PostgreSQL** (via Supabase) | Relational DB, matches the naturally relational shape of NBA data (players, teams, games, events, predictions) and is well-supported by Prisma and Supabase's managed hosting. |

### Authentication

| Choice | Why |
|---|---|
| **BetterAuth** | Established authentication library, satisfying the brief's ban on hand-rolled auth (§2.1). Replaces the initial scaffold's Passport.js local-strategy. BetterAuth mounts its own route set at `/api/auth/*` and manages sessions via cookies. See [ADR-002](decisions/adr-002-auth.md) for the full history. |

### Security & HTTP

| Choice | Why |
|---|---|
| **Helmet** | Sets security-related HTTP response headers (`X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`, etc.) — one-line middleware import that hardens the API against common web vulnerabilities without manual header configuration. |
| **CORS** (`cors` package) | Cross-origin resource sharing middleware — required because the frontend (Cloudflare Pages) and API (Render) are on different domains in production. Configured in `main.ts` with an explicit allowlist of origins from `auth.config.ts`. |

### Validation

| Choice | Why |
|---|---|
| **Zod** | Schema validation for incoming data (feedback submissions, query parameters). Provides runtime type-checking with descriptive rejection errors. Confirmed built — used in the feedback controller for request body validation. |

### Environment

| Choice | Why |
|---|---|
| **dotenv** | Loads environment variables from `.env` files into `process.env` at startup. Used in `main.ts` (`import "dotenv/config"`) so the API reads `DATABASE_URL`, `BETTER_AUTH_SECRET`, `PORT`, etc. from the environment rather than hardcoding configuration. |

### Development tools

| Choice | Why |
|---|---|
| **nodemon** | Hot-reload during development — watches `src/` for changes and auto-rebuilds/restarts the API. Used by the `dev` script: `nodemon --watch src --ext ts --exec "npm run build && node dist/main.js"`. |
| **tsx** | TypeScript execution for scripts that don't go through the Nest build pipeline — specifically `prisma/seed.ts` for database seeding. Runs TS directly via `node --loader tsx` without a separate compile step. |
| **Supertest** | HTTP assertion library for E2E tests — makes real HTTP requests against the test NestJS application and asserts on status codes, response bodies, and headers. Used in every `test/**/*.e2e-spec.ts` file. |
| **ESLint 10** + **typescript-eslint 8.66** | Linting for the API. Flat config in ESM (`eslint.config.js`) composing `@eslint/js` + `typescript-eslint` recommended. Enforced in CI — a lint error fails the build. See [CI/CD Pipeline](ci-cd.md). |
| **Vitest 4** + **@vitest/coverage-v8** | Test runner and coverage. v8 provider instruments at the V8 level for accurate coverage without Babel overhead. See [Testing](testing.md). |

## Frontend (apps/web)

### Framework

| Choice | Why |
|---|---|
| **React 19** | Team familiarity, large ecosystem, straightforward to keep the frontend fully decoupled from the backend (non-monolithic requirement) since it only talks to the API over HTTP. |
| **Vite 8** | Fast dev server and build tool, minimal config compared to older bundlers. Uses `@vitejs/plugin-react` for Fast Refresh in development and optimised production builds. |
| **TypeScript 6** | Type-safe JavaScript for the frontend — catches prop-type mismatches, missing route params, and API response shape errors at compile time rather than in the browser. |
| **React Router 7** (`react-router-dom`) | Client-side routing with eight routes: `/`, `/players`, `/players/:playerId`, `/teams`, `/teams/:teamId`, `/predictions`, `/optimizer`, `/games/:gameId`. |

### Styling & UI

| Choice | Why |
|---|---|
| **Tailwind CSS 4** (`@tailwindcss/vite`) | Utility-first styling — quick to build consistent, responsive layouts without hand-writing a separate stylesheet per component, which matters given six people are touching the frontend. v4 uses the Vite plugin directly (no `tailwind.config.js` needed) with `@theme` custom properties. |
| **shadcn/ui** | Component library, paired with Tailwind. Confirmed built — used for Teams pages, general UI components, and the court view. Components are copied into the repo (not installed as a package), so they're fully customisable. Configured via `components.json` with the `base-nova` style and neutral base colour. |
| **Radix UI** (`@radix-ui/react-slot`) | Headless UI primitives that shadcn/ui builds on. `react-slot` specifically lets components compose their props onto a single DOM element — used by shadcn's `Button`, `Card`, and other polymorphic components. |
| **Lucide React** (`lucide-react`) | Icon library — consistent, tree-shakeable SVG icons used across the UI (navigation, stat cards, error states). Chosen over Heroicons/Feather for better coverage of sports-relevant icons and a cleaner visual weight at small sizes. |
| **class-variance-authority** (`cva`) | Component variant system — defines typed variant props for shadcn components (e.g. `Button` with `variant="default" | "outline" | "ghost"` and `size="sm" | "md" | "lg"`). Keeps variant logic co-located with the component instead of scattered across className strings. |
| **clsx** + **tailwind-merge** | Conditional class name utilities. `clsx` conditionally joins class strings; `tailwind-merge` resolves Tailwind class conflicts (e.g. `px-4 px-6` → `px-6`). Used together in the `cn()` helper that every shadcn component calls. |
| **Recharts 3** | Charting library used for the player-profile radar and points-trend charts — themed against the same CSS custom properties as the rest of the UI rather than hardcoded colours. |

### Data fetching

| Choice | Why |
|---|---|
| **TanStack Query 5** (`@tanstack/react-query`) | Data-fetching/caching against the API. Handles loading states, background refetching, cache invalidation, and optimistic updates. Confirmed built — added as part of the frontend foundations work. |
| **BetterAuth** (client SDK) | Frontend auth client — manages session state, provides `signIn`, `signOut`, `getSession` helpers. The session cookie is sent automatically with `credentials: "include"` on every `fetch` call. |

### Testing

| Choice | Why |
|---|---|
| **Vitest 4** + **@vitest/coverage-v8** | Test runner and coverage for the frontend — same tooling as the API for consistency. |
| **React Testing Library** (`@testing-library/react`) | Component testing — tests components the way users interact with them (by role, label, text) rather than implementation details. Used in every `src/**/*.spec.tsx` file. |
| **@testing-library/user-event** | Simulates real user interactions (click, type, tab) in component tests — more realistic than `fireEvent` because it fires the full sequence of browser events a real user would trigger. |
| **@testing-library/jest-dom** | Custom DOM matchers (`toBeInTheDocument()`, `toHaveTextContent()`, `toBeVisible()`) — makes test assertions readable and specific to DOM elements. |
| **jsdom** | In-browser DOM environment for Vitest — provides `document`, `window`, and DOM APIs without a real browser, so component tests can render and query React trees. |
| **oxlint** | Linting for the frontend — dramatically faster than ESLint on large React codebases (written in Rust). Configured via `.oxlintrc.json`. Enforced in CI. |

## Python services

### Data ingestion (apps/ingestion)

| Choice | Why |
|---|---|
| **`nba_api` 1.11** | Free, MIT-licensed Python client for stats.nba.com. Covers historical seasons, the current season, and a live in-progress-game endpoint — broader and more current than the StatsBomb EPL data the team originally considered (limited to two dated seasons). See [NBA vs EPL pitch](decisions/index.md) for the full comparison. |
| **`psycopg2-binary`** | PostgreSQL adapter for Python — the ingestion service writes directly to the same Supabase database that the NestJS API reads from. Binary wheel avoids needing a C compiler on the deployment target. |
| **`python-dotenv`** | Loads `.env` files for the ingestion scripts — reads `DATABASE_URL` and other config from the environment, matching the API's `dotenv` convention. |

### Prediction (apps/predictor)

| Choice | Why |
|---|---|
| **NumPy 2** | Numerical computation for the Elo rating system and Four Factors analysis — vectorised array operations for calculating rolling averages, exponential decay weighting, and regression coefficients. Used in `elo.py` and `four_factors.py`. |
| **`psycopg2-binary`** | Same as ingestion — reads game data from Postgres to compute predictions, writes results back to the `GamePrediction` table. |
| **`python-dotenv`** | Environment variable loading, same convention as other services. |
| **pytest** | Unit tests for the prediction logic — `test_four_factors.py` validates the Four Factors weight calculations and edge cases. |

### Optimisation (apps/optimizer)

| Choice | Why |
|---|---|
| **PuLP 2.9** | Python linear programming library — solves the MILP (Mixed Integer Linear Program) for the fantasy lineup optimiser. Given player salary costs and predicted fantasy points, PuLP finds the lineup that maximises total predicted points under a salary cap. Chosen over OR-Tools for a simpler API and pure-Python installation. |
| **`psycopg2-binary`** | Reads player predictions from Postgres, writes the optimised lineup back to the `Lineup` and `LineupSlot` tables. |
| **`python-dotenv`** | Environment variable loading, same convention as other services. |

### Python services: shared architecture

`nba_api` is a Python package, and the backend is NestJS/TypeScript. This is resolved by running `nba_api` as a separate Python ingestion service (`apps/ingestion`) that writes directly to Postgres — NestJS then reads from the same database. The two languages coexist without needing an inter-process bridge because Postgres is the shared data layer. See [Architecture](design/architecture.md) for the full diagram.

Per the brief's requirement that statistics be derived from individual event records rather than stored totals, `nba_api`'s precomputed advanced stats (offensive rating, PIE, usage%) are used only for **verification** — the team calculates these itself from event-level data.

## Infrastructure

| Choice | Why |
|---|---|
| **Docker Compose** | Runs Postgres locally with one command, so every team member's dev environment matches without a manual install. |
| **Gitea** | Version control, hosted per the university's own requirement to use university-provided infrastructure — see [Git Methodology](git-methodology.md). |
| **Gitea Actions** (CI) | Pipeline defined in `.gitea/workflows/ci.yml`, running lint → typecheck → test as three parallel jobs (`api`, `web`, `coverage`) on every push and PR, pinned to Node 24. Confirmed built. Chosen simply because it's built into the Gitea instance already hosting the code — no second platform to register runners with. See [CI/CD Pipeline](ci-cd.md). |
| **MkDocs Material + GitHub Pages** | Documentation site. MkDocs Material was chosen over Docusaurus/mdBook for a lower setup cost with strong out-of-the-box search, admonitions, and theming; GitHub Pages is used specifically for the docs site's static hosting (separate from Gitea, which hosts the actual codebase). |
| **Cloudflare Pages** | Static CDN hosting for the frontend SPA (`apps/web` build output). Global edge network, managed TLS, auto-deploy from GitHub mirror. Per [ADR-003](decisions/adr-003-hosting-topology.md). |
| **Render** | NestJS API hosting (Node.js web service, free tier). Auto-deploy from GitHub mirror. A pinger service keeps the instance warm to avoid cold-start delays. Per [ADR-003](decisions/adr-003-hosting-topology.md). |
| **Supabase** | Managed PostgreSQL hosting with built-in connection pooling and a free tier. Supabase's auto REST/Auth/Storage APIs are deliberately unused — the NestJS API stays the only HTTP path to the data. Per [ADR-003](decisions/adr-003-hosting-topology.md). |

## Planned / not yet adopted

These were confirmed as team decisions but are not yet in the codebase:

| Choice | Why | Status |
|---|---|---|
| **Redis + BullMQ** | Batch submission/ingestion processing and incremental recomputation jobs. | Planned if ingestion pipeline needs batch/scheduled processing |
| **Redis** (cache / rate limiting) | Doubles as the cache layer and a rate-limit store — relevant given `nba_api` itself is rate-limit-sensitive (see [Security](security.md)). | Planned if rate limiting is needed |
| **S3-compatible storage** (MinIO for self-hosted/course use) | For any exported/versioned data artifacts. | Planned if needed |
| **@nestjs/swagger** | Auto-generated OpenAPI 3.0 spec and Swagger UI from NestJS controller decorators. | Setup documented — see [API Reference](api-reference.md) |

!!! success "CI/CD host confirmed: Gitea + GitHub mirror"
    CI runs on Gitea Actions, matching everywhere else the repo is described. CD is now live: the Gitea repo is mirrored to GitHub, which triggers auto-deploys to Cloudflare Pages (frontend) and Render (API). The docs site deploys separately via GitHub Pages from the docs repo.

## Confirmed: local dev proxy

`vite.config.ts` proxies `/api/*` to `http://localhost:4000` (with the `/api` prefix stripped before forwarding), so `apiClient.ts`'s relative `/api/v1/...` calls do reach the NestJS backend in dev. In production, `VITE_API_BASE_URL` points at the Render API URL directly.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite]*
