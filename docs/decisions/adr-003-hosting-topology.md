# ADR-003: Hosting topology

**Status:** Accepted, in use since 2026-08-19.

## Decision

| Part | Host | Why |
|---|---|---|
| Web app (React) | **Cloudflare Pages** | Static files on a global network with HTTPS, free. Pages Functions proxy `/api` and `/auth` to the API. |
| API (NestJS) | **Render**, free web service | Runs a long-lived Node server, which the API is. A pinger keeps it awake. |
| Database | **Supabase**, managed PostgreSQL | Standard Postgres with a connection pooler, free. |
| Profile pictures | **Supabase Storage**, private bucket | Keeps images out of the database. |
| Python jobs (ingestion, predictor, optimizer, valuation) | **A team member's computer** | stats.nba.com blocks cloud networks, so ingestion has to run from a home connection. |

Supabase is used only as a database and a private file bucket. Its generated API and its auth are not used, and the browser never talks to it: every request goes through our own API routes, which keeps this within the brief (§2.1).

Both apps deploy automatically from a GitHub mirror of the Gitea repo. The database design is [ADR-001](adr-001-database.md); the diagram is on [Architecture](../design/architecture.md#deployment-diagram).

## Context

1. **The API has to run continuously.** It is one long-running server, not per-request functions, so serverless hosts fit poorly.
2. **The web app and API are on different domains,** and Safari, Firefox and Brave block or partition cross-site cookies, which broke sign-in.
3. **The API must be the only way to the data** (§2.1), which rules out using a generated database API.
4. **The code is on the university's Gitea,** and most hosts deploy only from GitHub.
5. **The budget is zero.** Every service needed a free plan that lasts the semester.

The first plan was Microsoft Azure on a student subscription. Its region policy blocked Static Web Apps in every region offered, and Microsoft grants no exceptions to student subscriptions, so the team moved to the hosts above on 19 Aug.

## How the browser reaches the API

The browser only ever talks to `sportsanalytics.pages.dev`. Pages Functions (`functions/api/[[path]].ts` and `functions/auth/[[path]].ts` in the app repo) forward `/api/*` and `/auth/*` to Render. This makes the session cookie first-party, which fixed Google sign-in on Safari, Firefox and Brave. The proxy also adds the site's own API key, so signed-out visitors can browse while the API requires a key or a session.

## Configuration

Secrets live in each host's dashboard, never in the repo.

| Host | Variables |
|---|---|
| Render | `DATABASE_URL`, `DIRECT_URL`, `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `WEB_ORIGIN`, `SUPABASE_URL`, `SUPABASE_SECRET_KEY`, `SUPABASE_AVATARS_BUCKET` |
| Cloudflare Pages | `API_ORIGIN` (the Render URL), `SITE_PROXY_API_KEY`. `VITE_API_BASE_URL` is left unset so the app uses the proxy. |

## Database deployment

| Environment | Where | Data kept? | Schema applied by |
|---|---|---|---|
| Local development | Docker `postgres:16-alpine` on port 55432 | Yes | `npx prisma migrate dev` |
| Local tests | A second container on port 55433 | No | The test suite, before it starts |
| CI | A container per run ([CI/CD](../ci-cd.md)) | No | The test suite |
| Production | Supabase | Yes | The API, on every start |

Every environment is built from the same migrations, so each test run also proves the full history applies to an empty database.

**Connection strings.** `DATABASE_URL` uses Supabase's pooler in transaction mode (port 6543), which lends a connection per request; session mode hit Supabase's connection limit. `DIRECT_URL` (port 5432) is used for migrations, because Prisma's migration lock needs a direct connection. The Python jobs use the pooler in session mode.

**Schema changes.** Render starts the API with `npx prisma migrate deploy && npm start`. It applies only committed, reviewed migrations, and does nothing if the database is up to date. **If a migration fails, the API doesn't start,** rather than running new code against an old schema.

**Loading data.** Deploying changes the structure, not the data. NBA data reaches production in one of two ways, both run from a home connection:

1. **Direct:** a team member runs ingestion with its connection string pointed at production.
2. **Queued:** an admin clicks *Pull data* in the web app, which queues an `IngestionRequest`; `pull_worker.py` claims and runs it.

Ingestion upserts on NBA IDs, so re-running it is safe, and it can land a batch for admin review instead of publishing it. The predictor and optimizer are re-run afterwards. A full pull takes 35–45 minutes; single-purpose backfill scripts fill new columns without a full re-run. Details: [Data Ingestion](../design/ingestion.md).

**Profile pictures.** Users upload through `POST /v1/me/avatar`; only the API talks to Supabase Storage, with a server-only secret key. The database stores the file path. Viewers get a signed link that expires after an hour.

**Backups and limits.**

- **There are no automatic backups.** Supabase's free plan has none. Migrations rebuild the structure and ingestion rebuilds the NBA data, but **user data could not be recovered.**
- The free plan allows **500 MB**. On 27 Sep the database was at 392 MB, 323 MB of it one season of play-by-play, which is why only 2025-26 keeps play-by-play ([ADR-005](adr-005-play-by-play-storage.md)).
- Supabase pauses inactive free projects. The pinger doesn't prevent that: `/v1/health` doesn't query the database.

## Alternatives considered

| Option | Why not |
|---|---|
| **Azure** (Static Web Apps, App Service, Container Apps, Azure Postgres) | Blocked by the student subscription's region policy. Keeping only the database there would have meant a second account to manage. |
| **Neon or Railway** for the database | Considered with Supabase on 18 Aug. Supabase won on its built-in pooler and free plan. All three run standard Postgres, so switching would mean changing two connection strings and running the migrations. |
| **Fly.io** for the API | Its free tier is an allowance, not a permanent free plan, so an always-on server would eventually be billed. |
| **Render Starter** ($7/month) | Avoids the free plan's sleep, but the pinger does the same for free. |
| **Render scheduled jobs** for the Python jobs | stats.nba.com blocks cloud networks, so ingestion fails on any cloud host. |
| **Vercel** for the web app | Its serverless model doesn't suit the API, and Cloudflare Pages deployed more easily from the mirror. |

## Consequences

**Benefits**

- All of production runs on free plans.
- Schema changes reach production with no manual step, and a broken migration stops the API instead of damaging data.
- Development, test and production databases share one migration history.
- One origin for the browser, so sign-in works on every major browser.

**Drawbacks**

- **No backups.** Losing the Supabase project would lose all user data. This is the biggest open risk.
- **Data is only as fresh as the last manual pull,** and a new feature can show empty values until its data job runs.
- **The direct pull points a laptop at production.** If it isn't pointed back, the next local test writes to production.
- **Three dashboards** (Cloudflare, Render, Supabase), each with its own settings.
- **Deploys depend on the GitHub mirror.** If it stops syncing, nothing deploys.
- **The Supabase secret key bypasses all of Supabase's access rules,** so it is kept only on Render.

## Open questions

1. **Backups:** a scheduled `supabase db dump` stored elsewhere, or a paid plan?
2. **Fresh data:** an admin can queue a pull, but someone still has to run the worker. Who, and how often?
3. **Region:** the API runs in Render's default region (Oregon). Would a closer one be noticeably faster from South Africa?
4. **Domains:** custom domains would allow a shared parent domain for the cookies instead of the proxy.

## Sources

In the app repo: `render.yaml`, `functions/`, `apps/web/src/lib/apiBase.ts`, `apps/api/src/me/avatar-storage.service.ts`, `apps/ingestion/README.md` and `docker-compose.yml`. External: [Supabase: Database Backups](https://supabase.com/docs/guides/platform/backups).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
