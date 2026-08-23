# Roadmap

!!! success "Unblocked"
    Same source as [Feature Tiers](feature-tiers.md) — the team's plain-English answer on what the ML component does. This maps those tiers onto the brief's actual sprint dates.

## Sprint 1 — infra & foundation (due 2026-08-25) — COMPLETE

Sprint 1 delivered a working increment across the full stack:

**Backend & data**
- NestJS/Prisma/Postgres API with BetterAuth (Google OAuth) migration
- `nba_api` ingestion service (`apps/ingestion`) — fetches teams, rosters, games, and box scores
- GamePrediction model + predictor service (Elo win probability, Four Factors margin)
- `GET /v1/games/:id/prediction` endpoint with e2e tests
- Fantasy-lineup optimizer (`apps/optimizer`) with MILP solve
- Player and team search endpoints
- Authenticated route and API protection (predictions, games, optimizer)
- Real team logos and player headshots from nba.com's CDN
- Player bio fields from CommonPlayerInfo endpoint

**Frontend**
- React/Vite/Tailwind frontend with TanStack Query + shadcn/ui
- Teams list and profile pages
- Predictions page with court view visualisation
- Game detail page with win probability and predicted top scorers
- Optimizer page (auth-gated)
- Top navbar replacing sidebar navigation
- Landing page with hero section

**Infrastructure & CI/CD**
- Gitea Actions CI pipeline (lint, typecheck, test coverage)
- Code coverage reporting with combined HTML dashboard
- ADR-003: hosting topology decided (Cloudflare Pages + Render + Supabase)
- API deployed to Render, frontend deployed to Cloudflare Pages
- Gitea → GitHub mirror for auto-deploy

**Documentation**
- MkDocs documentation site with GitHub Pages deploy
- Full audit of all doc pages against actual codebase
- AI/Codex usage transcripts added

See the [Sprint Log](../sprint-log.md) for the complete task-by-task record.

## Sprint 2 — real data + expanded features (due 2026-09-15)

- Get `nba_api` ingestion flowing with real full-league data into Supabase (not just mock seed data)
- Swagger/OpenAPI setup for API documentation alongside this docs site
- Hosting topology pinger (per client meeting 2026-08-21)
- Database integration with signup functionality
- `axe-core` accessibility checks in CI
- Redis/BullMQ, Zod if the ingestion pipeline needs batch/scheduled processing
- Frontend polish and responsiveness pass

## Sprint 3 — model maturity (due 2026-09-29)

- **Intermediate-tier** prediction: player-level predictions, predicted-vs-actual accuracy view
- This is the sprint the client flagged as the realistic target for "a working ML model" — treat this as the sprint where prediction quality actually needs to be defensible in a review, not just present
- Target accuracy: 75–80% (per client meeting), with 64% as an achievable baseline
- Refine prediction model beyond Elo + Four Factors as real data accumulates

## Submission (due 2026-10-11)

- **Advanced-tier** recommendation layer, if time allows — this is explicitly the highest-risk, most-optional item; the client's own guidance ("last two weeks are mostly touch-ups if ML isn't ready") suggests treating this as a stretch goal, not a commitment
- Polish, responsiveness/accessibility pass, final documentation pass
- Review client's reference project ([RaceIQ](https://github.com/Race1q/RaceIQ)) for inspiration

## Still open

- Whether the advanced-tier recommendation layer is a real commitment or an explicit stretch goal — worth deciding as a team rather than leaving implicit
- Individual ownership of Sprint 2/3 work (Sprint 1 was completed with initiative-based task selection per the [Methodology](../methodology.md))

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
