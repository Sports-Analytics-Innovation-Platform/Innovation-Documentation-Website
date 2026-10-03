# Architecture Overview

The platform is not a monolith. The web app (`apps/web`) and the API (`apps/api`) are separate applications, deployed separately, that talk only over HTTP. Python services write to Postgres as separate processes and never go through the API.

| Component | What it is | Host |
|---|---|---|
| `apps/web` | React single-page app | Cloudflare Pages ([sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/)) |
| `apps/api` | NestJS REST API, versioned under `/v1/` | Render ([Swagger UI](https://sportsanalytics-api.onrender.com/api/docs)) |
| Database | PostgreSQL with Prisma | Supabase |
| `apps/ingestion`, `apps/predictor`, `apps/optimizer`, `apps/valuation` | Python batch jobs | Run from a team member's machine |
| CI | Lint, typecheck and tests on every push | Gitea Actions |
| Docs | This site | GitHub Pages |

Why these hosts: [ADR-003: Hosting Topology](../decisions/adr-003-hosting-topology.md).

## Deployment diagram

![Deployment diagram](diagrams/deployment.svg)

The production path is Cloudflare Pages → Render → Supabase. Pages Functions proxy `/api` and `/auth` on the web app's own origin, so the session cookie is first-party. Ingestion runs on a team member's machine because stats.nba.com blocks cloud networks: the API queues a pull and the pull worker runs it ([Data Ingestion](ingestion.md)).

## Frontend (`apps/web`)

- **React 19 and Vite**, **Tailwind CSS v4** (theme tokens in `index.css`) and shadcn/ui components.
- **React Router** with 16 routes. The [UI Overview](wireframes.md#pages) lists them.
- **TanStack Query** for fetching and caching. Every call goes through one wrapper in `lib/apiClient.ts`, with `credentials: "include"` for the session cookie.
- **Recharts** for the traits radar and the points trend.

## Backend (`apps/api`)

- **NestJS, Prisma and PostgreSQL** ([ADR-001](../decisions/adr-001-database.md)). One module per feature, each with a controller and a service; every service uses the shared `PrismaService`.
- **BetterAuth** with Google OAuth, mounted at `/auth/*` ([ADR-002](../decisions/adr-002-auth.md)).
- **Guards:** public reads need a session or an `X-API-Key`; `/v1/me/*` and the optimizer need a session; `/v1/admin/*` needs the `ADMIN` role (`RolesGuard`). The [API Reference](../api-reference.md#authentication) has the details.
- **One error envelope** for every error response ([API Design](api-design.md)).
- **An in-process response cache** for public reads ([ADR-004](../decisions/adr-004-caching-strategy.md), [Performance](performance.md)).
- **Health check** at [`/v1/health`](https://sportsanalytics-api.onrender.com/v1/health). A pinger keeps the free Render instance warm.

### Class diagram

![Backend class diagram](diagrams/class-diagram.svg)

Drawn in Sprint 1: controllers depend on their services, and every service goes through `PrismaService`. The modules added since (admin, datasets, API keys, custom statistics, Become Pro) follow the same pattern.

### Sequence diagram: `GET /v1/games/:id/prediction`

![Sequence diagram: game prediction request](diagrams/sequence-game-prediction.svg)

One request end to end: the CORS check against `WEB_ORIGIN`, the session cookie, and the two different 404s (no such game, or no prediction yet). The prediction itself is written earlier by `apps/predictor`, never during the request.

## Python services

| Service | What it does | Writes to |
|---|---|---|
| `apps/ingestion` | Pulls teams, rosters, games, box scores and play-by-play with `nba_api`. Can land a batch for admin review (`--review`). `pull_worker.py` runs pulls queued from the admin page. | NBA data and ingestion tables |
| `apps/predictor` | Elo win probability and Four Factors margin for each game | `GamePrediction`, `GamePredictionRun` |
| `apps/optimizer` | Projects fantasy points and picks five players under a salary cap with MILP (PuLP/CBC) | `PlayerPrediction`, `Lineup`, `LineupSlot` |
| `apps/valuation` | Fits the Become Pro draft-slot model on real NBA rookie seasons. The API applies it whenever a user's season changes. | `ProspectValuationModel` |

The tables are on the [ERD](erd.md).

## Local development

Docker Compose runs Postgres; the API and the web app each run their own dev server against it. See [Getting Started](../getting-started.md).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
