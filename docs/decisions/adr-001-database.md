# ADR-001: Database

**Status:** Accepted, in use since 2026-08-06.

## Decision

All data lives in one **PostgreSQL** database. Its schema is defined in `apps/api/prisma/schema.prisma` and changed only through **Prisma** migrations.

- The **API** (NestJS) is the only part users reach. It reads and writes through Prisma.
- The **Python jobs** (ingestion, predictor, optimizer, valuation) write to the same database directly, following the schema Prisma owns.
- The database is never exposed to browsers. Every read goes through a hand-written API route.

Where the database runs is [ADR-003](adr-003-hosting-topology.md). Every table and column is on the [ERD](../design/erd.md).

## Context

- **The data is relational.** Players belong to teams, a game has two teams, and each stat line belongs to one player in one game. Foreign keys and joins model this; a document store would duplicate it.
- **Statistics are calculated, not entered.** The brief asks for stats derived from underlying records (§2.4). Season averages and shooting percentages are aggregates over per-game rows, which is what SQL is for.
- **Auto-generated backends are banned** (§2.1, naming Firebase and Supabase). The database had to be plain storage behind our own API.
- **Two languages write to it.** If both TypeScript and Python could change the tables, they would drift. Making Prisma migrations the only owner of the schema prevents that.
- **Prisma fits the stack.** It generates TypeScript types from the schema, so API code is checked against the real tables at compile time, and BetterAuth ([ADR-002](adr-002-auth.md)) has a Prisma adapter, so accounts share the same database and migration history.
- **Schema changes get reviewed.** Each migration is a SQL file that goes through the same pull-request review as code.

## Alternatives considered

| Option | Why not |
|---|---|
| **Firebase (Firestore)** | On the first technology list (13 Aug), dropped on 14 Aug. The brief bans it (§2.1), and its documents would have duplicated our highly connected data. |
| **Drizzle ORM** | Listed as the alternative ("PostgreSQL + Prisma (or Drizzle)"). Prisma was kept because the starter code used it, BetterAuth has a Prisma adapter, and its migrations are reviewable SQL with no extra tooling. |
| **Other Postgres hosts** | Azure, Neon, Railway and Supabase all run standard Postgres, so the host doesn't affect this decision. Compared in [ADR-003](adr-003-hosting-topology.md#alternatives-considered). |

## Design rules in the schema

These rules are built into the schema or the shared queries, so no single page or script can break them.

