# CI/CD Pipeline

Continuous integration runs on **Gitea Actions**, defined by a single workflow file at `.gitea/workflows/ci.yml` in the code monorepo (`sdp.ms.wits.ac.za`). It runs on every **push** and every **pull request**, and satisfies the brief's CI/CD requirement (§2.1) — see [Requirements Traceability](requirements.md).

Before this workflow existed there was no CI of any kind in the repo — no `.gitea/`, no `.github/workflows/`, no `.gitlab-ci.yml`. This page describes what now runs, why it's shaped the way it is, and what it deliberately does *not* do yet.

!!! note "Where the source of truth lives"
    The workflow file itself lives in the code repo, not in this docs repo. This page describes it; if the two ever disagree, the file wins and this page is the bug.

## What runs

Two **independent, parallel jobs** — `api` and `web` — because the monorepo root `package.json` does not use npm workspaces (it drives sub-apps via `--prefix` scripts) and each app carries its own `package-lock.json`. Separate lockfiles mean separate installs, so a shared job would be a lie about the dependency graph.

Each job runs the same three stages, in order: **lint → typecheck → test**.

| Stage | `apps/api` | `apps/web` |
|---|---|---|
| Install | `npm ci` | `npm ci` |
| Codegen | `npx prisma generate` | — |
| Lint | `npm run lint` (ESLint 10, flat config) | `npm run lint` (oxlint + `.oxlintrc.json`) |
| Typecheck | `npx tsc --noEmit` | `npx tsc -b --noEmit` |
| Test | `npm test -- --passWithNoTests` (Vitest) | `npm run test --if-present` |

