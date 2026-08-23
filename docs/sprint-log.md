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

---
