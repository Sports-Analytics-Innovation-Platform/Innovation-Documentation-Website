# Roadmap

The plan for each sprint, what was delivered, and what is left. Task-by-task detail, with owners, is in the [Sprint Log](../sprint-log.md); how the work maps to the brief is in [Feature Tiers](feature-tiers.md).

## Plan and delivery

| Sprint | Planned | Delivered | Not delivered |
|---|---|---|---|
| **1: Foundation** (to 25 Aug) | Stack, auth, NBA data, a first prediction model, hosting, CI | NestJS API with BetterAuth (Google); `nba_api` ingestion; Elo and Four Factors game predictor; lineup optimizer; Teams, Predictions, game detail and Optimizer pages; CI with coverage; API on Render and web app on Cloudflare Pages; this docs site | Password reset ([ADR-002](../decisions/adr-002-auth.md)) |
| **2: Real data and features** (to 15 Sep) | Full-league data, API documentation, a keep-awake pinger, accessibility checks, the signed-in home page, UI polish | Three seasons of real data; live Swagger UI; pinger; axe-core checks in CI; postseason views and advanced stats; player comparison; player matchup projections (planned for Sprint 3, merged 12 Sep); signed-in Home (Beat the Model, watchlist, saved items, model-accuracy card, leaderboard); response caching and indexes ([Performance](performance.md)) | Redis and BullMQ: not needed ([Tech Stack](../tech-stack.md#considered-and-not-used)) |
| **3: Model maturity** (to 29 Sep) | Player-level predictions, a defensible model, 75–80% accuracy (64% as the baseline) | The brief's event-sourcing requirements (review before publication, corrections with undo, the pull worker, versioned dataset releases, API keys with rate limits, custom statistics, API versioning); [Become Pro](../become-pro/index.md); read-aloud and screen-reader support; user survey and interviews; load test and Lighthouse audit | 75–80% accuracy: the live model is at 65.8% (below) |
| **Submission** (11 Oct) | Polish and final documentation | [Player Archetypes](../player-archetypes/index.md); this documentation pass | — |

The first plan also had an **ML recommendation layer** as the advanced tier. It was dropped when the brief's event-sourcing requirements became the priority in Sprint 3; the client had advised that the last weeks could be touch-ups if the ML wasn't ready.

## Prediction accuracy

The client set a target of 75–80%, with 64% as an achievable baseline (21 Aug), and said 60% was acceptable if the model was deployed (7 Sep). On 3 Oct, the live model-accuracy route reported:

| Measure | Value |
|---|---|
| Games evaluated | 3,781 |
| Accuracy | **65.8%** |
| Always picking the home team | 54.8% |
| Brier score (lower is better) | 0.213 |

The model is calibrated: in each confidence band, the predicted and actual win rates are within about 3 percentage points.

## After submission

In rough order of importance:

1. **Backups.** The free database has none, so user data can't be recovered ([ADR-003](../decisions/adr-003-hosting-topology.md#open-questions)).
2. **Fresh data without a person.** Pulls still need someone to run the pull worker at home.
3. **Password reset,** through an email-and-password option alongside Google.
4. **CI gaps:** secret scanning, the Python tests and a build step ([CI/CD](../ci-cd.md#not-in-ci-yet)).
5. **A better model,** towards the client's 75–80% target.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
