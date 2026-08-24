# CI/CD Pipeline

Continuous integration runs on **Gitea Actions**, defined by a single workflow file at `.gitea/workflows/ci.yml` in the code monorepo (`sdp.ms.wits.ac.za`). It runs on every **push** and every **pull request**, and satisfies the brief's CI/CD requirement (§2.1) — see [Requirements Traceability](requirements.md).

Before this workflow existed there was no CI of any kind in the repo — no `.gitea/`, no `.github/workflows/`, no `.gitlab-ci.yml`. This page describes what now runs, why it's shaped the way it is, and what it deliberately does *not* do yet.

!!! note "Where the source of truth lives"
    The workflow file itself lives in the code repo, not in this docs repo. This page describes it; if the two ever disagree, the file wins and this page is the bug.

## What runs

**Three independent, parallel jobs**: `api`, `web`, and `coverage`. None of them declares `needs:`, so all three start together — a lint failure in `api` does **not** skip the coverage run, and you get every failure from a single push instead of discovering them one job at a time.

| Job | Display name | Timeout | What it does |
|---|---|---|---|
| `api` | *API — lint, typecheck* | 15 min | Static checks on `apps/api` |
| `web` | *Web — lint, typecheck* | 15 min | Static checks on `apps/web` |
| `coverage` | *API and Web — test coverage* | 20 min | Runs both test suites against a real Postgres, merges the reports, uploads the artifact |

`api` and `web` are split rather than merged because the monorepo root `package.json` does not use npm workspaces (it drives sub-apps via `--prefix` scripts) and each app carries its own `package-lock.json`. Separate lockfiles mean separate installs, so a shared job would be a lie about the dependency graph. Each sets `defaults.run.working-directory` to its own app.

The `coverage` job is the exception: it needs *both* apps installed in one workspace so the combined report can be built, so it uses `npm ci --prefix` per app instead of a working directory.

### Static checks

| Stage | `apps/api` | `apps/web` |
|---|---|---|
| Install | `npm ci` | `npm ci` |
| Codegen | `npm run prisma:generate` | — |
| Lint | `npm run lint`, behind an `eslint.config.*` probe (see below) | `npm run lint` (oxlint + `.oxlintrc.json`) |
| Typecheck | `npx tsc --noEmit -p tsconfig.json` | `npx tsc -b --noEmit` |

### Coverage

The `coverage` job runs, in order: install API deps → install web deps → `npm run prisma:generate --prefix apps/api` → `npm run test:cov --prefix apps/api` → `npm run test:cov --prefix apps/web` → `npm run coverage:report` (root script, merging both) → upload the `coverage-report` directory as a build artifact.

Runner: all three jobs use `runs-on: default` — the label the Wits `act_runner` actually registers, not GitHub's `ubuntu-latest`. `actions/checkout@v4` and `actions/setup-node@v4` pinned to **Node 24** (matching the team's local Node v24.14.0 / npm 11.9.0).

Abridged shape of the workflow — the real file is the authority:

```yaml
name: CI

on:
  push:
  pull_request:

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  api:
    runs-on: default
    timeout-minutes: 15
    defaults:
      run:
        working-directory: apps/api
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 24
      - run: npm ci
      - run: npm run prisma:generate   # prerequisite of the typecheck, made explicit
      - run: |                         # lint, guarded — see "the two pre-existing gaps"
          if ls eslint.config.* 2>/dev/null; then npm run lint; else echo "::warning::..."; fi
      - run: npx tsc --noEmit -p tsconfig.json

  web:
    runs-on: default
    timeout-minutes: 15
    defaults:
      run:
        working-directory: apps/web
    steps: [checkout, setup-node@24, npm ci, npm run lint, npx tsc -b --noEmit]

  coverage:
    runs-on: default
    timeout-minutes: 20
    services:
      postgres:
        image: postgres:16-alpine
        env: { POSTGRES_USER: postgres, POSTGRES_PASSWORD: postgres, POSTGRES_DB: nba_analytics_test }
        options: >-
          --health-cmd pg_isready --health-interval 10s
          --health-timeout 5s --health-retries 5
    env:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/nba_analytics_test?schema=public
      BETTER_AUTH_SECRET: ci-secret-not-for-production-use
      BETTER_AUTH_URL: http://localhost:4000
      WEB_ORIGIN: http://localhost:5173
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 24 }
      - run: npm ci --prefix apps/api
      - run: npm ci --prefix apps/web
      - run: npm run prisma:generate --prefix apps/api
      - run: npm run test:cov --prefix apps/api
      - run: npm run test:cov --prefix apps/web
      - run: npm run coverage:report
      - uses: actions/upload-artifact@v3.2.2   # not v4 — see "Gitea version constraints"
        with:
          name: coverage-report
          path: coverage-report
          if-no-files-found: error
```

