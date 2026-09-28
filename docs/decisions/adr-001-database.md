# ADR-001: Database

- **Status:** Accepted. In use since the project was first set up on 2026-08-06.
- **Last updated:** 2026-09-28

!!! info "Updated for Sprint 3 (15–27 September 2026)"
    The table groups, enums and schema change history below were checked against `schema.prisma` and the migration files on 2026-09-28. Sprint 3 added design rules 12 to 16. The four player archetype tables are on branch `player-archetypes`, which is in review; everything else described here is on `main`.

## Summary

The platform stores all of its data in a single **PostgreSQL** database. The structure of that database (its *schema*) is defined in one file and changed only through **Prisma** migrations.

This document explains why that choice was made, what else was considered, how the data is organised, and the design rules the schema follows. Where the database runs in production is covered separately in [ADR-003: Hosting Topology](adr-003-hosting-topology.md).

### Terms used in this document

| Term | Meaning here |
|---|---|
| Schema | The definition of every table, column and relationship. It lives in `apps/api/prisma/schema.prisma` in the source repository. |
| Prisma | An ORM (object-relational mapper) for TypeScript. It generates typed database code from the schema and manages migrations. |
| Migration | A dated SQL file that changes the schema in one step, such as adding a column. Migrations are applied in order, so any empty database can be brought up to the current schema by running them. |
| Ingestion | The project's Python scripts (`apps/ingestion`) that download NBA data from stats.nba.com and write it to the database. |
| Predictor and optimizer | Two more Python scripts. The predictor (`apps/predictor`) estimates the outcome of games; the optimizer (`apps/optimizer`) suggests fantasy-basketball lineups. Both write their results to the database. |
| Upsert | Insert a row if it doesn't exist yet, otherwise update the existing row. |

## Decision

The platform uses PostgreSQL as its only database, with Prisma managing the schema.

- The **API** (`apps/api`, built with NestJS) is the only part of the system that users reach. It reads and writes the database through Prisma.
- The **Python scripts** (ingestion, predictor and optimizer) write to the same database directly, following the schema that Prisma defines.
- The database is never exposed to browsers or other outside clients. Every request for data goes through routes written by hand in the API.

## Context

Several properties of the project shaped this decision.

- **The data is relational.** Players belong to teams, each game involves two teams, and each line of statistics belongs to one player in one game. A relational database models these links with foreign keys and joins them efficiently. A document database would need the same information copied into several places.
- **Statistics are calculated, not just stored.** Season averages and advanced figures such as true shooting percentage are computed by combining many per-game rows. Joining, filtering and aggregating rows is exactly what SQL databases are designed for.
- **The brief asks for statistics derived from underlying records** (§2.4), rather than totals entered by hand. A relational schema makes it straightforward to keep the raw per-game records and calculate totals from them.
- **The brief bans backends that generate an API automatically** (§2.1), naming Firebase and Supabase as examples. The database therefore had to act purely as storage behind the project's own API.
- **Two programming languages write to the database.** The API is written in TypeScript and the data scripts in Python. If both could change the table structure, the two would drift apart. Making Prisma migrations the single owner of the schema prevents that.
- **Prisma suits the rest of the technology stack.** It generates TypeScript types from the schema, so the API's code is checked against the real tables when it is compiled. The authentication library, BetterAuth ([ADR-002](adr-002-auth.md)), provides a Prisma adapter, so user accounts live in the same database and the same migration history as everything else.
- **Schema changes can be reviewed.** Each migration is a SQL file committed to the repository, so it goes through the same pull-request review as any other code ([Git Methodology](../git-methodology.md)).

## Alternatives considered

This section is reconstructed from recorded team meetings and AI-assistant transcripts kept in the source repository (`docs/transcripts/`). Where those records don't explain why an option was rejected, this is stated rather than guessed.

### Firebase (Firestore)

Firebase was part of the first rough technology list the team agreed on 2026-08-13 (React, Node.js, Firebase). It was dropped on 2026-08-14 for two reasons:

1. The brief names Firebase as a banned system, because it generates API endpoints automatically (§2.1). Using it would have meant relying on a narrow reading of that rule.
2. Firestore stores documents rather than related tables. This project's data is highly connected, so statistics that span players, teams and games would have needed duplicated data kept in sync by hand.

### Drizzle ORM

When the team settled the technology stack, Drizzle was listed as the alternative to Prisma ("PostgreSQL + Prisma (or Drizzle)"). The records don't include a detailed comparison. Prisma was kept because:

