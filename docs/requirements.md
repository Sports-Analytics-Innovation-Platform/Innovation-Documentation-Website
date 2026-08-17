# Requirements Traceability

This table maps every key requirement from the COMS3011A brief (§2.1) to the component that satisfies it, the owner responsible, and the tracking issue. It's reviewed at each milestone against the deployed environment — see [Definition of Done](definition-of-done.md).

| Brief requirement | How we satisfy it | Component | Owner | Issue |
|---|---|---|---|---|
| **Version control** | Git-compliant VCS, hosted on Gitea per the university requirement | Whole repo | Josh | TBD |
| **Responsiveness & accessibility** | Some responsive breakpoints in use (`apps/web/src` has 6 files using Tailwind's `sm:`/`md:`/`lg:` prefixes) and some keyboard/focus handling (skip-link, focus-visible styling in `App.tsx`/`Navbar.tsx`/`RecentResultWidget.tsx`). **`axe-core` is not wired in anywhere** — it's an aspirational item in the README's "not done yet" list, not a real check. A real accessibility pass (automated + manual) is still outstanding | `apps/web` | TBD | TBD |
| **CI/CD** | Gitea Actions workflow (`.gitea/workflows/ci.yml`) via a self-hosted Docker runner — runs lint + typecheck + test for both apps on every push/PR, including a real Postgres service container for the API's e2e suite (added 2026-08-17 after the first real run of it exposed the API job failing with no database to connect to). Coverage merging/dashboard (`scripts/build-coverage-report.mjs`) is built and runnable manually (`npm run coverage:report`) but not wired into CI | Whole repo | Kiran | TBD |
| **Non-monolithic front-end and back-end** | Separate `apps/api` (NestJS) and `apps/web` (React + Vite) apps, communicating only over HTTP, independently deployable | `apps/api`, `apps/web` | TBD | TBD |
| **Hand-written API** | All endpoints implemented directly in NestJS controllers/services — no auto-generated CRUD layer (e.g. no Supabase/Firebase-style generation) | `apps/api` | TBD | TBD |
| **Authentication & security** | Sign-in via BetterAuth (Google OAuth), an established auth library rather than hand-rolled auth. ⚠️ Password reset is unresolved for OAuth-only sign-in (see [ADR-002](decisions/adr-002-auth.md)); **self-service account deletion is not yet built** — see [Security](security.md) | `apps/api` (auth module) | TBD | TBD |
| **Integration with an external API** | `nba_api` (Python client for stats.nba.com) planned as the core data source; statistics to be derived from individual event data rather than precomputed league totals, per the brief. **Ingestion not yet started** — all current data is mock-seeded | `apps/api` (ingestion) | TBD | TBD |
| **Documentation website** | This MkDocs Material site, deployed via GitHub Pages static hosting | Docs site | Adrian | TBD |

## Notes

- **"TBD" owners/issues** are placeholders — fill these in as work is assigned on the Gitea issue board, and keep this table in sync rather than letting it drift from what's actually tracked.
- The **hand-written API requirement** is a specific risk area: `nba_api` itself is fine to depend on for *data*, but no part of our own API surface may be auto-generated from the database schema — every route in `apps/api` must be explicitly written.
- This table should be walked row-by-row with the client tutor at each milestone, per the brief's Milestone 4 expectation ("verify every brief requirement... each row has evidence linked").

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
