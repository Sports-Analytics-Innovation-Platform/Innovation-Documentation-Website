# Rubric Quick Links

One page mapping every rubric criterion (from the COMS3011A project brief) to where the evidence for it actually lives, so a marking tutor doesn't have to hunt across the site or the codebase. Grouped by milestone, in the brief's own order.

!!! warning "Honest about gaps, not just links"
    Where a criterion doesn't have real evidence yet, that's stated plainly with a ⚠️ below. Sprint 1 is complete with most criteria having solid evidence. The remaining open gaps are: **User Feedback** (Milestones 2–3), **Improvement** (Milestone 3), and **Performance** evaluations (Milestones 3–4, which are properties of the running system rather than documentation).

## Milestone 1: Sprint 1 (due 2026-08-25)

| Criterion | Weight | Evidence |
|---|---|---|
| Version Control | 10% | Repo itself (Gitea, not linkable from this site) + [Git Methodology](git-methodology.md) for how it's used. All members have committed code |
| CI/CD (brief §2.1, not a weighted line) | — | [CI/CD Pipeline](ci-cd.md) — Gitea Actions running lint/typecheck/test on every push and PR. CD is live: frontend auto-deploys to [Cloudflare Pages](https://sportsanalytics.pages.dev/), API auto-deploys to [Render](https://sportsanalytics-api.onrender.com/health) via GitHub mirror ([ADR-003](decisions/adr-003-hosting-topology.md) accepted and implemented) |
| Documentation Site | 10% | This site — deployed via GitHub Pages on every push to `main` |
| Getting Started / Dev Guides | 5% | [Getting Started](getting-started.md) |
| Work Tracker | 5% | [Gitea Projects Board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) (requires Gitea authorisation) |
| Git Methodology | 5% | [Git Methodology](git-methodology.md) |
| Project Methodology | 10% | [Methodology](methodology.md) |
| Tech Stack | 5% | [Tech Stack](tech-stack.md) — fully listed and motivated |
| Stakeholder Interaction | 10% | [Client Meetings](meetings/client/index.md), [Scrum](meetings/scrum/index.md) |
| Initial Design & Dev Plan | 20% | [Architecture](design/architecture.md), [ERD](design/erd.md), [API Design](design/api-design.md), [UI Overview](design/wireframes.md), [Feature Tiers](design/feature-tiers.md), [Roadmap](design/roadmap.md) |
| Implementation | 20% | Live webapp at [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/), live API at [sportsanalytics-api.onrender.com](https://sportsanalytics-api.onrender.com/health), codebase on Gitea |

## Milestone 2: Sprint 2 (due 2026-09-15)

| Criterion | Weight | Evidence |
|---|---|---|
| Core Features | 25% | [Feature Tiers](design/feature-tiers.md) (basic tier built and deployed) + [Requirements Traceability](requirements.md) |
| Automated Testing | 10% | Vitest + Supertest (API, against real Postgres) and Vitest + React Testing Library (web). CI `coverage` job runs both suites against a disposable Postgres service container — no `--passWithNoTests` tolerance. See [CI/CD Pipeline](ci-cd.md) |
| Stakeholder Reviews | 10% | [Client Meetings](meetings/client/index.md), [Scrum](meetings/scrum/index.md) |
| API | 15% | [API Design](design/api-design.md) — live at [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health). External integration: `nba_api` via [ingestion service](design/architecture.md) |
| User Feedback | 10% | ⚠️ No page exists. No formal user-testing/feedback-collection process is documented anywhere on this site |
| Project Methodology | 10% | [Methodology](methodology.md) — this rubric line specifically wants evidence of *active* following, not just the document existing |
| Bug Tracker | 5% | [Gitea Projects Board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) (requires Gitea authorisation) |
| Database Documentation | 5% | [ERD](design/erd.md), [ADR-001: Database](decisions/adr-001-database.md) — Supabase managed Postgres, live and connected |
| Third-Party Code Documentation | 5% | [Tech Stack](tech-stack.md) lists and motivates all libraries; [Architecture](design/architecture.md) documents how each third-party service (Cloudflare, Render, Supabase, BetterAuth, Prisma) is integrated |
| Testing Documentation | 5% | [CI/CD Pipeline](ci-cd.md) documents the automated testing procedure (what runs, how, against which database). See also Automated Testing above |

## Milestone 3: Sprint 3 (due 2026-09-29)

The brief lists weights only, no descriptions, for this milestone — criteria names are assumed to carry the same meaning as their Sprint 2 counterparts.

| Criterion | Weight | Evidence |
|---|---|---|
| User Feedback | 10% | ⚠️ Same gap as Sprint 2 — no formal feedback collection process documented yet |
| Automated Testing | 10% | Vitest + Supertest (API) and Vitest + React Testing Library (web), both running in CI against real Postgres — [CI/CD Pipeline](ci-cd.md) |
| Feature Implementation | 20% | [Feature Tiers](design/feature-tiers.md) (basic tier deployed; intermediate tier is the target per the [Roadmap](design/roadmap.md)) |
| API Implementation | 20% | [API Design](design/api-design.md) — live at [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health), with `nba_api` external integration via ingestion |
| Performance | 5% | ⚠️ Property of the running system — [live webapp](https://sportsanalytics.pages.dev/) and [live API](https://sportsanalytics-api.onrender.com/health) can be evaluated directly |
| Improvement | 5% | ⚠️ Would need a changelog or before/after comparison — doesn't exist yet |
| Documentation | 15% | This site (deployed at [docs](https://sports-analytics-innovation-platform.github.io/Innovation-Documentation-Website/)), [API Design](design/api-design.md), [ERD](design/erd.md), [Architecture](design/architecture.md) |
| Project Methodology | 15% | [Methodology](methodology.md), [Sprint Log](sprint-log.md) |

## Milestone 4: Project Submission (due 2026-10-11)

| Criterion | Category | Weight | Evidence |
|---|---|---|---|
| Data | Database | 3% | [ERD](design/erd.md) — 12 models on Supabase managed Postgres, live |
| Deployment | Database | 2% | [ADR-003: Hosting Topology](decisions/adr-003-hosting-topology.md) — accepted and implemented. Supabase managed Postgres with connection pooling over TLS |
| Structure | Database | 5% | [ERD](design/erd.md), [ADR-001: Database](decisions/adr-001-database.md) |
| Availability | API | 3% | Live at [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health) — kept responsive by a pinger service |
| Architecture | API | 5% | [Architecture](design/architecture.md) — non-monolithic, `apps/web` and `apps/api` independently deployed, HTTP-only communication |
| Deployment | API | 2% | [ADR-003](decisions/adr-003-hosting-topology.md) — Render auto-deploy from GitHub mirror. Live at [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health) |
| Performance | API | 5% | ⚠️ Property of the running system — [live API](https://sportsanalytics-api.onrender.com/health) can be evaluated directly. A pinger keeps the Render instance warm so responses are immediate |
| Design | API | 10% | [API Design](design/api-design.md) — versioned under `/v1/`, hand-written NestJS controllers/services |
| Accessibility | App | 5% | ⚠️ Skip-to-content link, `aria-label`, responsive breakpoints built — see [UI Overview](design/wireframes.md). `axe-core` audit planned for Sprint 2 |
| Aesthetics | App | 3% | [UI Overview](design/wireframes.md) — dark theme with CSS custom properties, themed charts |
| User Experience | App | 5% | [UI Overview](design/wireframes.md), live at [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/) — 8 routes, navbar navigation, court view visualisation |
| Deployment | App | 2% | [ADR-003](decisions/adr-003-hosting-topology.md) — Cloudflare Pages auto-deploy from GitHub mirror. Live at [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/) |
| Performance | App | 5% | ⚠️ Property of the running system — [live webapp](https://sportsanalytics.pages.dev/) can be evaluated directly. Vite-built static SPA on Cloudflare's global edge network |
| Features | App | 10% | [Feature Tiers](design/feature-tiers.md) — basic tier deployed: player/team browsing, search, predictions, court view, fantasy optimizer, ingestion service |
| Responsiveness | App | 5% | [UI Overview](design/wireframes.md) — mobile/tablet/desktop breakpoints documented. Live at [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/) |
| Structure | App | 5% | [Architecture](design/architecture.md) — non-monolithic, separately deployed frontend and backend |
| Git Methodology | Misc | 5% | [Git Methodology](git-methodology.md) |
| Integration | Misc | 7% | `nba_api` (Python client for stats.nba.com) integrated via [ingestion service](design/architecture.md) (`apps/ingestion`) — fetches teams, rosters, games, and box scores into Postgres. See also [Tech Stack](tech-stack.md) |
| Testing | Misc | 8% | Vitest + Supertest (API, against real Postgres) and Vitest + React Testing Library (web). CI runs both suites in the `coverage` job — [CI/CD Pipeline](ci-cd.md) |
| Tools | Misc | 5% | [Tech Stack](tech-stack.md), [AI Usage Ledger](ai-usage.md), [CI/CD Pipeline](ci-cd.md) |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite]*
