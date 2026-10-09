# NBA Analytics & Optimisation Engine

COMS3011A Sport Analytics Tool: project documentation. The code lives in the team's Gitea organisation. This site is built from the `docs/` folder of this repository and deploys on every push to `main`.

- [:material-web: **Live Webapp**](https://sportsanalytics.pages.dev/){ .md-button .md-button--primary } frontend on Cloudflare Pages
- [:material-api: **Live API**](https://sportsanalytics-api.onrender.com/api/docs){ .md-button } Swagger UI on Render
- [:material-server: **Source repo**](https://sdp.ms.wits.ac.za/innovation/sportsanalytics) on Gitea
- [:material-view-dashboard: **Project board**](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) on Gitea (needs a Gitea login)

## Start here

1. **[Demo Guide](demo-guide.md):** a 10-minute walkthrough of the live app.
2. **[Rubric Quick Links](rubric-links.md):** every rubric criterion linked to its evidence.
3. **[Getting Started](getting-started.md):** run the project locally.

## Status

Sprints 1–3 are complete and the project is being submitted as finished.

| | What was built |
|---|---|
| **Sprint 1** | NestJS API with Google sign-in, React frontend for players and teams, `nba_api` ingestion, Elo and Four Factors predictions, MILP fantasy optimizer, CI/CD, deployment on Cloudflare Pages, Render and Supabase |
| **Sprint 2** | Player comparison, postseason views and advanced stats, bug tracker, accessibility checks in CI, Swagger UI, response caching, first user survey (11 responses) |
| **Sprint 3** | Admin event corrections with an audit trail, reviewed ingestion with a pull worker, versioned dataset releases, API keys with rate limits and quotas, custom statistics, [Become Pro](become-pro/index.md), and improvements driven by user feedback |
| **Submission week** | [Player Archetypes](player-archetypes/index.md), a [Live tab](design/wireframes.md#live) for games in progress, [page tutorials](design/wireframes.md#page-tutorials), and the API-key check moved off the database ([Performance](design/performance.md#the-api-key-check-without-the-database-9-oct)) |

[Feature Tiers](design/feature-tiers.md) maps this to the brief's tiers. The [Sprint Log](sprint-log.md) has the detail, and the [Roadmap](design/roadmap.md) covers what's next.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
