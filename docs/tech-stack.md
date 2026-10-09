# Tech Stack

Every runtime dependency in the app repo, with why it was chosen. Versions are from `apps/api/package.json`, `apps/web/package.json` and each Python service's `requirements.txt` on `main`.

## Backend (`apps/api`)

| Choice | Version | Why |
|---|---|---|
| **NestJS** | 10 | Modules, controllers, services and guards give six people one shape for the hand-written API the brief requires (§2.1). The API started as plain Express and moved to NestJS on 6 Aug. |
| **Express** | 5 | The HTTP server under NestJS. Version 5 is needed for the `*splat` wildcard routes (`/auth/*splat` and the 404 catch-all). |
| **TypeScript** | 5.9 | Type checks against Prisma's generated client; NestJS's decorators need it. |
| **reflect-metadata**, **RxJS** | –, 7 | Required by NestJS's dependency injection and interceptors. Not used directly. |
| **Prisma** | 5.22 | One schema file (40 tables, 7 enums), generated TypeScript types, and migrations as reviewable SQL. See [ADR-001](decisions/adr-001-database.md). |
| **PostgreSQL** | 16 locally | NBA data is relational (players, teams, games, plays), and season stats are aggregates, which SQL does well. Hosted on Supabase. |
| **BetterAuth** | 1.3 | An established auth library, as the brief requires (§2.1). Google sign-in, with sessions in Postgres through its Prisma adapter; mounted at `/auth/*`. See [ADR-002](decisions/adr-002-auth.md). |
| **Zod** | 4 | Validates every write route's request body and returns a readable reason when it rejects one. |
| **Helmet** | 8 | Standard security headers in one line. |
| **cors** | – | Allows only the site's own origins. In production the browser reaches the API through the same-origin proxy ([ADR-003](decisions/adr-003-hosting-topology.md#how-the-browser-reaches-the-api)), so this mainly covers local development. |
| **@nestjs/swagger** | 7 | Generates the OpenAPI spec and the live Swagger UI at `/api/docs` from the controllers ([API Reference](api-reference.md)). |
| **@nestjs/schedule** | 4 | Checks the pull schedule every hour ([Data Ingestion](design/ingestion.md)). |
| **@supabase/supabase-js** + **ws** | 2 | Stores profile pictures in a private Supabase bucket. Only the API uses it, with a server-side key. `ws` is passed to it because its constructor needs a WebSocket on Node versions before 22. |
| **multer** | 2 | Receives the profile-picture upload (`POST /v1/me/avatar`). |
| **dotenv** | 17 | Reads `.env` into the environment, so no setting is hard-coded. |

**Development:** nodemon (restart on change), tsx (runs `prisma/seed.ts`), Supertest (end-to-end HTTP tests), ESLint 10 with `typescript-eslint` 8, and Vitest 4 with `@vitest/coverage-v8`. See [Testing](testing.md) and [CI/CD](ci-cd.md).

## Frontend (`apps/web`)

| Choice | Version | Why |
|---|---|---|
| **React** | 19 | The team knew it, and a single-page app talks to the API only over HTTP, which keeps the front end separate from the back end. |
| **Vite** | 8 | Fast dev server and build with little configuration. In development it forwards `/api` to the local API. |
| **TypeScript** | 6 | Catches wrong props and API response shapes at build time. |
| **React Router** | 7 | Client-side page routes ([UI Overview](design/wireframes.md)). |
| **TanStack Query** | 5 | Caching, loading states and refetching for API calls, instead of hand-written `useEffect` fetching. |
| **BetterAuth client** | 1.6 | Sign-in, sign-out and the current session, matching the API's auth library. |
| **Tailwind CSS** | 4 | Utility classes keep six people's pages consistent without a stylesheet per component. Theme colours are CSS variables. |
| **shadcn/ui** (with Radix Slot, class-variance-authority, clsx, tailwind-merge) | – | Accessible components copied into the repo, so they can be changed freely. The helper libraries handle variants and merging class names. |
| **Lucide** | 1 | Consistent SVG icons; only the ones used are bundled. |
| **Recharts** | 3 | The trait radars, points trend and landing-page charts, styled with the same theme variables. |

**Testing:** Vitest 4 with coverage, React Testing Library (tests by role and label, as a user would), `user-event`, `jest-dom`, jsdom, and axe-core for automated accessibility checks. **Linting:** oxlint, which is much faster than ESLint on a React codebase.

## Python services

The Python jobs write to the same Postgres database as the API, so the two languages need no bridge between them. They run on a team member's computer, because stats.nba.com blocks cloud networks ([ADR-003](decisions/adr-003-hosting-topology.md)).

| Service | Libraries | Why |
|---|---|---|
| **Ingestion** (`apps/ingestion`) | `nba_api` 1.11, `requests` 2.32 | `nba_api` is a free, MIT-licensed client for stats.nba.com covering past and current seasons, which was broader than the football data first considered ([pitch](decisions/index.md)). `requests` fetches betting-market odds. |
| **Predictor** (`apps/predictor`) | NumPy 2.1 | Elo ratings and the Four Factors model for game predictions. |
| **Optimizer** (`apps/optimizer`) | PuLP 2.9 | Solves the fantasy lineup as an integer program: most projected points under the salary cap. Simpler to install than OR-Tools. |
| **Valuation** (`apps/valuation`) | NumPy 2.1 | Least-squares fit for the [Become Pro](become-pro/valuation-model.md) draft-slot model. About 140 training rows support little more, and every coefficient stays readable. The API applies the model itself in TypeScript. |
| **Similarity** (`apps/similarity`) | scikit-learn 1.5, NumPy 2.1 | K-Means clustering and principal components for [Player Archetypes](player-archetypes/index.md). |
| All five | `psycopg2-binary` 2.9, `python-dotenv` 1.0, `pytest` 8.3 | Direct Postgres access, settings from `.env`, and unit tests. |

Box-score totals are derived from the stored play-by-play. Plus-minus, usage and ratings are stored as the NBA publishes them, because they need data the platform doesn't hold ([ADR-001](decisions/adr-001-database.md#design-rules-in-the-schema), rule 2).

## Infrastructure

| Choice | Why |
|---|---|
| **Gitea** and **Gitea Actions** | The university's required Git host; its built-in CI runs lint, typecheck and tests on every push ([CI/CD](ci-cd.md)). |
| **Cloudflare Pages** | Free static hosting for the web app; its Functions proxy `/api` and `/auth` to the API. |
| **Render** | Free hosting for a long-running Node server. |
| **Supabase** | Free managed Postgres with a connection pooler, used only as a database and a private file bucket. |
| **Docker Compose** | One command starts the local and test databases. |
| **MkDocs Material** on **GitHub Pages** | This site: search, admonitions and theming with little setup. |

Hosting choices and alternatives are in [ADR-003](decisions/adr-003-hosting-topology.md).

## Considered and not used

- **Redis** for caching: the API is a single instance, so an in-process cache does the same job ([ADR-004](decisions/adr-004-caching-strategy.md)).
- **Redis and BullMQ** for job queues: queued pulls are rows in Postgres, claimed by the pull worker.
- **Object storage** for dataset releases: each release stores its CSV in its database row.
- **Firebase**: banned by the brief, and a poor fit for relational data ([ADR-001](decisions/adr-001-database.md#alternatives-considered)).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Opus 5.5]*
