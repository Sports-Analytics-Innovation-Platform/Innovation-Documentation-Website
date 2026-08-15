# ERD

!!! warning "Inferred, not confirmed"
    Nothing here comes from the actual `schema.prisma` — I don't have it. This is reverse-engineered from the frontend's TypeScript types (`apps/web/src/types/nba.ts`) and the API responses they describe. Frontend types describe the *API's response shape*, which is not guaranteed to match the *database schema* one-for-one (e.g. `SeasonAverages` looks aggregated/computed, not a raw table). Treat every entity below as a hypothesis to confirm against the real schema, not documentation of it.

## Entities inferred from `nba.ts`

**Team**
- `id`, `nbaTeamId` (external NBA API ID), `name`, `abbreviation`, `city`, `conference`, `division`, `logoUrl`

**Player**
- `id`, `nbaPlayerId`, `firstName`, `lastName`, `position`, `heightInches`, `weightLbs`, `jerseyNumber`, `headshotUrl`, `teamId` (→ Team)

**GameLogEntry** (per player, per game)
- `gameId`, `gameDate`, `points`
- Only `points` is exposed to the frontend chart, but a real per-game stat row for the brief's "derived from individual events" requirement would plausibly need rebounds, assists, etc. too — worth confirming whether those fields already exist server-side and just aren't wired into this chart yet, or don't exist at all.

**SeasonAverages** (likely *computed*, not stored)
- `gamesPlayed`, `pointsPerGame`, `reboundsPerGame`, `assistsPerGame`, `stealsPerGame`, `blocksPerGame`, `turnoversPerGame`, `fieldGoalPercentage`, `threePointPercentage`, `freeThrowPercentage`
- The brief requires stats to be derived from event data rather than stored totals — if this is implemented as intended, these numbers are calculated on read from a `GameLogEntry`-like table, not persisted as their own row. **Confirm this is actually true** — it's exactly the kind of requirement that's easy to violate accidentally by caching an aggregate.

## Proposed relationship diagram

```
Team (1) ──────────< (many) Player
Player (1) ──────────< (many) GameLogEntry [event-level, per game]
Player (1) ──────────  SeasonAverages [derived/computed, not necessarily its own table]
```

## Known gaps — not in the frontend types at all

- **User** — no model for authenticated users appears anywhere in the frontend code, but Passport auth exists per [ADR-002](../decisions/adr-002-auth.md). This table has to exist somewhere in `schema.prisma`; it's just not visible from the frontend.
- **Roles / permissions** — if the proposed RBAC in [Security](../security.md) is real, there's presumably a role field or a join table somewhere, unconfirmed.
- **Any table backing the "optimisation" side of the product** — nothing in the current frontend code touches optimisation at all; the whole feature is still undefined (see the open question in [Feature Tiers](feature-tiers.md), not yet written).

## What would close this gap

The actual `schema.prisma` file, or even just a `prisma migrate` diff/output pasted in — that would let this page go from "inferred" to "confirmed" in one pass.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
