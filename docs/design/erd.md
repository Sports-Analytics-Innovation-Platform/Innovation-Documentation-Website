# ERD

This page describes every table in the platform's PostgreSQL database: what each one holds, what its columns mean, and how the tables relate to each other. The reasons behind the design are explained in [ADR-001: Database](../decisions/adr-001-database.md), and where the database runs in [ADR-003: Hosting Topology](../decisions/adr-003-hosting-topology.md).

!!! success "Checked against the schema"
    Every table, column, constraint and index on this page was checked against `apps/api/prisma/schema.prisma` in the source repository, as of its most recent migration, `20260913140000_game_prediction_versioning` (13 September 2026).

!!! warning "Stale as of 2026-09-23 — schema has grown from 20 to 31 models"
    A large batch of work since 13 September added tables this page doesn't cover: `IngestionBatch` and `EventCorrection` (the submission-review/correction workflow — see the "still open" note below, which is now wrong), `ApiConsumer`/`ApiKey`/`ApiUsageLog` (external API key issuance and rate limiting), `DatasetRelease` (versioned, checksummed dataset snapshots), `CustomStatistic` (analyst-defined statistics), `IngestionRequest`/`IngestionWorker` (a queued ingestion job), and `GameMarketOdds` (the second external API integration). Not rewritten in full here given the size of this page — see `apps/api/prisma/schema.prisma` directly, or [ADR-001](../decisions/adr-001-database.md)'s matching currency note, or [Feature Tiers](feature-tiers.md) for what these tables back.

    Since then, Become Pro (PR #192) has added four more tables, bringing the schema to 35 models. Unlike the ones above, those four are documented on this page, under [Become Pro](#become-pro), checked against `schema.prisma` on 2026-09-27.

## Diagram

![Database ERD](diagrams/database-erd.svg)

Click the diagram to enlarge it. The diagram's source file is `docs/diagrams/database-erd.puml` in the source repository.

**How to read it**

- `*` marks a required column; `?` marks an optional one that may be empty (`NULL`).
- `PK` is a primary key and `FK` a foreign key (a link to a row in another table).
- Lines below a table's dotted divider list its unique constraints and indexes.
- On the connecting lines, a crow's foot means "many", a bar means "exactly one", and a circle means "zero" is allowed. For example, one team has zero or many players.

## Overview

This page documents **24 tables** and **4 enums** (fixed lists of allowed values), in six groups. Each group is written by exactly one part of the system, except Become Pro, where the API and the valuation script each write their own tables.

| Group | Tables | Written by |
|---|---|---|
| [NBA data](#nba-data) | `Team`, `Player`, `Game`, `GameEvent`, `PlayerGameStat` | The ingestion scripts (`apps/ingestion`), which download data from stats.nba.com |
| [Game predictions](#game-predictions) | `GamePrediction`, `GamePredictionRun` | The predictor script (`apps/predictor`) |
| [Fantasy lineups](#fantasy-lineups) | `PlayerPrediction`, `Lineup`, `LineupSlot` | The optimizer script (`apps/optimizer`) |
| [Accounts](#accounts) | `User`, `Session`, `Account`, `Verification` | BetterAuth, the authentication library |
| [Personal data](#personal-data) | `UserFollowedPlayer`, `GamePick`, `SavedComparison`, `SavedComparisonPlayer`, `SavedLineup`, `SavedLineupSlot` | The API, when a signed-in user saves something |
| [Become Pro](#become-pro) | `ProspectSeason`, `ProspectGame`, `ProspectValuation`, `ProspectValuationModel` | The API writes seasons, games and valuations when a user logs their own games; the valuation script (`apps/valuation`) writes the trained model |

The API reads every group, but never writes to the NBA data, game prediction or fantasy lineup tables, or to `ProspectValuationModel`.

Unless stated otherwise, every table's primary key is `id`, a generated UUID.

## NBA data

### Team

One row per NBA team.

| Column | Type | Notes |
|---|---|---|
| `nbaTeamId` | int, unique | The NBA's own team ID. Ingestion uses it to update existing teams instead of creating duplicates. |
| `name`, `abbreviation`, `city` | string | For example "Lakers", "LAL", "Los Angeles" |
| `conference`, `division` | string | |
| `logoUrl` | string, optional | Not currently filled in |

### Player

One row per player.

| Column | Type | Notes |
|---|---|---|
| `nbaPlayerId` | int, unique | The NBA's own player ID |
| `firstName`, `lastName`, `position` | string | |
| `heightInches`, `weightLbs` | int, optional | |
| `jerseyNumber`, `headshotUrl` | string, optional | |
| `teamId` | → `Team`, optional | The player's **current** team, from the latest roster. For the team a player played for in a past game, see `PlayerGameStat.teamId`. |
| `birthDate` | datetime, optional | Biography fields, all optional. They come from a separate NBA endpoint and are filled in by a backfill script. |
| `school`, `country`, `lastAffiliation`, `rosterStatus` | string, optional | |
| `seasonExp`, `draftYear`, `draftRound`, `draftNumber` | int, optional | `seasonExp` is years of NBA experience |

**Index:** `teamId`, for loading a team's roster.

### Game

One row per game.

| Column | Type | Notes |
|---|---|---|
| `nbaGameId` | string, unique | The NBA's own game ID |
| `gameDate` | datetime | |
| `season` | string | For example "2025-26" |
| `homeTeamId`, `awayTeamId` | → `Team` | |
| `homeScore`, `awayScore` | int, optional | Empty until the game has been played |
| `seasonType` | `SeasonType`, default `REGULAR` | Regular season, play-in, playoffs or Finals. Every statistics query filters on this, so figures from different parts of the season never mix. All games loaded before this column was added were regular-season games, so the default is correct for them. |
| `playoffRound` | int, optional | 1–4 for playoff and Finals games; empty otherwise |

**Indexes:** `seasonType`; `gameDate`; `(homeTeamId, gameDate)` and `(awayTeamId, gameDate)`, for a team's schedule in date order.

### GameEvent

Raw play-by-play: one row per event in a game, such as a shot or a foul. This is the underlying event record that the brief asks statistics to be traced back to.

| Column | Type | Notes |
|---|---|---|
| `gameId` | → `Game` | |
| `sequence`, `period` | int | The event's position in the game and the quarter it happened in |
| `clock` | string | Game clock at the time of the event |
| `eventType`, `description` | string | |
| `playerId` | string, optional | Stored as plain text, not a link to `Player` |
| `createdAt` | datetime | |

**Index:** `(gameId, sequence)`, for reading a game's events in order.

### PlayerGameStat

One player's box score for one game. Season averages, true shooting percentage, effective field-goal percentage and assist-to-turnover ratio are calculated from these rows each time they are requested (by the API's `/v1/players/:id/stats` route); there is no table of stored averages. Those calculations were checked against the NBA's own published figures and matched to three decimal places.

| Column | Type | Notes |
|---|---|---|
| `playerId` | → `Player` | |
| `gameId` | → `Game` | |
| `teamId` | → `Team`, optional | The team the player played for **in this game**, which can differ from their current team after a trade. Empty for some rows loaded before this column was added. |
| `minutes`, `points`, `rebounds`, `assists`, `steals`, `blocks`, `turnovers` | int | |
| `fieldGoalsMade`, `fieldGoalsAttempted` | int | |
| `threesMade`, `threesAttempted` | int | |
| `freeThrowsMade`, `freeThrowsAttempted` | int | |
| `offensiveRebounds`, `defensiveRebounds` | int, optional | |
| `plusMinus` | int, optional | Points scored minus points conceded while the player was on court |
| `usagePercentage` | float, optional | Share of the team's plays used by the player while on court |
| `offensiveRating`, `defensiveRating` | float, optional | Points produced and allowed per 100 possessions, as published by the NBA |

The optional statistics columns are empty for rows loaded before those columns existed, or where the NBA's data didn't include them. Empty means "not recorded", which is different from zero; the website shows "—".

**Unique:** `(playerId, gameId)`, so a player has at most one row per game. **Index:** `gameId`, for loading a game's box score.

## Game predictions

### GamePrediction

The current prediction for each game: one row per game, replaced each time the predictor runs.

| Column | Type | Notes |
|---|---|---|
| `gameId` | → `Game`, unique | |
| `homeWinProbability` | float | The home team's chance of winning (0 to 1), from the Elo rating model |
| `homeTeamEloPre`, `awayTeamEloPre` | float | Each team's Elo rating (a strength score) going into the game |
| `predictedMarginHome` | float, optional | Predicted home score minus away score, from the Four Factors model. Empty if either team has no completed games to learn from. |
| `marginMethod` | string, optional | `regression` (a fitted model) or `heuristic` (fixed weights, used when there is too little data) |
| `modelVersion` | string, default `unversioned` | Which version of the model produced this prediction |
| `createdAt` | datetime | |

### GamePredictionRun

A permanent history of predictions: one row per game per model version. Older predictions stay available after the model changes.

| Column | Type | Notes |
|---|---|---|
| `gameId` | → `Game` | |
| `modelVersion` | string | |
| `homeWinProbability`, `homeTeamEloPre`, `awayTeamEloPre` | float | As in `GamePrediction` |
| `predictedMarginHome`, `marginMethod` | optional | As in `GamePrediction` |
| `createdAt` | datetime | |

**Unique:** `(gameId, modelVersion)`. Re-running the same model version updates its row instead of adding another. **Index:** `gameId`.

## Fantasy lineups

### PlayerPrediction

A player's predicted fantasy points for their next game. Each optimizer run adds a new row per player, and the newest row is used.

| Column | Type | Notes |
|---|---|---|
| `playerId` | → `Player` | |
| `predictedFantasyPoints` | float | |
| `salary` | int | A made-up fantasy "cost" calculated from the prediction, not a real market price |
| `asOf` | datetime | When the prediction was made |

**Index:** `(playerId, asOf)`, for finding each player's latest prediction.

### Lineup

The best lineup found by one optimizer run: the 5 players with the highest total predicted points within a salary budget.

| Column | Type |
|---|---|
| `totalPredictedPoints` | float |
| `totalSalary`, `budget` | int |
| `createdAt` | datetime |

### LineupSlot

One player in a `Lineup`.

| Column | Type |
|---|---|
| `lineupId` | → `Lineup` |
| `playerId` | → `Player` |

**Unique:** `(lineupId, playerId)`, so a player can't appear twice in one lineup.

## Accounts

These four tables follow the schema required by [BetterAuth](https://better-auth.com/docs/concepts/database), the authentication library ([ADR-002](../decisions/adr-002-auth.md)). `role`, `username`, `avatarUrl` and `favoriteTeamId` on `User` are the project's own additions.

### User

| Column | Type | Notes |
|---|---|---|
| `name` | string | |
| `email` | string, unique | |
| `emailVerified` | boolean, default `false` | |
| `image` | string, optional | Profile image URL from Google |
| `role` | `Role`, default `USER` | Permission level. Signing up or updating a Google profile can't change it. |
| `username` | string, optional, unique | Chosen after first sign-in. Empty until the user finishes setting up their profile. |
| `avatarUrl` | string, optional | Location of an uploaded profile picture in private file storage. Despite the name, this is not a web link; the API creates a temporary link when needed. |
| `favoriteTeamId` | → `Team`, optional | The user's favourite team |
| `createdAt`, `updatedAt` | datetime | |

### Session

A signed-in browser session.

| Column | Type | Notes |
|---|---|---|
| `userId` | → `User` | |
| `token` | string, unique | |
| `expiresAt` | datetime | |
| `ipAddress`, `userAgent` | string, optional | |
| `createdAt`, `updatedAt` | datetime | |

### Account

One row per sign-in method linked to a user. Google is currently the only one.

| Column | Type | Notes |
|---|---|---|
| `userId` | → `User` | |
| `accountId`, `providerId` | string | The user's ID at the provider, and which provider it is |
| `accessToken`, `refreshToken`, `idToken`, `scope` | string, optional | Tokens issued by the provider |
| `accessTokenExpiresAt`, `refreshTokenExpiresAt` | datetime, optional | |
| `password` | string, optional | Unused, because users sign in with Google |
| `createdAt`, `updatedAt` | datetime | |

### Verification

Short-lived codes for flows such as email verification. Part of BetterAuth's required schema, but unused while Google is the only sign-in method. [ADR-002](../decisions/adr-002-auth.md) discusses what this means for the brief's password-reset requirement.

| Column | Type |
|---|---|
| `identifier`, `value` | string |
| `expiresAt`, `createdAt`, `updatedAt` | datetime |

## Personal data

Every table in this group records a choice a user made. No NBA statistics are stored here, apart from the deliberate copies of model figures described under `GamePick` and `SavedLineup`. When a user deletes their account, all of their rows here are deleted with it.

### UserFollowedPlayer

A player the user follows. This table has no `id` column; each row is identified by its user and player.

| Column | Type |
|---|---|
| `userId` | → `User` |
| `playerId` | → `Player` |
| `createdAt` | datetime |

**Unique:** `(userId, playerId)`, so a user can follow a player only once.

### GamePick

A user's call on who won a completed game, made without seeing the score, and compared with the model's prediction. Drawn games are excluded, because there is no winner to pick.

| Column | Type | Notes |
|---|---|---|
| `userId` | → `User` | |
| `gameId` | → `Game` | |
| `pickedTeamId` | string | The team the user picked. Checked to be one of the game's two teams when saved, but not a database link. |
| `outcome` | `PickOutcome` | Whether the call was right |
| `modelHomeWinProbabilityAtPick` | float | The model's numbers **at the moment of the call** |
| `modelPredictedMarginAtPick` | float, optional | |
| `homeTeamEloAtPick`, `awayTeamEloAtPick` | float | |
| `createdAt` | datetime | |

The model's numbers are copied into this table because `GamePrediction` is replaced every time the predictor runs. Without the copy, a user's record against the model would change after the fact.

**Unique:** `(userId, gameId)`. A call is final: a second call on the same game is rejected, not treated as a change. **Index:** `(userId, createdAt)`.

### SavedComparison

A named set of players the user compared side by side.

| Column | Type |
|---|---|
| `userId` | → `User` |
| `name` | string |
| `createdAt` | datetime |

**Index:** `(userId, createdAt)`.

### SavedComparisonPlayer

One player in a `SavedComparison`.

| Column | Type | Notes |
|---|---|---|
| `savedComparisonId` | → `SavedComparison` | |
| `playerId` | → `Player` | |
| `position` | int | The player's left-to-right place in the comparison, not their basketball position |

**Unique:** `(savedComparisonId, playerId)`.

### SavedLineup

A fantasy lineup the user chose to keep.

| Column | Type | Notes |
|---|---|---|
| `userId` | → `User` | |
| `name` | string | Required when saving |
| `sourceLineupId` | string, optional | Reserved for linking to the optimizer run the lineup came from. Currently unused. |
| `totalPredictedPointsAtSave` | float | Totals **at the moment of saving** |
| `totalSalaryAtSave`, `budgetAtSave` | int | |
| `createdAt` | datetime | |

**Index:** `(userId, createdAt)`.

### SavedLineupSlot

One player in a `SavedLineup`.

| Column | Type | Notes |
|---|---|---|
| `savedLineupId` | → `SavedLineup` | |
| `playerId` | → `Player` | |
| `predictedPointsAtSave`, `salaryAtSave` | float, int | The player's figures **at the moment of saving** |

**Unique:** `(savedLineupId, playerId)`.

Saved lineups copy their figures for the same reason as `GamePick`: the optimizer keeps producing new predictions, so figures looked up later would no longer match what the user saved. The saved figures are also what lets the home page show how a lineup's predictions have changed since it was saved.

## Become Pro

The tables behind [Become Pro](../become-pro/index.md): a user's own seasons and self-reported box scores, what each season is projected to be worth, and the trained model that projection comes from. Added by migration `20260923000000_add_become_pro` (PR #192). Nothing about the NBA data tables changed to make room for them. They are private to their owner; no other user ever reads them. The diagram above predates them.

### ProspectSeason

One league year at one competition level for one user.

| Column | Type | Notes |
|---|---|---|
| `userId` | → `User` | |
| `season` | string | For example "2025-26", the same format as `Game.season`, so a user's season and an NBA season are labelled identically |
| `competitionLevel` | `CompetitionLevel` | Where the season was played. Required, because it scales the valuation. |
| `position` | string | |
| `teamName` | string, optional | |
| `createdAt`, `updatedAt` | datetime | |

**Unique:** `(userId, season)`, so logging the same league year twice is refused rather than splitting a game log in two. **Index:** `(userId, createdAt)`.

### ProspectGame

One self-reported box score.

| Column | Type | Notes |
|---|---|---|
| `seasonId` | → `ProspectSeason` | |
| `gameDate` | datetime | |
| `opponent` | string | |
| `minutes`, `points`, `rebounds`, `assists`, `steals`, `blocks`, `turnovers` | int | |
| `fieldGoalsMade`, `fieldGoalsAttempted` | int | |
| `threesMade`, `threesAttempted` | int | |
| `freeThrowsMade`, `freeThrowsAttempted` | int | |
| `createdAt`, `updatedAt` | datetime | |

It mirrors the `PlayerGameStat` columns a person can actually know about their own game, and deliberately leaves out plus-minus, usage and the two ratings, which need the possession context of a tracked game. That is why a user's derived line shows "—" for those four rather than a zero.

**Unique:** `(seasonId, gameDate, opponent)`: two games against the same opponent on the same day is a double entry, not a doubleheader. **Index:** `(seasonId, gameDate)`.

### ProspectValuation

What one season is projected to be worth, written by the API every time the season's games change. A new row is added only when a figure actually changes, so the history can be plotted over time.

| Column | Type | Notes |
|---|---|---|
| `seasonId` | → `ProspectSeason` | |
| `projectedDraftSlot` | int, optional | The draft pick the model projects |
| `projectedValueUsd`, `projectedValueLowUsd`, `projectedValueHighUsd` | int, optional | The pick priced on the rookie salary scale, with its range. Empty, never zero, when there is no figure. |
| `rookieScaleYear` | string | Which published scale produced the dollars, e.g. "2026-27", so a stored figure is never re-read against a newer scale |
| `levelFactor`, `levelFactorBasis` | float, string | The competition-level factor applied, and the sentence explaining it |
| `drivers` | JSON | Sentences written by the API naming what moved the figure most; the website shows them word for word |
| `comparablePlayerIds`, `comparableScores` | string list, float list | The real NBA rookies closest to this season, and how close. Plain ids rather than database links, so they survive a player being re-ingested. |
| `slotAlumniPlayerIds` | string list | Real players drafted at the projected pick |
| `modelVersion` | string | |
| `modelId` | → `ProspectValuationModel`, optional | The exact trained model behind the figure. Comparing it with the newest model is how the API notices a stale valuation and redoes it. |
| `computedAt` | datetime | |

**Index:** `(seasonId, computedAt)`, for the latest valuation and the value-over-time history.

### ProspectValuationModel

One trained draft-slot model, written by `apps/valuation/train_valuation_model.py`. The API always applies the newest one.

| Column | Type | Notes |
|---|---|---|
| `modelVersion` | string | |
| `bundle` | JSON | Everything needed to apply the model: the coefficients, the rookie scale, the competition-level factors with their bases, the range widths and the comparable-player index. Each of those is therefore defined once, in Python. |
| `trainingRows`, `mae`, `rankCorrelation` | int, float, float | How good the fit was, stored with the model so every figure can be traced back to it. Both measures are in-sample. |
| `fittedAt` | datetime | |

**Index:** `fittedAt`, for finding the newest model.

The split is deliberate. Training needs the whole NBA rookie dataset and only has to happen when NBA data changes, so Python does it. Applying a fitted linear model is a dot product that has to happen the moment a user logs a game, which only the always-on API can do. See [Valuation Model](../become-pro/valuation-model.md).

## Enums

| Enum | Values | Used by |
|---|---|---|
| `Role` | `PUBLIC`, `USER`, `ANALYST`, `ADMIN` | `User.role` |
| `SeasonType` | `REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS` | `Game.seasonType` |
| `PickOutcome` | `CORRECT`, `MISSED` | `GamePick.outcome` |
| `CompetitionLevel` | `NCAA_D1`, `NCAA_D2`, `NCAA_D3`, `NAIA`, `JUCO`, `INTERNATIONAL_PRO`, `SEMI_PRO`, `HIGH_SCHOOL`, `REC` | `ProspectSeason.competitionLevel`. The API's validation list is built from this enum, so the two can't drift apart. |

## Indexes for common queries

An index lets the database find matching rows without reading the whole table. Each table's indexes are listed with the table above. Five of them were added together on 13 September 2026 (migration `add_query_indexes`, PR #124). Before then, `Game` was indexed only on `seasonType`, even though most of the website's queries find games by date or by team:

| Table | Index | Speeds up |
|---|---|---|
| `Game` | `gameDate` | Lists of games, which are shown in date order almost everywhere |
| `Game` | `(homeTeamId, gameDate)` | A team's results and recent form: games found by team, then sorted by date |
| `Game` | `(awayTeamId, gameDate)` | The same, for games the team played away |
| `PlayerGameStat` | `gameId` | Loading one game's box score. The existing unique constraint on `(playerId, gameId)` already covers loading by player. |
| `Player` | `teamId` | Loading a team's roster |

See [Performance](performance.md) for the measured effect of these indexes, and for which data is cached instead.

## Relationships

```
Team (1) ──────< (many) Player              [current team, optional]
Team (1) ──────< (many) Game                [as home team]
Team (1) ──────< (many) Game                [as away team]
Team (1) ──────< (many) PlayerGameStat      [team in that game, optional]
Team (1) ──────< (many) User                [favourite team, optional]
Game (1) ──────< (many) GameEvent
Game (1) ──────< (many) PlayerGameStat
Game (1) ────── (0..1) GamePrediction
Game (1) ──────< (many) GamePredictionRun
Player (1) ────< (many) PlayerGameStat
Player (1) ────< (many) PlayerPrediction
Player (1) ────< (many) LineupSlot
Lineup (1) ────< (many) LineupSlot

User (1) ──────< (many) Session
User (1) ──────< (many) Account

User (1) ──────< (many) UserFollowedPlayer >───── (1) Player
User (1) ──────< (many) GamePick           >───── (1) Game
User (1) ──────< (many) SavedComparison
User (1) ──────< (many) SavedLineup
SavedComparison (1) ─< (many) SavedComparisonPlayer >─ (1) Player
SavedLineup (1) ─────< (many) SavedLineupSlot       >─ (1) Player

User (1) ──────< (many) ProspectSeason
ProspectSeason (1) ──< (many) ProspectGame
ProspectSeason (1) ──< (many) ProspectValuation >── (0..1) ProspectValuationModel
```

### What happens on delete

| Deleting | Effect on related rows |
|---|---|
| A user | Their sessions, linked accounts, followed players, game calls, saved comparisons, saved lineups and Become Pro seasons are all deleted |
| A Become Pro season | Its games and valuations are deleted |
| A trained valuation model | Valuations made with it keep their figures; only their link to the model is cleared, so pruning old models never deletes a user's history |
| A saved comparison or saved lineup | Its player rows are deleted |
| An optimizer lineup | Its player rows are deleted |
| A player or game | Users' follows, calls and saved items that refer to it are deleted |
| A team | Users with it as their favourite team are left with no favourite; per-game rows that recorded it keep their other data |
| A team, player or game that NBA statistics, events or predictions depend on | Refused, so those records never lose what they were calculated from |

## Still open

- ~~Submitting and reviewing statistics.~~ — **built, checked 2026-09-23.** `IngestionBatch` (a submission record — accepted/rejected event counts, `PENDING_REVIEW`/`COMPLETED`/`REJECTED` status, reviewer, notes) and `EventCorrection` (an append-only audit trail of individual event edits, with `revertsCorrectionId` linking an undo back to what it reverted) now implement exactly this. Not documented on this page yet — see [ADR-001](../decisions/adr-001-database.md)'s currency note above and `apps/api/prisma/schema.prisma` directly for the real column list.
- **Multi-human-submitter approval**, specifically — still genuinely not built, and deliberately so per the schema's own doc comment: this project has one automated "submitter" (the ingestion pipeline itself, source-tagged per batch), not many competing human ones. Don't confuse this with the line above, which is a different and now-closed gap.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5] (2026-09-23: flagged staleness, corrected the "submission/review not built" claim — it is), Claude-Code[Claude Opus 5.5]*
