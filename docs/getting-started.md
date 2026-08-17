# Getting Started

This page gets a new contributor from a clean clone to a running app. Steps below are verified against the actual repo (`apps/api`, `apps/web`, `apps/optimizer`), not drafted from the tech stack sheet.

## Prerequisites

- **Node.js 20+** and **npm** — for both `apps/api` (NestJS) and `apps/web` (React + Vite)
- **Docker** and **Docker Compose** — runs Postgres locally (a native or WSL-hosted Postgres on port 5432 works too, if you point `DATABASE_URL` at it instead)
- **Git**, with access to the project's GitLab repository (`sdp.ms.wits.ac.za`)
- **Python 3** — only if you're running `apps/optimizer` (the MILP lineup optimizer)

## 1. Clone the repo

```bash
git clone https://sdp.ms.wits.ac.za/innovation/sportsanalytics.git
cd sportsanalytics
```

## 2. Start Postgres

```bash
docker compose up -d postgres
```

This runs Postgres on `localhost:55432` (not 5432, to avoid clashing with any Postgres already installed on your machine).

## 3. Set up the API

```bash
cd apps/api
cp .env.example .env
npm install
npm run prisma:migrate   # creates tables
npm run prisma:seed      # loads mock NBA players/teams/games/stats
npm run dev              # starts on http://localhost:4000
```

Sign-in uses Google OAuth via [BetterAuth](https://better-auth.com) — see [Security](security.md) for what's needed to enable it. Without `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET` in `.env`, the API still runs (everything except signing in works), it just logs a startup warning.

## 4. Set up the frontend

In a separate terminal:

```bash
cd apps/web
npm install
npm run dev               # starts on http://localhost:5173
```

The Vite dev server proxies `/api/*` to `http://localhost:4000` (see `vite.config.ts`'s `server.proxy` block) — confirmed working, the frontend never needs to know the API's real port.

## 5. (Optional) Set up the optimizer

The lineup optimizer (`apps/optimizer`) is a separate Python service — see its own [README](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/-/blob/main/apps/optimizer/README.md) for the full setup. Short version:

```bash
cd apps/optimizer
python -m venv .venv
.venv\Scripts\activate        # Windows; source .venv/bin/activate on macOS/Linux
pip install -r requirements.txt
cp .env.example .env          # points DATABASE_URL at the same Postgres instance
python predict.py             # writes a PlayerPrediction row per player
python optimize.py            # reads predictions, writes a Lineup
```

The API's `GET /v1/optimizer/lineup` only reads what these scripts write — it doesn't run them itself.

## 6. Open the app

Go to `http://localhost:5173` and browse to **Players** to see seeded mock NBA data with derived season averages.

## 7. Run tests

Both apps use Vitest. The API's tests run against a real, disposable Postgres database rather than a mocked Prisma client.

```bash
npm run db:up:test              # one-off: start the disposable test database

cd apps/api
cp .env.test.example .env.test  # one-off
npm test                        # or npm run test:cov for coverage

cd apps/web
npm test                        # or npm run test:cov for coverage
```

## Troubleshooting

| Problem | Likely cause |
|---|---|
| API can't connect to Postgres | Docker Compose isn't running, or `DATABASE_URL` in `apps/api/.env` doesn't match — check it points at port `55432` (or your Postgres's actual port, if not using Docker) |
| Prisma migration fails | Database not reachable yet — wait a few seconds after starting Postgres before migrating, or check `docker compose logs` |
| Frontend shows no data | The API isn't running, or `apps/api/.env` is missing — check the API terminal for errors first |

If you hit something not covered here, add it to this table once you've solved it — that's the point of this page.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
 