## Why it's built this way

Each of these is a decision that changes behaviour, not a style preference.

- **Split by app, not one job.** Two lockfiles, two dependency trees, two toolchains (ESLint vs oxlint, TS 5.9 vs TS 6.0). Running them in parallel also means a web-only failure doesn't hide behind a slow API install.
- **Coverage is its own job, not a stage inside `api` and `web`.** Tests are the slow, stateful part — they need a database container and a 20-minute budget. Keeping them separate means the fast static checks report back in a couple of minutes instead of queueing behind Postgres coming up, and the combined report can only be built somewhere both apps are installed.
- **`concurrency` with `cancel-in-progress`.** A newer push supersedes an in-flight run. This matters specifically because the Gitea runner is shared and self-hosted — stale runs queueing behind each other cost everyone else wall-clock time.
- **npm caching is deliberately omitted.** `setup-node`'s `cache: npm` routes through `@actions/cache`, whose restore errors reach `core.setFailed` — so if the `act_runner` cache server is unreachable, the step *fails* rather than degrading to an uncached install. The pipeline was built to work unconditionally rather than to save ~45s. The workflow carries the exact commented-out block to enable it (`cache: npm` plus `cache-dependency-path: apps/api/package-lock.json`) once the runner's cache server is confirmed.
- **`prisma generate` is an explicit step**, in both the `api` job and the `coverage` job. The API typecheck genuinely cannot pass without a generated Prisma client — `src/` references model fields, so `tsc` fails outright — so it's a visible step rather than an implicit reliance on npm's `postinstall` side-effect. Generation itself needs **no database**: it reads `schema.prisma` only. Postgres is needed by the *tests*, not by codegen, which is why the `api` job has no service container.
- **`--noEmit` for API, `tsc -b` for web.** The API's `build` script emits to `dist/`, so the check uses `--noEmit -p tsconfig.json` to typecheck without producing artifacts. `apps/web/tsconfig.json` is solution-style (`files: []` plus `references` to the app and node projects), so `tsc -b` is mandatory — a plain `tsc --noEmit` there would check *nothing* and pass silently.
- **CD is live.** The Gitea repo is mirrored to GitHub, which triggers auto-deploys: Cloudflare Pages for the frontend, Render for the API. See the [CD Pipeline](#cd-pipeline) section below.

## The test database

The API e2e suite needs a real, disposable Postgres, so the `coverage` job declares a `postgres:16-alpine` **service container** with `POSTGRES_DB: nba_analytics_test` and a `pg_isready` healthcheck (10s interval, 5s timeout, 5 retries) so steps don't start against a database that isn't listening yet.

!!! warning "The connection string uses `postgres:5432`, not `localhost`"
    Jobs on this runner execute **inside containers**, not directly on the host. That puts the job and the service on the same container network, so the service is reached by its **hostname** (`postgres`) on its **normal container port** (5432) — there is no host port mapping to go through. Copying `localhost:5432` in from a GitHub-hosted example is the classic way to break this job.

Four environment variables are set at job level, so every step in the job sees them:

| Variable | Value | Why |
|---|---|---|
| `DATABASE_URL` | `postgresql://postgres:postgres@postgres:5432/nba_analytics_test?schema=public` | Points Prisma at the service container |
| `BETTER_AUTH_SECRET` | `ci-secret-not-for-production-use` | BetterAuth refuses to boot without one. The value is deliberately a self-documenting dummy — it is **not** a secret, is not read from Gitea Actions secrets, and must never be reused outside CI (see [Security](security.md)) |
| `BETTER_AUTH_URL` | `http://localhost:4000` | The API's own origin, as the test process sees it |
| `WEB_ORIGIN` | `http://localhost:5173` | CORS origin the API expects from the Vite dev server |

Note that the workflow itself runs **no migration step** — no `prisma migrate deploy`, no `db push`. Applying the schema to the fresh container is the test harness's responsibility, not the pipeline's. If the e2e suite ever starts failing on missing tables, that's the thing to check first.

## Gitea version constraints

Two places where the workflow is shaped by the server, not by preference:

- **`actions/upload-artifact` is pinned to `v3.2.2`, not `v4`.** The Gitea instance is **1.24.7**, which implements the *legacy* artifact protocol. `upload-artifact@v4` talks to a v4 backend and needs a patched action to work against Gitea at all. Staying on v3 keeps the step working with the stock action; revisit when the server is upgraded.
- **`runs-on: default`.** Self-hosted `act_runner` instances register whatever labels their config gives them, and this one registers `default`. `ubuntu-latest` would simply never be picked up.

`if-no-files-found: error` on the upload is a deliberate choice: if `coverage:report` silently produces nothing, the job fails instead of uploading an empty artifact and reporting green.

## Handling the two pre-existing gaps

The repo had two gaps at the time CI was introduced, both handled inside the workflow without changing anything else:

**1. `apps/api` had no ESLint config.** `npm run lint` failed outright with *"ESLint couldn't find an eslint.config.(js|mjs|cjs) file"*. Rather than blanket `continue-on-error` (which masks genuine failures forever) or a hard fail (a red pipeline on arrival for a pre-existing gap), the lint step **probes for `eslint.config.*`**: if absent it emits a `::warning::` and passes; if present it runs the real lint and lets a non-zero exit propagate.

!!! success "The lint guard is now enforcing, not warning"
    `apps/api/eslint.config.js` has since been added (see below), so the guard takes its live branch. **A lint error now fails the build.** The warning branch remains only as a safety net, and its message spells out the fix for anyone who hits it.

**2. No test files existed anywhere in either `src` tree**, so the API's `vitest run` exited 1 with *"No test files found"* and the API step was given `--passWithNoTests` as a temporary accommodation.

!!! success "`--passWithNoTests` has been retired"
    Tests now exist. The `api` and `web` jobs no longer run tests at all — the `coverage` job runs `npm run test:cov` in both apps against a real database, with no tolerance flag. An empty or failing suite now fails the pipeline.

## The API ESLint config

Adding CI surfaced that `apps/api` was missing its lint config entirely. Fixing it needed **no new packages** — `eslint@10.8.0`, `@eslint/js@10.0.1`, and `typescript-eslint@8.66.0` were already devDependencies; only the config file was absent. (typescript-eslint 8.66 was confirmed to run under ESLint 10, which was the main compatibility risk.)

`apps/api/eslint.config.js` is a **flat config in ESM** (the package is `"type": "module"`), composing `@eslint/js` + `typescript-eslint` recommended. Two rule overrides are functional rather than stylistic, and both are documented here so nobody "tidies them up" later:

| Rule | Setting | Why |
|---|---|---|
| `no-undef` | `off` for TypeScript files | On TS it only produces false positives — Node globals (`process`, `console`) and type-only names. TypeScript's own checker covers undefined identifiers properly. |
| `@typescript-eslint/no-empty-object-type` | `off`, scoped to `**/*.d.ts` | Without it, `express.d.ts:5` is flagged for `interface User extends PrismaUser {}`. That empty interface is *correct* — it's the canonical Express declaration-merging idiom that types `req.user`. Scoping the exception to `.d.ts` keeps the rule active everywhere it's meaningful. |

Across the whole API source that left exactly **one** genuine error: an unused `cors` import in `apps/api/src/main.ts`, since deleted. It was verified dead first — `cors` appears nowhere else in `src/`, and CORS is actually configured through Nest's built-in `app.enableCors({...})` at `main.ts:19`.

## What each job enforces

| Check | Enforced? | Notes |
|---|---|---|
| API lint | ✅ | Live branch of the guard; a lint error fails the build |
| API typecheck | ✅ | `tsc --noEmit`, after Prisma codegen |
| Web lint | ✅ | oxlint |
| Web typecheck | ✅ | `tsc -b` — solution-style config, build mode required |
| API tests | ✅ | Against a real Postgres service container |
| Web tests | ✅ | |
| Combined coverage report | ⚠️ produced, **not gated** | The report is built and uploaded; nothing fails on a low coverage number |

That last row is the one to read carefully. CI proves the suites **run and pass**; it does not yet enforce any coverage floor. Download the `coverage-report` artifact from the run to see the actual numbers — a green `coverage` job on its own says nothing about how much of the code is exercised.

## CD Pipeline

Continuous deployment is live, running via a Gitea → GitHub mirror that triggers auto-deploys on each host:

| Component | Host | Deploy trigger | Live URL |
|---|---|---|---|
| Frontend (`apps/web`) | Cloudflare Pages | GitHub mirror push → Cloudflare auto-deploy (builds `apps/web` with `npm run build`, serves `dist/`) | [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/) |
| API (`apps/api`) | Render | GitHub mirror push → Render auto-deploy (build with `npm run build`, start with `npm run start:prod`) | [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health) |
| Docs site | GitHub Pages | Separate docs repo push to `main` → GitHub Actions build and deploy (`.github/workflows/deploy-docs.yml`) | [Docs site](https://sports-analytics-innovation-platform.github.io/Innovation-Documentation-Website/) |
| Database | Supabase | Prisma migrations run manually or from CI (Render free tier doesn't support `preDeployCommand`) | — |
| Python services | Render (planned) | Not yet auto-deployed — planned as Render Cron Jobs or Background Workers | — |

The Gitea → GitHub push mirror was set up during Sprint 1 (week of 18 Aug). GitHub is used only as a deploy trigger — the source of truth remains on Gitea, and all CI (lint, typecheck, test) still runs on Gitea Actions.

Per [ADR-003](decisions/adr-003-hosting-topology.md): Cloudflare Pages provides a global CDN with managed TLS for the SPA; Render hosts the NestJS API as a long-running Node.js web service (free tier, kept warm by a pinger); Supabase provides managed Postgres with connection pooling over TLS.

## What CI does not do yet

Other pages on this site describe CI steps that are planned but **not in `ci.yml` today**. Stated plainly so nobody assumes coverage that doesn't exist:

- **No build step.** Lint, typecheck, and test only — nothing verifies that `apps/api` or `apps/web` actually builds in the CI pipeline (the build is verified by the CD deploy step instead).
- **No coverage threshold.** See above: reported, not gated.
- **No secret scanning.** [Git Methodology](git-methodology.md) and [Security](security.md) describe `gitleaks`/`trufflehog` as a PR backstop — that's the intent, not yet the implementation. The manual pre-commit check is currently the only line of defence.
- **No `axe-core` accessibility checks.** [Requirements Traceability](requirements.md) lists these for `apps/web`; they aren't wired up yet.

## Local parity

CI runs the same commands you can run locally, with two wrinkles worth knowing.

To reproduce the `coverage` job you need a Postgres to point at, and the same four environment variables the job sets. Locally the host *is* `localhost` (you are not inside the runner's container network), so the connection string differs from the one in the workflow:

```bash
docker run --rm -d --name nba-test-db -p 5432:5432 \
  -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=nba_analytics_test postgres:16-alpine

export DATABASE_URL='postgresql://postgres:postgres@localhost:5432/nba_analytics_test?schema=public'
export BETTER_AUTH_SECRET='local-dev-only'
export BETTER_AUTH_URL='http://localhost:4000'
export WEB_ORIGIN='http://localhost:5173'

npm run test:cov --prefix apps/api
npm run test:cov --prefix apps/web
npm run coverage:report        # writes ./coverage-report
```

!!! warning "Phantom typecheck errors from a stale local install"
    A local `apps/api` checkout that predates recent dependency additions produces ~19 bogus errors — `Cannot find module '@nestjs/passport'`, `'bcryptjs'`, `'passport'` — for packages that *are* declared in `package.json`, plus missing Prisma model fields (e.g. `passwordHash`, declared at `schema.prisma:24`, absent from the generated client).

    These are **not real**. They come from a stale `node_modules` and a stale generated client, and were disproved by running a clean install in isolation: `npm ci` (exit 0, 364 packages, ~43s — also proving the lockfile is in sync) then `npx prisma generate` then `npx tsc --noEmit` (exit 0).

    Fix your checkout with:

    ```bash
    cd apps/api
    npm ci
    npm run prisma:generate
    ```

    This is exactly why CI installs from the lockfile and regenerates the client on every run — it never inherits anyone's stale tree.

## Open items

Tracked here rather than lost in a chat log:

1. **Enable npm caching** via the commented-out block once the runner's cache server is confirmed reachable. This is now the only remaining runner-environment unknown — the `runs-on` label question is settled (`default`).
2. **Move to `actions/upload-artifact@v4`** once the Gitea server is upgraded past 1.24.7 and exposes the v4 artifact backend.
3. **Gate on coverage.** Add a threshold so the `coverage` job fails below an agreed floor, instead of only proving the suites pass. Agree the number first — a threshold set above current coverage lands as an immediately-red pipeline.
4. **Confirm where the test schema comes from.** The workflow runs no migration against the service container; if that is handled by the test harness it should be stated in the testing docs, and if it isn't, the job needs a `prisma migrate deploy` step.
5. **Remove the now-unused `cors` dependency** (`apps/api/package.json:32`). Deliberately deferred: it touches the lockfile, so it belongs in its own change.
6. **Consider stricter linting** by swapping to `recommendedTypeChecked` + `projectService`. Expect considerably more findings — worth its own pass rather than bundling into unrelated work.
7. **Add the missing stages** as they become real: build (in CI), secret scanning, `axe-core`.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5], Qoder [Qoder Lite]*
