# Sprint Log


## Team

| Name ||
|---|---|
| Owen Pace 
| Josh Sawyer 
| Adrian Draxl | Scrum Master|
| Kiran Soodyall
| Daniel Passos 
| Sanele H. 

---

## Sprint 1 — Project Setup & Direction

### Week of 4 Aug

| Task | Type | Owner |
|---|---|---|
| Scaffold the NBA analytics platform base project | `feat` | Josh Sawyer |
| Migrate the API backend from Express to NestJS | `refactor` | Owen Pace |
| Replace Passport auth with BetterAuth (Google OAuth) | `feat` | Owen Pace |
| Build Teams pages, TanStack Query, shadcn/ui, court theme | `feat` | Owen Pace |
| Write backend + frontend test suites with real-Postgres integration tests | `test` | Owen Pace |
| Stand up GitLab CI pipeline with a coverage dashboard | `chore` | Owen Pace |
| Log AI chat transcript for course attribution requirement | `docs` | Owen Pace |
| Stand up MkDocs documentation site with GitHub Pages deploy workflow | `chore` | Adrian Draxl |

### Week of 11 Aug

| Task | Type | Owner |
|---|---|---|
| Implement landing page with hero section and navigation | `feat` | Kiran Soodyall |
| Draft git methodology, project methodology, and project overview docs | `docs` | Owen Pace |
| Add games endpoint, top navbar, and hero redesign | `feat` | Owen Pace |
| Build prediction-model + MILP lineup optimizer | `feat` | Owen Pace |
| Move docs from the marketing site into the repo; normalize AI usage ledger | `docs` | Adrian Draxl |
| Reconcile frontend branch — merge hero/navbar with a11y patterns, clean up optimizer | `refactor` | Owen Pace |
| Merge duplicate AI usage ledgers | `chore` | Owen Pace |
| Ship public landing page + top navigation (PR #10) | `feat` | Kiran Soodyall |
| Add CI configuration and ESLint setup for API and Web (PR #32) | `feat` | Kiran Soodyall |
| Add Docker runner setup for CI | `chore` | Kiran Soodyall |
| Match navbar/hero layout to reference mockup | `style` | Owen Pace |
| Remove dead GitLab CI leftovers, add apps/api ESLint config | `chore` | Owen Pace |
| Fix stale "what's not done yet" list in README; add Postgres service to CI | `fix` | Owen Pace |
| Fix API tests to reach Postgres by service name, not localhost | `fix` | Owen Pace |
| Organize meeting docs (client vs. standup), clean up meeting page TOC | `docs` | Adrian Draxl |
| Write core docs site content — coding conventions, git/project methodology, definition of done, requirements traceability, tech stack, security, getting started, architecture overview, ERD, API design, UI overview, ADR-001 (database), ADR-002 (auth) | `docs` | Adrian Draxl |
| Fix broken internal doc links breaking the strict MkDocs build | `fix` | Owen Pace |
| Update CI/CD documentation with pipeline details | `docs` | Kiran Soodyall |
| Correct docs site to match the real repo (Gitea not GitLab, BetterAuth not Passport); fix stale CI claims and an orphaned meetings page | `fix` | Owen Pace |

### Week of 18 Aug

| Task | Type | Owner |
|---|---|---|
| Build ingestion service scaffold with team fetching | `feat` | Josh Sawyer |
| Add roster, game, and boxscore ingestion | `feat` | Josh Sawyer |
| Add ingestion orchestrator + README (PR #34) | `feat` | Josh Sawyer |
| Add GamePrediction model + predictor service (Elo win probability, Four Factors margin) | `feat` | Josh Sawyer |
| Add GET /v1/games/:id/prediction endpoint + e2e tests | `feat` | Josh Sawyer |
| Fix future-game data leaking into historical predictions | `fix` | Josh Sawyer |
| Add Predictions page and game detail page (win probability, predicted top scorers) | `feat` | Josh Sawyer |
| Add court view visualizing predicted top scorers by position | `feat` | Josh Sawyer |
| Show real team logos + player headshots from nba.com's CDN (PR #37) | `feat` | Josh Sawyer |
| Join predictions into GET /v1/games — kill the one-request-per-game pattern | `fix` | Josh Sawyer |
| Add coverage reporting to Gitea Actions (PR #40) | `chore` | Daniel Passos |
| Fix regression training-data leakage | `fix` | Daniel Passos |
| Add player and team search (PR #42) | `feat` | Daniel Passos |
| Protect authenticated routes and API endpoints (PR #44) | `feat` | Daniel Passos |
| Write ADR-003 (hosting topology); evaluate Azure, pivot to Cloudflare Pages + Render + Supabase | `docs` | Adrian Draxl |
| Deploy API to Render, web to Cloudflare Pages; fix Render Blueprint, build, and tsconfig issues (PR #45) | `feat` | Adrian Draxl |
| Sync BetterAuth trustedOrigins with CORS allowed origins; fix CI test timeouts; restore VITE_API_BASE_URL | `fix` | Adrian Draxl |
| Harden npm ci against CI runner network stalls; land CORS trusted-origins fix (PR #46) | `fix` | Owen Pace |
| Pull player bio fields from the CommonPlayerInfo endpoint; fix seed.ts player IDs and frontend mocks | `feat` | Sanele H. |
| Migrate ADR docs from project repo to doc site; resolve MkDocs strict-mode build failures | `fix` | Adrian Draxl |
| Add AI/Codex usage transcripts to docs site (PR #1, PR #2) | `docs` | Daniel Passos |
| Diagnose Google OAuth `statemismatch` failures on slow sign-ins; add self-service account deletion | `fix` | Sanele H. |
| Unify the redesigned landing-page header across all app routes (PR #50) | `feat` | Daniel Passos |
| Publish deployment, ERD, class, and sequence diagrams (PlantUML → SVG) to the docs site with click-to-zoom | `docs` | Josh Sawyer |
| Standardise AI declaration format across all docs pages; add live links, demo guide, and Sprint 1 rubric update | `docs` | Adrian Draxl |
| Link documentation site from README; fix stale scaffold status | `docs` | Owen Pace |
| Correct stale "not yet built" claims in getting-started.md | `docs` | Adrian Draxl |
| Set up a cron-job.org pinger to keep the free-tier Render API instance warm | `chore` | Sanele H. |

---

## Sprint 2 — Feature Expansion & Documentation

### Week of 25 Aug

| Task | Type | Owner |
|---|---|---|
| Update methodology.md to reflect current process | `docs` | Owen Pace |
| Add outstanding AI transcripts and ledger entries | `docs` | Sanele H., Josh Sawyer |
| Re-land player bio fields after an earlier revert; run Prisma migrations automatically on API start (PR #52) | `fix` | Sanele H. |
| Add per-game season averages and a player-compare endpoint; build the player comparison page (PR #53) | `feat` | Sanele H. |
| Show minutes and free-throw attempts on player profiles, with a link into the comparison page | `feat` | Sanele H. |
| Document the player comparisons feature and its data audit | `docs` | Sanele H. |

### Week of 1 Sep

| Task | Type | Owner |
|---|---|---|
| Add bouncing basketball loading animation (PR #80) | `feat` | Owen Pace |
| Let users temporarily edit a stat and see the effect on related numbers, then reset (PR #81, #85) | `feat` | Owen Pace |
| Set up bug tracking with a custom Gitea issue template and label scheme (PR #82) | `chore` | Sanele H. |
| Fix player age not appearing on all player comparisons | `fix` | Sanele H. |
| Switch CI to the course-provided `sdp-runner-1`; target the `ubuntu-latest` label the Wits runners actually register; move the disposable CI Postgres off the default port/hostname | `fix` | Owen Pace |
| Add season type and playoff round to games; ingest play-in, playoff, and finals games; keep postseason games out of prediction/optimizer models | `feat` | Sanele H. |
| Filter player and game endpoints by season type; add season-segment views to the web app; seed a mock postseason for local dev and tests (→ PR #89) | `feat` | Sanele H. |
| Store plus-minus, usage, and player ratings per game; derive true shooting%, eFG%, and assist-to-turnover; surface advanced stats on profile, splits, and comparison pages (→ PR #90) | `feat` | Sanele H. |
| Add Swagger/OpenAPI documentation to the API (→ PR #86) | `feat` | Adrian Draxl |
| Add testing, stakeholder interactions, and API reference pages; audit tech stack; update rubric quick links; draft the user feedback survey; sync docs site with actual codebase state | `docs` | Adrian Draxl |

### Week of 8 Sep

| Task | Type | Owner |
|---|---|---|
| Build the signed-in home dashboard shell ("The Locker") with placeholder data — watchlist, followed teams, jump-back-in rail (PR #87); wired to real endpoints later in the sprint by PR #94 | `feat` | Kiran Soodyall |
| Fix CI Postgres container startup, reachability, and concurrency handling | `fix` | Kiran Soodyall |
| Protect the personalised home login flow (PR #91) | `fix` | Daniel Passos |
| Add axe-core automated accessibility checks (PR #92) | `test` | Daniel Passos |
| Hold team meeting 5: agreed on a profile page (account settings, password reset, data deletion), a "coach mode" self-upload stats feature, deferring Row Level Security to sprint-end, and scheduled final Sprint 2 review for 2026-09-14 | `docs` | Adrian Draxl |
| Merge postseason-view (#89) and advanced-player-stats (#90) into main, resolving the stacked-branch history between them | `chore` | Sanele H. |
| Resolve a real merge conflict in `players.controller.ts` between the Swagger decorators and the season-type work, then merge swagger-openapi (#86) into main | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Update landing-page hero photos and add a real home-page screenshot to the hero cascade (PR #88) | `feat` | Kiran Soodyall |
| Found and removed a live Gitea runner registration token (`ci-runner/data/.runner`, accidentally committed on PR #88) before merging; token rotation on the server is still pending | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Pre-warm the API with a `/health` ping on mount to reduce Render cold-start failures during Google sign-in — the OAuth state row's 10-minute expiry is hardcoded in `better-auth` with no config option (confirmed up to the latest 1.7.4) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Build the user-owned personalisation layer behind the signed-in home page: one migration (`20260910134345_home_personalization`) adding seven tables and a `PickOutcome` enum with zero ALTERs on any NBA-data table, 15 new `/v1/analytics/*` and `/v1/me/*` routes, and the Beat the Model, watchlist, followed-team results, model-accuracy ledger, leaderboard and saved-shelf features (PR #94) | `feat` | Kiran Soodyall |
| Add `OriginCheckGuard` as a global `APP_GUARD` and shared Zod body validation (`parseBody`) alongside the API's first write routes; scope every `/v1/me/*` query to the session user id so one user can never read or delete another's rows (PR #94) | `feat` | Kiran Soodyall |
| Remove the "Add to Locker" and "Jump Back In" home sections — both were layout with nothing behind them — and add an "Add to watchlist" control to the player profile page as the new entry point into the watchlist (PR #94) | `refactor` | Kiran Soodyall |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude Code[Claude Sonnet 5], Claude-Code[Claude Opus 5]*
