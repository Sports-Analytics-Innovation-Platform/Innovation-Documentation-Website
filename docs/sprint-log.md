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
| Ingest two more historical seasons (2023-24, 2024-25 — ~3,780 games total), fix a trade-attribution bug that misjoined a traded player's past boxscores onto their current team, retune Elo's K-factor (20→25) and home-court advantage (75→40), and add a season-boundary rating reset, each validated by chronological train/validation splits (PR #95) | `feat` | Josh Sawyer |
| Overhaul the Predictions page — model highlights, recent-results track record, "Top 5 to watch" player cards, and a "How it works" explainer (PR #95) | `feat` | Josh Sawyer |
| Revert the saved-comparisons/lineups merge (#94) after it conflicted with the parallel home-page personalisation work still in flight (PR #98) | `fix` | Kiran Soodyall |
| Add onboarding (username, favorite team, suggested players to follow) and a Profile page (avatar upload, followed-players list, hard account deletion); new `GET/PATCH /v1/me`, `/v1/me/avatar`, `/v1/me/followed-players/:playerId` routes (PR #96, completed in #99) | `feat` | Josh Sawyer |
| Make read-only game endpoints public so the signed-out landing page can load live matches; refresh shared auth, protected-route, error, and loading states (PR #99) | `feat` | Josh Sawyer |
| Stop requiring the OAuth state cookie on cross-domain sign-in — Safari, Firefox, and Brave's stricter third-party cookie handling was silently dropping it before Google's callback redirect could read it back, causing intermittent `state_mismatch` failures (PR #100) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Enforce an 80% coverage threshold (statements/branches/functions/lines) across API and web; add the API-client tests needed to clear it (PR #101, #102) | `test` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Full landing page redesign — full-bleed responsive photos, scroll-triggered reveal animations, marquee bands, an animated "How We Predict" explainer; unify `AuthBootScreen` so the boot loader no longer plays twice on protected routes (PR #103) | `feat` | Josh Sawyer |
| Keep Google OAuth pointed at the deployed Render API when `VITE_API_BASE_URL` is unset, instead of silently falling back to a broken origin (PR #104) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Add per-opponent matchup projections (`GET /v1/players/:id/matchup-projection`, shrunk toward each player's overall rate) and a player-page overhaul with scroll animations (PR #105, shipped to `main` in #106) | `feat` | Josh Sawyer |
| Let users temporarily edit a stat on their own profile page to see the ripple effect on related numbers, then reset back to the real values (PR #107) | `feat` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Diagnose and fix Google sign-in for Safari/Firefox/Brave: the OAuth CSRF state cookie was set via a cross-origin `fetch()`, which strict third-party cookie policies silently drop before it can be read back on Google's callback; switch CI to the course-provided `sdp-runner-1` runner and move the disposable CI Postgres off the default port/hostname so it stops colliding with other groups' runs (PR #107) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Proxy `/api` and `/auth` through same-origin Cloudflare Pages Functions so the session cookie set during the OAuth callback is first-party instead of genuinely cross-site — closes the sign-in failure the state-cookie fix above got past but didn't fully resolve (PR #108) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Add saved lineups end-to-end — snapshot the optimizer board's per-slot predictions/salary and derived totals at save time, with solver-mirroring validation (exactly 5 players, position minimums, salary cap, no duplicates) (PR #111) | `feat` | Josh Sawyer |
| Remove duplicate coverage thresholds left in the web vite config (PR #110) | `fix` | Josh Sawyer |
| Move the Cloudflare Pages Functions proxy from `apps/web/functions/` to the repo root — the Pages project's configured root directory never found them there, so `/auth/get-session` was returning the SPA's HTML shell instead of a proxied response (PR #112) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Add a test for `meApi`'s suggested-players query-string branch to bring web branch coverage back over the enforced 80% threshold (PR #113) | `test` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Dedupe two conflicting `SavedLineup` type declarations left behind by the home-page merge, unblocking the `tsc -b` web build (PR #114) | `fix` | Josh Sawyer |
| Retry `prisma migrate deploy` in the e2e test suite's global setup to absorb an intermittent CI failure with no corresponding code change (PR #116) | `fix` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Trigger a Cloudflare Pages rebuild after removing a stale `VITE_API_BASE_URL` override that was silently bypassing the same-origin proxy fix and baking a direct cross-origin API URL into the build (PR #117) | `chore` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Fix lint errors on the home-page backend branch ahead of merge (PR #119) | `fix` | Kiran Soodyall |
| Restyle Compare and Teams — the last two pages still on the original dark app-shell theme — to match the locker design system (PR #118) | `style` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Add filtering, sorting, Elo ratings, records, and recent-form/follow-team controls to the Teams directory; add team profile pages and a team-records API; add player suggestions to comparison search and improve the radar visualisation (PR #123) | `feat` | Josh Sawyer |
| Cut database round trips: an in-process response cache for public reads (single-flight, never caches errors or nulls, bounded, disabled under Vitest), five redundant-query consolidations (splits 5→2, compare 8→2 for four players, stats 3→2, prediction 2→1, leaderboard counts 3→2), a 5-minute React Query `staleTime` with no refetch on window focus, a BetterAuth session cookie cache removing the per-request `Session`/`User` read, and new indexes on `Game`, `PlayerGameStat` and `Player`. Measured: most public endpoints drop to zero queries on a repeat call. `PredictionsService` deleted as dead code (PR #124) | `perf` | Kiran Soodyall |
| Fix Beat the Model and the leaderboard: the card kept showing the game just called while the next loaded, replayed a called game after navigating back, and stuck on a 409; it now drops the called game immediately, loads the next behind the graded result, always revalidates on mount, and skips an already-called game. The server breaks same-date ties on id so "next game" is stable, and two same-named users no longer swap leaderboard rows (PR #124) | `fix` | Kiran Soodyall |
| Fix onboarding follow/unfollow persistence, block finishing onboarding while a preference save is still pending, add error feedback for failed writes, and refresh profile/watchlist/favourite-team locker data after changes (PR #125, closes #67) | `fix` | Daniel Passos |
| Build an admin page (Teams/Players editing, user role management and deletion) behind the RBAC guard infrastructure (`RolesGuard`, `@Roles()`, the `ADMIN` role) that already existed in the schema but had no endpoint using it — the API's first write access to `Team`/`Player` rows (PR #120, open) | `feat` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Add model versioning to `GamePrediction` — a `modelVersion` column plus an append-only `GamePredictionRun` history table, so a prediction stays reproducible after the Elo/Four Factors model changes instead of being silently overwritten; first step of a series closing rubric gaps identified in the predictions feature (PR #126, open) | `feat` | Owen Pace (Claude Code, Claude Sonnet 5) |
| Field the Sprint 2 user feedback survey (Google Forms via WhatsApp), collect 7 responses, and publish the findings, quantitative analysis, and feedback-to-action traceability table on the docs site; raw responses archived with emails redacted | `docs` | Adrian Draxl |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude Code[Claude Sonnet 5], Claude-Code[Claude Opus 5]*
