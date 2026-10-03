# Feature Tiers

How the platform meets each tier of the brief (COMS3011A Project 3, "Sport Analytics Tool"). Every tier is built around statistics derived from individual events, so that is how this page is organised. The prediction model, optimizer and other extras are listed separately under [Beyond the brief](#beyond-the-brief).

✅ done · ⚠️ partly done, gap named · ❌ not built

## Basic tier

> "Every statistic it publishes should be derived from a record of the individual events... Only approved submitters should be able to submit... A submission... should be checked against the platform's event schema before it is accepted... Every published statistic should be traceable back to the events and the submission behind it." (§1.1.1)

| Requirement | | How |
|---|---|---|
| Statistics derived from events | ✅ | Box-score lines are summed from stored plays (`derive_player_game_stats.py`, mirrored in TypeScript for corrections). Play-by-play is kept for 2025-26 only; older seasons are box scores, because of the 500 MB database limit ([ADR-005](../decisions/adr-005-play-by-play-storage.md)). |
| Only approved submitters | ✅ | Data enters only through ingestion. On the site, only an `ADMIN` can start a pull. |
| Checked against the event schema, with reasons | ✅ | `event_validation.py` checks every play and counts each rejection by reason on its batch ([Data Ingestion](ingestion.md#batches)). |
| Traceable to events and submission | ✅ | Every stored play points to the batch that wrote it; every manual edit is recorded in `EventCorrection` with who, when and why. |
| A corrected event updates its statistics | ✅ | A correction re-derives the affected players' lines in the same transaction. |
| API for fixtures, events and statistics, filtered and paged | ✅ | `/v1/games`, `/v1/games/:id/events`, `/v1/players/:id/stats`, `/v1/players/leaders`, `/v1/players/aggregates` ([API Reference](../api-reference.md)). |
| Stable identifiers | ✅ | Every public id is a UUID, never reused. |
| Export a filtered slice | ✅ | CSV from `/v1/players/export`, `/v1/games/export` and dataset releases. |

**Basic tier: complete.**

## Intermediate tier

> "A batch should be staged and validated before it lands... Resubmitting a batch should not double-count anything, and a batch that fails part way through should resume rather than restart. Submissions should pass a review before publication... a change to the event data should only cause the figures that depend on it to be recomputed... Queries should meet a stated response time... The API should be versioned, should issue keys... and hold them to rate limits and quotas... with repeated reads served from cache... datasets should become releases..." (§1.1.2)

| Requirement | | How |
|---|---|---|
| Batches staged and validated | ✅ | Each game's run is an `IngestionBatch`, validated before its plays are saved. |
| Resubmission doesn't double count | ✅ | Plays are upserted on game and sequence number. |
| A failed batch resumes | ✅ | Each batch keeps a checkpoint and each game commits as it finishes (PR #186). |
| Review before publication | ✅ | A game is hidden from every public read while any of its batches is pending, running, failed or rejected (PR #184; [Data Ingestion](ingestion.md#review-and-publication)). |
| Only dependent figures recomputed | ✅ | A correction recomputes only the players in the corrected game (`plan-stat-recompute.ts`). |
| Impossible data caught; corrections keep a history | ✅ | Correction rules refuse, for example, credit left on the wrong player. Corrections are append-only; an undo is a new correction. |
| Figures checked against reference results | ⚠️ | Unit tests match the NBA's published advanced stats to three decimal places. There is no automated test replaying a whole game against its published box score. |
| A stated response time | ⚠️ | Target: P95 under 300 ms. Measured on 29 Sep: **P95 495 ms** with 10 concurrent users on Render's free plan. Indexes were chosen from measured query counts ([Performance](performance.md)). |
| Versioned API | ✅ | `/v1/`, `Accept-Version`, and `Deprecation`/`Sunset` headers ([API Design](api-design.md)). |
| Keys, rate limits and quotas | ✅ | Per-minute limits and daily quotas per key, counted in the database. |
| Repeated reads from cache | ✅ | An in-memory cache with lifetimes by how often data changes ([ADR-004](../decisions/adr-004-caching-strategy.md)). |
| Dataset releases | ✅ | Each release stores its CSV, a description of every field and a SHA-256 checksum. A later correction marks it stale instead of rewriting it. |

**Intermediate tier: complete except the response-time target.**

## Advanced tier

> "An analyst should be able to define a new statistic over the event schema itself... the platform should also accept a feed from a fixture in progress, and should cope with events that arrive late or out of order... say what a statistic was as of a given date... let a consumer see what changed between two dataset releases... hand large requests off as jobs... offer a feed of changes... retiring versions along a published deprecation path, testing its own contracts, and showing each consumer what it has used... flagging events that look wrong against the history, reconciling submitters that disagree." (§1.1.3)

| Requirement | | How |
|---|---|---|
| Analyst-defined statistics | ✅ | Formulas over seven event-derived fields, parsed without `eval`, versioned on each edit, for `ANALYST` and `ADMIN` only. |
| Statistics as of a date | ✅ | `GET /v1/players/:id/stats?asOf=` counts only games finished by then. |
| Differences between releases; a change feed | ✅ | `GET /v1/datasets/diff` and `GET /v1/datasets/changes?since=`. |
| Corrections reach aggregates and releases | ✅ | Season figures update at once; affected releases are marked stale in the same transaction. |
| Deprecation path and contract tests | ✅ | `GET /health` retires on 31 Mar 2027. A test compares the API with its own OpenAPI document; it isn't consumer-driven. |
| A feed from a game in progress | ⚠️ | `GET /v1/games/:id/live` returns new plays since a sequence number (PR #161), but plays arrive only when ingestion runs, after the game. |
| Late or out-of-order events | ⚠️ | Late plays are reordered within a pull (PR #148). A conflicting play from another source is rejected, not merged. |
| Showing each consumer its usage | ⚠️ | Every keyed request is logged, but the UI shows only a total count. |
| Flagging events that look wrong | ⚠️ | Impossible lines are flagged (for example, more threes than field goals), but not outliers against a player's history. |
| Large requests as jobs | ❌ | Consumer requests run to completion; only admin pulls are queued. |
| Reconciling disagreeing submitters | ❌ | There is one automated submitter, so there is nothing to reconcile. |

**Advanced tier: 5 done, 4 partly done, 2 not built.**

## Beyond the brief

| Feature | What it does |
|---|---|
| **Game predictions** | Elo win probability and a Four Factors margin, with every model run kept. **65.8%** accurate over 3,781 games, against 54.8% for always picking the home team ([Roadmap](roadmap.md#prediction-accuracy)). |
| **Betting-market odds** | Bookmakers' averaged win probability (The Odds API), shown next to the model's. |
| **Lineup optimizer** | The best five-player fantasy lineup under a salary cap, solved as an integer program. |
| **Personal features** | Beat the Model picks with a leaderboard, followed players and teams, saved comparisons and lineups. |
| **[Become Pro](../become-pro/index.md)** | A user logs their own games and sees the NBA draft pick their season most resembles, its rookie salary, and the three most similar NBA rookies. Private to each user. |
| **[Player Archetypes](../player-archetypes/index.md)** | In review, not merged. Nine playing styles found by clustering 2025-26 box-score rates, with a league style map. |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
