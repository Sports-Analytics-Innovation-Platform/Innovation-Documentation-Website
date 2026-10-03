# ADR-005: Play-by-play storage

- **Status:** Accepted on 2026-09-27.
- **Decided by:** Sanele H.
- **Last updated:** 2026-09-28

## Summary

The database stores **play-by-play for one season only: 2025-26**, both its regular season and its postseason. The two older seasons in the database, 2023-24 and 2024-25, are stored as **box scores only**.

The reason is database size. Production runs on Supabase's free plan ([ADR-003](adr-003-hosting-topology.md)), which limits the database to **500 MB**. One season of play-by-play takes about **320 MB**, which is already most of that limit. A second season would not fit.

The feature this limits most is **play corrections**. An admin can only correct a play the database holds, so corrections are available for 2025-26 games only.

### Terms used in this document

| Term | Meaning here |
|---|---|
| Play-by-play | The NBA's record of every action in a game: each shot, rebound, foul, turnover and so on. The ingestion scripts download it from stats.nba.com's `PlayByPlayV3` endpoint and store one `GameEvent` row per action. |
| Box score | One line of totals per player per game, such as points, rebounds and assists. Stored as `PlayerGameStat` rows. |
| Play correction | The admin tool (PR #182) for fixing a single play, for example the wrong player, the wrong shot value or the wrong assist credit. Every statistic that depends on the play is then brought back in line. |
| Free plan | Supabase's no-cost plan, which the production database runs on. |

## Context

### What play-by-play is for

The brief asks for every published statistic to be derived from a record of individual events, and to be traceable back to those events and the submission that brought them in ([Feature Tiers](../design/feature-tiers.md)). Play-by-play is that record. For a game that has it:

- its box-score lines are derived from its plays (`apps/ingestion/derive_player_game_stats.py`);
- each play records the ingestion run that wrote it (`GameEvent.batchId` → `IngestionBatch`);
- an admin can correct any play.

A play correction works directly on the stored plays. When an admin saves one, the API does all of the following in a single database transaction (`applyCorrectionPlan` in `apps/api/src/admin/admin-events.service.ts`):

1. updates the play's `GameEvent` row;
2. re-derives the box-score lines of the players the play affects, and no one else (`PlayerGameStat`);
3. marks that season's published dataset releases as stale (`DatasetRelease.isStale`);
4. records the old and new values, who made the change and why (`EventCorrection`).

A game without stored plays has nothing for this process to work on.

### How much space play-by-play takes

Measured on the production database with a read-only connection on 25 and 27 September 2026:

| Measure | Value |
|---|---|
| Free plan database limit | 500 MB |
| Database size on 2026-09-27 | 392 MB |
| `GameEvent` table, including its indexes (2025-26 plays only) | 323 MB, about 82% of the database |
| 2025-26 plays stored | about 625,000 |
| Plays per game | about 472 in the regular season, about 483 in the postseason |
| Space per play, including indexes | about 517 bytes |
| One postseason (about 90 games) | about 24 MB with play-by-play; about 1.4 MB as box scores only |

Play-by-play is about 94% of the space a game takes up. The box scores are small by comparison.

Starting from the 392 MB in use, this is what each option would cost:

| What would be loaded | Estimated database size | Fits within 500 MB? |
|---|---|---|
| Nothing more (the decision below) | 392 MB | Yes, 108 MB to spare |
| 2023-24 and 2024-25 postseasons, box scores only | about 395 MB | Yes |
| 2023-24 and 2024-25 postseasons, with play-by-play | about 440 MB | Only just, 60 MB to spare |
| Play-by-play for one older regular season (1,230 games) | about 690 MB | No |
| Play-by-play for both older regular seasons | about 990 MB | No, about twice the limit |

The spare space isn't only for play-by-play. Other tables grow with normal use: every request made with an API key adds an `ApiUsageLog` row, every published dataset release stores its CSV (about 90 KB each), and users add accounts, saved items and Become Pro games.

### What happens at the limit

According to [Supabase's documentation on database size](https://supabase.com/docs/guides/platform/database-size), a free-plan project that goes over its limit is put into **read-only mode**. Every write then fails, not just ingestion and corrections. Signing in writes a `Session` row, so signing in would fail too, along with game picks, saved comparisons and lineups, and Become Pro. Running out of space would break most of the site, not one feature.

### Why the older seasons have no play-by-play

The 2023-24 and 2024-25 regular seasons were loaded with `apps/ingestion/ingest_historical_season.py`, to give the Elo prediction model a history of results. Their box scores are complete (82 games for every team), but neither season has any `GameEvent` rows.

Several `PlayerGameStat` columns are also empty for those seasons. None of the 49,409 older player rows has the offensive/defensive rebound split or plus-minus, measured in the team-profiles feasibility study on 2026-09-25. The rebound split is derived from play-by-play, so it can't exist without it. Usage and the two ratings are loaded by `backfill_advanced_stats.py`, which only runs for the current season.

## Decision

**Play-by-play (`GameEvent` rows) is stored for the 2025-26 season only, for its regular season and its postseason.**

- 2023-24 and 2024-25 stay as box scores only.
- Any further older data is loaded without play-by-play. For now that means the 2023-24 and 2024-25 postseasons (see [Loading older postseasons](#loading-older-postseasons)).

Sanele H. made the decision on 2026-09-27, while planning to load the older postseasons, after the production database was measured at 392 MB.

## Alternatives considered

### Store play-by-play for every season

About 990 MB, twice the limit. Rejected.

### Also store play-by-play for the older postseasons

About 440 MB. It fits, but it would leave only 60 MB for everything else that grows, and nothing towards the next season. The only thing it would add is the ability to correct plays in two past postseasons, which nobody has asked for. Rejected.

### Move to a paid Supabase plan

A paid plan would remove the limit. Rejected because the project's budget is effectively zero (constraint 5 in [ADR-003](adr-003-hosting-topology.md#context)), and the whole production system runs on free plans.

### Not evaluated

These were not formally compared. They are recorded here so it's clear they weren't considered, rather than implying they were:

- **Making each play smaller**, for example by dropping the `description` text or an index. The unique index on `(gameId, sequence)` must stay, because it is what makes re-ingesting a game overwrite its plays instead of duplicating them. Even halving the size of a play would add about 150 MB for one more regular season, which would still take the database past 500 MB.
- **Keeping older play-by-play outside the database**, for example in files or in a second free database. A correction changes the play, the affected statistics and the audit trail in one transaction. With the plays stored somewhere else, that wouldn't be possible.

## Consequences

### Play corrections

- **Corrections are available for 2025-26 games only**, in both the regular season and the postseason.
- **Older games are still listed in the admin Corrections tab**, because its season filter offers every season. Their *Plays* column shows 0, and opening one shows a message that there are no plays to correct. The message suggests pulling the game's play-by-play again. For an older season, don't: see [Scripts not to run against production](#scripts-not-to-run-against-production).
- **"Recalculate stats" leaves an older game unchanged.** It only re-derives players who appear in at least one play, so on a game with no plays it recomputes nobody and changes nothing (`planStatRecompute` in `apps/api/src/admin/plan-stat-recompute.ts`). It can't overwrite an older box score with zeros.

### Traceability

Statistics for 2023-24 and 2024-25 trace back to the NBA's box score for each game, not to individual plays. The brief's requirement that a statistic be traceable to its underlying events is therefore met for 2025-26 only. Nothing is typed in as a season total in any season: every figure still comes from a per-game record.

### Other features

- **[Player archetypes](../player-archetypes/index.md)** can only be fitted on 2025-26. The model needs the offensive/defensive rebound split and usage, which the older seasons don't have, so every older player would be excluded as incomplete data.
- **Offensive and defensive rebounds and plus-minus** show "—" for games in the older seasons.
- **The public play-by-play route**, `GET /v1/games/:id/events`, returns no events for older games.

### Loading older postseasons

`apps/ingestion/ingest_postseason.py` has `--season` and `--skip-play-storage` options (PR #197, merged on 2026-09-28). This is the route for loading the 2023-24 and 2024-25 postseasons within this decision:

```
python ingest_postseason.py --season 2023-24 --skip-play-storage
python ingest_postseason.py --season 2024-25 --skip-play-storage
```

With `--skip-play-storage`, each game's play-by-play is still downloaded and used to derive its box-score lines, but the plays themselves are not saved. That keeps each postseason to about 1.4 MB instead of about 24 MB. The cost is that these games can't be corrected, and the public `GET /v1/games/:id/events` route returns nothing for them. The script never writes rosters, so it is safe to run for a past season.

### Scripts not to run against production

Each of these would store a past season's play-by-play, against this decision:

- **`ingest.py --season` with an older season**, including a pull queued on the site's **Pull Data** form with an older season filled in, since the pull worker runs `ingest.py`. It has no option to skip play storage. Without a date range it saves the plays of each team's 15 most recent games and the whole postseason, about 120 MB. With a date range covering the season it saves about 320 MB. Either is more than the 108 MB left.
- **`ingest_historical_season.py` with an older season.** It saves the plays for every game it loads (it calls `run_ingestion_batch` from `play_by_play.py`): about 300 MB.
- **`ingest_postseason.py --season` without `--skip-play-storage`.** It saves the postseason's plays, about 22 MB. That would fit, but it goes against this decision and uses space the next season will need.

Before loading anything into production, check how much space is left (see [How to check the size](#how-to-check-the-size)).

### The next season

The 2026-27 season won't fit alongside 2025-26 with both seasons' play-by-play: about 392 MB plus about 320 MB is roughly 710 MB. Before 2026-27's play-by-play is loaded, the team has to choose one of the following, and this ADR should be revisited at that point:

- **Delete 2025-26's play-by-play.** Its box scores, statistics and correction history stay, but its games can no longer be corrected. PostgreSQL doesn't shrink the database as soon as rows are deleted. `VACUUM` makes the space reusable, and only `VACUUM FULL`, which locks the table while it runs, gives it back.
- **Load 2026-27 without play-by-play.** The current season could then not be corrected, which is the opposite of today.
- **Move to a paid plan.**

## How to check the size

Run these with a read-only connection. The Supabase dashboard also shows the database size.

```sql
-- The whole database, as counted against the free plan's limit
SELECT pg_size_pretty(pg_database_size(current_database()));

-- The play-by-play table, including its indexes
SELECT pg_size_pretty(pg_total_relation_size('"GameEvent"'));

-- Plays stored per season
SELECT g.season, count(*) AS plays
FROM "GameEvent" e
JOIN "Game" g ON g.id = e."gameId"
GROUP BY g.season
ORDER BY g.season;
```

## Sources

- Measurements of the production database, taken with a read-only connection on 2026-09-25 and 2026-09-27.
- The team-profiles feasibility study (branch `team-profiles-feasibility`, 2026-09-25), for the counts of older player rows.
- In the source repository: `apps/api/src/admin/admin-events.service.ts`, `apps/api/src/admin/plan-stat-recompute.ts`, `apps/web/src/components/admin/CorrectionGamePicker.tsx`, `apps/web/src/components/admin/GamePlayByPlayPanel.tsx`, `apps/ingestion/ingest.py`, `apps/ingestion/ingest_postseason.py`, `apps/ingestion/ingest_historical_season.py`, `apps/ingestion/play_by_play.py`, `apps/ingestion/backfill_advanced_stats.py` and `apps/similarity/player_seasons.py`.
- [Supabase: Database size](https://supabase.com/docs/guides/platform/database-size).
- [ADR-001: Database](adr-001-database.md), [ADR-003: Hosting Topology](adr-003-hosting-topology.md) and the [ERD](../design/erd.md#nba-data-and-models).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
