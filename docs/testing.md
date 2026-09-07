# Testing

This page describes what testing exists across the project, how it runs, the tooling behind it, and the team's test policy — what must be tested, at what level, and before what it must pass. It covers automated testing (unit, integration, end-to-end, and component), the CI test pipeline, coverage reporting, and the user feedback process.

## Testing strategy

The project uses a **three-layer** testing approach, each layer targeting a different failure mode:

| Layer | What it catches | Where it lives | Tooling |
|---|---|---|---|
| **Unit tests** | Logic bugs in individual services, utilities, and pure functions | `src/**/*.spec.ts` alongside source files | Vitest |
| **Integration / E2E tests** | Broken API routes, database query errors, auth guard failures, incorrect HTTP status codes | `apps/api/test/**/*.e2e-spec.ts` | Vitest + Supertest + real Postgres |
| **Component tests** | UI rendering bugs, broken user interactions, missing accessibility attributes | `src/**/*.spec.{ts,tsx}` alongside components | Vitest + React Testing Library + jsdom |

The principle: **unit tests for logic, integration tests for routes, component tests for UI** — with no layer doing another's job. A unit test never boots the database; an E2E test never imports a React component.

## API tests

### Unit tests

Unit specs live next to the source files they exercise (`src/**/*.spec.ts`) and test individual services and utilities in isolation. They do **not** start a Nest application, connect to a database, or make HTTP requests.

| Spec file | What it tests |
|---|---|
| `src/common/all-exceptions.filter.spec.ts` | The global exception filter maps errors to correct HTTP status codes |
| `src/common/pagination.spec.ts` | Pagination helper produces correct offsets and page counts |
| `src/common/roles.guard.spec.ts` | Role-based guard allows/denies access correctly |
| `src/players/stats.service.spec.ts` | Player stats service computes derived statistics |
| `src/predictions/predictions.service.spec.ts` | Predictions service logic |

### End-to-end tests

