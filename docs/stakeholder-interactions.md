# Stakeholder Interaction Log

This page is a consolidated record of every formal interaction with the project's client/stakeholder — **Kovendan Raman** (tutor/client) — across both sprints. It documents dates, attendees, topics discussed, feedback received from the client, and the actions the team took in response. Raw transcripts are linked where available.

The purpose of this log is to evidence the **Stakeholder Interaction** rubric criterion (10%) by showing that the team engaged the client regularly, acted on his feedback, and can trace decisions back to specific conversations.

## Stakeholders

| Role | Name | Interaction channel |
|---|---|---|
| Client / Tutor | Kovendan Raman | Weekly in-person meetings (Tue/Fri labs), WhatsApp |
| Scrum Master | Adrian Draxl | Internal scrum meetings, WhatsApp |
| Team | Owen Pace, Josh Sawyer, Kiran Soodyall, Daniel Passos, Sanele H. | Internal scrum meetings, WhatsApp, Gitea |

---

## Sprint 1 interactions

### 2026-08-12 — Internal Scrum (Sprint 1 kickoff)

| | |
|---|---|
| **Type** | Internal team standup |
| **Attendees** | Adrian, Daniel, Owen, Sanele, Kiran, Josh |
| **Duration** | Full team meeting |

**Agenda:** Sprint 1 requirements, documentation, Git methodology, coding conventions, feature implementation strategy, work tracker setup, and stakeholder meeting preparation.

**Key decisions:**

- Adopted **CalVer (date-based) versioning** for easier historical tracking
- Agreed on a **Scrum adaptation** with three sprints and initiative-based task allocation
- Confirmed **Gitea Projects + Issues** as the work tracker
- Decided to build features **vertically** (full-stack per feature) rather than in horizontal layers
- Team members must announce which feature they're working on in WhatsApp to avoid conflicts

**Actions taken:**

| Action | Owner | Completed? |
|---|---|---|
| Maintain documentation site and incorporate project documents | Adrian | ✅ |
| Assist with documentation diagrams | Owen, Josh | ✅ |
| Keep work tracker accurate and assign tasks | Sanele | ✅ |
| Implement CI/CD workflow (YAML, coverage, linting, testing) | Kiran | ✅ |
| Maintain repository hygiene and Git/coding conventions | Josh | ✅ |
| Push project overview and additional documents to repo | Owen | ✅ |
| Arrange tutor meeting | Daniel | ✅ |

---

### 2026-08-13 — Client meeting (Sprint 1 scope alignment)

| | |
|---|---|
| **Type** | Client meeting |
| **Attendees** | Adrian, Kovendan Raman (client), Pace, Sanele, Josh |
| **Absent** | Kiran, Daniel (stuck in traffic) |

**Agenda:** Sprint 1 scope, MVP status, database schema, external API integration, documentation, Git workflow, hosting/deployment, front-end design, ML expectations, and ongoing meeting cadence.

**Client feedback received:**

| # | Feedback / guidance | How the team responded |
|---|---|---|
| 1 | Sprint 1 should focus on **infrastructure and rubric requirements**; additional features where time allows | Team prioritised pipeline, API, auth, docs site, and CI over visual polish |
| 2 | Use an **NBA API** rather than the previously considered EPL football API | Team adopted `nba_api` (Python) as the data source — confirmed free, MIT-licensed, well-maintained |
| 3 | Database schema should **correspond closely with the external API structure** | Schema was aligned with `nba_api` response shapes (players, teams, games, events) |
| 4 | Deploy documentation site using **static web hosting** | Docs deployed to GitHub Pages via MkDocs |
| 5 | Demonstrate **ongoing progress** during meetings rather than waiting until deadline | Team has demonstrated work at every subsequent meeting |
| 6 | Core functionality and infrastructure take priority over **visual polish** in Sprint 1 | Team focused on working features over design refinement |
| 7 | **AI-assisted development is encouraged** ("vibe coding"), but review generated components like schema and env config | Team logged all AI usage per the [AI Usage Ledger](ai-usage.md) and reviewed generated code |
| 8 | ML component not needed in Sprint 1 — expect it around **Sprint 3** | ML/prediction work deferred; Sprint 1 focused on data pipeline and API |
| 9 | Set up a **basic API as a safety net** — previous groups received credit for this even when not explicitly in the rubric | NestJS API was built and deployed to Render |
| 10 | Aim beyond minimum rubric requirements — **80–100% coverage mindset** | Team adopted this as a guiding principle |
| 11 | Documentation-focused contributions are **valid team contributions** | Adrian's documentation role was confirmed and formalised |
| 12 | ML deployment can be **substantially harder** than running locally — be aware of backend deployment challenges | Team planned separate Python services rather than embedding ML in the Node API |
| 13 | Front-end design looked good — **satisfied with current direction** | No design changes required |
| 14 | No testing requirement in Sprint 1, but AI useful for generating test cases later | Testing planned for Sprint 2 |
| 15 | Avoid **excessively purple** interfaces (associates purple with AI-generated UIs) | Team used teal colour scheme instead |

