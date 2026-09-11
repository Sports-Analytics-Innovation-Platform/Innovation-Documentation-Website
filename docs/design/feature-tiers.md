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

- **Player comparison endpoint**: `GET /v1/players/compare` returns side-by-side derived season averages for 2–4 players, for a chosen season segment. Per-player game-outcome prediction (e.g. predicted points/rebounds/assists for an upcoming matchup) is planned for a later sprint.
- **Postseason views**: games and player stats are tagged with a `seasonType` (`REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS`) and `playoffRound`. Player profiles, the comparison page, and the players list can all switch to a postseason-only view via a `SeasonSegmentControl`, backed by `GET /v1/players/:id/stats/splits` (every segment in one request) so the UI never shows a regular-season figure mislabelled as a playoff one. Postseason games are deliberately excluded from the prediction and optimizer models — a playoff matchup isn't statistically like a regular-season one.
- **Advanced player stats**: true shooting%, effective FG%, and assist-to-turnover ratio are derived on read from existing boxscore fields (verified against `BoxScoreAdvancedV3` to three decimal places). Plus-minus, usage%, offensive rating, and defensive rating are ingested per game from the same advanced boxscore and stored on `PlayerGameStat`, fetched leaguewide rather than per game to stay within the ingestion API's rate limit. All four are nullable — `null` for games ingested before these columns existed, since a real 0% usage rate is a different fact than a missing one.
- **Model accuracy shown, not just asserted** — a view comparing predicted vs. actual outcomes for games that have already happened, so the prediction isn't just a number nobody can verify. This also gives something concrete to show in Sprint 3 reviews.
- Richer feature set feeding the model where the data supports it (recent form, home/away split, head-to-head history) — exact feature list still open.
- **Target accuracy**: 75–80% (per client meeting 2026-08-21), with 64% as an achievable baseline; the team reported ~65% after seeding two additional seasons (team meeting, 2026-09-10).

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