E2E specs live in `apps/api/test/` and exercise the **full NestJS application** — real routing, real middleware, real exception filter, real Prisma queries against a disposable Postgres database. They use [Supertest](https://github.com/ladjs/supertest) to make HTTP requests against the running test app.

| Spec file | What it tests |
|---|---|
| `test/health.e2e-spec.ts` | `GET /health` returns 200 with `{ status: "ok" }` |
| `test/players.e2e-spec.ts` | Player CRUD endpoints — list, detail, filtering, pagination |
| `test/teams.e2e-spec.ts` | Team endpoints — list, detail, roster lookup |
| `test/games.e2e-spec.ts` | Game endpoints — list, detail, events, box scores |
| `test/optimizer.e2e-spec.ts` | Optimizer endpoint — lineup generation, validation |
| `test/not-found.e2e-spec.ts` | Unknown routes return 404 via the `NotFoundController` catch-all |

### Test infrastructure

**`create-test-app.ts`** — Boots the real `AppModule` on an Express 5 adapter (matching the production `main.ts` setup exactly, including the `ExpressAdapter` to avoid Nest's bundled Express 4 fallback). Applies the global exception filter. Returns the `INestApplication` for Supertest to drive.

```typescript
// Simplified — see apps/api/test/create-test-app.ts for the full version
export async function createTestApp(): Promise<INestApplication> {
  const server = expressFactory();
  server.use(expressFactory.json());
  const app = await NestFactory.create(AppModule, new ExpressAdapter(server), { logger: false });
  app.useGlobalFilters(new AllExceptionFilter());
  await app.init();
  return app;
}
```

**`test-db.ts`** — A shared `PrismaClient` instance and a `resetDatabase()` helper that truncates all tables in foreign-key-safe order. Called from `afterEach` in every E2E spec so tests never depend on leftover state.

**`global-setup.ts`** — Runs `npx prisma migrate deploy` once before the entire test suite, applying all migrations to the disposable test database. Throws if `DATABASE_URL` is not set, forcing the developer to configure `.env.test` first.

### Database isolation

E2E tests run **serially** (`fileParallelism: false` in `vitest.config.ts`) against a single shared Postgres database. This is deliberate: parallel spec files would race each other's inserts and truncations, causing flaky unique-constraint failures. The 20-second `testTimeout` and `hookTimeout` accommodate the database round-trips.

The test database is **disposable** — it is created fresh in CI (a `postgres:16-alpine` service container) and pointed at locally via `.env.test` (copied from `.env.test.example`). No test data persists between runs.

## Frontend tests

### Component tests

Frontend specs live next to the components and pages they test (`src/**/*.spec.{ts,tsx}`) and use **React Testing Library** with the **jsdom** environment. The philosophy is to test components the way a user would interact with them — by role, label, and text content — rather than by implementation details like state or instance methods.

| Spec file | What it tests |
|---|---|
| `src/App.spec.tsx` | Root app renders, routing mounts correctly |
| `src/components/AuthStatus.spec.tsx` | Auth status displays login/logout state |
| `src/components/CourtView.spec.tsx` | Court visualization renders player positions |
| `src/components/ErrorState.spec.tsx` | Error state component displays message and retry |
| `src/components/Pagination.spec.tsx` | Pagination controls emit correct page changes |
| `src/components/PlayerHeadshot.spec.tsx` | Player headshot image loads with fallback |
| `src/components/PlayersFilterBar.spec.tsx` | Filter bar captures search and filter input |
| `src/components/ProtectedRoute.spec.tsx` | Protected route redirects unauthenticated users |
| `src/components/RecentResultWidget.spec.tsx` | Recent result widget displays latest game outcome |
| `src/components/StatTile.spec.tsx` | Stat tile renders value and label |
| `src/components/TeamBadge.spec.tsx` | Team badge renders team abbreviation/logo |
| `src/components/landing/LandingMatchWidget.spec.tsx` | Landing page match widget renders |
| `src/components/landing/Marquee.spec.tsx` | Marquee scrolling component |
| `src/lib/nbaApi.spec.ts` | API client functions construct correct URLs |
| `src/lib/utils.spec.ts` | Utility functions (cn class merger, etc.) |
| `src/pages/GameDetailPage.spec.tsx` | Game detail page loads and displays game data |
| `src/pages/LandingPage.spec.tsx` | Landing page renders hero, recent games, features |
| `src/pages/OptimizerPage.spec.tsx` | Optimizer page form submission and result display |
| `src/pages/PlayersListPage.spec.tsx` | Players list renders, filters, and paginates |
| `src/pages/PredictionsPage.spec.tsx` | Predictions page displays forecast data |
| `src/pages/TeamsListPage.spec.tsx` | Teams list renders team cards |
| `src/pages/ComparePage.spec.tsx` | Player comparison page renders and compares multiple players |

### Test setup

The web test environment is configured in `vite.config.ts`:

- **Environment:** `jsdom` — provides a DOM without a real browser
- **Setup file:** `src/test/setup.ts` — imports `@testing-library/jest-dom` for matchers like `toBeInTheDocument()`, `toHaveTextContent()`
- **Globals:** enabled — `describe`, `it`, `expect` available without imports
- **Timeout:** 30 seconds (higher than API tests, since jsdom rendering can be slower)

## Running tests

### Locally

**API tests** require a disposable Postgres database. The fastest way:

```bash
# Start a throwaway Postgres
docker run --rm -d --name nba-test-db -p 55433:5432 \
  -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=nba_analytics_test postgres:16-alpine

# Configure the test environment
cd apps/api
cp .env.test.example .env.test
# .env.test points at localhost:55433 (see the mapped port above)

# Run tests
npm run test          # run once, exit
npm run test:watch    # watch mode
npm run test:cov      # run with coverage report
```

**Web tests** have no external dependencies:

```bash
cd apps/web
npm run test          # run once, exit
npm run test:watch    # watch mode
npm run test:cov      # run with coverage report
```

### In CI

All tests run automatically on every push and pull request via the `coverage` job in the Gitea Actions pipeline. See [CI/CD Pipeline](ci-cd.md) for the full breakdown. Summary:

| Step | What happens |
|---|---|
| Service container | `postgres:16-alpine` starts with a healthcheck (`pg_isready`) |
| Environment | `DATABASE_URL` points at the service container's hostname (`postgres:5432`, not `localhost`) |
| Schema | `prisma migrate deploy` runs via the API's `global-setup.ts` before any test |
| API tests | `npm run test:cov --prefix apps/api` — all unit + E2E specs |
| Web tests | `npm run test:cov --prefix apps/web` — all component specs |
| Coverage merge | `npm run coverage:report` (root script) combines both into a single HTML report |
| Artifact | The merged `coverage-report/` directory is uploaded as a CI artifact |

!!! warning "Tests are enforced; coverage is not (yet)"
    CI **fails** if any test fails — there is no `--passWithNoTests` tolerance. However, CI does **not** fail on low coverage numbers. The coverage report is generated and uploaded, but no threshold gate exists yet. See [CI/CD Pipeline — Open items](ci-cd.md#what-ci-does-not-do-yet).

## Coverage

Both apps use **Vitest's v8 coverage provider**, which instruments code at the V8 level (more accurate than Babel-based instrumentation). Reports are generated in multiple formats:

| Format | Purpose |
|---|---|
| `text` | Summary table printed to the CI log |
| `html` | Browsable report for local development |
| `json` / `json-summary` | Machine-readable for merging and tooling |
| `cobertura` | XML format for CI integrations |

### What is excluded from coverage

| App | Excluded | Why |
|---|---|---|
| API | `src/main.ts` | Bootstrap wiring — no logic to test |
| API | `src/**/*.module.ts` | Nest module declarations — structural, not behavioural |
| API | `src/**/*.spec.ts` | Test files themselves |
| API | `src/auth/**` | BetterAuth manages this; no custom logic to cover |
| API | `src/prisma/**` | Generated Prisma client |
| Web | `src/main.tsx` | React DOM bootstrap |
| Web | `src/vite-env.d.ts` | Type declarations only |
| Web | `src/components/ui/**` | shadcn/ui primitives — third-party code |
| Web | `src/**/*.{test,spec}.{ts,tsx}` | Test files themselves |
| Web | `src/test/**` | Test setup and helpers |

## Test policy

This is what the team agrees to, enforced via the [Definition of Done](definition-of-done.md) and PR review:

### What must be tested

| Change type | Minimum test requirement |
|---|---|
| **New API endpoint** | At least one E2E spec covering the happy path (correct status code, response shape) and one error case (400, 404, or 403) |
| **New service method** | Unit test covering the core logic; integration test if it touches the database |
| **Bug fix** | A regression test that fails without the fix and passes with it |
| **New UI component** | Component test verifying it renders with expected props and key user interactions work |
| **Auth / role change** | E2E test confirming protected routes reject unauthenticated access |
| **Configuration change** | No test required, but must be verified manually on staging |

### Test conventions

- **File naming:** `*.spec.ts` for unit and component tests; `*.e2e-spec.ts` for integration tests
- **Location:** Unit/component specs live next to source files; E2E specs live in `apps/api/test/`
- **Isolation:** Each E2E spec calls `resetDatabase()` in `afterEach` — never depend on state from another test
- **No test-only shortcuts:** E2E tests boot the real application via `createTestApp()`, not a stripped-down mock — what CI tests is what production runs
- **Deterministic:** Tests must pass consistently. Flaky tests are treated as bugs and fixed or quarantined immediately
- **Fast feedback:** Unit tests should run in seconds; E2E tests under the 20-second timeout

### What is not required (yet)

- **Coverage threshold** — CI produces coverage numbers but does not gate on them. An agreed floor (e.g. 70% line coverage) should be set before Sprint 2 ends, once current coverage is measured.
- **Visual regression tests** — not planned for Sprint 2
- **axe-core accessibility scans in CI** — agreed for a later sprint; accessibility is checked manually per the Definition of Done
- **Performance / load tests** — not in scope for the current milestone

## User feedback process

Beyond automated testing, the project collects **formal user feedback** to validate that the product meets stakeholder needs. This satisfies the rubric's "extensive user testing with formal feedback collection" criterion.

### Feedback collection methods

| Method | Tool | Status |
|---|---|---|
| **Structured survey** | Google Forms distributed via WhatsApp | Drafted — see [User Feedback Survey](feedback-survey.md) |
| **Client meeting notes** | Meeting minutes with action items | Ongoing — see [Meetings](meetings/index.md) |

### How feedback is integrated

1. Feedback is collected via the survey during the testing window
2. Responses are reviewed by the team and categorised (bug, feature request, UX improvement, data accuracy issue)
3. Actionable items are converted into Gitea issues and prioritised in the sprint backlog
4. Changes made in response to feedback are documented in the [Sprint Log](sprint-log.md) with a link back to the originating feedback
5. The full feedback methodology, questions asked, distribution plan, findings, and integration actions are documented on the [User Feedback Survey](feedback-survey.md) page

!!! note "Feedback methodology page"
    The dedicated feedback methodology page is created once the Google Forms survey has been distributed and responses collected. It will include the survey instrument, sharing method, raw findings, and a traceability table showing which feedback items led to which changes.

## Bug tracking

Bugs discovered through testing (automated or manual) are tracked in the **Gitea issue tracker** at [sdp.ms.wits.ac.za/innovation/sportsanalytics/issues](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/issues). Each bug is:

- Labelled with its severity (critical / major / minor / cosmetic)
- Linked to the PR that fixes it (via commit message or PR description)
- Moved through the project board columns (Open → In Progress → Review → Done)

The bug tracker is also used to track feature requests and user feedback items that result in code changes, ensuring full traceability from feedback to fix.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite]*
