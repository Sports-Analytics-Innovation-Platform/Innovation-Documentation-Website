# Stakeholder Interaction Log

What the client asked for at each meeting and what the team did about it. The full minutes, with links to the raw transcripts, are on [Client Meetings](meetings/client/index.md) and [Scrum Meetings](meetings/scrum/index.md).

## Stakeholders

| Role | Name | Channel |
|---|---|---|
| Client and tutor | Kovendan Raman | In-person meetings in lab sessions, WhatsApp |
| Scrum Master | Adrian Draxl | Scrum meetings, WhatsApp |
| Team | Owen Pace, Josh Sawyer, Kiran Soodyall, Daniel Passos, Sanele H. | Scrum meetings, WhatsApp, Gitea |

## Meetings

| Sprint | Client meetings | Internal scrums |
|---|---|---|
| Sprint 1 | 13, 18 and 21 Aug | 12 and 23 Aug |
| Sprint 2 | 7 Sep | 7, 10 and 14 Sep |
| Sprint 3 | 27 Sep | 28 Sep |

## Client feedback and what the team did

| Date | The client said | What the team did | Evidence |
|---|---|---|---|
| 13 Aug | Put infrastructure and the rubric first; polish later | Pipeline, API, sign-in, docs site and CI came first | [Sprint Log](sprint-log.md) |
| 13 Aug | Use an NBA API, not the football API considered earlier | Adopted `nba_api` (free, MIT-licensed) | [Data Ingestion](design/ingestion.md) |
| 13 Aug | Model the schema on the external API | The Prisma schema follows `nba_api`'s players, teams, games and events | [ERD](design/erd.md) |
| 13 Aug | Host the docs as a static site | MkDocs on GitHub Pages | This site |
| 13 Aug | Build a basic API as a safety net | NestJS API on Render | [API Reference](api-reference.md) |
| 13 Aug | Use AI, but review what it generates | Every AI session logged and its output reviewed | [AI Usage Ledger](ai-usage.md) |
| 13 Aug | ML can wait until Sprint 3, and deploying it is harder than running it locally | ML runs as separate Python services that write to Postgres | [Architecture](design/architecture.md#python-services) |
| 13 Aug | Avoid a purple, "AI-looking" interface | A warm dark theme with an orange accent | [UI Overview](design/wireframes.md) |
| 18 Aug | By 24 Aug: our own API, database integration, code coverage, diagrams and a project board | All five done | [CI/CD](ci-cd.md), [Architecture](design/architecture.md), [project board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) |
| 21 Aug | Load real API data, not only seed data | Three NBA seasons ingested into Supabase | [Data Ingestion](design/ingestion.md) |
| 21 Aug | Integrate sign-up with the database | Google sign-up, sign-in and account deletion with BetterAuth and Prisma. **No password reset:** Google sign-in leaves no password to reset | [ADR-002](decisions/adr-002-auth.md) |
| 21 Aug | Add a pinger to show the services are up | cron-job.org calls [`/v1/health`](https://sportsanalytics-api.onrender.com/v1/health) every 10 minutes | [Architecture](design/architecture.md#backend-appsapi) |
| 21 Aug | Document the API with Swagger | Swagger UI generated from the code | [Swagger UI](https://sportsanalytics-api.onrender.com/api/docs) |
| 21 Aug | Aim for 75–80% prediction accuracy, with 64% as a baseline | Elo and Four Factors predictor; live accuracy on the Home page's Model accuracy card | [Improvements Made](improvements.md) |
| 21 Aug | Look at the client's reference project, RaceIQ | Reviewed for UX ideas | — |
| 7 Sep | 60% accuracy is acceptable if the model is deployed | Model live on the Predictions page | [Predictions](https://sportsanalytics.pages.dev/predictions) |
| 7 Sep | Measure performance with Lighthouse, aiming for 80+ | See 27 Sep | [Performance](design/performance.md#lighthouse-scores-2026-09-29) |
| 7 Sep | Use another group's API, or have one use ours | Not done: later confirmed as not needed, because it isn't in the rubric | — |
| 7 Sep | Password reset is missing (about 80% for authentication without it); ask Brendan | Not built: Brendan said it isn't needed | [ADR-002](decisions/adr-002-auth.md) |
| 7 Sep | Send a simple user survey with a link to the web app | Google Forms survey, 8–15 Sep, 11 responses | [User Feedback Survey](feedback-survey.md) |
| 7 Sep | Deriving last-five-games stats from event data is acceptable | Kept that approach on player profiles | — |
| 7 Sep | The bug tracker can be simple | Gitea issues with a bug-report form and labels | [Testing](testing.md#bug-tracking) |
| 7 Sep | Player images from the NBA CDN are fine; stats must be stored | No change: images come from the CDN, stats are in Postgres | — |
| 27 Sep | Run Lighthouse on data-heavy pages, not only the landing page | Home, Teams, Players and Admin scored 88–95 for performance on mobile and 100 for accessibility | [Performance](design/performance.md#lighthouse-scores-2026-09-29) |
| 27 Sep | Re-test the API now that keys are required, including rate limits | Live API tested with a consumer key: missing and invalid keys and rate limits | [AI Usage Ledger](ai-usage.md) (27 Sep) |
| 27 Sep | Stress-test the API | One 10-user run (29 Sep) sent no API key, so it measured only the health route. Lighthouse and query counts are the performance evidence instead | [Performance](design/performance.md) |
| 27 Sep | Ship and document at least one change that came from user feedback | Seven shipped, each with before and after | [Improvements Made](improvements.md) |

At the 27 Sep meeting the client said the docs site needed no changes and the Git and meeting process was being followed, and closed with "I'm happy. There's nothing left."

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
