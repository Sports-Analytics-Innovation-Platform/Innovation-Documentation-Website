# ERD

!!! success "Confirmed from `schema.prisma`"
    This page previously reverse-engineered entities from frontend TypeScript types. It's now built directly from the real `apps/api/prisma/schema.prisma` and its BetterAuth-related migration (see [ADR-002](../decisions/adr-002-auth.md)).

!!! warning "One caveat on freshness"
    The schema snapshot I have predates some later frontend/backend work described in the team's own dev log (e.g. a `GET /v1/games` endpoint, broadened mock data). The core entities below are unlikely to have changed shape, but if new tables were added after this snapshot, they won't appear here — re-confirm against the live `schema.prisma` if this page is being relied on for the group report.

## Entities

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

## Auth entities (added for the BetterAuth migration — see ADR-002)

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
Player (1) ────< (many) PlayerGameStat

User (1) ──────< (many) Session
User (1) ──────< (many) Account
```

## Still open

- **Any table backing "optimisation"** — nothing in the schema touches optimisation at all. See the open question in [Feature Tiers](feature-tiers.md) (still a stub) — this needs the team's plain-English answer on what's actually being optimised before a schema addition makes sense.
- **Submissions / review workflow** — an early feature-breakdown draft mentioned approved-submitter roles and a review flow, but nothing matching that exists in the schema or codebase as of this snapshot. Don't assume it's coming unless the team confirms it's still planned.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
