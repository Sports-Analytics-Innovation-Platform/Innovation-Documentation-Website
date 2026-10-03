# Rubric Quick Links

Every rubric criterion in the COMS3011A brief, linked to its evidence, grouped by milestone in the brief's order. Live evidence: the [web app](https://sportsanalytics.pages.dev/), the [API's Swagger UI](https://sportsanalytics-api.onrender.com/api/docs) and the [Gitea repo](https://sdp.ms.wits.ac.za/innovation/sportsanalytics) (needs a login).

## Milestone 1: Sprint 1 (due 2026-08-25)

| Criterion | Weight | Evidence |
|---|---|---|
| Version Control | 10% | The Gitea repo, used as described in [Git Methodology](git-methodology.md). Every member has committed code. |
| CI/CD (brief §2.1, unweighted) | — | [CI/CD Pipeline](ci-cd.md): lint, typecheck and tests on every push. The web app and API auto-deploy ([ADR-003](decisions/adr-003-hosting-topology.md)). |
| Documentation Site | 10% | This site, deployed by GitHub Actions on every push to `main` |
| Getting Started | 5% | [Getting Started](getting-started.md) |
| Work Tracker | 5% | [Gitea project board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) (needs a login) |
| Git Methodology | 5% | [Git Methodology](git-methodology.md) |
| Project Methodology | 10% | [Methodology](methodology.md) and the [Sprint Log](sprint-log.md) |
| Tech Stack | 5% | [Tech Stack](tech-stack.md): every choice listed with its reason |
| Stakeholder Interaction | 10% | [Stakeholder Interactions](stakeholder-interactions.md), [Client Meetings](meetings/client/index.md), [Scrum Meetings](meetings/scrum/index.md) |
| Initial Design & Dev Plan | 20% | [Architecture](design/architecture.md), [ERD](design/erd.md), [API Design](design/api-design.md), [UI Overview](design/wireframes.md), [Feature Tiers](design/feature-tiers.md), [Roadmap](design/roadmap.md) |
| Implementation | 20% | The live [web app](https://sportsanalytics.pages.dev/) and [API](https://sportsanalytics-api.onrender.com/api/docs). Walk through it with the [Demo Guide](demo-guide.md). |

## Milestone 2: Sprint 2 (due 2026-09-15)

| Criterion | Weight | Evidence |
|---|---|---|
| Core Features | 25% | [Feature Tiers](design/feature-tiers.md) (Basic tier complete) and [Requirements Traceability](requirements.md) |
| Automated Testing | 10% | [Testing](testing.md): Vitest and Supertest against real Postgres, run in CI by the `coverage` job |
| Stakeholder Reviews | 10% | [Stakeholder Interactions](stakeholder-interactions.md): every client meeting, the feedback given and the action taken |
| API | 15% | [API Reference](api-reference.md), [API Design](design/api-design.md) and the live [Swagger UI](https://sportsanalytics-api.onrender.com/api/docs) |
| User Feedback | 10% | [User Feedback Survey](feedback-survey.md): method, 11 responses, findings, and how each one was acted on |
| Project Methodology | 10% | [Methodology](methodology.md), [Sprint Log](sprint-log.md), [Scrum Meetings](meetings/scrum/index.md) |
| Bug Tracker | 5% | [Testing: Bug tracking](testing.md#bug-tracking). Example: issue [#83](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/issues/83), filed and closed. |
| Database Documentation | 5% | [ERD](design/erd.md) and [ADR-001: Database](decisions/adr-001-database.md) |
| Third-Party Code | 5% | [Tech Stack](tech-stack.md): every dependency in the API, the web app and the Python services, with its reason |
| Testing Documentation | 5% | [Testing](testing.md): procedure, policy, coverage and conventions |

## Milestone 3: Sprint 3 (due 2026-09-29)

| Criterion | Weight | Evidence |
|---|---|---|
| User Feedback | 10% | [User Feedback Survey](feedback-survey.md) (11 responses) and a hands-on [User Interview](user-interviews.md) (2026-09-27) |
| Automated Testing | 10% | [Testing](testing.md): 974 API and 717 web tests at the Become Pro hand-off, plus a 65-check live browser run, with an 80% coverage threshold in CI |
| Feature Implementation | 20% | [Feature Tiers](design/feature-tiers.md): Basic complete, Intermediate largely complete, Advanced partial. Also [Become Pro](become-pro/index.md) and [Player Archetypes](player-archetypes/index.md). |
| API Implementation | 20% | [API Reference](api-reference.md): 106 live operations, including admin, datasets, custom statistics, API keys and Become Pro |
| Performance | 5% | [Performance](design/performance.md): Lighthouse Performance 88–95 and Accessibility 100 on mobile, and caching with before/after query counts |
| Improvement | 5% | [Improvements Made](improvements.md): seven shipped changes, each with a before/after and the feedback item that prompted it |
| Documentation | 15% | This site, especially the [API Reference](api-reference.md), [ERD](design/erd.md) and [Architecture](design/architecture.md) |
| Project Methodology | 15% | [Methodology](methodology.md) and the [Sprint Log](sprint-log.md) |

## Milestone 4: Project Submission (due 2026-10-11)

| Criterion | Area | Weight | Evidence |
|---|---|---|---|
| Data | Database | 3% | [ERD](design/erd.md): 39 tables and 7 enums on Supabase Postgres |
| Deployment | Database | 2% | [ADR-003](decisions/adr-003-hosting-topology.md): Supabase, pooled connections over TLS |
| Structure | Database | 5% | [ERD](design/erd.md), [ADR-001](decisions/adr-001-database.md), [ADR-005](decisions/adr-005-play-by-play-storage.md) |
| Availability | API | 3% | [Live API](https://sportsanalytics-api.onrender.com/v1/health), kept warm by a pinger |
| Architecture | API | 5% | [Architecture](design/architecture.md): separate web, API and Python services that talk over HTTP |
| Deployment | API | 2% | [ADR-003](decisions/adr-003-hosting-topology.md): Render auto-deploys from the GitHub mirror |
| Performance | API | 5% | [Performance](design/performance.md#measured-result): repeat requests for public data issue no database statements. Why: [ADR-004](decisions/adr-004-caching-strategy.md) |
| Design | API | 10% | [API Design](design/api-design.md) and the [API Reference](api-reference.md): hand-written, versioned under `/v1/`, one error envelope |
| Accessibility | App | 5% | Lighthouse Accessibility **100** on every page tested ([Home](assets/lighthouse/homepage-mobile.jpg), [Teams](assets/lighthouse/teams-mobile.jpg), [Players](assets/lighthouse/players-mobile.jpg), [Admin](assets/lighthouse/admin-mobile.jpg)). `axe-core` runs in CI. See [UI Overview](design/wireframes.md#accessibility). |
| Aesthetics | App | 3% | [UI Overview](design/wireframes.md), with screenshots |
| User Experience | App | 5% | [UI Overview](design/wireframes.md) and the [Demo Guide](demo-guide.md) |
| Deployment | App | 2% | [ADR-003](decisions/adr-003-hosting-topology.md): Cloudflare Pages auto-deploys from the GitHub mirror |
| Performance | App | 5% | [Lighthouse](design/performance.md#lighthouse-scores-2026-09-29) Performance 88–95 on mobile, and [frontend caching](design/performance.md#frontend-caching) |
| Features | App | 10% | [Feature Tiers](design/feature-tiers.md) and the [Demo Guide](demo-guide.md) |
| Responsiveness | App | 5% | [UI Overview](design/wireframes.md#responsive-design), with phone screenshots |
| Structure | App | 5% | [Architecture](design/architecture.md) |
| Git Methodology | Misc | 5% | [Git Methodology](git-methodology.md) |
| Integration | Misc | 7% | `nba_api` through the [ingestion service](design/ingestion.md), and The Odds API for bookmaker lines ([Feature Tiers](design/feature-tiers.md#beyond-the-brief)) |
| Testing | Misc | 8% | [Testing](testing.md) and [CI/CD Pipeline](ci-cd.md) |
| Tools | Misc | 5% | [Tech Stack](tech-stack.md), [CI/CD Pipeline](ci-cd.md), [AI Usage Ledger](ai-usage.md) |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
