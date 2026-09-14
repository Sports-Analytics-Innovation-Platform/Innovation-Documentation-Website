# Performance

How the platform keeps its database work down, what was measured, and what the caching buys. Written after a dedicated optimisation pass on branch `DBCallRate` (PR #124, 13 September 2026).

## Why this matters on this stack

The API is a single **Render free-tier instance** talking to **Supabase Postgres through the transaction pooler**. Two costs follow from that: every round trip pays real network latency, and large row pulls consume Supabase compute and egress on a free plan.

The property that makes caching safe here — rather than merely convenient — is that the NBA data is **almost read-only**. Games, stats, predictions and lineups change only when the Python batch jobs (`apps/ingestion`, `apps/predictor`, `apps/optimizer`) run. Between runs the same query returns the same answer, so a short TTL costs nothing in correctness. The one class of data that *does* change constantly is the user-owned personalisation layer, and that is exactly the data this cache never touches.

## What the audit found

Before this pass, **nothing was cached anywhere**. Every page view, tab refocus and signed-in request reached Postgres.

| Problem | Detail |
|---|---|
| Frontend refetch storm | React Query's defaults (`staleTime: 0`, `refetchOnWindowFocus: true`) re-ran all 41 `useQuery` calls on mount and on every tab switch |
| Per-request session lookup | Every signed-in request re-read the `Session` and `User` tables through BetterAuth |
| Heavy public reads, per request | `/players/leaders` and sorted player lists scanned the whole player table plus a `groupBy` over roughly 80k boxscore rows; both analytics routes scanned every completed game; `/teams/elo-ratings` and `/games/seasons` ran full scans |
| Redundant queries | `/players/:id/stats` ran the same query twice, `/stats/splits` ran four, `/players/compare` ran two per player, and `/games/:id/prediction` re-fetched a prediction it had already joined |
| Missing indexes | `Game` was indexed only on `seasonType` — nothing on `gameDate`, `homeTeamId` or `awayTeamId`, which is what most queries filter and sort on |

## The response cache

A small in-process cache in `apps/api/src/cache/`, with **no new dependencies**. Four properties are worth documenting because each one is a decision, not an accident:

- **Single-flight.** When several identical requests arrive at once, one query runs and all of them share its result. This matters because the frontend fires parallel identical requests for upcoming games.
- **Errors and `null` are never stored.** A failed load or a "not found" result is not cached, so a transient database error or a 404 cannot stick around for the length of a TTL.
- **Bounded at roughly 500 entries**, oldest evicted first, so memory stays predictable on a 512 MB instance.
- **Disabled automatically under Vitest**, and manually via `API_CACHE_DISABLED=true`. The e2e specs truncate and reseed one shared database; a cache that survived between specs would serve a previous test's data.

### Time-to-live

| Constant | Value | Covers |
|---|---|---|
| `REFERENCE_DATA_TTL_MS` | 1 hour | Teams, seasons — effectively fixed for a season |
| `DERIVED_DATA_TTL_MS` | 5 minutes | Stats, games, predictions, Elo ratings, optimizer lineups |
| `USER_ACTIVITY_TTL_MS` | 60 seconds | Leaderboard user counts |

### What is cached, and what deliberately is not

Only **public, non-user-specific reads** are cached: games and game detail, seasons, teams, Elo ratings, suggested players, player lookups and stats, matchup projections, model accuracy, leaderboard counts, and optimizer lineups and predictions.

**Nothing under `/v1/me` is cached.** Those responses are per-user, change often, and are already cheap — and caching anything keyed on a session would be the easiest way to serve one user another user's data. The rule is simpler to keep than to audit.

Caching is applied at the **service layer rather than as a controller interceptor**, so a shared sub-result is cached once and reused by every route that needs it, and cache keys contain only the parameters that actually change the answer. Two consequences show up in the measurements below: sorted and paginated player lists reuse the same cached intermediate as the leaders band, and the leaderboard reuses the evaluated-games set that the model-accuracy route already loaded.

**Invalidation** is targeted rather than time-based where it matters: a successful `POST /v1/me/picks` invalidates the leaderboard keys immediately, so a user's own rank updates the moment they make a call, despite the 60-second TTL.

Cached values are **shared references**, so callers must treat them as read-only. Every current caller builds new objects rather than mutating, and the service carries a comment saying so.

Why an in-process cache rather than Redis: see [ADR-004](../decisions/adr-004-caching-strategy.md).

## Query consolidation

Separately from caching — and worth keeping separate, because these savings hold even on a cache miss — several endpoints were issuing more queries than they needed:

| Endpoint | Before | After | How |
|---|---|---|---|
| `GET /v1/players/:id/stats` | 3 | 2 | Fetch the season stats once and derive both the averages and the game log from the same rows |
| `GET /v1/players/:id/stats/splits` | 5 | 2 | One query across all segments, grouped by `seasonType` in Node, instead of one query per segment |
| `GET /v1/players/compare` (4 players) | 8 | 2 | One `findMany` over the id set plus one batched stats query, rather than two queries per player |
| `GET /v1/games/:id/prediction` | 2 | 1 | Return the prediction already joined by the game lookup instead of re-fetching it |
| Leaderboard user counts | 3 | 2 | A single `groupBy` on `[userId, outcome]`, folding calls and correct counts in Node |

`PredictionsService` had no remaining callers once `/games/:id/prediction` stopped re-fetching, so it was deleted rather than left as dead code.

## Frontend caching

`apps/web/src/main.tsx` now configures the React Query client with a **5-minute `staleTime`** and **`refetchOnWindowFocus: false`**, deliberately matched to the API's derived-data TTL so the two layers do not disagree about how fresh the data is.

This does not delay updates after a user action: every mutation in the app already calls `invalidateQueries`, and invalidation refetches active queries regardless of `staleTime`. Following a player, making a pick, editing a note and saving a lineup all still reflect immediately.

## Session lookups

BetterAuth is configured with a **5-minute session cookie cache**, so a signed-in request verifies a signed cookie instead of reading the `Session` and `User` tables every time. This carries a real trade-off, documented in [Security](../security.md#session-cookie-cache) and [ADR-004](../decisions/adr-004-caching-strategy.md): a session revoked from another device, or a role changed directly in the database, can take up to five minutes to take effect. Sign-out and account deletion clear the cookie immediately and are unaffected.

## Indexes

Migration `add_query_indexes` adds what the remaining uncached queries actually filter and sort on:

| Table | Index |
|---|---|
| `Game` | `[gameDate]` |
| `Game` | `[homeTeamId, gameDate]` |
| `Game` | `[awayTeamId, gameDate]` |
| `PlayerGameStat` | `[gameId]` — the existing `@@unique([playerId, gameId])` already covers lookups by player |
| `Player` | `[teamId]` |

Render applies these on the next deploy through the existing `prisma migrate deploy` start command.

## Measured result

Queries logged per request against a local development database holding real ingested data — first call, then an immediate second call. Prisma logs each joined table as its own statement, so these are **SQL statements, not requests**:

| Endpoint | First call | Second call |
|---|---|---|
| `GET /v1/games?status=completed` | 6 | 0 |
| `GET /v1/games/seasons` | 1 | 0 |
| `GET /v1/teams/elo-ratings` | 6 | 0 |
| `GET /v1/players/leaders` | 3 | 0 |
| `GET /v1/players?sort=ppg` | 0 | 0 |
| `GET /v1/analytics/model-accuracy` | 2 | 0 |
| `GET /v1/analytics/leaderboard` | 1 | 0 |
| `GET /v1/players/:id/stats` | 1 | 0 |
| `GET /v1/players/:id/stats/splits` | 3 | 1 |

Three rows need reading carefully rather than at face value:

- **Sorted player lists cost nothing even on a first call**, because changing sort, order, page or minimum-games reuses the intermediate already loaded for the leaders band. Only a change to team, position, search or season segment reaches the database.
- **The leaderboard needed only its own pick counts**, because the set of scored games had already been cached by the model-accuracy call.
- **Splits still runs one query on a repeat visit.** It was consolidated from four queries to one but deliberately left uncached, so this row shows consolidation working rather than caching working.

!!! note "Measured locally, not against production"
    These counts come from a local development database with real ingested data, not from the deployed Render and Supabase instances. They are an accurate measure of *how many statements the API issues*, which is what the work set out to change. They are not a latency benchmark of the production stack, and nothing here should be quoted as one.

## The staleness contract

Worth stating plainly, since it is the price paid for everything above:

- Public NBA data can be **up to 5 minutes stale** after an ingestion or predictor run — 1 hour for teams and seasons.
- A **session revoked elsewhere stays valid for up to 5 minutes**; sign-out and account deletion are immediate.
- **Nothing a user owns is ever stale.** Watchlist, picks, notes, saved comparisons and saved lineups are never cached, and a new pick invalidates the leaderboard immediately.

## Measuring it yourself

Two environment variables exist for this, both documented in `.env.example`:

| Variable | Effect |
|---|---|
| `PRISMA_LOG_QUERIES=true` | Logs every SQL statement the API issues, so query counts can be compared before and after a change |
| `API_CACHE_DISABLED=true` | Turns the response cache off entirely, to confirm a result is genuinely cached rather than coincidentally fast |

Load Home, Predictions, Players (leaders plus a sort) and a player profile, then repeat. A second visit or a tab switch should log no queries for public data.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5]*
