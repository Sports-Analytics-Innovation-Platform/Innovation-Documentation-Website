# API Design

## Confirmed endpoints

Confirmed directly from the backend controllers (`apps/api/src/players/players.controller.ts`, `teams/teams.controller.ts`) as of my code snapshot — not just what the frontend happens to call:

| Method | Path | Query params | Returns |
|---|---|---|---|
| `GET` | `/v1/players` | `teamId`, `position`, `page`, `pageSize` | `PagedResult<Player>` |
| `GET` | `/v1/players/:id` | — | `Player`, or `404 NOT_FOUND` |
| `GET` | `/v1/players/:id/stats` | — | `{ playerId, seasonAverages, gameLog }`, both derived at request time from `PlayerGameStat` rows |
| `GET` | `/v1/teams` | (same list-query pattern as players) | `PagedResult<Team>` |
| `GET` | `/v1/teams/:id` | — | `Team`, or `404 NOT_FOUND` |

The team's own dev log also mentions a `GET /v1/games` endpoint added later — not yet confirmed from source, since it postdates this snapshot.

All requests go through a single wrapper (`apiClient.ts`):

```ts
const API_BASE_URL = "/api";
fetch(`${API_BASE_URL}${path}`, { credentials: "include" })
```

So the full path hit on the wire is `/api/v1/players`, etc. — `/api` is the Vite dev proxy prefix (confirmed, see [Tech Stack](../tech-stack.md)) in front of the versioned `/v1/` routes.

## Response shapes (from `types/nba.ts`)

**`PagedResult<T>`** — the pagination envelope used at minimum by `/v1/players`:

```ts
{ data: T[], page: number, pageSize: number, total: number }
```

**`Player`**:

```ts
{
  id, nbaPlayerId, firstName, lastName, position,
  heightInches, weightLbs, jerseyNumber, headshotUrl,
  teamId, team: Team | null
}
```

**`PlayerStatsResponse`**:

```ts
{ playerId, seasonAverages: SeasonAverages, gameLog: GameLogEntry[] }
```

(See [ERD](erd.md) for the full field lists of `SeasonAverages` and `GameLogEntry` — same caveat applies: these are response shapes, not confirmed DB structure.)

## Error handling

**Confirmed from source** (`common/api-exception.ts`, `common/all-exceptions.filter.ts`): every error response uses a structured envelope, applied globally:

```ts
{ error: { code: string, message: string } }
```

`ApiException` bodies pass through as-is; other Nest `HttpException`s get wrapped with a generic `HTTP_ERROR` code; anything unhandled becomes a `500` with `INTERNAL_ERROR`, logged server-side. This resolves what was previously an open gap — the envelope exists and matches the shape this doc had proposed. `apiClient.ts` on the frontend hasn't been confirmed to actually parse `error.code`/`error.message` yet rather than just checking `response.ok` — worth a quick check before assuming the frontend surfaces these usefully.

## Pagination

**Confirmed from source** (`common/pagination.ts`): `?page=` and `?pageSize=`, both optional. Defaults: `page=1`, `pageSize=25`, capped at `pageSize=100`. Applies to `/v1/players` and `/v1/teams`.

## Not yet confirmed / not built

- **Versioning beyond `/v1/`** — no `/v2/` or deprecation policy exists yet, which is fine at this stage but worth deciding before it matters.
- **Write endpoints** — nothing in the current code writes data (no POST/PUT/PATCH/DELETE calls exist). If the proposed analyst/admin roles in [Security](../security.md) are real, these don't exist yet.
- **Auth endpoints** — not confirmed from this snapshot; BetterAuth typically mounts its own route set (e.g. `/api/auth/*`) rather than routes written by hand in a controller — worth confirming the actual paths once someone can check.
- **OpenAPI/Swagger spec** — not confirmed to exist. The client mentioned Swagger as a tool they used previously, not something this team has adopted yet.
- **Prediction/recommendation endpoints** — not built yet, see [Feature Tiers](feature-tiers.md).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
