# API Design

## Confirmed endpoints

Confirmed directly from the backend controllers in `apps/api/src/`:

| Method | Path | Auth | Query params | Returns |
|---|---|---|---|---|
| `GET` | `/health` | None | — | `{ status: "ok" }` |
| `GET` | `/v1/players` | None (public) | `teamId`, `position`, `search`, `seasonType`, `participated`, `page`, `pageSize` | `PagedResult<Player>` |
| `GET` | `/v1/players/:id` | None (public) | — | `Player`, or `404 NOT_FOUND` |
| `GET` | `/v1/players/:id/stats` | None (public) | `seasonType` | `{ playerId, seasonType, seasonAverages, gameLog }` — `seasonAverages` derived at request time from that segment's `PlayerGameStat` rows |
| `GET` | `/v1/players/:id/stats/splits` | None (public) | — | `{ playerId, splits }` — the same derived line as `:id/stats`, for every `seasonType` at once |
| `GET` | `/v1/players/compare` | None (public) | `ids`, `seasonType` | `{ seasonType, players: [{ player, seasonAverages }] }` for 2–4 players |
| `GET` | `/v1/teams` | None (public) | `search`, `page`, `pageSize` | `PagedResult<Team>` |
| `GET` | `/v1/teams/:id` | None (public) | — | `Team`, or `404 NOT_FOUND` |
| `GET` | `/v1/games` | `SessionAuthGuard` | `seasonType`, `page`, `pageSize` | Game list with predictions joined in |
| `GET` | `/v1/games/:id` | `SessionAuthGuard` | — | Game detail with win probability, predicted margin, and predicted top scorers |
| `GET` | `/v1/games/:id/prediction` | `SessionAuthGuard` | — | `GamePrediction` (Elo win probability, Four Factors margin), or `404` if no prediction generated yet |
| `GET` | `/v1/optimizer/lineup` | `SessionAuthGuard` | — | Latest `Lineup` with `LineupSlot` entries, or `404` if no lineup generated yet |
| `GET` | `/v1/optimizer/predictions/:playerId` | `SessionAuthGuard` | — | Predicted fantasy points for one player by NBA player ID, or `404` |
| `GET` | `/v1/analytics/model-accuracy` | None (public) | — | Model accuracy, Brier score, always-pick-home baseline, games evaluated, forward-prediction count, and calibration bands |
| `GET` | `/v1/analytics/leaderboard` | None (public) | — | Accuracy ranking of users, with the Elo model included as a benchmark row |
| `GET` | `/v1/me/challenge/next` | `SessionAuthGuard` | — | A completed game with its final score withheld, excluding every game the caller has already picked |
| `POST` | `/v1/me/picks` | `SessionAuthGuard` | — | Submits a pick, grades it against the result and the model, and returns the score |
| `GET` | `/v1/me/picks/record` | `SessionAuthGuard` | — | The caller's head-to-head record against the model, on the same games |
| `GET` | `/v1/me/watchlist` | `SessionAuthGuard` | `page`, `pageSize` | Followed players with derived averages, recent scoring, and the caller's notes |
| `GET` | `/v1/me/watchlist/ids` | `SessionAuthGuard` | — | Followed player ids only, so a follow button can render its own state in one request |
| `POST` | `/v1/me/follows/players/:playerId` | `SessionAuthGuard` | — | Follow a player (idempotent) |
| `PATCH` | `/v1/me/follows/players/:playerId` | `SessionAuthGuard` | — | Replace the scouting note; `404` rather than creating the follow |
| `DELETE` | `/v1/me/follows/players/:playerId` | `SessionAuthGuard` | — | Unfollow; returns `{ playerId, removed }` |
| `PUT` | `/v1/me/follows/teams/:teamId` | `SessionAuthGuard` | — | Follow a team, optionally as the caller's one primary team |
| `DELETE` | `/v1/me/follows/teams/:teamId` | `SessionAuthGuard` | — | Unfollow a team |
| `GET` | `/v1/me/teams/results` | `SessionAuthGuard` | — | Recent results for the caller's teams, oriented to their side (`yourTeam`/`opponent`, `won`) |
| `GET` `POST` `DELETE` | `/v1/me/saved/comparisons` | `SessionAuthGuard` | — | Saved player comparisons |
| `GET` `POST` `DELETE` | `/v1/me/saved/lineups` | `SessionAuthGuard` | — | Saved lineups, with drift since save |

The `/v1/analytics/*` and `/v1/me/*` routes were added in PR #94 (merged 2026-09-11) to back the signed-in home page. They are the API's first write endpoints — see [Auth model](#auth-model) below for how they are scoped, and [API Reference](../api-reference.md) for full request/response shapes.

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
{ playerId, seasonType, seasonAverages: DerivedSeasonAverages, gameLog: GameLogEntry[] }
```

`seasonType` echoes back the resolved segment (defaults to `REGULAR`) so a caller can't mislabel a chart it already rendered against a different segment. See [ERD](erd.md) for the full field lists of `DerivedSeasonAverages` and `GameLogEntry`.

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

Two of the routes added in PR #94 sit on the public side deliberately. `GET /v1/analytics/model-accuracy` and `GET /v1/analytics/leaderboard` return the same response to everyone — they describe the model and the board, not the caller — so gating them would add nothing. The leaderboard reads only each user's `id` and `name`; email addresses are never selected.

Everything under `/v1/me/*` is **scoped to the session user id in the same `where` clause as the resource id**, so a request can never reach another user's rows even with a valid session and a guessed id. Where a resource exists but belongs to someone else, the response is `404`, not `403` — a `403` would confirm that the row exists, which is itself a disclosure. Because these are the API's first state-changing routes, they sit behind an `OriginCheckGuard` registered globally as an `APP_GUARD`; see [Security](../security.md) for why CORS alone does not cover this.

### Request flow: `GET /v1/games/:id/prediction`

See the [Architecture Overview](architecture.md#sequence-diagram-get-v1gamesidprediction) for the full sequence diagram, which walks this auth-gated request end to end — including why it works cross-origin in production (Cloudflare Pages calling Render) and the two-step 404 (game not found vs. game found but not yet predicted). PlantUML source: `docs/diagrams/sequence-game-prediction.puml` in the main app repo.

## Not yet built

- **Analyst/admin write access** — the `/v1/me/*` routes added in PR #94 are the API's first write endpoints, but they only ever write a user's *own* choices: follows, notes, picks, and saved comparisons and lineups. Nothing writes or corrects NBA data, so the proposed analyst/admin roles in [Security](../security.md) still have no endpoint behind them.
- **Versioning beyond `/v1/`** — no `/v2/` or deprecation policy exists yet, which is fine at this stage but worth deciding before it matters.

## OpenAPI / Swagger documentation

The full API reference is documented on the [API Reference (Swagger)](../api-reference.md) page, including the Swagger UI endpoint (`/api/docs`), all request/response shapes, authentication requirements, and the `@nestjs/swagger` setup instructions for the code repo.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5]*
