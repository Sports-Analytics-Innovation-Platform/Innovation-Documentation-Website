# Getting Started

This page should get a new contributor from a clean clone to a running app in under 15 minutes. If it doesn't, that's a bug in this page — fix it, don't just work around it.

!!! warning "Placeholder values"
    Repo URLs, exact env variable names, and a couple of commands below are placeholders (`<...>`) — this page was drafted from the tech stack and the initial scaffold described in the [AI Usage Ledger](ai-usage.md), not from the actual repo. Swap in the real values and delete this note once verified.

## Prerequisites

- **Node.js** (LTS) and **npm** or **pnpm** — for both `apps/api` (NestJS) and `apps/web` (React + Vite)
- **Docker** and **Docker Compose** — runs Postgres locally
- **Git**, with access to the project's Gitea repository
- **Python 3** — only if you're working on the `nba_api` ingestion piece (see the open question in [Tech Stack](tech-stack.md))

## 1. Clone the repo

```bash
git clone <gitea-repo-url>
cd <repo-name>
```

## 2. Set up environment variables

```bash
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env
```

Fill in the values `.env.example` documents (database connection string, Passport/session secret, etc.) — never commit the filled-in `.env` files. See [Security](security.md) for why.

## 3. Start Postgres

```bash
docker compose up -d
```

This brings up Postgres locally so both apps can connect to it. Confirm it's running with `docker compose ps`.

## 4. Set up the database

From `apps/api`:

```bash
cd apps/api
npm install
npx prisma migrate dev
npx prisma db seed   # loads mock NBA seed data, if a seed script exists
```

## 5. Run the backend

```bash
npm run start:dev
```

The NestJS API should now be running (default `http://localhost:3000` unless configured otherwise). Check the OpenAPI/Swagger docs, once set up, at `/api` or `/docs`.

## 6. Run the frontend

From `apps/web`, in a separate terminal:

```bash
cd apps/web
npm install
npm run dev
```

Vite's dev server will print the local URL (default `http://localhost:5173`).

## 7. Run tests

```bash
# from apps/api
npm run test

# from apps/web
npm run test
```

(Test setup isn't finalised yet — see the open item in [Tech Stack](tech-stack.md). Update this section once a framework is chosen.)

## Troubleshooting

| Problem | Likely cause |
|---|---|
| API can't connect to Postgres | Docker Compose isn't running, or the connection string in `apps/api/.env` doesn't match the Compose service | 
| Prisma migration fails | Database not reachable yet — wait a few seconds after `docker compose up -d` before migrating, or check `docker compose logs` |
| Frontend shows no data / requests to `/api/...` 404 or fail with a CORS error | The frontend calls a **relative** path (`/api/v1/...`), not `http://localhost:3000`. This only works if `vite.config.ts` proxies `/api` to the backend — if that proxy isn't configured, local dev won't work at all. Check for a `server.proxy` block in `vite.config.ts` first before assuming your own env is broken. |

!!! warning "Unconfirmed: dev proxy setup"
    See [Tech Stack](tech-stack.md#not-yet-decided) — whether `vite.config.ts` actually proxies `/api` to the backend hasn't been confirmed. If it doesn't exist yet, add it before this getting-started guide can be verified end-to-end.

If you hit something not covered here, add it to this table once you've solved it — that's the point of this page.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
 