# Requirements Traceability

This table maps every key requirement from the COMS3011A brief (§2.1) to the component that satisfies it, the owner responsible, and the tracking issue. It's reviewed at each milestone against the deployed environment — see [Definition of Done](definition-of-done.md).

| Brief requirement | How we satisfy it | Component | Owner | Issue |
|---|---|---|---|---|
| **Version control** | Git-compliant VCS, hosted on GitLab per the university requirement | Whole repo | Josh | TBD |
| **Responsiveness & accessibility** | Responsive layouts (mobile/tablet/desktop breakpoints), keyboard navigation, `axe-core` checks in CI | `apps/web` | TBD | TBD |
| **CI/CD** | GitLab CI pipeline — test + coverage on every push, coverage report published to GitLab Pages. Lint/typecheck not wired into CI yet (run manually) | Whole repo | Kiran | TBD |
| **Non-monolithic front-end and back-end** | Separate `apps/api` (NestJS) and `apps/web` (React + Vite) apps, communicating only over HTTP, independently deployable | `apps/api`, `apps/web` | TBD | TBD |
| **Hand-written API** | All endpoints implemented directly in NestJS controllers/services — no auto-generated CRUD layer (e.g. no Supabase/Firebase-style generation) | `apps/api` | TBD | TBD |
| **Authentication & security** | Sign-in via BetterAuth (Google OAuth), an established auth library rather than hand-rolled auth. Password reset doesn't apply under OAuth-only sign-in; **self-service account deletion not yet built** — see [Security](security.md) | `apps/api` (auth module) | TBD | TBD |
| **Integration with an external API** | `nba_api` (Python client for stats.nba.com) planned as the core data source; statistics to be derived from individual event data rather than precomputed league totals, per the brief. **Ingestion not yet started** — all current data is mock-seeded | `apps/api` (ingestion) | TBD | TBD |
| **Documentation website** | This MkDocs Material site, deployed via GitHub Pages static hosting | Docs site | Adrian | TBD |

## Notes

- **"TBD" owners/issues** are placeholders — fill these in as work is assigned on the GitLab issue board, and keep this table in sync rather than letting it drift from what's actually tracked.
- The **hand-written API requirement** is a specific risk area: `nba_api` itself is fine to depend on for *data*, but no part of our own API surface may be auto-generated from the database schema — every route in `apps/api` must be explicitly written.
- This table should be walked row-by-row with the client tutor at each milestone, per the brief's Milestone 4 expectation ("verify every brief requirement... each row has evidence linked").

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
