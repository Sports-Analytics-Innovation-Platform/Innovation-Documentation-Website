# Getting Started

From a clean clone to a running app in about 15 minutes.

## Prerequisites

- **Node.js 24** and npm, for `apps/api` (NestJS) and `apps/web` (React and Vite)
- **Docker** with Compose, for Postgres
- **Git**, with access to the Gitea repository
- **Python 3**, only for the data jobs: ingestion, predictor, optimizer, the [Become Pro model](become-pro/valuation-model.md#operating-it), or [Player Archetypes](player-archetypes/index.md) (`apps/similarity`)

## 1. Clone the repo

```bash
git clone https://sdp.ms.wits.ac.za/innovation/sportsanalytics.git
cd sportsanalytics
```

## 2. Set up environment variables

```bash
cp apps/api/.env.example apps/api/.env
```

Fill in the values:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | Postgres connection string (default: `postgresql://postgres:postgres@localhost:55432/nba_analytics?schema=public`) |
| `PORT` | API port (default: `4000`) |
| `WEB_ORIGIN` | CORS origin for the frontend (default: `http://localhost:5173`, comma-separated for multiple) |
| `BETTER_AUTH_SECRET` | Generate with `openssl rand -base64 32` |
| `BETTER_AUTH_URL` | API origin (default: `http://localhost:4000`) |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID, from the [Google Cloud Console](https://console.cloud.google.com/apis/credentials) |
| `GOOGLE_CLIENT_SECRET` | Google OAuth client secret |
| `SUPABASE_URL`, `SUPABASE_SECRET_KEY`, `SUPABASE_AVATARS_BUCKET` | Profile picture uploads only |
| `PRISMA_LOG_QUERIES`, `API_CACHE_DISABLED` | Optional, for [measuring performance](design/performance.md#measuring-it-yourself) |
| `INGESTION_MODE` | Optional. `queue` sends local pulls through the queue and pull worker |

Then `cp apps/web/.env.example apps/web/.env`. Public data needs an API key, which the dev proxy adds from `SITE_PROXY_API_KEY`. Without it, signed-out pages get `401 API_KEY_REQUIRED`; signed-in pages work, because a session is enough. Seeding the database (step 4) creates one: `npx prisma db seed` prints a `SITE_PROXY_API_KEY=...` line to paste into `apps/web/.env`. The key is random on every run, and each run revokes the previous one, so paste the newest line. Only a hash of each key is stored, so a key can't be inserted into the database by hand.

Never commit a filled-in `.env` ([Security](security.md#secrets)).

## 3. Start Postgres

```bash
docker compose up -d
```

This starts Postgres 16 on port 55432, and a throwaway test database on 55433.

## 4. Set up the database

From `apps/api`:

```bash
cd apps/api
npm install
npx prisma migrate dev
npx prisma db seed   # loads mock NBA seed data and prints SITE_PROXY_API_KEY for apps/web/.env
```

## 5. Run the backend

```bash
npm run start:dev
```

The API runs at `http://localhost:4000`; `http://localhost:4000/v1/health` returns `{"status":"ok"}`.

## 6. Run the frontend

From `apps/web`, in a separate terminal:

```bash
cd apps/web
npm install
npm run dev
```

The app runs at `http://localhost:5173`. Vite forwards `/api` and `/auth` to the API, as Cloudflare does in production.

## 7. Run tests

```bash
# from apps/api (Vitest and Supertest; end-to-end tests use the test database)
npm run test

# from apps/web (Vitest and React Testing Library)
npm run test
```

## 8. Run the same checks CI runs

Before pushing, run what [CI](ci-cd.md) runs:

```bash
# apps/api
npm ci
npx prisma generate     # required before typecheck; needs no database
npm run lint            # ESLint 10, flat config
npx tsc --noEmit

# apps/web
npm ci
npm run lint            # oxlint
npx tsc -b --noEmit     # -b is required: tsconfig.json is solution-style
```

## Troubleshooting

| Problem | Likely cause |
|---|---|
| API can't connect to Postgres | Docker isn't running, or `DATABASE_URL` doesn't point at port 55432 |
| Prisma migration fails | Postgres is still starting; wait a few seconds, or check `docker compose logs` |
| Signed-out pages show no data | `SITE_PROXY_API_KEY` in `apps/web/.env` isn't the key the latest seed printed (step 4). Restart the web app after changing it. |
| Google sign-in fails | Check the Google variables, and that `http://localhost:4000/auth/callback/google` is a registered redirect URI |
| `tsc` can't find modules that are in `package.json` | Run `npm ci && npm run prisma:generate` in `apps/api` |
| Every `/api` and `/auth` call returns **502** | The API crashed at startup; check its log. A common cause is a `.env` still holding `.env.example`'s placeholder `SUPABASE_URL` |
| `train_valuation_model.py` says `DATABASE_URL` is not set | Deliberate: it reads only `apps/valuation/.env`, because the root `.env` points at production |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
