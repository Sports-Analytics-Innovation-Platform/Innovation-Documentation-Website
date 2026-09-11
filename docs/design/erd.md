# ERD

!!! success "Confirmed from `schema.prisma`"
    The core, prediction, and auth entities below are built directly from the real `apps/api/prisma/schema.prisma`, verified field by field against the current schema — including the two Sprint 2 migrations that added season segments (`add_game_season_type`) and advanced per-game stats (`add_advanced_player_game_stats`). The **Personalisation entities** section carries a weaker claim and says so in its own note; read that before citing it.

## Diagram

See the [Architecture Overview](architecture.md#database-erd) for the full visual ERD, grouped by concern (auth, domain, optimizer, predictor). PlantUML source lives alongside the main app repo's diagrams (`docs/diagrams/database-erd.puml`).

## Core entities

**Team**
`id`, `nbaTeamId` (unique, external NBA API ID), `name`, `abbreviation`, `city`, `conference`, `division`, `logoUrl?`

**Player**
`id`, `nbaPlayerId` (unique), `firstName`, `lastName`, `position`, `heightInches?`, `weightLbs?`, `jerseyNumber?`, `headshotUrl?`, `teamId?` → Team

**Game**
`id`, `nbaGameId` (unique), `gameDate`, `season`, `homeTeamId` → Team, `awayTeamId` → Team, `homeScore?`, `awayScore?`, `seasonType` (`SeasonType` enum: `REGULAR`/`PLAY_IN`/`PLAYOFFS`/`FINALS`, default `REGULAR`), `playoffRound?`
Indexed on `seasonType`. Every pre-existing row backfilled to `REGULAR` on migration — `nba_api`'s `LeagueGameFinder` was always called with `season_type_nullable="Regular Season"` before this migration, so the default is correct for old data, not a guess.

**GameEvent** — raw play-by-play; the source of truth every derived stat traces back to, per the brief's requirement that statistics come from event records, not typed totals
`id`, `gameId` → Game, `sequence`, `period`, `clock`, `eventType`, `playerId?`, `description`, `createdAt`
Indexed on `[gameId, sequence]`.

**PlayerGameStat** — per-game boxscore, derived from `GameEvent` rows, never entered by hand
`id`, `playerId` → Player, `gameId` → Game, `minutes`, `points`, `rebounds`, `assists`, `steals`, `blocks`, `turnovers`, `fieldGoalsMade/Attempted`, `threesMade/Attempted`, `freeThrowsMade/Attempted`, `plusMinus?`, `usagePercentage?`, `offensiveRating?`, `defensiveRating?`, `offensiveRebounds?`, `defensiveRebounds?`
Unique on `[playerId, gameId]`. The six advanced columns are all nullable by design, fetched leaguewide from the advanced boxscore rather than per game to stay within `nba_api`'s rate limit — `null` means the row predates these columns or the advanced boxscore was unavailable, which is a different fact than a real 0% usage rate or an even plus-minus.

!!! success "Confirmed: season averages, true shooting%, eFG%, and assist-to-turnover are computed on read, not stored"
    `/v1/players/:id/stats` (in `players.controller.ts`) computes `seasonAverages` and `gameLog` at request time from `PlayerGameStat` rows via `statsService`. There is no `SeasonAverages` table in the schema — it was never a stored model, only an API response shape. True shooting%, effective FG%, and assist-to-turnover ratio are derived the same way from existing boxscore fields (verified against `BoxScoreAdvancedV3` to three decimal places) rather than given their own columns. This confirms the brief's "derived from events, not stored totals" requirement is actually being followed, not just documented as an intent.

## Prediction entities

Added to support game-outcome prediction (`apps/predictor`) and the fantasy-lineup optimizer (`apps/optimizer`). NestJS only reads these tables — the Python services write to them directly.

**GamePrediction** — one row per game (unique on `gameId`, upserted on rerun)
`id`, `gameId` (unique) → Game, `homeWinProbability` (Elo-based, in [0, 1]), `homeTeamEloPre`, `awayTeamEloPre`, `predictedMarginHome?` (Four Factors-based, home minus away, in points), `marginMethod?` ("regression" or "heuristic"), `createdAt`

**PlayerPrediction** — one row per player per optimizer run (append-and-take-latest shape)
`id`, `playerId` → Player, `predictedFantasyPoints`, `salary` (synthetic DFS-style cost, not a real market price), `asOf`
Indexed on `[playerId, asOf]`.

**Lineup** — a single optimizer run's chosen lineup (MILP solve)
`id`, `totalPredictedPoints`, `totalSalary`, `budget`, `createdAt`

**LineupSlot** — one player slot in a lineup
`id`, `lineupId` → Lineup (cascade delete), `playerId` → Player
Unique on `[lineupId, playerId]`.

## Auth entities

Added for the BetterAuth migration — see [ADR-002](../decisions/adr-002-auth.md).

**User**
`id`, `name`, `email` (unique), `emailVerified`, `image?`, `role` (`PUBLIC`/`USER`/`ANALYST`/`ADMIN`, project-specific RBAC field layered on BetterAuth's schema, non-writable by the OAuth flow itself), `createdAt`, `updatedAt` — has many `Session`, many `Account`

**Session**
`id`, `userId` → User (cascade delete), `token` (unique), `expiresAt`, `ipAddress?`, `userAgent?`, `createdAt`, `updatedAt`

**Account** — one row per linked sign-in method (currently just Google)
`id`, `userId` → User (cascade delete), `accountId`, `providerId`, `accessToken?`, `refreshToken?`, token expiries, `scope?`, `idToken?`, `password?`, `createdAt`, `updatedAt`

**Verification** — short-lived tokens (e.g. email verification); present because it's part of BetterAuth's core schema, currently unused while Google OAuth is the only provider — see the password-reset risk flagged in [ADR-002](../decisions/adr-002-auth.md)

## Personalisation entities

Added by a single migration, `20260910134345_home_personalization` (PR #94, merged 2026-09-11), to back the signed-in home page. Two properties of this layer are worth stating before the field lists:

- **Zero ALTERs on any NBA-data table.** The personalisation layer sits entirely alongside the existing schema. Nothing about how `Game`, `Player`, or `PlayerGameStat` behave changed to accommodate it, so none of the ingestion or prediction code had to be touched.
- **The only genuinely new rows are records of a user's own choices.** No NBA statistic is copied into this layer, and nothing derived is stored — the watchlist averages and scoring trends are computed at request time from existing `PlayerGameStat` rows, so a newly ingested game shows up immediately rather than waiting for a recompute.

!!! note "Verified from the migration's design notes, not line by line from `schema.prisma`"
    Unlike the sections above, this one was written from the migration description rather than read field by field out of the schema. The table purposes, the frozen columns, and the constraints called out below are accurate; the surrounding field lists are the documented subset, not a guaranteed-complete column listing. A verification pass against `schema.prisma` is still owed here, and this note should be replaced with the standard "Confirmed" admonition once someone has done it.

**FollowedPlayer** — a player on a user's watchlist, plus that user's own free-text scouting note
`userId` → User, `playerId` → Player, `note?` (free text, 500 characters)
One row per user per player. The note belongs to the follow, not to the player — two users following the same player each keep their own.

**FollowedTeam** — a team a user tracks
`userId` → User, `teamId` → Team, `isPrimary`
At most one of a user's followed teams may have `isPrimary` set.

**GamePick** — a user's call on a game, with the model's prediction frozen at pick time
`userId` → User, `gameId` → Game, the picked side, `outcome` (`PickOutcome`), and four frozen columns: `modelHomeWinProbabilityAtPick`, `modelPredictedMarginAtPick`, `homeTeamEloAtPick`, `awayTeamEloAtPick`

!!! info "Why the model's numbers are copied here instead of joined"
    `GamePrediction` is append-and-take-latest — a later predictor run can change what the "current" prediction for a game is. If the head-to-head record re-derived the model's numbers at read time, a rerun would silently change what the user was graded against, and a record they had already seen would quietly rewrite itself. Freezing the four values at pick time is what makes "you beat the model on this game" a durable statement rather than one that depends on when you ask.

**SavedComparison** / **SavedComparisonPlayer** — a named set of players saved from the Compare tab
`SavedComparison`: `userId` → User, a user-supplied name. `SavedComparisonPlayer`: `savedComparisonId` → SavedComparison, `playerId` → Player.

**SavedLineup** / **SavedLineupSlot** — a saved optimizer lineup
`SavedLineup`: `userId` → User. `SavedLineupSlot`: `savedLineupId` → SavedLineup, `playerId` → Player, plus two frozen columns — `salaryAtSave` and `predictedPointsAtSave`.

The same reasoning as `GamePick` applies: `PlayerPrediction` is also append-and-take-latest, so a slot's salary and predicted points are captured at save time. That frozen pair is precisely what makes the "drift since you saved this" line on the home page computable, and honest — without it there is no baseline to have drifted from.

**PickOutcome** (enum) — `CORRECT` / `MISSED`
Games that ended in a tie are excluded from the challenge entirely rather than given a third outcome value: there is no correct call to make on one.

## Relationship diagram

```
Team (1) ──────< (many) Player
Team (1) ──────< (many) Game [as home team]
Team (1) ──────< (many) Game [as away team]
Game (1) ──────< (many) GameEvent
Game (1) ──────< (many) PlayerGameStat
Game (1) ────── (0..1) GamePrediction
Player (1) ────< (many) PlayerGameStat
Player (1) ────< (many) PlayerPrediction
Player (1) ────< (many) LineupSlot

Lineup (1) ────< (many) LineupSlot

User (1) ──────< (many) Session
User (1) ──────< (many) Account

User (1) ──────< (many) FollowedPlayer >───── (1) Player
User (1) ──────< (many) FollowedTeam   >───── (1) Team
User (1) ──────< (many) GamePick       >───── (1) Game
User (1) ──────< (many) SavedComparison
User (1) ──────< (many) SavedLineup

SavedComparison (1) ─< (many) SavedComparisonPlayer >─ (1) Player
SavedLineup (1) ─────< (many) SavedLineupSlot      >─ (1) Player
```

## Still open

- **Submissions / review workflow** — an early feature-breakdown draft mentioned approved-submitter roles and a review flow, but nothing matching that exists in the schema or codebase. Don't assume it's coming unless the team confirms it's still planned.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5]*
