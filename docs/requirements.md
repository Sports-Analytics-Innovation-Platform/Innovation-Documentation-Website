# Requirements Traceability

Each technical requirement of the COMS3011A brief (§2.1), the component that meets it, and its owner. How the brief's feature tiers are met is on [Feature Tiers](design/feature-tiers.md).

| Brief requirement | How we satisfy it | Component | Owner |
|---|---|---|---|
| **Version control** | Git-compliant VCS, hosted on Gitea per the university requirement. Repo mirrored to GitHub for CI/CD auto-deploy | Whole repo | Josh Sawyer |
| **Responsiveness & accessibility** | Responsive layouts (mobile/tablet/desktop breakpoints), keyboard navigation with skip-to-content link, `axe-core` automated accessibility checks running in CI (PR #92) | `apps/web` | Owen Pace |
| **CI/CD** | Gitea Actions workflow (`.gitea/workflows/ci.yml`) — lint, typecheck, test as parallel `api`/`web`/`coverage` jobs on every push and PR. CD is live: frontend auto-deploys to Cloudflare Pages, API auto-deploys to Render via GitHub mirror. Full detail: [CI/CD Pipeline](ci-cd.md) | Whole repo | Kiran Soodyall |
| **Non-monolithic front-end and back-end** | Separate `apps/api` (NestJS) and `apps/web` (React + Vite) apps, communicating only over HTTP, independently deployed (Cloudflare Pages + Render) | `apps/api`, `apps/web` | Owen Pace |
| **Hand-written API** | All endpoints implemented directly in NestJS controllers/services — no auto-generated CRUD layer (e.g. no Supabase/Firebase-style generation) | `apps/api` | Owen Pace |
| **Authentication & security** | Sign-up, sign-in and account deletion through BetterAuth with Google, an established library rather than hand-written auth. Personal routes need a session; public data needs a session or an API key. Password reset is left to Google ([ADR-002](decisions/adr-002-auth.md), [Security](security.md)) | `apps/api` (auth module) | Owen Pace |
| **Integration with an external API** | `nba_api` (Python client for stats.nba.com) as the core data source; ingestion service (`apps/ingestion`) fetches teams, rosters, games, and box scores into Postgres. Statistics derived from individual event data rather than precomputed league totals, per the brief | `apps/ingestion`, `apps/api` | Josh Sawyer |
| **Documentation website** | This MkDocs Material site, deployed via GitHub Pages static hosting on every push to `main` | Docs site | Adrian Draxl |

`nba_api` is only a data source. No part of the platform's own API is generated from the database schema: every route in `apps/api` is written by hand.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