**Actions from this meeting:**

| Action | Owner | Completed? |
|---|---|---|
| Continue project infrastructure, database, hosting, core functionality | Team | ✅ |
| Investigate and integrate NBA API | Pace / Team | ✅ |
| Align database schema with API structure | Team | ✅ |
| Prepare basic API functionality | Team | ✅ |
| Build documentation site and required docs | Team (Adrian) | ✅ |
| Deploy docs via static hosting | Team | ✅ |
| Continue front-end development from mock-data implementation | Team | ✅ |
| Demonstrate progress at future client meetings | Team | ✅ |
| Assign Gitea issues to team members | Sanele | ✅ |

!!! note "Raw transcript"
    [2026-08-13-client.txt](transcripts/meeting-transcripts/2026-08-13-client.txt)

---

## Sprint 2 interactions

### 2026-08-18 — Client meeting (Sprint 2 priorities)

| | |
|---|---|
| **Type** | Client meeting |
| **Attendees** | Adrian, Owen, Josh, Sanele, Kovendan Raman (client) |

**Agenda:** Review of progress so far; features to be implemented in the coming week.

**Client feedback received:**

| # | Feedback / guidance | How the team responded |
|---|---|---|
| 1 | Key Sprint 2 priorities identified: **custom API, database integration, code coverage, diagrams, project board** | All five items were addressed during the week of 18 Aug |
| 2 | All action items due by **2026-08-24** | Team completed all items before the deadline |

**Actions from this meeting:**

| Action | Owner | Completed? |
|---|---|---|
| Implement the team's own API | Team | ✅ — NestJS API with full CRUD endpoints |
| Download and use API data from a database | Team | ✅ — `nba_api` ingestion into Supabase Postgres |
| Set up code coverage reporting in Gitea Actions | Team | ✅ — `coverage` job with v8 provider, merged HTML report |
| Create project diagrams (architecture, ERD, etc.) | Team | ✅ — Architecture, ERD, wireframes on docs site |
| Set up and maintain a project board | Team | ✅ — Gitea Projects board with issues assigned |

---

### 2026-08-21 — Client meeting (Feature review and Sprint 2 direction)

| | |
|---|---|
| **Type** | Client meeting |
| **Attendees** | Adrian, Josh, Daniel, Sanele, Kovendan Raman (client) |

**Agenda:** Progress update and feature review; discussion of upcoming features and ML goals.

**Client feedback received:**

| # | Feedback / guidance | How the team responded |
|---|---|---|
| 1 | **Search bar feature is complete** — confirmed | Team moved on to next priorities |
| 2 | Implement a **second API** to expand functionality, using buckets to pull data | Team planned the ingestion pipeline as the second data source |
| 3 | **Swagger** should be used alongside the documentation site for API documentation | Swagger/OpenAPI setup planned for Sprint 2 (in progress) |
| 4 | ML predictive model target accuracy: **75–80%**, with 64% as a baseline | Team's Four Factors model and Elo predictor target this range |
| 5 | Implement a **hosting topology pinger** to show all services are healthy | Pinger implemented — see [Architecture](design/architecture.md) |
| 6 | **Signup functionality** must be integrated with the database | Sign-up, sign-in, password reset, and account deletion implemented with BetterAuth + Prisma |
| 7 | Database must include **real API data** (not just seed data) | Full-league NBA data ingested into Supabase |
| 8 | Review the client's **reference project** ([RaceIQ](https://github.com/Race1Q/RaceIQ)) for inspiration | Team reviewed the project for UX and feature ideas |

**Actions from this meeting:**

| Action | Owner | Completed? |
|---|---|---|
| Add a pinger to the hosting topology | Team | ✅ |
| Implement the predictive ML model (target: 75–80% accuracy) | Team | ⏳ Sprint 3 |
| Have the team's API consume another API | Team | ✅ — `nba_api` ingestion service |
| Integrate signup functionality with the database | Team | ✅ |
| Ensure the database includes API data | Team | ✅ — full-league ingestion |
| Set up Swagger for API documentation | Team | ⏳ In progress |
| Review client's reference project (RaceIQ) | Team | ✅ |

