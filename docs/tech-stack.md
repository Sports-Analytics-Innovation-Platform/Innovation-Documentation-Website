# Tech Stack

Every component below is listed with why it was chosen, not just what it is — this doc exists specifically to score the brief's "Tech Stack" criterion (motivated stack, not just named).

## Backend

| Choice | Why |
|---|---|
| **NestJS** (TypeScript) | Started as a plain Express + Prisma API, migrated to NestJS on 2026-08-06 for its structured module/controller/service pattern, built-in dependency injection, guards for auth, and a global exception filter — this gives a consistent shape to the hand-written API the brief requires, rather than routes accumulating ad hoc across an Express app as the team grows to six people working in parallel. |
| **Prisma** (ORM) | Type-safe query builder and migration tool over Postgres. Keeps the schema in one place (`schema.prisma`), generates TypeScript types automatically, and makes migrations reviewable in PRs rather than hand-written SQL drifting from the actual DB state. **Confirmed in use** — a real `schema.prisma` and migration (`20260806085439_init`) exist in the repo. |
| **PostgreSQL** | Relational DB, matches the naturally relational shape of NBA data (players, teams, games, events) and is well-supported by Prisma and every hosting option we're likely to use. |

!!! warning "Open question: Prisma vs. Drizzle"
    The team's own stack list still poses this as `Prisma (or Drizzle)`. What's actually built and merged is Prisma — but the team has told me this isn't formally decided, just what's in place so far. Treat Prisma as "current default," not "final decision," until someone says otherwise.
| **BetterAuth** (Google OAuth) | Established authentication library, satisfying the brief's ban on hand-rolled auth (§2.1). Replaces the initial scaffold's Passport.js local-strategy — the team's confirmed target stack always specified BetterAuth + Google OAuth; the scaffold shipped with Passport first and was migrated on top of it. See [ADR-002](decisions/adr-002-auth.md) for the full history, including a real ⚠️ compliance risk this migration introduces. |

## Data source

| Choice | Why |
|---|---|
| **`nba_api`** (Python client for stats.nba.com) | Free, MIT-licensed, actively maintained, and covers historical seasons, the current season, and a live in-progress-game endpoint — broader and more current than the StatsBomb EPL data the team originally considered (limited to two dated seasons). See [NBA vs EPL pitch](decisions/index.md) for the full comparison. |

!!! warning "Open question: `nba_api` is Python, the API is TypeScript"
    `nba_api` is a Python package, but the backend is NestJS/TypeScript. This doc doesn't yet capture how the two talk to each other — e.g. a separate scheduled Python ingestion script that writes to Postgres directly, which NestJS then reads, versus something else. Worth pinning down and folding into [Architecture](design/architecture.md) once decided, since it affects the non-monolithic and hand-written-API requirements too.

Per the brief's requirement that statistics be derived from individual event records rather than stored totals, `nba_api`'s precomputed advanced stats (offensive rating, PIE, usage%) are used only for **verification** — the team calculates these itself from event-level data.

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
| **Gitea** | Version control, hosted per the university's own requirement to use university-provided infrastructure — see [Git Methodology](git-methodology.md). |
| **Vitest** | Unit testing on both apps — **confirmed built**: Vitest + Supertest for backend integration tests (against a real disposable Postgres DB), Vitest + React Testing Library on the frontend. |
| **MkDocs Material + GitHub Pages** | Documentation site. MkDocs Material was chosen over Docusaurus/mdBook for a lower setup cost with strong out-of-the-box search, admonitions, and theming; GitHub Pages is used specifically for the docs site's static hosting (separate from Gitea, which hosts the actual codebase). |

!!! warning "CI/CD host — needs confirming"
    The team's own dev log describes "a GitLab CI pipeline... publishing to GitLab Pages" for test coverage, but everywhere else (including the same log) the codebase is on **Gitea**, not GitLab. This is most likely Gitea Actions (which looks similar to GitHub/GitLab-style YAML pipelines) described loosely as "GitLab CI" — but I'm not guessing which. Confirm which host is actually running the pipeline before this goes in the group report.

## Not yet decided

- **Production hosting** for the API and frontend — no equivalent to the FPL plan's Hetzner VPS decision has been made yet for the NBA project. See [ADR-003](decisions/adr-003-hosting-topology.md) (still a stub).

## Confirmed: local dev proxy

`vite.config.ts` proxies `/api/*` to `http://localhost:4000` (with the `/api` prefix stripped before forwarding), so `apiClient.ts`'s relative `/api/v1/...` calls do reach the NestJS backend in dev. This was an open question in a prior pass — it's now confirmed from the actual file.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
