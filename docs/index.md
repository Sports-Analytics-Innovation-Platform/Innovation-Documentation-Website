# NBA Analytics & Optimisation Engine

COMS3011A — Sport Analytics Tool project documentation.

This site is the public documentation for the project, covering architecture, API reference,
process, and decision records. Source code lives in the team's Gitea organisation; this site
is built from the `docs/` folder in this repository and deployed automatically on every push
to `main`.

## Live links

- [:material-web: **Live Webapp**](https://sportsanalytics.pages.dev/){ .md-button .md-button--primary } — deployed frontend on Cloudflare Pages
- [:material-api: **Live API**](https://sportsanalytics-api.onrender.com/health){ .md-button } — backend on Render (try `/health`)
- [:material-server: **Gitea Source Repo**](https://sdp.ms.wits.ac.za/innovation/sportsanalytics) — the team's code repository
- [:material-view-dashboard: **Project Board**](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8){ .md-button } — Gitea Projects (requires Gitea authorisation)

## Start here

- [Getting Started](getting-started.md) — run the project locally
- [Architecture Overview](design/architecture.md) — system design
- [Methodology](methodology.md) — how we work
- [Decisions](decisions/index.md) — why we chose what we chose
- [Rubric Quick Links](rubric-links.md) — marking a milestone? Start here

!!! tip "Marking tutor? Start here"
    The fastest path through the project:

    1. **[Demo Guide](demo-guide.md)** — a 10-minute step-by-step walkthrough of the live app, mapped to the rubric
    2. **[Rubric Quick Links](rubric-links.md)** — every rubric criterion linked to its evidence
    3. **[Live Webapp](https://sportsanalytics.pages.dev/)** — try it yourself
    4. **[Live API](https://sportsanalytics-api.onrender.com/health)** — check `/health` for a quick backend verification

## Status

Sprint 1 is complete. The platform ships a working increment: NestJS API with BetterAuth (Google OAuth), React frontend with player/team browsing and search, game listings, predictions with Elo win-probability and Four Factors scoring, a fantasy-lineup optimizer, `nba_api` ingestion service, CI/CD pipeline (Gitea Actions), and production deployment on Cloudflare Pages (frontend), Render (API), and Supabase (Postgres). See the [Sprint Log](sprint-log.md) for the full record of what was delivered and the [Roadmap](design/roadmap.md) for what's next.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite]*
