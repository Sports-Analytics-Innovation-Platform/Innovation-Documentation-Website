# CI/CD Pipeline

CI runs on **Gitea Actions** from one workflow, `.gitea/workflows/ci.yml` in the app repo, on every push and pull request. CD runs from a GitHub mirror of the Gitea repo, which Cloudflare Pages and Render deploy from. This meets the brief's CI/CD requirement (§2.1).

## CI: three parallel jobs

| Job | Timeout | Steps |
|---|---|---|
| `api` (*API — lint, typecheck*) | 15 min | `npm ci` → `npm run prisma:generate` → `npm run lint` (ESLint) → `npx tsc --noEmit -p tsconfig.json` |
| `web` (*Web — lint, typecheck*) | 15 min | `npm ci` → `npm run lint` (oxlint) → `npx tsc -b --noEmit` |
| `coverage` (*API and Web — test coverage*) | 20 min | Start a disposable Postgres → install both apps → `prisma generate` → `npm run test:cov` in the API, then the web app → merge both reports → upload the `coverage-report` artifact → remove the database |

All three run on `ubuntu-latest` (the label of the course's shared runner) with Node 24. None depends on another, so one push reports every failure at once. A newer push to the same branch or pull request cancels the run in progress.

## What fails the build

| Check | Fails on |
|---|---|
| API lint | Any ESLint error (`@eslint/js` and `typescript-eslint` recommended) |
| Web lint | Any oxlint error |
| Typecheck | Any TypeScript error in either app |
| API tests | Any failing unit or end-to-end test, run against a real Postgres |
| Web tests | Any failing component test |
| Coverage | Either app below **80%** lines, statements, functions or branches, set in `apps/api/vitest.config.ts` and `apps/web/vite.config.ts` |
| Report | The merged coverage report being empty (`if-no-files-found: error`) |

Test counts and coverage figures are on [Testing](testing.md).

## Why it is built this way

- **One job per app.** The repo doesn't use npm workspaces: each app has its own lockfile and its own linter (ESLint for the API, oxlint for the web app).
- **Tests in their own job.** They are slow and need a database, so the static checks report back in minutes without waiting for Postgres.
- **`prisma generate` is an explicit step.** The API doesn't typecheck without the generated client. Generating it reads only `schema.prisma` and needs no database.
- **`tsc -b` for the web app.** Its `tsconfig.json` only references other projects, so a plain `tsc --noEmit` would check nothing and pass.
- **No npm cache.** On this runner, an unreachable cache server fails the step instead of falling back to a normal install. The workflow keeps the cache settings commented out until that is fixed.
- **`upload-artifact@v3.2.2-node20`, not v4.** The university's Gitea (1.24.7) only supports the older artifact protocol, and the runner rejects actions built for Node 24.

### The test database

The shared runner runs jobs on the host network, so a `services:` Postgres container never got a hostname and its fixed port clashed with other groups' jobs. Three attempts failed that way.

Instead, a step starts `postgres:16-alpine` as an ordinary container, with its data in memory, and tries three ways to reach it, in order: by name on the job's Docker network, by IP on that network, then through a randomly published port on the host. Each is checked with a TCP probe and `pg_isready` before it is used, and the working address is exported as `DATABASE_URL`. A final step removes the container even if the tests fail.

The test suite applies the migrations itself (`global-setup.ts` runs `prisma migrate deploy`), so the workflow has no migration step. The job's other environment variables are dummies: `BETTER_AUTH_SECRET` is `ci-secret-not-for-production-use` and is never used outside CI.

## CD

| Component | Host | Deploys when | Live |
|---|---|---|---|
| Web app | Cloudflare Pages | A push to `main`, via the GitHub mirror; Pages builds `apps/web` | [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/) |
| API | Render | The same push; Render builds, then starts with `npx prisma migrate deploy && npm start` | [`/v1/health`](https://sportsanalytics-api.onrender.com/v1/health) |
| Database schema | Supabase | Pending migrations are applied each time the API starts | — |
| Docs | GitHub Pages | A push to `main` in the docs repo runs `mkdocs build --strict` and publishes | This site |
| Python jobs | A team member's computer | Run by hand; stats.nba.com blocks cloud hosts | — |

GitHub is only a deploy trigger: the code lives on Gitea, and CI runs there. Hosting choices are explained in [ADR-003](decisions/adr-003-hosting-topology.md).

## Not in CI yet

- **No build step.** CI lints, typechecks and tests; the build itself is checked by the deploy.
- **No Python tests.** The four Python services' `pytest` suites run locally only.
- **No secret scanning.** [Security](security.md) relies on review and the pre-commit check.
- **No npm cache** and **no `upload-artifact@v4`**, until the runner's cache server and the Gitea version allow them.

## Running CI's tests locally

See [Testing: Running tests](testing.md#running-tests). After pulling new dependencies, run `npm ci` and `npm run prisma:generate` in `apps/api` before typechecking, as CI does, so a stale `node_modules` or Prisma client doesn't produce false errors.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5], Qoder[Qoder Lite], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
