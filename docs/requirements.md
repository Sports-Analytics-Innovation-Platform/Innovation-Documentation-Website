# Requirements Traceability

This table maps every key requirement from the COMS3011A brief (§2.1) to the component that satisfies it, the owner responsible, and the tracking issue. It's reviewed at each milestone against the deployed environment — see [Definition of Done](definition-of-done.md).

| Brief requirement | How we satisfy it | Component | Owner | Issue |
|---|---|---|---|---|
| **Version control** | Git-compliant VCS, hosted on Gitea per the university requirement. Repo mirrored to GitHub for CI/CD auto-deploy | Whole repo | Josh Sawyer | Gitea org |
| **Responsiveness & accessibility** | Responsive layouts (mobile/tablet/desktop breakpoints), keyboard navigation with skip-to-content link, `axe-core` checks planned for Sprint 2 | `apps/web` | Owen Pace | — |
| **CI/CD** | Gitea Actions workflow (`.gitea/workflows/ci.yml`) — lint, typecheck, test as parallel `api`/`web`/`coverage` jobs on every push and PR. CD is live: frontend auto-deploys to Cloudflare Pages, API auto-deploys to Render via GitHub mirror. Full detail: [CI/CD Pipeline](ci-cd.md) | Whole repo | Kiran Soodyall | — |
| **Non-monolithic front-end and back-end** | Separate `apps/api` (NestJS) and `apps/web` (React + Vite) apps, communicating only over HTTP, independently deployed (Cloudflare Pages + Render) | `apps/api`, `apps/web` | Owen Pace | — |
| **Hand-written API** | All endpoints implemented directly in NestJS controllers/services — no auto-generated CRUD layer (e.g. no Supabase/Firebase-style generation) | `apps/api` | Owen Pace | — |
| **Authentication & security** | Sign up, sign in, account deletion via BetterAuth (Google OAuth) — an established auth library rather than hand-rolled auth. Authenticated route protection on predictions, games, and optimizer endpoints. ⚠️ Password reset is unresolved for OAuth-only sign-in, see [ADR-002](decisions/adr-002-auth.md) | `apps/api` (auth module) | Owen Pace | — |
| **Integration with an external API** | `nba_api` (Python client for stats.nba.com) as the core data source; ingestion service (`apps/ingestion`) fetches teams, rosters, games, and box scores into Postgres. Statistics derived from individual event data rather than precomputed league totals, per the brief | `apps/ingestion`, `apps/api` | Josh Sawyer | — |
| **Documentation website** | This MkDocs Material site, deployed via GitHub Pages static hosting on every push to `main` | Docs site | Adrian Draxl | — |

## Notes

- The **hand-written API requirement** is a specific risk area: `nba_api` itself is fine to depend on for *data*, but no part of our own API surface may be auto-generated from the database schema — every route in `apps/api` must be explicitly written.
- This table should be walked row-by-row with the client tutor at each milestone, per the brief's Milestone 4 expectation ("verify every brief requirement... each row has evidence linked").

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