- the project's starting codebase already used it;
- BetterAuth provides a ready-made Prisma adapter;
- Prisma's migration tool produces reviewable SQL files without extra tooling.

### Where to host the database

The hosting providers considered (Azure Database for PostgreSQL, Neon, Railway and Supabase) are compared in [ADR-003](adr-003-hosting-topology.md#database-hosting). All of them run standard PostgreSQL, so the choice of host doesn't affect this decision.

## How the data is organised

On `main`, the schema contains **35 tables** (Prisma calls them *models*) and **7 enums** (fixed lists of allowed values). Branch `player-archetypes`, in review, adds 4 more tables. The [ERD](../design/erd.md) lists every column.

The tables fall into nine groups. Most groups have a single part of the system that writes to them, which keeps responsibility for each kind of data clear. Where a group has more than one writer, the table says so, and the exceptions are deliberate.

| Group | What it holds | Tables | Written by |
|---|---|---|---|
| NBA data | Teams, players, games, play-by-play events and per-game player statistics | `Team`, `Player`, `Game`, `GameEvent`, `PlayerGameStat` | Ingestion scripts; also admin corrections through the API (rule 12) |
| Ingestion and review | A record of every ingestion run and every correction, queued data pulls, pull workers and the automatic-pull schedule | `IngestionBatch`, `EventCorrection`, `IngestionRequest`, `IngestionWorker`, `IngestionSchedule` | Ingestion scripts and the pull worker; admins through the API |
| Game predictions | Each game's predicted winner and score margin, a history of past predictions, and the betting market's view | `GamePrediction`, `GamePredictionRun`, `GameMarketOdds` | Predictor; the market-odds script writes `GameMarketOdds` |
| Fantasy lineups | Predicted fantasy points for each player, and suggested lineups | `PlayerPrediction`, `Lineup`, `LineupSlot` | Optimizer only |
| Player archetypes (in review) | Playing-style groups, each player's place among them, and similar players | `Archetype`, `PlayerArchetype`, `PlayerArchetypeMembership`, `PlayerSimilarity` | Similarity script only |
| Accounts | Users, sign-in sessions and linked Google accounts | `User`, `Session`, `Account`, `Verification` | BetterAuth (the authentication library) |
| Personal data | Players a user follows, the games they have called, and comparisons and lineups they saved | `UserFollowedPlayer`, `GamePick`, `SavedComparison`, `SavedComparisonPlayer`, `SavedLineup`, `SavedLineupSlot` | The API, when a signed-in user saves something |
| Publishing and API access | Dataset releases, analyst-defined statistics, and API consumers with their keys and usage | `DatasetRelease`, `CustomStatistic`, `ApiConsumer`, `ApiKey`, `ApiUsageLog` | The API |
| Become Pro | A user's own seasons, games and projected value, and the trained valuation model | `ProspectSeason`, `ProspectGame`, `ProspectValuation`, `ProspectValuationModel` | The API; the valuation script writes the model |

The API reads from every group. It never writes the models' output: predictions, lineups, archetypes or the valuation model. Its only writes to NBA data are admin corrections (rule 12).

| Enum | Allowed values | Used for |
|---|---|---|
| `Role` | `PUBLIC`, `USER`, `ANALYST`, `ADMIN` | A user's permission level |
| `SeasonType` | `REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS` | Which part of the season a game belongs to |
| `PickOutcome` | `CORRECT`, `MISSED` | Whether a user's call on a game was right |
| `IngestionBatchStatus` | `RUNNING`, `COMPLETED`, `FAILED`, `PENDING_REVIEW`, `REJECTED` | Where an ingestion run stands, including review |
| `IngestionRequestStatus` | `QUEUED`, `RUNNING`, `SUCCEEDED`, `FAILED`, `CANCELLED` | Where a queued data pull stands |
| `IngestionFrequency` | `NEVER`, `HOURLY`, `DAILY`, `WEEKLY` | How often data is pulled automatically |
| `CompetitionLevel` | `NCAA_D1`, `NCAA_D2`, `NCAA_D3`, `NAIA`, `JUCO`, `INTERNATIONAL_PRO`, `SEMI_PRO`, `HIGH_SCHOOL`, `REC` | Where a Become Pro season was played |

## Design rules in the schema

These rules are built into the database structure itself, so application code can't accidentally break them.

### 1. Internal IDs are separate from NBA IDs

Every row has its own generated ID (a UUID). The NBA's own IDs for teams, players and games are stored in separate columns, each of which must be unique. Ingestion uses the NBA ID to decide whether a row already exists, so running it again updates existing rows instead of creating duplicates. Links between tables use the internal IDs, so they don't depend on how the NBA formats its IDs.

### 2. Figures that can be calculated are not stored

Season averages, true shooting percentage, effective field-goal percentage and assist-to-turnover ratio are calculated when they are requested, from the per-game rows in `PlayerGameStat`. The calculations were checked against the NBA's own published figures and matched to three decimal places. Storing them as well would create two copies of the same number that could disagree.

Plus/minus, usage rate and offensive and defensive ratings **are** stored, exactly as the NBA publishes them. They depend on information this database doesn't hold, such as possessions and which players were on court, so calculating them locally would produce a different, invented statistic.

### 3. An empty value means "not recorded", not zero

The advanced statistics columns on `PlayerGameStat` (plus/minus, usage rate, the two ratings and the offensive/defensive rebound split) allow empty (`NULL`) values. Games loaded before these columns were added have no data for them. Filling them with `0` would be wrong: a plus/minus of 0 means the scores were level while the player was on court, not that nobody measured it. The website shows "—" for empty values.

### 4. Regular-season and playoff figures can't mix

Every game has a `seasonType`: regular season, play-in, playoffs or Finals. Every statistics query filters on it, so a playoff view can't include regular-season numbers. Because the rule lives in the database query rather than in the page that displays the results, no individual screen can forget to apply it.

The Finals have their own value, rather than being recorded as "playoffs, round 4", because "a player's Finals games" is a common request and is simpler to query directly. The playoff round is also stored, for views broken down by round.

### 5. A player's team is recorded for each game

Players get traded, so the team a player is on now isn't necessarily the team they played for in an older game. `PlayerGameStat.teamId` records the team for that specific game.

Before this column was added on 2026-09-09, team statistics looked up each player's *current* team instead. A check at the time found that this credited about 7.7% of per-game rows to the wrong team.

### 6. Saved records keep a copy of what the user saw

Model output changes over time. The predictor overwrites each game's prediction every time it runs, and the optimizer regularly produces new player projections. When a user calls a game or saves a lineup, the numbers they were looking at are therefore copied into their saved record:

- `GamePick` keeps the model's win probability, its predicted margin and both teams' Elo ratings (a strength score for each team) at the moment the user made the call.
- `SavedLineup` and `SavedLineupSlot` keep the salaries and predicted points at the moment the lineup was saved.

Without these copies, a user's saved call or lineup would quietly change whenever the models were re-run.

### 7. Past predictions are kept when the model changes

`GamePrediction` holds one current prediction per game, which is what the website shows. `GamePredictionRun` keeps every prediction made for each game, labelled with the version of the model that produced it. When the prediction model is improved, older predictions stay available exactly as they were first published.

### 8. What happens when a row is deleted

Each relationship has a deletion rule that matches what the data means:

| When this is deleted | What happens to related rows | Why |
|---|---|---|
| A user | All of that user's sessions, follows, calls and saved items are deleted too | The brief requires users to be able to delete their accounts (§2.1). Removing everything in one step leaves no personal data behind. |
| A player or game | Users' follows, calls and saved items that refer to it are deleted too | Otherwise a single saved item could stop the ingestion scripts from replacing out-of-date NBA data. |
| A team | Users who chose it as their favourite team are left with no favourite | Having a favourite team is optional. |
| A player or game that has statistics or predictions | The deletion is refused | Statistics and predictions must not lose the records they were calculated from. |
| An admin's account | Their batch reviews, corrections, dataset releases and queued pulls are kept; only the link to the admin is cleared | The history of changes to the data must outlive the person who made them (rule 12). |

### 9. Each fact is stored in one place

For a short time, the schema had two separate ways of recording which players a user follows. The sign-up flow wrote to one, while the home page read from the other, so users who followed players during sign-up saw an empty watchlist on the home page.

The duplicate tables were removed on 2026-09-12, leaving one table for followed players and one column for a user's favourite team. A private note per followed player, which only the removed table supported, was dropped deliberately.

### 10. Indexes match the queries the website runs

An index lets the database find matching rows without scanning a whole table. On 2026-09-13, indexes were added for the queries the website runs most often:

| Index | Speeds up |
|---|---|
| `Game(gameDate)` | Lists of upcoming and recent games, and any statistics shown in date order |
| `Game(homeTeamId, gameDate)` and `Game(awayTeamId, gameDate)` | A team's schedule and results. The database combines the two to find games where the team played either at home or away. |
| `PlayerGameStat(gameId)` | A single game's box score |
| `Player(teamId)` | A team's roster |

Earlier indexes cover each game's season type, the order of events within a game, each player's latest projection, and each user's saved items in date order.

### 11. User roles and profile pictures are protected

- A user's `role` (their permission level) defaults to a normal user. The authentication library is configured so that neither signing up nor a Google profile update can change it; only a direct change to the database can.
- Profile pictures are kept in a private file-storage bucket, not in the database. The database stores only each file's location. When a profile is viewed, the API creates a temporary link to the picture, so no permanent public link to it ever exists ([ADR-003](adr-003-hosting-topology.md#profile-picture-storage)).

### 12. Statistics trace back to their submission, and corrections leave a history

Each ingestion run for a game is an `IngestionBatch` row, and each play records the run that wrote it. Runs are added rather than overwritten, so past rejections stay visible.

An admin correction is the only way the API changes NBA data, and the one deliberate exception to "each group has one writer". In a single transaction, a correction:

1. updates the play;
2. re-derives the statistics of only the players the play affects;
3. marks the season's dataset releases as stale;
4. records the old and new values, who made the change and why in `EventCorrection`.

Corrections are only ever added. An undo is a new correction that restores the old values, linked to the one it reverts, so the history is never lost.

### 13. Nothing is published before it is reviewed

A game is left out of every public read while any of its ingestion batches that hasn't been removed is pending review, running, failed or rejected. As with rule 4, the rule lives in the database query (`PUBLISHED_GAME_FILTER`) rather than in each page, so no screen can forget to apply it.

### 14. Published releases never change

A dataset release stores its CSV and checksum when it is published. A later correction marks the release as stale instead of rewriting it, so a download always matches what was released. Before the CSV was stored (18 September 2026), releases were rebuilt from live data when downloaded; a stale release of that kind refuses to download rather than serve corrected data under its old version.

### 15. Keys are stored only as hashes

An API key is shown once, when it is created. Only its SHA-256 hash is stored, so a copy of the database contains no usable keys.

### 16. Storage has a budget

The production database runs on Supabase's free plan, which is limited to 500 MB, and one season of play-by-play takes about 320 MB of it. Play-by-play is therefore stored for 2025-26 only, and older seasons are kept as box scores ([ADR-005](adr-005-play-by-play-storage.md)). Tables that grow without limit, such as `ApiUsageLog`, count against the same budget.

## Schema change history

All migrations are in `apps/api/prisma/migrations/` and are applied in date order. There are 25 on `main`; branch `player-archetypes` adds a 26th.

| Date | Migration | Change |
|---|---|---|
| 2026-08-06 | `init` | First tables: teams, players, games, play-by-play events, per-game statistics and users |
| 2026-08-06 | `betterauth_schema` | Moved sign-in to BetterAuth: removed the starter code's password column and added sessions, linked accounts and verification tokens |
| 2026-08-13 | `lineup_optimizer` | Tables for player projections and suggested fantasy lineups |
| 2026-08-18 | `game_prediction` | Table for each game's predicted winner and margin |
| 2026-08-20 | `player_bio_fields` | Nine optional biography columns for players, such as birth date, school, country and draft details |
| 2026-09-07 | `add_game_season_type` | Season type and playoff round for games (rule 4). All existing games were regular-season games, so they correctly default to regular season. |
| 2026-09-07 | `add_advanced_player_game_stats` | Optional advanced statistics on per-game rows (rules 2 and 3) |
| 2026-09-09 | `add_player_game_stat_team_id` | Team recorded for each game (rule 5) |
| 2026-09-10 | `home_personalization` | Game calls, saved comparisons and saved lineups, plus two follow tables that were later removed (rule 9) |
| 2026-09-11 | `add_user_personalization` | Username, profile picture and favourite team for users, and the followed-players table |
| 2026-09-12 | `drop_superseded_follow_tables` | Removed the duplicate follow tables (rule 9) |
| 2026-09-13 | `add_query_indexes` | Indexes for common queries (rule 10) |
| 2026-09-13 | `game_prediction_versioning` | Model version on predictions, and the prediction history table (rule 7) |
| 2026-09-15 | `add_game_market_odds` | Table for the betting market's pre-game win probability, from The Odds API |
| 2026-09-16 | `derive_stats_from_game_events` | Real play-by-play: five new `GameEvent` columns, a unique key on game and sequence, and the ingestion-run table `IngestionBatch` (rule 12). Deleted the placeholder rows that were all `GameEvent` had held until then. |
| 2026-09-16 | `intermediate_brief_features` | Review states for ingestion runs (rule 13), the corrections history `EventCorrection` (rule 12), API consumers, keys and usage log (rule 15), and dataset releases (rule 14) |
| 2026-09-16 | `ingestion_schedule_and_batch_delete` | The automatic-pull schedule, and soft delete for ingestion runs |
| 2026-09-16 | `mark_stale_dataset_releases` | A stale flag on dataset releases (rule 14) |
| 2026-09-16 | `add_ingestion_resume_checkpoint` | A checkpoint on ingestion runs, so a failed run resumes instead of restarting |
| 2026-09-16 | `add_custom_statistics` | Analyst-defined statistics |
| 2026-09-17 | `add_user_api_keys` | A link from API consumers to users, so signed-in users can create their own keys |
| 2026-09-18 | `store_dataset_release_csv` | Each release's CSV stored when it is published (rule 14) |
| 2026-09-18 | `add_ingestion_request_queue` | The queue of data pulls and the pull-worker table ([ADR-003](adr-003-hosting-topology.md)) |
| 2026-09-19 | `add_event_correction_revert_link` | A link from an undo to the correction it reverts (rule 12) |
| 2026-09-22 | `add_player_archetypes` | *In review, on branch `player-archetypes`.* The four player archetype tables ([Player Archetypes](../player-archetypes/index.md)) |
| 2026-09-23 | `add_become_pro` | The four Become Pro tables ([Become Pro](../become-pro/index.md)) |

## Consequences

### How a schema change reaches production

1. A developer edits `schema.prisma`.
2. Prisma generates a migration file from the change (`npx prisma migrate dev`).
3. The migration is reviewed with the rest of the pull request.
4. Once merged, the production server applies it automatically the next time the API starts ([ADR-003](adr-003-hosting-topology.md#schema-changes)).

### Benefits

- Every change to the database has a reviewed, dated file, so an empty database can be rebuilt to the current structure at any time. The automated tests do exactly that on every run.
- The API's code is checked against the real table structure when it is compiled, so a renamed or removed column is caught before release.
- Because each group of tables has a single writer, the data scripts and the website's user features can change independently of each other.

### Drawbacks

- **The Python scripts aren't checked automatically.** Prisma's type checking covers only the TypeScript API. If a migration renames a column the Python scripts use, the failure only appears when a script runs, so these changes must be checked by hand.
- **Applied migrations can't be edited.** Prisma records a fingerprint of each migration it applies and refuses to continue if one has changed. Mistakes must be corrected with a new migration, as the 2026-09-12 migration did.
- **Deploying code doesn't load data.** A new column reaches production automatically, but it stays empty until someone runs the matching data script ([ADR-003](adr-003-hosting-topology.md#loading-production-data)).
- **The sample-data script erases real data.** `prisma/seed.ts` deletes all games and statistics before inserting sample data, so it must never be run against the production database.
- **The free plan's size limit decides what can be stored.** With one season of play-by-play taking about 320 MB of the 500 MB limit, the older seasons are box scores only and can't be corrected, and a decision is needed before the next season's play-by-play is loaded ([ADR-005](adr-005-play-by-play-storage.md)).
- **Generated migrations need reading before they are committed.** The schema file and the database differ in two small, harmless ways (listed on the [ERD](../design/erd.md#known-differences-between-the-schema-and-the-database)), and `prisma migrate dev` adds changes to "fix" both to the next migration it generates.

## Sources

- `apps/api/prisma/schema.prisma` and `apps/api/prisma/migrations/` in the source repository.
- Transcripts in the source repository's `docs/transcripts/` folder: `2026-08-14-adrian-claude-doc-website.txt` (Firebase) and `OwenPace_01.txt` (Prisma or Drizzle).
- [ADR-002: Auth](adr-002-auth.md), [ADR-003: Hosting Topology](adr-003-hosting-topology.md), [ADR-005: Play-by-play storage](adr-005-play-by-play-storage.md) and the [ERD](../design/erd.md).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5] (2026-09-23: flagged the table/enum counts and migration history as stale, not rewritten in full), Claude-Code[Claude Opus 5.5] (2026-09-28: updated for Sprint 3)*