| # | Rule | How |
|---|---|---|
| 1 | Internal IDs are separate from NBA IDs | Every row has a UUID. NBA IDs sit in their own unique columns, so re-running ingestion updates rows instead of duplicating them. |
| 2 | Figures that can be calculated are not stored | Averages, TS%, eFG% and AST/TO are computed from `PlayerGameStat` on request, and match the NBA's published figures to three decimal places. Plus/minus, usage and ratings **are** stored as published, because they need data we don't hold. |
| 3 | Empty means "not recorded", not zero | Advanced stat columns are nullable. A plus/minus of 0 means something, so missing values stay `NULL` and the site shows "—". |
| 4 | Regular season and playoffs never mix | Every game has a `seasonType` (regular, play-in, playoffs, Finals) and every stats query filters on it. |
| 5 | A player's team is recorded per game | `PlayerGameStat.teamId`, added 9 Sep. Before it, team stats used each player's current team, which credited about 7.7% of rows to the wrong team. |
| 6 | Saved records keep what the user saw | `GamePick` copies the win probability, margin and both Elo ratings; `SavedLineup` copies salaries and projections. Re-running a model doesn't change a saved call or lineup. |
| 7 | Old predictions survive a model change | `GamePrediction` holds the current prediction; `GamePredictionRun` keeps every run, labelled with its model version. |
| 8 | Deletes match what the data means | Deleting a user deletes all their data (§2.1). Deleting a player or game removes saved items that point at it, but is refused if stats or predictions depend on it. Deleting an admin keeps their corrections and releases. Full table on the [ERD](../design/erd.md#what-happens-on-delete). |
| 9 | Each fact is stored once | Two follow tables briefly coexisted, so sign-up and Home disagreed. The duplicates were dropped on 12 Sep. |
| 10 | Indexes match the queries | Added 13 Sep: `Game(gameDate)`, `Game(homeTeamId, gameDate)`, `Game(awayTeamId, gameDate)`, `PlayerGameStat(gameId)`, `Player(teamId)`. See [Performance](../design/performance.md). |
| 11 | Roles and profile pictures are protected | Sign-up and Google profile updates can't change `role`. Pictures sit in a private bucket and are served through short-lived signed links. |
| 12 | Corrections leave a history | Each ingestion run is an `IngestionBatch`. An admin correction updates the play, re-derives only the affected players' stats, marks releases stale and records old and new values in `EventCorrection`, in one transaction. An undo is a new correction, linked to the one it reverts. |
| 13 | Nothing is published before review | A game is hidden from every public read while any of its batches is pending, running, failed or rejected (`PUBLISHED_GAME_FILTER`). |
| 14 | Published releases never change | A release stores its CSV and checksum. A later correction marks it stale rather than rewriting it. |
| 15 | Keys are stored only as hashes | An API key is shown once; only its SHA-256 hash is stored. |
| 16 | Storage has a budget | Supabase's free plan allows 500 MB and one season of play-by-play takes about 320 MB, so only 2025-26 keeps play-by-play ([ADR-005](adr-005-play-by-play-storage.md)). |

## Schema change history

All 26 migrations are in `apps/api/prisma/migrations/`, applied in date order.

| Date | Migration | Change |
|---|---|---|
| 6 Aug | `init` | Teams, players, games, events, per-game stats and users |
| 6 Aug | `betterauth_schema` | BetterAuth sessions, accounts and verification tokens; the starter's password column removed |
| 13 Aug | `lineup_optimizer` | Player projections and lineups |
| 18 Aug | `game_prediction` | Game predictions |
| 20 Aug | `player_bio_fields` | Nine player biography columns |
| 7 Sep | `add_game_season_type` | Season type and playoff round (rule 4) |
| 7 Sep | `add_advanced_player_game_stats` | Nullable advanced stats (rules 2 and 3) |
| 9 Sep | `add_player_game_stat_team_id` | Team per game (rule 5) |
| 10 Sep | `home_personalization` | Game picks, saved comparisons and lineups |
| 11 Sep | `add_user_personalization` | Username, picture, favourite team, followed players |
| 12 Sep | `drop_superseded_follow_tables` | Duplicate follow tables removed (rule 9) |
| 13 Sep | `add_query_indexes` | Indexes (rule 10) |
| 13 Sep | `game_prediction_versioning` | Prediction history (rule 7) |
| 15 Sep | `add_game_market_odds` | Betting-market win probability |
| 16 Sep | `derive_stats_from_game_events` | Real play-by-play columns and `IngestionBatch` (rule 12) |
| 16 Sep | `intermediate_brief_features` | Review states, `EventCorrection`, API keys and dataset releases (rules 12–15) |
| 16 Sep | `ingestion_schedule_and_batch_delete` | Pull schedule and soft-deleted batches |
| 16 Sep | `mark_stale_dataset_releases` | Stale flag on releases (rule 14) |
| 16 Sep | `add_ingestion_resume_checkpoint` | Failed runs resume instead of restarting |
| 16 Sep | `add_custom_statistics` | Analyst-defined statistics |
| 17 Sep | `add_user_api_keys` | Users can create their own keys |
| 18 Sep | `store_dataset_release_csv` | Releases store their CSV (rule 14) |
| 18 Sep | `add_ingestion_request_queue` | Queued pulls and the pull worker |
| 19 Sep | `add_event_correction_revert_link` | Undo links to the correction it reverts (rule 12) |
| 22 Sep | `add_player_archetypes` | The four [Player Archetypes](../player-archetypes/index.md) tables |
| 23 Sep | `add_become_pro` | The four [Become Pro](../become-pro/index.md) tables |

## Consequences

**How a change ships:** edit `schema.prisma`, generate a migration with `npx prisma migrate dev`, review it in the pull request, and the production API applies it on its next start ([ADR-003](adr-003-hosting-topology.md#database-deployment)).

**Benefits**

- An empty database can be rebuilt to the current schema at any time. The end-to-end tests do this on every run.
- A renamed or removed column breaks the API's build, not production.
- Each group of tables has one writer, so the Python jobs and the API's user features change independently.

**Drawbacks**

- **Python isn't type-checked against the schema.** A renamed column breaks a script only when it runs.
- **Applied migrations can't be edited.** Mistakes need a new migration.
- **Deploying doesn't load data.** A new column stays empty until its data job runs.
- **`prisma/seed.ts` erases real data,** so it must never run against production.
- **The 500 MB limit decides what is stored.** Older seasons are box scores only and can't be corrected.
- **Generated migrations must be read.** `prisma migrate dev` tries to "fix" two harmless schema differences ([ERD](../design/erd.md#known-differences-between-the-schema-and-the-database)).

## Sources

`schema.prisma` and the migrations in the app repo; the transcripts `2026-08-14-adrian-claude-doc-website.txt` (Firebase) and `OwenPace_01.txt` (Prisma or Drizzle).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
