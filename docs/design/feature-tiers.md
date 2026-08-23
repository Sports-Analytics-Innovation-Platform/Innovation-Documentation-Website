# Feature Tiers

!!! success "Unblocked"
    Previously blocked on "what does the optimisation engine actually optimise?" The team's answer: compile team/player analytics from real game data, use ML to **predict** outcomes, then (as an advanced-tier feature) **recommend** actions that improve a team's win chances. Tiers below follow the brief's own definitions (§1.1): Basic = MVP, Intermediate = functional and useful, Advanced = market-ready.

## Basic tier (MVP)

Core analytics platform — built and deployed:

- Team/player browsing with event-derived season stats (`GameEvent` → `PlayerGameStat`, confirmed in schema). Teams list and profile pages are live with real team logos and player headshots from nba.com's CDN.
- Player and team search across the full dataset with pagination (server-side search query params).
- Auth via BetterAuth (Google OAuth) — see [ADR-002](../decisions/adr-002-auth.md).
- Game listings with `GET /v1/games` endpoint (auth-gated).
- **Team-level matchup prediction**: Elo-based home win probability and Four Factors-based predicted score margin, computed by `apps/predictor` and stored in the `GamePrediction` table. Exposed via `GET /v1/games/:id/prediction`. Shown in the UI on the Predictions page and the game detail page.
- **Court view** visualisation showing predicted top scorers by position on a basketball court.
- **Fantasy-lineup optimizer**: `apps/optimizer` predicts per-player fantasy points and solves a 5-player lineup under a salary cap via MILP (PuLP/CBC). Exposed via `GET /v1/optimizer/lineup` and shown on the Optimizer page (auth-gated).
- **`nba_api` ingestion service**: `apps/ingestion` fetches teams, rosters, games, and box scores from stats.nba.com into Postgres. Orchestrated by `ingest.py`.

## Intermediate tier (functional and useful)

- **Player-level prediction** added alongside team-level: predicted individual performance (e.g. points/rebounds/assists) for an upcoming or hypothetical matchup, per the team's confirmed scope.
- **Model accuracy shown, not just asserted** — a view comparing predicted vs. actual outcomes for games that have already happened, so the prediction isn't just a number nobody can verify. This also gives something concrete to show in Sprint 3 reviews.
- Richer feature set feeding the model where the data supports it (recent form, home/away split, head-to-head history) — exact feature list still open.
- **Target accuracy**: 75–80% (per client meeting 2026-08-21), with 64% as an achievable baseline.

## Advanced tier (market-ready)

- **Recommendation layer** — the actual "optimisation" in the product name: given a team's current roster/available lineup options, recommend the lineup or rotation that maximises predicted win probability. The fantasy-lineup optimizer (built in Sprint 1) is a stepping stone toward this — it already solves a lineup selection problem under constraints, but for DFS-style fantasy points rather than win probability.
- Possibly a "what-if" simulator: user picks a hypothetical lineup change, sees the predicted effect.
- Model comparison/versioning if time allows.

## Still open

- Exact model/algorithm choices at each tier beyond Elo + Four Factors (current Basic tier implementation)
- Exact feature set feeding the prediction model for Intermediate tier
- Whether the recommendation layer (advanced tier) is in-scope for this semester at all, or explicitly a stretch goal — see [Roadmap](roadmap.md) for how this maps to sprint dates, including the client's own guidance on ML timing
- Whether the current Four Factors regression model or heuristic approach will be superseded by a more sophisticated model as more real data flows through the ingestion pipeline

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
