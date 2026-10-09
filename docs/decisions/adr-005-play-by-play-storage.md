# ADR-005: Play-by-play storage

**Status:** Accepted on 27 Sep, decided by Sanele H.

## Decision

Store **play-by-play (`GameEvent` rows) for the 2025-26 season only**, regular season and postseason. The older seasons in the database, 2023-24 and 2024-25, stay as **box scores only**, and any older data loaded later is loaded without plays.

## Context

**Play-by-play** is the NBA's record of every action in a game, one `GameEvent` row per shot, rebound, foul and so on. A **box score** is one line of totals per player per game (`PlayerGameStat`).

The brief asks for every statistic to be derived from individual events and traceable back to them ([Feature Tiers](../design/feature-tiers.md)). Play-by-play is that record. For a game that has it, the box-score lines are derived from its plays, each play records the ingestion run that wrote it, and an admin can correct any play: one transaction updates the play, re-derives the affected players' lines, marks that season's dataset releases stale, and records who changed what and why.

**The limit is space.** Production is on Supabase's free plan, capped at **500 MB** ([ADR-003](adr-003-hosting-topology.md)). Measured read-only on production on 27 Sep:

| Measure | Value |
|---|---|
| Database size | 392 MB |
| `GameEvent` with its indexes (2025-26 only) | 323 MB, about 82% of the database |
| 2025-26 plays | about 625,000, about 517 bytes each |
| One postseason (about 90 games) | about 24 MB with plays; about 1.4 MB as box scores only |

| What would be loaded | Estimated size | Fits in 500 MB? |
|---|---|---|
| Nothing more (this decision) | 392 MB | Yes, 108 MB spare |
| Older postseasons, box scores only | about 395 MB | Yes |
| Older postseasons, with plays | about 440 MB | Only just |
| Plays for one older regular season | about 690 MB | No |
| Plays for both older regular seasons | about 990 MB | No |

The spare space isn't only for plays: API usage logs, dataset release CSVs and user data all grow. And a free-plan database over its limit becomes **read-only** ([Supabase](https://supabase.com/docs/guides/platform/database-size)), so every write fails, including sign-in, which writes a session row. Running out of space would break most of the site.

The two older seasons were loaded as box scores to give the Elo model a history of results. They have no plays, and so no offensive/defensive rebound split or plus-minus either.

## Alternatives considered

| Option | Why not |
|---|---|
| **Plays for every season** | About 990 MB, twice the limit. |
| **Plays for the older postseasons too** | Fits, but leaves 60 MB for everything else, only to allow correcting old playoff games, which nobody has asked for. |
| **A paid Supabase plan** | The project's budget is effectively zero ([ADR-003](adr-003-hosting-topology.md#context)). |
| **Smaller plays, or plays stored outside the database** | Not evaluated. Halving a play's size would still not fit another regular season, and plays stored elsewhere couldn't be corrected in the same transaction as their statistics. |

## Consequences

- **Corrections are possible for 2025-26 games only.** Older games still appear in the admin Corrections tab, but show 0 plays and nothing to correct. "Recalculate stats" leaves them unchanged.
- **Traceability to events holds for 2025-26 only.** Older statistics trace back to each game's NBA box score. No season has typed-in season totals: every figure comes from a per-game record.
- **Older games** show "—" for offensive and defensive rebounds and plus-minus, and `GET /v1/games/:id/events` returns nothing for them.
- **[Player Archetypes](../player-archetypes/index.md)** can only be fitted on 2025-26, because it needs the rebound split and usage.

### Loading older postseasons

`ingest_postseason.py --season <season> --skip-play-storage` (PR #197) downloads each game's plays to derive its box scores, then doesn't save them, so a postseason costs about 1.4 MB instead of 24 MB.

Since PR #200 (8 Oct), a pull queued on the site's **Pull Data** form also runs with `--skip-play-storage`, for any season. It derives the stats but saves no plays, so new 2025-26 games pulled this way can't be corrected either.

**Don't run these against production for an older season:** `ingest.py --season` run by hand without `--skip-play-storage`, `ingest_historical_season.py`, or `ingest_postseason.py` without `--skip-play-storage`. Each saves the plays, and the first two would need more than the 108 MB left.

### The next season

2026-27's plays won't fit alongside 2025-26's (about 710 MB). Before loading them, the team must choose, and revisit this ADR:

- **Delete 2025-26's plays.** Its statistics and correction history stay, but its games can no longer be corrected. Postgres only returns the space after `VACUUM FULL`, which locks the table.
- **Load 2026-27 without plays,** so the current season can't be corrected.
- **Move to a paid plan.**

To check the size, run `SELECT pg_size_pretty(pg_database_size(current_database()));` on a read-only connection, or look at the Supabase dashboard.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
