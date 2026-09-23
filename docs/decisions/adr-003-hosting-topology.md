# ADR-003: Hosting topology

- **Status:** Accepted. Implemented on 2026-08-19.
- **Last updated:** 2026-09-14

## Summary

The platform is split across three hosting services, plus scripts that team members run on their own computers:

| Part of the system | Where it runs |
|---|---|
| Website (the React front end) | Cloudflare Pages |
| API (the NestJS back end) | Render |
| Database and profile-picture storage | Supabase |
| Data scripts (ingestion, predictor, optimizer) | A team member's computer, run by hand |

This document explains how these parts connect, how the database is deployed and kept up to date, which other hosts were considered, and the risks of the current setup. The design of the database itself is covered in [ADR-001: Database](adr-001-database.md).

## Change log

| Date | Change |
|---|---|
| 2026-09-23 | Documented the queued ingestion pull (`IngestionRequest`/`pull_worker.py`), added alongside the original direct-script method — an admin can now trigger a pull from the web UI, though a human-run worker on a home connection still has to claim it. |
| 2026-09-14 | Updated to match the live deployment. Schema changes are now applied automatically when the API starts, the API uses two database connection strings, Supabase file storage is used for profile pictures, the data scripts run on a team member's computer, and the database has no automatic backups. Added the [Database deployment](#database-deployment) section. |
| 2026-08-24 | A *pinger* (a service that sends the API a request at regular intervals) now stops it from going to sleep, removing the start-up delay described under [Render's free plan](#renders-free-plan). |
| 2026-08-19 | The first plan, based on Microsoft Azure, was replaced with Cloudflare Pages, Render and Supabase (see [Azure](#azure)). |

## Context

The platform has four parts:

- **The website** is a React single-page application. Building it produces static files (HTML, JavaScript and CSS) that any web host can serve.
- **The API** is a NestJS server. It is the only way the website, or anyone else, can read or change data.
- **The database** is PostgreSQL ([ADR-001](adr-001-database.md)).
- **The data scripts** are Python programs that download NBA data and calculate predictions, writing the results straight to the database.

Several requirements and constraints shaped where these could run:

1. **The API has to run continuously.** It is built as one long-running server process, not as short functions started for each request, so "serverless" hosting platforms are a poor fit.
2. **Sign-in relies on cookies across two web addresses.** The website and the API are served from different domains. Login cookies therefore have to be configured for cross-site use (`SameSite=None; Secure`), and the API needs a fixed address.
3. **The brief requires the API to be the only way to reach the data** (§2.1). Services that automatically generate an API from a database, such as Firebase and Supabase's own API, can't be used for that purpose.
4. **The source code is hosted on the university's Gitea server.** Most hosting services deploy automatically from GitHub, but not from Gitea.
5. **The budget is effectively zero.** Every service needed a free or student plan that would last for the whole semester.

The team first planned to use Microsoft Azure through its student subscription. Azure's student subscriptions only allow certain regions, and that policy blocked Azure Static Web Apps (the service meant to host the website) in every region offered. Microsoft doesn't grant exceptions for student subscriptions, and the other planned Azure services were at risk of the same restriction. Fly.io was considered next, but its free allowance isn't a permanent free plan for a server that has to stay on all semester. The team therefore moved to the services below.

## Decision

**The website is hosted on Cloudflare Pages, the API on Render, and the database on Supabase. The data scripts are run by hand from a team member's computer.**

Supabase is used for two things only: a managed PostgreSQL database, and one private storage bucket for profile pictures. Supabase's automatically generated database API and its built-in authentication are not used, and the browser never connects to Supabase at all. All requests for data or files go through routes written by hand in the project's API, which keeps the setup within the brief's rule (§2.1).

The data scripts can't run on a cloud host, because stats.nba.com, the source of the NBA data, blocks requests coming from cloud providers' networks (see [Loading production data](#loading-production-data)).

### How the parts connect

```
                           User's browser
                                 |
               +-----------------+------------------+
               |                                    |
               v                                    v
     +-------------------+              +-----------------------+
     | Website           |              | API                   |
     | Cloudflare Pages  |              | Render                |
     | (static files)    |              | (NestJS server, kept  |
     +-------------------+              |  awake by a pinger)   |
                                        +-----------+-----------+
                                                    |
                        database queries, schema    |   profile picture
                        changes                     |   uploads and links
                                                    v
                                 +-------------------------------------+
                                 | Supabase                            |
                                 |   PostgreSQL database               |
                                 |   Private profile-picture bucket    |
                                 +-------------------------------------+
                                                    ^
                                                    | direct database connection,
                                                    | run by hand
                                 +-------------------------------------+
                                 | Team member's computer              |
                                 | (home internet connection)          |
                                 |   Ingestion  <---  stats.nba.com    |
                                 |   Predictor and optimizer           |
                                 +-------------------------------------+
```

The browser downloads the website from Cloudflare, and the website then calls the API. The browser never talks to Supabase directly.

### Where each part runs

| Part | Service | Why this service |
|---|---|---|
| Website | Cloudflare Pages | Serves the static files from a global network with HTTPS included, on a generous free plan. It deploys automatically from a GitHub mirror of the Gitea repository (an automatically updated copy). |
| API | Render web service (free plan) | Runs a long-lived Node.js server, which the API requires. The free plan puts the server to sleep after 15 minutes without requests, so a pinger sends it requests at regular intervals to keep it awake. Login sessions are stored in the database, so users stay signed in if the server restarts. Render also deploys automatically from the GitHub mirror, using the settings in `render.yaml`. |
| Database | Supabase (managed PostgreSQL) | Standard PostgreSQL on a free plan, with a built-in connection pooler (explained under [Connection strings](#connection-strings)). All connections are encrypted. See [Database hosting](#database-hosting) for the other providers considered. |
| Profile pictures | Supabase Storage (private bucket) | Keeps uploaded images out of the database. See [Profile picture storage](#profile-picture-storage). |
| Data scripts | A team member's computer | stats.nba.com blocks cloud providers' networks, so the ingestion script must run from a home internet connection. The predictor and optimizer run straight afterwards, on the same computer. |

### Configuration and secrets

Passwords, keys and addresses are stored as environment variables in each hosting service's dashboard. They are never committed to either repository.

| Where | Variables |
|---|---|
| Render (API) | `DATABASE_URL` and `DIRECT_URL` (database connections), `BETTER_AUTH_SECRET` and `BETTER_AUTH_URL` (sign-in), `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` (Google sign-in), `WEB_ORIGIN` (the website's address, which the API accepts requests from), and `SUPABASE_URL`, `SUPABASE_SECRET_KEY` and `SUPABASE_AVATARS_BUCKET` (profile-picture storage) |
| Cloudflare Pages (website) | `VITE_API_BASE_URL` — the API's address, built into the website's files |

For sign-in to work across the two domains:

- `WEB_ORIGIN` must be the website's exact production address, and `VITE_API_BASE_URL` and `BETTER_AUTH_URL` the API's.
- Google's sign-in settings must list `https://<api-domain>/auth/callback/google` as an allowed redirect address.
- Login cookies must be marked `Secure` and `SameSite=None`, or browsers won't send them from the website to the API.

Local development is unaffected: developers run the database in Docker and start both apps on their own machine.

## Database deployment

### Environments

Every copy of the database is built from the same migration files ([ADR-001](adr-001-database.md#schema-change-history)); only the connection details differ.

| Environment | Where it runs | Is the data kept? | How the schema is applied |
|---|---|---|---|
| Local development | Docker container (`postgres:16-alpine`) on port `55432` | Yes, in a Docker volume | Developer runs `npx prisma migrate dev` |
| Local tests | A second Docker container on port `55433` | No, it starts empty every time | The test suite applies all migrations before it starts |
| Automated tests (CI) | A temporary database created for each test run ([CI/CD Pipeline](../ci-cd.md)) | No, removed after the run | The test suite, as above |
| Production | Supabase | Yes, managed by Supabase | Applied automatically when the API starts |

Tests use a separate, disposable database so they can never damage development data. Because every test run builds its database from nothing, each run also confirms that the full migration history still works on an empty database.

### Connection strings

PostgreSQL can only handle a limited number of open connections at once. A **connection pooler** sits in front of the database and shares a small number of real connections among many clients. Supabase provides one, but the pooler can't do everything a direct connection can, so the API is given two addresses:

| Variable | Connects to | Used for | Why |
|---|---|---|---|
| `DATABASE_URL` | Supabase's pooler in *transaction mode* (port `6543`) | Everything the API does while serving requests | The API makes many short database requests. Transaction mode lends out a real connection only for the length of each request. An earlier setting (*session mode*) kept one real connection per client and quickly hit Supabase's connection limit. |
| `DIRECT_URL` | The database itself (port `5432`) | Applying schema changes | Prisma locks the database while it applies migrations, and the pooler's transaction mode doesn't support that kind of lock. |

The data scripts also use the direct connection.

### Schema changes

Render starts the API with `npx prisma migrate deploy && npm start`, and has done since 2026-08-29. This command applies any migrations that production hasn't received yet, then starts the server.

- It only applies migration files that were reviewed and committed. It never creates new changes of its own, and does nothing if the database is already up to date, so it is safe to run on every start.
- **If a migration fails, the API doesn't start.** This is deliberate: running new code against a database that is missing the columns it expects would cause errors, or save incorrect data.
- Migrations that have already been applied must never be edited. Prisma checks each applied migration's fingerprint on every start, and would refuse to start the API if one had changed.

### Loading production data

Deploying the code updates the database's *structure* automatically, but not its *data*. NBA data reaches production one of two ways, as of 2026-09-23:

1. **Direct (original method).** A team member runs the ingestion script from a home internet connection, with its database address temporarily pointed at the production database, then switches it back afterwards. Still necessary because stats.nba.com blocks cloud providers' networks, so the pull itself can never run on Render.
2. **Queued (added this sprint).** An admin triggers a pull from the web app's admin UI. This writes an `IngestionRequest` row rather than running anything immediately; `apps/ingestion/pull_worker.py`, polling from wherever it's running (still someone's home machine — the cloud-IP block applies here too, this just moves *where in the process* a human is involved, not whether one still is), claims the request and runs the pull. This is what the deployed API itself does when an admin clicks "Pull Data" in production, since the API process can't run `nba_api` calls directly either.

Either way: the script upserts every row using the NBA's own IDs (idempotent — running it again updates existing rows rather than duplicating them), and can land a batch as `PENDING_REVIEW` for admin approval instead of auto-publishing (see [Feature Tiers](../design/feature-tiers.md)). The predictor and optimizer are then re-run so predictions and lineups reflect the new data.

**New columns need a separate data run.** A migration can add a column to production, but only a data script can fill it. On 2026-09-02, for example, the player biography columns had been deployed but were empty in production; running `backfill_player_bios.py` filled them for 530 players the same day. Smaller single-purpose scripts (`backfill_player_bios.py`, `backfill_advanced_stats.py` and `ingest_postseason.py`) exist so production can be filled in without repeating the full 25–35 minute ingestion.

**The sample-data script must never be run against production.** `npm run prisma:seed` deletes every game and all game statistics before inserting sample data.

### Profile picture storage

Profile pictures are kept in a Supabase Storage bucket called "profile pictures". This is the only Supabase feature used besides the database, and it is set up so the API remains the only way to reach user data:

- The bucket is **private**. Nobody can download a file from it without a signed link.
- Users upload pictures to the project's own API route (`POST /v1/me/avatar`). Only the API talks to Supabase Storage, using a secret key that exists only on the server.
- The database stores the file's location, not a web link. When a profile is viewed, the API creates a link that expires after one hour, and reuses it for up to 55 minutes so that an expired link is never handed out.
- The secret key bypasses Supabase's access rules, so it is kept only in Render's settings and is never sent to the website.

The brief bans services that *generate API endpoints*. Here, Supabase Storage is used only as a place to keep files behind a route the team wrote, the same role Amazon S3 would play.

### Backups and limits

- **The database has no automatic backups.** Supabase backs up paid projects daily, but free projects get no automatic backups and no point-in-time recovery ([Supabase documentation](https://supabase.com/docs/guides/platform/backups)). For free projects, Supabase recommends regularly exporting the database with `supabase db dump` and keeping the export somewhere else. This isn't done yet (see [Open questions](#open-questions)).
- **What could be recovered if the database were lost:**
    - the *structure*, fully, by re-running the migrations;
    - the *NBA data*, by re-running ingestion (about 25–35 minutes, plus the backfill scripts);
    - but **not user data**. Accounts, followed players, calls and saved lineups and comparisons exist only in production.
- **Free-plan limits.** Supabase's free plan limits the database's size and pauses projects that have been inactive for a while. The pinger doesn't prevent this: it calls the API's `/health` route, which doesn't query the database. Only real use of the website keeps the database active.

## Alternatives considered

### Azure

Azure was the original choice for everything: Static Web Apps for the website, App Service for the API, Container Apps for the data scripts, and Azure Database for PostgreSQL for the database. The student subscription's region policy blocked Static Web Apps in every region offered, and put the other services at risk of the same block, so Azure was abandoned.

Keeping only the database on Azure would still have depended on that subscription, and would have meant managing a separate Azure account alongside Render, so the database moved as well.

### Database hosting

When the team moved the database off a local Docker container on 2026-08-18, three managed PostgreSQL providers were considered: **Supabase, Neon and Railway**. Supabase was chosen because:

- **It runs standard PostgreSQL.** The existing schema, migrations and Python scripts worked without changes; only the connection address changed.
- **It includes a connection pooler,** which a server making many short database requests needs.
- **It has a free plan,** which the rest of this setup also relies on.

The team's records don't include a detailed comparison of Neon and Railway. Both also run standard PostgreSQL, which makes this decision easy to reverse: moving to another provider would mean changing the two connection strings and running the migrations there.

Supabase is named in the brief as an example of a banned service (§2.1), because it can generate an API automatically. Using it only as a database and a private file bucket, behind the project's own API, is what keeps this setup within the rules (see [Profile picture storage](#profile-picture-storage)).

### Fly.io

Fly.io was considered for the API and data scripts, because its servers can be configured to stay on permanently. However, its free tier is a usage allowance rather than a permanent free plan, so a server running all semester would eventually be billed. Render's free plan was preferred.

### Render's free plan

Render's free plan puts a server to sleep after 15 minutes without requests. The next request then waits about 30 seconds while the server starts up again. Render's Starter plan ($7 per month) avoids this, but wasn't chosen. The team initially accepted the delay, then on 2026-08-24 added a pinger that keeps the server awake.

### Running the data scripts on Render

The original plan ran the ingestion, predictor and optimizer scripts on a schedule using Render's scheduled jobs. This was dropped because stats.nba.com blocks requests from cloud providers' networks (a widely reported problem in the `nba_api` library's issue tracker), so ingestion fails on any cloud host. The predictor and optimizer could run on Render, but they only have new work to do straight after ingestion, so they run on the same computer.

### Vercel

Vercel was considered first for the website because it is simple to set up. Its servers run as serverless functions, which suit the long-running API poorly. Vercel could have hosted the website alone, but Cloudflare Pages offers the same benefits and was easier to deploy to from this project's repositories.

## Consequences

### Benefits

- The entire production system runs on free plans.
- The API runs as the long-running server it was designed to be, and the pinger keeps it responsive.
- Schema changes reach production without any manual step, and a broken migration stops the API from starting instead of damaging data.
- Development, testing and production databases are all built from the same migration files, so they share exactly the same structure.
- The website is served from Cloudflare's global network, so it loads quickly wherever users are.

### Drawbacks

- **No automatic backups.** If the Supabase project were lost, all user data would be lost permanently. This is the biggest open risk in the current setup.
- **Production data can lag behind the code.** Because data is loaded by hand, a newly deployed feature can show empty values until someone runs the matching data script.
- **Data is only as fresh as the last manual run.** Nothing refreshes the NBA data on a schedule.
- **Loading data involves a risky temporary change.** A team member's computer is briefly pointed at the production database. If it isn't switched back, the next local test would write to production.
- **Three providers to manage.** Cloudflare, Render and Supabase each have their own dashboard and their own copy of the settings.
- **Deployments depend on the GitHub mirror.** If the copy from Gitea to GitHub stops updating, neither the website nor the API will deploy.
- **Cookie settings are easy to break.** If the cross-site cookie settings are wrong, sign-in fails in production without an obvious error.
- **One key has broad access.** The Supabase secret key bypasses all of Supabase's access rules, so anyone who obtained Render's settings could read the storage bucket as well as the database.

### Completed follow-ups

- Automatic deployment of the website and API: **done**, from the GitHub mirror.
- Applying migrations in production: **done**, automatically on API start since 2026-08-29.
- Scheduled data scripts on Render: **dropped**, because stats.nba.com blocks cloud networks.
- Listing production settings and secrets: **done**, see [Configuration and secrets](#configuration-and-secrets).

## Open questions

1. **Backups.** Should the database be exported on a schedule (for example, a weekly automated `supabase db dump` saved outside Supabase), or should the project move to Supabase's paid plan before final submission? User data can't be recreated if it is lost.
2. **Keeping data current.** ⚠️ Partially answered 2026-09-23: an admin can now *trigger* a pull from the web app rather than only via direct script access (see [Loading production data](#loading-production-data)), but a human still has to be running `pull_worker.py` on a home connection to actually claim and execute it — the "who, how often" question is softer than it was, not closed.
3. **Server region.** The API runs in Render's default region (Oregon, USA). Would a region closer to South Africa noticeably improve response times?
4. **Domain names.** Should both apps get custom domain names, or keep the default `onrender.com` and `pages.dev` addresses for the demonstration?

## Sources

In the source repository:

- `apps/api/src/main.ts` — how the API server starts, which is why it needs a long-running host.
- `render.yaml` — Render's build and start settings, including applying migrations on start and the notes on the two connection strings.
- `apps/api/prisma/schema.prisma` — the two database connection settings.
- `apps/api/.env.example` — the format of every connection and storage variable.
- `apps/api/src/me/avatar-storage.service.ts` — the only code that uses Supabase Storage.
- `apps/ingestion/README.md` — why ingestion must run from a home connection, how long it takes, and the backfill scripts.
- `docker-compose.yml` — the local development and test databases.

External:

- [Supabase: Database Backups](https://supabase.com/docs/guides/platform/backups) — which backups each Supabase plan includes.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5] (2026-09-23: documented the queued ingestion pull)*