Runner: `ubuntu-latest`, `actions/checkout@v4`, `actions/setup-node@v4` pinned to **Node 24** (matching the team's local Node v24.14.0 / npm 11.9.0). Each job has `timeout-minutes: 15`.

Abridged shape of the workflow — the real file is the authority:

```yaml
on: [push, pull_request]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  api:  # runs in parallel with `web`
    runs-on: ubuntu-latest
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
      - run: npx prisma generate     # prerequisite of the typecheck, made explicit
      - run: npm run lint
      - run: npx tsc --noEmit
      - run: npm test -- --passWithNoTests
```

## Why it's built this way

Each of these is a decision that changes behaviour, not a style preference.

- **Split by app, not one job.** Two lockfiles, two dependency trees, two toolchains (ESLint vs oxlint, TS 5.9 vs TS 6.0). Running them in parallel also means a web-only failure doesn't hide behind a slow API install.
- **`concurrency` with `cancel-in-progress`.** A newer push supersedes an in-flight run. This matters specifically because the Gitea runner is shared and self-hosted — stale runs queueing behind each other cost everyone else wall-clock time.
- **npm caching is deliberately omitted.** `setup-node`'s `cache: npm` routes through `@actions/cache`, whose restore errors reach `core.setFailed` — so if the `act_runner` cache server is unreachable, the step *fails* rather than degrading to an uncached install. The Wits runner's cache configuration couldn't be inspected, so the pipeline was built to work unconditionally rather than to save ~45s. The workflow carries a commented-out block to enable caching once the runner's cache server is confirmed.
- **`prisma generate` is an explicit step.** The API typecheck genuinely cannot pass without a generated Prisma client, so it's a visible step rather than an implicit reliance on npm's `postinstall` side-effect. It needs **no database and no `DATABASE_URL`** — it reads `schema.prisma` only, which is why CI needs no Postgres service container.
- **`--noEmit` for API, `tsc -b` for web.** The API's `build` script emits to `dist/`, so the check uses `--noEmit` to typecheck without producing artifacts. `apps/web/tsconfig.json` is solution-style (`files: []` plus `references`), so `tsc -b` is mandatory — a plain `tsc --noEmit` there would check *nothing* and pass silently.
- **No CD stages.** Scope was CI only, and there is no deploy target: production hosting is still undecided (see [ADR-003](decisions/adr-003-hosting-topology.md)). Deployment stages get added when a host exists, not before.

## Handling the two pre-existing gaps

The repo had two gaps at the time CI was introduced, both handled inside the workflow without changing anything else:

**1. `apps/api` had no ESLint config.** `npm run lint` failed outright with *"ESLint couldn't find an eslint.config.(js|mjs|cjs) file"*. Rather than blanket `continue-on-error` (which masks genuine failures forever) or a hard fail (a red pipeline on arrival for a pre-existing gap), the lint step **probes for `eslint.config.*`**: if absent it emits a `::warning::` and passes; if present it runs the real lint and lets a non-zero exit propagate.

!!! success "The lint guard is now enforcing, not warning"
    `apps/api/eslint.config.js` has since been added (see below), so the guard takes its live branch. **A lint error now fails the build.** The warning branch remains only as a safety net.

**2. No test files existed anywhere in either `src` tree.** The API's `vitest run` exited 1 with *"No test files found"*. The API step passes `--passWithNoTests`; the web step uses `npm run test --if-present`, which self-activates the moment a `test` script is added to `apps/web/package.json`.

`--passWithNoTests` tolerates **zero** tests, not *failing* tests — this was verified by injecting a deliberately failing probe test, confirming exit 1, then removing it. It is a temporary accommodation: see [Open items](#open-items).

## The API ESLint config

Adding CI surfaced that `apps/api` was missing its lint config entirely. Fixing it needed **no new packages** — `eslint@10.8.0`, `@eslint/js@10.0.1`, and `typescript-eslint@8.66.0` were already devDependencies; only the config file was absent. (typescript-eslint 8.66 was confirmed to run under ESLint 10, which was the main compatibility risk.)

`apps/api/eslint.config.js` is a **flat config in ESM** (the package is `"type": "module"`), composing `@eslint/js` + `typescript-eslint` recommended. Two rule overrides are functional rather than stylistic, and both are documented here so nobody "tidies them up" later:

| Rule | Setting | Why |
|---|---|---|
| `no-undef` | `off` for TypeScript files | On TS it only produces false positives — Node globals (`process`, `console`) and type-only names. TypeScript's own checker covers undefined identifiers properly. |
| `@typescript-eslint/no-empty-object-type` | `off`, scoped to `**/*.d.ts` | Without it, `express.d.ts:5` is flagged for `interface User extends PrismaUser {}`. That empty interface is *correct* — it's the canonical Express declaration-merging idiom that types `req.user`. Scoping the exception to `.d.ts` keeps the rule active everywhere it's meaningful. |

Across the whole API source that left exactly **one** genuine error: an unused `cors` import in `apps/api/src/main.ts`, since deleted. It was verified dead first — `cors` appears nowhere else in `src/`, and CORS is actually configured through Nest's built-in `app.enableCors({...})` at `main.ts:19`.

## Current status

| Stage | API before CI work | API now | Web |
|---|---|---|---|
| Lint | ❌ no config | ✅ exit 0 | ✅ exit 0 |
| Typecheck | ✅ (on a clean install) | ✅ exit 0 | ✅ exit 0 |
| Test | ❌ exit 1 (no test files) | ✅ exit 0 (no tests yet) | ✅ exit 0 (no test script yet) |

Green here means *"the pipeline runs and the code is clean"* — it does **not** yet mean *"the code is tested."* No test files exist in either app. Treat a green test stage as a placeholder until real suites land.

## What CI does not do yet

Other pages on this site describe CI steps that are planned but **not in `ci.yml` today**. Stated plainly so nobody assumes coverage that doesn't exist:

- **No build step.** Lint, typecheck, and test only.
- **No secret scanning.** [Git Methodology](git-methodology.md) and [Security](security.md) describe `gitleaks`/`trufflehog` as a PR backstop — that's the intent, not yet the implementation. The manual pre-commit check is currently the only line of defence.
- **No `axe-core` accessibility checks.** [Requirements Traceability](requirements.md) lists these for `apps/web`; they aren't wired up.
- **No deployment.** CI only — see the "No CD stages" note above.

## Local parity

CI runs the same commands you can run locally, with one wrinkle worth knowing.

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

1. **Verify the `ubuntu-latest` runner label** against the Gitea runner's actual registration. This is the single most likely line to need changing on first run — self-hosted `act_runner` instances often register different labels.
2. **Enable npm caching** via the commented-out block once the runner's cache server is confirmed reachable.
3. **Drop `--passWithNoTests`** as soon as real API tests land, so an accidentally-empty run fails instead of quietly passing.
4. **Remove the now-unused `cors` dependency** (`apps/api/package.json:32`). Deliberately deferred: it touches the lockfile, so it belongs in its own change.
5. **Consider stricter linting** by swapping to `recommendedTypeChecked` + `projectService`. Expect considerably more findings — worth its own pass rather than bundling into unrelated work.
6. **Add the missing stages** as they become real: build, secret scanning, `axe-core`, and eventually CD once hosting is decided.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5]*
