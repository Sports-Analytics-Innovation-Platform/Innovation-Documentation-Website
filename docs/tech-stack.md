# Tech Stack

Every component below is listed with why it was chosen, not just what it is — this doc exists specifically to score the brief's "Tech Stack" criterion (motivated stack, not just named).

## Backend

| Choice | Why |
|---|---|
| **NestJS** (TypeScript) | Started as a plain Express + Prisma API, migrated to NestJS on 2026-08-06 for its structured module/controller/service pattern, built-in dependency injection, guards for auth, and a global exception filter — this gives a consistent shape to the hand-written API the brief requires, rather than routes accumulating ad hoc across an Express app as the team grows to six people working in parallel. |
| **Prisma** (ORM) | Type-safe query builder and migration tool over Postgres. Keeps the schema in one place (`schema.prisma`), generates TypeScript types automatically, and makes migrations reviewable in PRs rather than hand-written SQL drifting from the actual DB state. |
| **PostgreSQL** | Relational DB, matches the naturally relational shape of NBA data (players, teams, games, events) and is well-supported by Prisma and every hosting option we're likely to use. |
| **Passport.js** | Established authentication library rather than hand-rolled auth, satisfying the brief's requirement to rely on established practices for auth (§2.1). Handles sign up, sign in, and session/token strategy; password reset and account deletion are built on top of it in our own API. |

## Data source

| Choice | Why |
|---|---|
| **`nba_api`** (Python client for stats.nba.com) | Free, MIT-licensed, actively maintained, and covers historical seasons, the current season, and a live in-progress-game endpoint — broader and more current than the StatsBomb EPL data the team originally considered (limited to two dated seasons). See [NBA vs EPL pitch](../decisions/index.md) for the full comparison. |

!!! warning "Open question: `nba_api` is Python, the API is TypeScript"
    `nba_api` is a Python package, but the backend is NestJS/TypeScript. This doc doesn't yet capture how the two talk to each other — e.g. a separate scheduled Python ingestion script that writes to Postgres directly, which NestJS then reads, versus something else. Worth pinning down and folding into [Architecture](../design/architecture.md) once decided, since it affects the non-monolithic and hand-written-API requirements too.

Per the brief's requirement that statistics be derived from individual event records rather than stored totals, `nba_api`'s precomputed advanced stats (offensive rating, PIE, usage%) are used only for **verification** — the team calculates these itself from event-level data.

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
| **Gitea** | Version control and CI/CD (Gitea Actions), hosted per the university's own requirement to use university-provided infrastructure — see [Git Methodology](git-methodology.md). |
| **MkDocs Material + GitHub Pages** | Documentation site. MkDocs Material was chosen over Docusaurus/mdBook for a lower setup cost with strong out-of-the-box search, admonitions, and theming; GitHub Pages is used specifically for the docs site's static hosting (separate from Gitea, which hosts the actual codebase). |

## Not yet decided

- **Production hosting** for the API and frontend — no equivalent to the FPL plan's Hetzner VPS decision has been made yet for the NBA project.
- **Testing framework** — not yet documented; needed for [Definition of Done](definition-of-done.md) item 2 and the Sprint 2 "Automated Testing" rubric line.

!!! danger "Open question: does local dev actually work end-to-end?"
    `apiClient.ts` fetches a **relative** path (`/api/v1/players`), not an absolute URL like `http://localhost:3000`. Vite's dev server and the NestJS API run on different ports, which means they're different origins — a relative fetch from the frontend will only reach the backend if something proxies `/api` through to it, typically a `server.proxy` entry in `vite.config.ts`. I don't have that file, so I can't confirm this exists. If it doesn't, local dev is broken as-is and either a Vite proxy needs adding or `apiClient.ts` needs an absolute base URL (from an env var). Worth checking before someone burns an hour on "why is my API request 404ing."

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
