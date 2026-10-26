# Sprint Log

What each member delivered, week by week, taken from the Gitea repo history. PR numbers refer to Gitea pull requests. How the sprints are run is in [Methodology](methodology.md).

**Team:** Owen Pace, Josh Sawyer, Adrian Draxl (Scrum Master), Kiran Soodyall, Daniel Passos, Sanele H.

---

## Sprint 1: Project setup and direction

### Week of 4 Aug

| Task | Type | Owner |
|---|---|---|
| Scaffold the base project | `feat` | Josh Sawyer |
| Move the API from Express to NestJS | `refactor` | Owen Pace |
| Replace Passport with BetterAuth (Google OAuth) | `feat` | Owen Pace |
| Teams pages, TanStack Query, shadcn/ui and the court theme | `feat` | Owen Pace |
| API and web test suites, with integration tests against real Postgres | `test` | Owen Pace |
| First CI pipeline with a coverage dashboard | `chore` | Owen Pace |
| Log the AI chat transcript | `docs` | Owen Pace |
| MkDocs site with a GitHub Pages deploy workflow | `chore` | Adrian Draxl |

### Week of 11 Aug

| Task | Type | Owner |
|---|---|---|
| Landing page and top navigation (PR #10) | `feat` | Kiran Soodyall |
| CI configuration, ESLint for API and web (PR #32), and a Docker runner | `chore` | Kiran Soodyall |
| Games endpoint, navbar and hero redesign | `feat` | Owen Pace |
| Prediction model and MILP lineup optimizer | `feat` | Owen Pace |
| Git and project methodology drafts; merge the duplicate AI ledgers | `docs` | Owen Pace |
| Fix CI: Postgres service, reach it by service name, remove GitLab leftovers | `fix` | Owen Pace |
| Core docs site pages: conventions, methodology, DoD, requirements, tech stack, security, getting started, architecture, ERD, API design, UI overview, ADR-001, ADR-002 | `docs` | Adrian Draxl |
| Organise the meeting docs into client and stand-up | `docs` | Adrian Draxl |
| Document the CI/CD pipeline | `docs` | Kiran Soodyall |
| Correct the docs to match the repo (Gitea, BetterAuth); fix the strict build | `fix` | Owen Pace |

### Week of 18 Aug

| Task | Type | Owner |
|---|---|---|
| Ingestion service: teams, rosters, games, box scores and an orchestrator (PR #34) | `feat` | Josh Sawyer |
| Game predictions (Elo win probability, Four Factors margin) and `GET /v1/games/:id/prediction` | `feat` | Josh Sawyer |
| Fix future games leaking into historical predictions | `fix` | Josh Sawyer |
| Predictions page, game detail page and court view | `feat` | Josh Sawyer |
| Team logos and player headshots from the nba.com CDN (PR #37) | `feat` | Josh Sawyer |
| Join predictions into `GET /v1/games` to remove one request per game | `fix` | Josh Sawyer |
| Coverage reporting in Gitea Actions (PR #40) | `chore` | Daniel Passos |
| Fix training-data leakage in the regression | `fix` | Daniel Passos |
| Player and team search (PR #42) | `feat` | Daniel Passos |
| Protect authenticated routes and endpoints (PR #44) | `feat` | Daniel Passos |
| One landing-page header across all routes (PR #50) | `feat` | Daniel Passos |
| Add AI transcripts to the docs site | `docs` | Daniel Passos |
| ADR-003: evaluate Azure, choose Cloudflare Pages, Render and Supabase | `docs` | Adrian Draxl |
| Deploy the API to Render and the web app to Cloudflare Pages (PR #45) | `feat` | Adrian Draxl |
| Sync BetterAuth trusted origins with CORS; fix CI timeouts | `fix` | Adrian Draxl, Owen Pace |
| Move the ADRs to the docs site; standard AI declaration on every page; Demo Guide | `docs` | Adrian Draxl |
| Player bio fields from `CommonPlayerInfo` | `feat` | Sanele H. |
| Diagnose slow-sign-in `state_mismatch` failures; self-service account deletion | `fix` | Sanele H. |
| A cron-job.org pinger to keep the Render API warm | `chore` | Sanele H. |
| Deployment, ERD, class and sequence diagrams on the docs site | `docs` | Josh Sawyer |

---

## Sprint 2: Feature expansion and documentation

### Week of 25 Aug

| Task | Type | Owner |
|---|---|---|
| Update the methodology page | `docs` | Owen Pace |
| Outstanding AI transcripts and ledger entries | `docs` | Sanele H., Josh Sawyer |
| Re-land player bio fields; run migrations on API start (PR #52) | `fix` | Sanele H. |
| Season averages, a compare endpoint and the comparison page (PR #53) | `feat` | Sanele H. |
| Minutes and free-throw attempts on profiles | `feat` | Sanele H. |

### Week of 1 Sep

| Task | Type | Owner |
|---|---|---|
| Loading animation (PR #80) | `feat` | Owen Pace |
| Edit a stat temporarily to see the effect on related figures (PRs #81, #85) | `feat` | Owen Pace |
| Move CI to the course runner `sdp-runner-1` | `fix` | Owen Pace |
| Bug tracking: a Gitea issue template and labels (PR #82) | `chore` | Sanele H. |
| Play-in, playoff and finals games, kept out of the models; season-segment views (PR #89) | `feat` | Sanele H. |
| Plus-minus, usage, ratings, TS%, eFG% and AST/TO on profile, splits and compare (PR #90) | `feat` | Sanele H. |
| Swagger UI for the API (PR #86) | `feat` | Adrian Draxl |
| Testing, stakeholder and API reference pages; draft the user survey | `docs` | Adrian Draxl |

### Week of 8 Sep

| Task | Type | Owner |
|---|---|---|
| Signed-in Home shell (PR #87) | `feat` | Kiran Soodyall |
| Home personalisation: Beat the Model, watchlist, followed teams, model accuracy, leaderboard and saved items; 7 tables and 15 routes (PR #94) | `feat` | Kiran Soodyall |
| Global origin check and Zod body validation; every `/v1/me/*` query scoped to the session user (PR #94) | `feat` | Kiran Soodyall |
| Response cache, fewer queries per endpoint, new indexes, and a session cookie cache (PR #124) | `perf` | Kiran Soodyall |
| Fix Beat the Model replaying or sticking on a called game (PR #124) | `fix` | Kiran Soodyall |
| CI Postgres fixes; landing hero photos (PR #88); lint fixes (PR #119) | `fix` | Kiran Soodyall |
| Protect the Home login flow (PR #91) | `fix` | Daniel Passos |
| axe-core accessibility checks (PR #92) | `test` | Daniel Passos |
| Fix onboarding follow state and save errors (PR #125) | `fix` | Daniel Passos |
| Two more seasons ingested; fix a traded-player attribution bug; retune Elo (PR #95) | `feat` | Josh Sawyer |
| Predictions page overhaul (PR #95) | `feat` | Josh Sawyer |
| Onboarding and the Profile page (PRs #96, #99) | `feat` | Josh Sawyer |
| Public game reads for the landing page (PR #99) | `feat` | Josh Sawyer |
| Landing page redesign (PR #103) | `feat` | Josh Sawyer |
| Per-opponent matchup projections (PRs #105, #106) | `feat` | Josh Sawyer |
| Saved lineups (PR #111) | `feat` | Josh Sawyer |
| Teams filters, Elo, records and team profiles (PR #123) | `feat` | Josh Sawyer |
| Merge postseason (#89) and advanced stats (#90) | `chore` | Sanele H. |
| Remove a committed runner token before merge | `fix` | Owen Pace |
| Fix Google sign-in on Safari, Firefox and Brave: same-origin proxy for `/api` and `/auth` (PRs #100, #104, #107, #108, #112, #117) | `fix` | Owen Pace |
| 80% coverage threshold on API and web (PRs #101, #102, #113) | `test` | Owen Pace |
| Admin page with role-based access (PR #120) | `feat` | Owen Pace |
| Prediction model versioning (PR #126) | `feat` | Owen Pace |
| Restyle Compare and Teams to the locker design (PR #118) | `style` | Owen Pace |
| Team meeting 5 | `docs` | Adrian Draxl |
| Sprint 2 user survey: 11 responses, findings and traceability | `docs` | Adrian Draxl |

---

## Sprint 3: Event sourcing, Become Pro and user feedback

### Week of 15 Sep

About 35 merged PRs, mostly the brief's Intermediate and Advanced requirements.

| Task | Type | Owner |
|---|---|---|
| Submission review, API keys, audit trail, career stats and dataset releases (PRs #136, #138, #140, #141) | `feat` | Owen Pace |
| Delete consumers and keys (PR #151); API keys required for public reads (PRs #171, #172) | `feat` | Owen Pace |
| Read aloud and screen reader support (PR #166) | `feat` | Owen Pace |
| Ingestion and download fixes (PRs #167–#170) | `fix` | Owen Pace |
| Brief audit: review now gates publication, resumable batches survive a crash (PRs #184, #186) | `fix` | Owen Pace |
| Re-derive stats after a correction; on-demand game replay (PR #139) | `feat` | Adrian Draxl |
| Flag impossible box-score rows (PR #142) | `feat` | Adrian Draxl |
| API deprecation, contract checks and version negotiation (PRs #144–#146) | `feat` | Daniel Passos |
| Dataset diffs, change feed and stale marking (PRs #147, #150, #155) | `feat` | Daniel Passos |
| Late-event reordering, aggregate and `asOf` stats, async and resumable ingestion (PRs #148, #149, #154, #156, #157) | `feat` | Daniel Passos |
| Games CSV export and live event feed (PRs #158, #161) | `feat` | Daniel Passos |
| Custom statistics with a sandboxed, versioned evaluator (PRs #159, #173) | `feat` | Daniel Passos |
| Animation polish (PR #174) | `fix` | Josh Sawyer |
| API keys moved into Profile; dataset sorting; admin batch filters (PRs #178–#180) | `feat` | Sanele H. |
| Admin event corrections with preview, validation and undo (PR #182) | `feat` | Sanele H. |
| Fix play-by-play vocabulary drift (PR #181) and teammate credit for shared surnames (PR #183) | `fix` | Sanele H. |

### Week of 22 Sep (to the Sprint 3 deadline, 29 Sep)

| Task | Type | Owner |
|---|---|---|
| Become Pro: private seasons and box scores, derived line, valuation model, NBA comparables (PR #192) | `feat` | Kiran Soodyall |
| Re-scope Become Pro to private only; fix bugs from a 65-check live browser run | `fix` | Kiran Soodyall |
| Train the valuation model (140 rookie seasons, MAE 10.9 picks) | `chore` | Kiran Soodyall |
| Past-season ingestion without overwriting rosters; career tab kept to one segment (PRs #196–#198) | `fix` | Sanele H. |
| Player Archetypes and the style map (PR #195, merged 7 Oct) | `feat` | Sanele H. |
| Database, ingestion and archetypes docs | `docs` | Sanele H. |
| Live API test transcript | `docs` | Daniel Passos |
| Survey analysis at 11 responses; first follow-up interview (F20–F31) | `docs` | Adrian Draxl |
| Load test, Lighthouse audit and the Improvements page | `docs` | Adrian Draxl |
| Full docs-site consistency audit | `docs` | Owen Pace |

---

## Submission week (to 11 Oct)

Sprint 4 closed with twenty-five pull requests, #204 to #228, merged between 8 and 10 October. The seven branches still open on 9 October all landed, so nothing was left in review at the deadline.

### Merged to `main`

| Task | Type | Owner |
|---|---|---|
| Player Archetypes merged (PR #195, 7 Oct) | `feat` | Sanele H. |
| The pull worker stops saving play-by-play (PR #200, 8 Oct) | `feat` | Sanele H. |
| Live tab: live, upcoming and recent games from the NBA's live feed (PR #201, 8 Oct) | `feat` | Sanele H. |
| Page tutorials: Home first (PR #202), then every page except the landing page (PR #203), 8 Oct | `feat` | Kiran Soodyall |
| READMEs describe the platform as built; the seed prints a local site-proxy key (PR #209, 8 Oct) | `docs` | Owen Pace |
| API keys and rate limits checked in memory, usage logged in batches (PR #204, 9 Oct) | `perf` | Owen Pace |
| OpenAPI response models, filters and key auth (PR #205, 9 Oct) | `docs` | Owen Pace |
| Every version of a custom statistic's formula kept (PR #206, 9 Oct) | `feat` | Owen Pace |
| The CI runner's registration file is no longer tracked (PR #207, 9 Oct) | `chore` | Owen Pace |
| Each page's code loaded only when its route is visited (PR #208, 9 Oct) | `perf` | Owen Pace |
| The Python services' tests run in CI (PR #210, 9 Oct) | `ci` | Owen Pace |
| Supabase's Data API closed to the `anon` and `authenticated` roles (PR #211, 9 Oct) | `fix` | Owen Pace |
| POPIA: privacy notice, data download, retention job, no stored session IPs, usernames on the public leaderboard, avatar deletion (PR #212, 9 Oct) | `feat` | Owen Pace |
| The landing scoreboard shows the biggest live game, picked by Elo (PR #213, 9 Oct) | `feat` | Sanele H. |
| A real game's play-by-play replayed against its official box score (PR #214, 9 Oct) | `test` | Josh Sawyer |
| A consumer's API usage broken down by endpoint and by day (PR #215, 10 Oct) | `feat` | Josh Sawyer |
| A stat line far outside a player's own history is flagged (PR #216, 9 Oct) | `feat` | Josh Sawyer |
| An analyst can submit a game's play-by-play by hand (PR #217, 9 Oct) | `feat` | Josh Sawyer |
| Large CSV exports run as a background job (PR #218, 9 Oct) | `feat` | Josh Sawyer |
| Stat abbreviations explained, Beat the Model and Become Pro introduced, predictions filters and headshots made readable (F7, F20, F21, F28, F29, F30) (PR #219, 10 Oct) | `feat` | Adrian Draxl |
| Public API guide and a route-by-route audit of what needs a key (F31) (PR #220, 10 Oct) | `docs` | Adrian Draxl |
| Optimizer: lock or exclude a player and solve again; three alternative lineups (F4, F6) (PR #221, 10 Oct) | `feat` | Adrian Draxl |
| Datasets: seasons that haven't started are hidden, an edited season downloads again (F25, F26) (PR #222, 10 Oct) | `fix` | Adrian Draxl |
| The landing page rewritten in plain language (F8) (PR #223, 10 Oct) | `docs` | Adrian Draxl |
| Accent-insensitive player search; the leaders band follows the list filters (F10, F22) (PR #224, 10 Oct) | `feat` | Adrian Draxl |
| Injury reports and expected return dates on team and player pages (F16) (PR #225, 10 Oct) | `feat` | Sanele H. |
| All-time career leaders, with player bios (F11) (PR #226, 10 Oct) | `feat` | Sanele H. |
| Teams: your own team above the list and an Elo explainer; Datasets explained on the page (F23, F24) (PR #227, 10 Oct) | `feat` | Sanele H. |
| Flaky count-up animation tests fixed (PR #228, 10 Oct) | `fix` | Sanele H. |

The merged work took the API from 109 to 120 live operations — injuries, all-time leaders, the optimizer solve, queued exports, the API-usage breakdown, custom-statistic versions, hand-submitted play-by-play and the account-data export — which is what [API Reference](api-reference.md) now lists.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude Code[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Opus 5.5]*
