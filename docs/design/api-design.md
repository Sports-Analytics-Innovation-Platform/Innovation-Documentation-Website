# API Design

!!! warning "Table below is a Sprint 1/2 snapshot — checked 2026-09-23"
    This table is missing everything added since (datasets, custom statistics, self-service API keys, the whole `/v1/admin/*` surface — admin corrections, batch review, consumer management), and its `Auth` column is wrong for the rows it does have: **every** public route below now requires either a session *or* an `X-API-Key` (mandatory since PR #172), not "None." The full, current endpoint list lives on [API Reference](../api-reference.md) — this page is kept for the request/response-shape and error-handling detail below, which is still accurate.

## Confirmed endpoints

Confirmed directly from the backend controllers in `apps/api/src/` — **as of PR #94 (2026-09-11)**; see [API Reference](../api-reference.md) for what's been added since:

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

⚠️ **Updated 2026-09-23 — was wrong about which routes need what.** Players, teams, games, analytics, and datasets are "public" in the sense of not needing a specific role — but since PR #172, **every** one of them needs either a signed-in session *or* a valid `X-API-Key`; there is no truly anonymous path anymore (`OptionalSessionGuard` + `ApiKeyGuard`). Predictions and the optimizer additionally require a session specifically — no API-key path exists for those two. Admin routes (`/v1/admin/*`) require the `ADMIN` role; custom statistics require `ANALYST` or `ADMIN`. This matches the frontend routing: `/players`, `/teams`, `/datasets` are reachable by anyone with a key or a session, while `/predictions`, `/optimizer`, `/games/:gameId`, and `/admin` are wrapped in `<ProtectedRoute>`.

Two of the routes added in PR #94 sit on the public side deliberately. `GET /v1/analytics/model-accuracy` and `GET /v1/analytics/leaderboard` return the same response to everyone — they describe the model and the board, not the caller — so gating them would add nothing. The leaderboard reads only each user's `id` and `name`; email addresses are never selected.

Everything under `/v1/me/*` is **scoped to the session user id in the same `where` clause as the resource id**, so a request can never reach another user's rows even with a valid session and a guessed id. Where a resource exists but belongs to someone else, the response is `404`, not `403` — a `403` would confirm that the row exists, which is itself a disclosure. Because these are the API's first state-changing routes, they sit behind an `OriginCheckGuard` registered globally as an `APP_GUARD`; see [Security](../security.md) for why CORS alone does not cover this.

### Request flow: `GET /v1/games/:id/prediction`

See the [Architecture Overview](architecture.md#sequence-diagram-get-v1gamesidprediction) for the full sequence diagram, which walks this auth-gated request end to end — including why it works cross-origin in production (Cloudflare Pages calling Render) and the two-step 404 (game not found vs. game found but not yet predicted). PlantUML source: `docs/diagrams/sequence-game-prediction.puml` in the main app repo.

## Not yet built

All three of the below were true as of PR #94 (2026-09-11) and are **no longer accurate as of 2026-09-23** — corrected rather than left to mislead:

- ~~Analyst write access~~ — **built.** `/v1/custom-statistics/*` (`ANALYST`/`ADMIN`) lets an analyst define and evaluate a statistic as an expression over a player's event-derived fields.
- ~~Admin write access~~ — **built, merged, and grown well past Team/Player/User management.** `/v1/admin/*` now also covers ingestion batch review/approval, per-event corrections (preview/apply/undo, with an audit trail), and external API-consumer/key management. PR #120 merged long ago; treat any doc still citing it as "open" as stale.
- ~~Versioning beyond `/v1/`~~ — **real machinery exists**, even without a `/v2/` yet: `api-version.guard.ts` enforces `/v1/` and `Accept-Version`, and `@DeprecateEndpoint` + `deprecation.interceptor.ts` emit real `Deprecation`/`Sunset`/`Link` headers (one live example: `GET /health`, deprecated in favour of `GET /v1/health`).

**Genuinely still not built:** an async job pattern for large consumer requests (every response, including dataset publication, is synchronous), and a live/late-arriving event feed (ingestion is still batch-per-game, run after the fact — a deliberate scope decision, not an oversight, since this project has one automated "submitter," not many competing human ones). See [Feature Tiers](feature-tiers.md) for the full current picture.

## OpenAPI / Swagger documentation

The full API reference is documented on the [API Reference (Swagger)](../api-reference.md) page, including the Swagger UI endpoint (`/api/docs`), all request/response shapes, authentication requirements, and the `@nestjs/swagger` setup instructions for the code repo.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5] (2026-09-23: corrected the auth model and "Not yet built" section, flagged the endpoint table as a Sprint 1/2 snapshot)*
