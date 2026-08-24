# API Design

## Confirmed endpoints

Confirmed directly from the backend controllers in `apps/api/src/`:

| Method | Path | Auth | Query params | Returns |
|---|---|---|---|---|
| `GET` | `/health` | None | — | `{ status: "ok" }` |
| `GET` | `/v1/players` | None (public) | `teamId`, `position`, `search`, `page`, `pageSize` | `PagedResult<Player>` |
| `GET` | `/v1/players/:id` | None (public) | — | `Player`, or `404 NOT_FOUND` |
| `GET` | `/v1/players/:id/stats` | None (public) | — | `{ playerId, seasonAverages, gameLog }`, both derived at request time from `PlayerGameStat` rows |
| `GET` | `/v1/teams` | None (public) | `search`, `page`, `pageSize` | `PagedResult<Team>` |
| `GET` | `/v1/teams/:id` | None (public) | — | `Team`, or `404 NOT_FOUND` |
| `GET` | `/v1/games` | `SessionAuthGuard` | varies | Game list with predictions joined in |
| `GET` | `/v1/games/:id` | `SessionAuthGuard` | — | Game detail with win probability, predicted margin, and predicted top scorers |
| `GET` | `/v1/games/:id/prediction` | `SessionAuthGuard` | — | `GamePrediction` (Elo win probability, Four Factors margin), or `404` if no prediction generated yet |
| `GET` | `/v1/optimizer/lineup` | `SessionAuthGuard` | — | Latest `Lineup` with `LineupSlot` entries, or `404` if no lineup generated yet |

BetterAuth mounts its own route set at `/api/auth/*` (sign in, sign out, session management, Google OAuth redirect). These are not hand-written NestJS controllers — they are managed by the BetterAuth library.

All requests go through a single wrapper (`apiClient.ts`):

```ts
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "/api";
fetch(`${API_BASE_URL}${path}`, { credentials: "include" })
```

In dev, `vite.config.ts` proxies `/api/*` to `http://localhost:4000`. In production, `VITE_API_BASE_URL` points at the Render API URL directly.

## Response shapes (from `types/nba.ts`)

**`PagedResult<T>`** — the pagination envelope used by `/v1/players` and `/v1/teams`:

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

(See [ERD](erd.md) for the full field lists of `SeasonAverages` and `GameLogEntry`.)

## Error handling

**Confirmed from source** (`common/api-exception.ts`, `common/all-exceptions.filter.ts`): every error response uses a structured envelope, applied globally:

```ts
{ error: { code: string, message: string } }
```

`ApiException` bodies pass through as-is; other Nest `HttpException`s get wrapped with a generic `HTTP_ERROR` code; anything unhandled becomes a `500` with `INTERNAL_ERROR`, logged server-side.

## Pagination

**Confirmed from source** (`common/pagination.ts`): `?page=` and `?pageSize=`, both optional. Defaults: `page=1`, `pageSize=25`, capped at `pageSize=100`. Applies to `/v1/players` and `/v1/teams`.

## Auth model

Player and team endpoints are **public** — no authentication required. Games, predictions, and optimizer endpoints are **auth-gated** via `SessionAuthGuard` — a valid BetterAuth session cookie is required. This matches the frontend routing: `/players` and `/teams` are accessible to anyone, while `/predictions`, `/optimizer`, and `/games/:gameId` are wrapped in `<ProtectedRoute>`.

### Request flow: `GET /v1/games/:id/prediction`

See the [Architecture Overview](architecture.md#sequence-diagram-get-v1gamesidprediction) for the full sequence diagram, which walks this auth-gated request end to end — including why it works cross-origin in production (Cloudflare Pages calling Render) and the two-step 404 (game not found vs. game found but not yet predicted). PlantUML source: `docs/diagrams/sequence-game-prediction.puml` in the main app repo.

## Not yet built

- **Write endpoints** — nothing in the current code writes data (no POST/PUT/PATCH/DELETE calls exist). If the proposed analyst/admin roles in [Security](../security.md) are real, these don't exist yet.
- **Versioning beyond `/v1/`** — no `/v2/` or deprecation policy exists yet, which is fine at this stage but worth deciding before it matters.
- **OpenAPI/Swagger spec** — planned for Sprint 2 per client meeting (2026-08-21). Not yet adopted.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
