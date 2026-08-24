# ERD

!!! success "Confirmed from `schema.prisma`"
    This page is built directly from the real `apps/api/prisma/schema.prisma` (257 lines, 12 models). Last verified against the current schema.

## Diagram

See the [Architecture Overview](architecture.md#database-erd) for the full visual ERD, grouped by concern (auth, domain, optimizer, predictor). PlantUML source lives alongside the main app repo's diagrams (`docs/diagrams/database-erd.puml`).

## Core entities

**Team**
`id`, `nbaTeamId` (unique, external NBA API ID), `name`, `abbreviation`, `city`, `conference`, `division`, `logoUrl?`

**Player**
`id`, `nbaPlayerId` (unique), `firstName`, `lastName`, `position`, `heightInches?`, `weightLbs?`, `jerseyNumber?`, `headshotUrl?`, `teamId?` → Team

**Game**
`id`, `nbaGameId` (unique), `gameDate`, `season`, `homeTeamId` → Team, `awayTeamId` → Team, `homeScore?`, `awayScore?`

**GameEvent** — raw play-by-play; the source of truth every derived stat traces back to, per the brief's requirement that statistics come from event records, not typed totals
`id`, `gameId` → Game, `sequence`, `period`, `clock`, `eventType`, `playerId?`, `description`, `createdAt`
Indexed on `[gameId, sequence]`.

**PlayerGameStat** — per-game boxscore, derived from `GameEvent` rows, never entered by hand
`id`, `playerId` → Player, `gameId` → Game, `minutes`, `points`, `rebounds`, `assists`, `steals`, `blocks`, `turnovers`, `fieldGoalsMade/Attempted`, `threesMade/Attempted`, `freeThrowsMade/Attempted`
Unique on `[playerId, gameId]`.

!!! success "Confirmed: season averages are computed on read, not stored"
    `/v1/players/:id/stats` (in `players.controller.ts`) computes `seasonAverages` and `gameLog` at request time from `PlayerGameStat` rows via `statsService`. There is no `SeasonAverages` table in the schema — it was never a stored model, only an API response shape. This confirms the brief's "derived from events, not stored totals" requirement is actually being followed, not just documented as an intent.

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
```

## Still open

- **Submissions / review workflow** — an early feature-breakdown draft mentioned approved-submitter roles and a review flow, but nothing matching that exists in the schema or codebase. Don't assume it's coming unless the team confirms it's still planned.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