---

### 2026-08-23 — Internal Scrum (Sprint 1 readiness check)

| | |
|---|---|
| **Type** | Internal team standup |
| **Attendees** | Owen, Adrian, Josh, Kiran, Sanele |

**Agenda:** Sprint 1 rubric walkthrough and readiness check; code coverage status; API structure; Google Auth login issue; methodology documentation; auth requirements gap; documentation updates; AI transcript collection.

**Key decisions:**

- Methodology updated to describe the process as a **dynamic, initiative-based Scrum adaptation** (not planned sprints with pre-assigned tasks)
- **Sprint Review** replaced with a **Sprint Reflection** using the Sprint Log (week-by-week tabulated record)
- A **project plan** maintained from Sprint Log + roadmap to show forward planning
- **Gitea project board** confirmed as the work tracker evidence
- Two critical Sprint 1 gaps identified: (1) Google Auth login broken on hosted site, (2) credential-based sign-up/password reset/account deletion not yet implemented

**Actions taken:**

| Action | Owner | Completed? |
|---|---|---|
| Fix Google Auth login (404 on hosted site) | Daniel / Team | ✅ |
| Implement credential-based sign-up, password reset, account deletion | Daniel (Sanele backup) | ✅ |
| Update documentation site to reflect current project state | Adrian | ✅ |
| Replace/update diagrams on docs site | Adrian | ✅ |
| Assign Gitea project board tasks to team members | Sanele | ✅ |
| Add remaining AI transcripts to docs site | Owen, Josh, Kiran, Sanele | ✅ |
| Ensure all team members familiar with tech stack for marking | All | ✅ |

!!! note "Raw transcript"
    [2026-08-23-team-standup.txt](transcripts/meeting-transcripts/2026-08-23-team-standup.txt)

---

## Summary of client feedback and team responses

This table traces every piece of significant client feedback to the concrete action the team took, demonstrating that stakeholder input directly shaped the project.

| Date | Client feedback | Team action | Evidence |
|---|---|---|---|
| 13 Aug | Use NBA API, not EPL | Adopted `nba_api` as data source | [Tech Stack](tech-stack.md), ingestion service |
| 13 Aug | Schema should match API structure | Aligned Prisma schema with `nba_api` shapes | [ERD](design/erd.md) |
| 13 Aug | Deploy docs via static hosting | MkDocs → GitHub Pages | [Docs site](https://sports-analytics-innovation-platform.github.io/Innovation-Documentation-Website/) |
| 13 Aug | Set up basic API as safety net | Built NestJS API, deployed to Render | [API health check](https://sportsanalytics-api.onrender.com/health) |
| 13 Aug | AI use encouraged, but review generated code | AI Usage Ledger maintained | [AI Usage](ai-usage.md) |
| 13 Aug | ML not needed until Sprint 3 | Deferred ML; focused on data pipeline | [Roadmap](design/roadmap.md) |
| 18 Aug | Code coverage in CI required | Added `coverage` job to Gitea Actions | [CI/CD](ci-cd.md) |
| 18 Aug | Project diagrams needed | Architecture, ERD, wireframes created | [Architecture](design/architecture.md) |
| 18 Aug | Project board needed | Gitea Projects board set up | [Project board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) |
| 21 Aug | Hosting topology pinger needed | Pinger service implemented | [Architecture](design/architecture.md) |
| 21 Aug | Signup must integrate with DB | BetterAuth + Prisma sign-up/reset/delete | Sprint 2 deliverables |
| 21 Aug | Swagger for API docs | `@nestjs/swagger` setup planned | Pending |
| 21 Aug | ML target: 75–80% accuracy | Four Factors + Elo model in development | [Predictions](https://sportsanalytics.pages.dev/predictions) |
| 21 Aug | Review RaceIQ reference project | Team reviewed for UX inspiration | — |

## Meeting cadence

| Period | Frequency | Format |
|---|---|---|
| Sprint 1 (4–25 Aug) | 3 client meetings + 2 internal scrums | In-person at Tue/Fri labs + WhatsApp coordination |
| Sprint 2 (26 Aug – 15 Sep) | Weekly client meetings + weekly scrums | Same format; meetings documented as they occur |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite]*
