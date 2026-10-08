# API Reference

The NBA Analytics API is a NestJS service hosted at **[sportsanalytics-api.onrender.com](https://sportsanalytics-api.onrender.com)**. The interactive reference is generated from the code, so it is always the source of truth:

- **Swagger UI:** [/api/docs](https://sportsanalytics-api.onrender.com/api/docs). Every operation, its parameters and response shapes, with "Try it out".
- **OpenAPI JSON:** [/api-json](https://sportsanalytics-api.onrender.com/api-json).

This page summarises the conventions every route shares and lists all 106 live operations, grouped by area. The spec comes from `@nestjs/swagger` decorators (`@ApiTags`, `@ApiOperation`, `@ApiQuery`, `@ApiResponse`) on each controller, wired up in `apps/api/src/main.ts`.

---

## Authentication

There are three ways a request is authorised. The **Auth** column in the tables below says which one each route needs.

| Auth | What the caller sends | Who uses it |
|---|---|---|
| **Key or session** | A session cookie, **or** an `X-API-Key` header | The public read routes: players, games, teams, analytics, datasets. A request with neither gets `401 API_KEY_REQUIRED`. |
| **Session** | `Cookie: better-auth.session_token=<token>` | Everything under `/v1/me/**` and `/v1/optimizer/*`. Acts only on the signed-in user's own data. |
| **Admin** / **Analyst** | A session whose user has the `ADMIN` (or `ANALYST`) role | `/v1/admin/**`, dataset publishing, custom statistics |

**Sessions.** Sign-in is BetterAuth with **Google OAuth only**; there is no email/password route. The web app calls `POST /auth/sign-in/social` with `provider: "google"`, Google redirects back, and the callback sets the session cookie. See [ADR-002: Auth](decisions/adr-002-auth.md).

**API keys.** There are two kinds:

- **Self-service:** created on the user's Profile page (`/v1/me/api-keys`). 60 requests per minute, 5,000 per day.
- **Admin-issued:** created for an external `ApiConsumer` in the admin Consumers tab, with that consumer's own limits.

Both limits are checked against the `ApiUsageLog` table on every keyed request, so they survive a server restart. Going over returns `429 RATE_LIMIT_EXCEEDED` (per minute) or `429 DAILY_QUOTA_EXCEEDED` (per day).

**Signed-out visitors on the web app** never hold a key. The site's Cloudflare Pages Function (`functions/api/[[path]].ts`) proxies their reads and attaches the server-held `SITE_PROXY_API_KEY`, which belongs to the consumer "NBA Analytics Web App (first-party)". See [Getting Started](getting-started.md).

```bash
curl -H "X-API-Key: <your key>" \
  "https://sportsanalytics-api.onrender.com/v1/players?search=nikola&pageSize=5"
```

---

## Errors

Every error uses one envelope, applied globally by `AllExceptionsFilter`:

```json
{ "error": { "code": "NOT_FOUND", "message": "Player not found" } }
```

| Status | Code | Meaning |
|---|---|---|
| `400` | `BAD_REQUEST` | Invalid query parameters or body. Some routes use a more specific code, such as `INVALID_LINEUP`, `INVALID_CORRECTION` or `INVALID_BOX_SCORE`. |
| `401` | `UNAUTHENTICATED` | Session route called without a valid session |
| `401` | `API_KEY_REQUIRED` | Key-or-session route called with neither |
| `401` | `UNAUTHORIZED` | The `X-API-Key` is unknown, revoked, or its consumer is inactive |
| `403` | `FORBIDDEN` | Signed in, but without the required role |
| `404` | `NOT_FOUND` | No such resource. Also returned for another user's data under `/v1/me/**`, so ids can't be probed. |
| `406` | `UNSUPPORTED_API_VERSION` | The `Accept-Version` header names a version other than `1` |
| `409` | `CONFLICT` and others | For example `USERNAME_TAKEN`, `CORRECTION_CONFLICT`, or a stale dataset with no stored file |
| `429` | `RATE_LIMIT_EXCEEDED` / `DAILY_QUOTA_EXCEEDED` | API-key per-minute or per-day limit reached |
| `500` | `INTERNAL_ERROR` | Unexpected error, logged server-side |

---

## Pagination and versioning

**Pagination.** List routes take `page` (default `1`) and `pageSize` (default `25`, maximum `100`), and return:

```json
{ "data": [ ... ], "page": 1, "pageSize": 25, "total": 530 }
```

**Versioning.** Every route lives under `/v1/`. A request whose `Accept-Version` header names another version gets `406`. The one deprecated route, `GET /health`, still answers but sends `Deprecation`, `Sunset` and `Link` headers pointing to `GET /v1/health`.

**Season segments.** Routes that take `seasonType` accept `REGULAR` (the default), `PLAY_IN`, `PLAYOFFS` or `FINALS`.

---

## Endpoints

### Health

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/health` | None | Service health check |
| `GET` | `/health` | None | Deprecated alias of `/v1/health` |

### Players

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/players` | Key or session | Paginated list. Filters: `search`, `teamId`, `position`, `minGames`. Sort with `sort` and `order`. |
| `GET` | `/v1/players/{id}` | Key or session | One player with their team |
| `GET` | `/v1/players/{id}/stats` | Key or session | Season averages and game log for one `seasonType` |
| `GET` | `/v1/players/{id}/stats/splits` | Key or session | The same season line for every segment at once |
| `GET` | `/v1/players/{id}/stats/career` | Key or session | Career totals, averages and a per-season breakdown |
| `GET` | `/v1/players/{id}/matchup-projection` | Key or session | Splits against each opponent, plus scoring projections for upcoming games |
| `GET` | `/v1/players/compare` | Key or session | Compare 2–4 players side by side (`ids=a,b,c`) |
| `GET` | `/v1/players/stats-batch` | Key or session | Season stats for several players in one query (`ids=a,b,c`) |
| `GET` | `/v1/players/leaders` | Key or session | Leader in PPG, RPG, APG and TS% after a games-played floor (`minGames`, `asOf`) |
| `GET` | `/v1/players/league-averages` | Key or session | League-wide averages for one segment |
| `GET` | `/v1/players/aggregates` | Key or session | A metric averaged by team or position (`groupBy`, `metric`) |
| `GET` | `/v1/players/export` | Key or session | The filtered list as CSV |

Example: `GET /v1/players/{id}/stats?seasonType=REGULAR` for Nikola Jokić. The response is shortened; `seasonAverages` has 24 fields.

```json
{
  "playerId": "f74d5514-1a30-4cf7-927a-0c396409f422",
  "seasonType": "REGULAR",
  "seasonAverages": {
    "gamesPlayed": 214, "minutesPerGame": 34.9, "pointsPerGame": 27.8,
    "reboundsPerGame": 12.6, "assistsPerGame": 9.9, "trueShootingPercentage": 66,
    "usagePercentage": 28.9, "offensiveRating": 126.1, "defensiveRating": 115.4
  },
  "gameLog": [
    { "gameId": "232b014b-...", "gameDate": "2023-10-24T00:00:00.000Z", "points": 29, "season": "2023-24" }
  ]
}
```

All statistics are derived by the team from play-by-play events, not copied from `nba_api` totals. See [Data Ingestion](design/ingestion.md).

### Games

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/games` | Key or session | Paginated list, most recent first |
| `GET` | `/v1/games/seasons` | Key or session | Seasons that have games, for season filters |
| `GET` | `/v1/games/{id}` | Key or session | Game detail with the prediction and predicted top scorers |
| `GET` | `/v1/games/{id}/prediction` | Key or session | Elo win probability and Four Factors breakdown |
| `GET` | `/v1/games/{id}/prediction/history` | Key or session | Every model version's prediction for this game |
| `GET` | `/v1/games/{id}/events` | Key or session | The ordered play-by-play events |
| `GET` | `/v1/games/{id}/live` | Key or session | Poll for new events in a game in progress |
| `GET` | `/v1/games/export` | Key or session | The filtered list as CSV |

### Teams

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/teams` | Key or session | Paginated list (`search`) |
| `GET` | `/v1/teams/{id}` | Key or session | One team |
| `GET` | `/v1/teams/records` | Key or session | Every team's win/loss record and recent form |
| `GET` | `/v1/teams/elo-ratings` | Key or session | Every team's current Elo rating |
| `GET` | `/v1/teams/{id}/suggested-players` | Key or session | The roster ranked by usage rate (`count`) |

### Optimizer and analytics

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/optimizer/lineup` | Session | The latest fantasy lineup from the optimizer |
| `GET` | `/v1/optimizer/predictions` | Session | Every player's latest projection |
| `GET` | `/v1/optimizer/predictions/{playerId}` | Session | One player's latest projection |
| `GET` | `/v1/analytics/model-accuracy` | Key or session | How the Elo model has scored on finished, predicted games |
| `GET` | `/v1/analytics/leaderboard` | Key or session | Users ranked by prediction accuracy, with the model as the benchmark |

### Me

Every `/v1/me/**` route reads the user from the session. None takes a user id, so a user can only reach their own data.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/me` | Session | Profile, role, favourite team and followed players |
| `PATCH` | `/v1/me` | Session | Change username and/or favourite team |
| `POST` | `/v1/me/avatar` | Session | Upload an avatar image (multipart) |
| `PUT` | `/v1/me/followed-players/{playerId}` | Session | Follow a player (idempotent) |
| `DELETE` | `/v1/me/followed-players/{playerId}` | Session | Unfollow a player (idempotent) |
| `GET` | `/v1/me/watchlist` | Session | Followed players with their season averages and recent points |
| `GET` | `/v1/me/teams/results` | Session | Recent results for the user's teams, from their team's side |
| `GET` | `/v1/me/challenge/next` | Session | A finished game the user hasn't called yet, with the score hidden |
| `POST` | `/v1/me/picks` | Session | Call that game. The response reveals the result and the final score. |
| `GET` | `/v1/me/picks/record` | Session | The user's record beside the model's record on the same games |
| `GET` | `/v1/me/lineups` | Session | Saved optimizer lineups, newest first |
| `POST` | `/v1/me/lineups` | Session | Save a lineup |
| `DELETE` | `/v1/me/lineups/{lineupId}` | Session | Delete a saved lineup |
| `GET` | `/v1/me/saved/comparisons` | Session | Saved player comparisons |
| `POST` | `/v1/me/saved/comparisons` | Session | Save a comparison |
| `DELETE` | `/v1/me/saved/comparisons/{id}` | Session | Delete a saved comparison |
| `GET` | `/v1/me/api-keys` | Session | The user's own API keys, limits and usage |
| `POST` | `/v1/me/api-keys` | Session | Create a key. The raw key is shown once. |
| `DELETE` | `/v1/me/api-keys/{keyId}` | Session | Revoke a key (kept, marked inactive) |
| `DELETE` | `/v1/me/api-keys/{keyId}/purge` | Session | Permanently delete a key |

### Become Pro

A signed-in user logs their own games and gets a projected draft pick, a rookie-scale value and comparable NBA rookies. See [Become Pro](become-pro/index.md) and the [Valuation Model](become-pro/valuation-model.md). Logging, correcting or removing a game, or editing a season, re-values the season before the response returns.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/me/become-pro` | Session | The full page: seasons, one season in detail (`seasonId`), valuation and comparables |
| `GET` | `/v1/me/become-pro/summary` | Session | Current projected value and trend, for the Home and Profile card |
| `POST` | `/v1/me/become-pro/seasons` | Session | Start a season |
| `PATCH` | `/v1/me/become-pro/seasons/{seasonId}` | Session | Edit a season |
| `DELETE` | `/v1/me/become-pro/seasons/{seasonId}` | Session | Delete a season and its games |
| `POST` | `/v1/me/become-pro/seasons/{seasonId}/games` | Session | Log a game |
| `PATCH` | `/v1/me/become-pro/games/{gameId}` | Session | Correct a logged game |
| `DELETE` | `/v1/me/become-pro/games/{gameId}` | Session | Remove a logged game |

Become Pro returns these errors:

- `400 INVALID_BOX_SCORE` for an impossible line, such as more makes than attempts, over 65 minutes, or a future date.
- `404 SEASON_NOT_FOUND` / `GAME_NOT_FOUND`, also for another user's ids.
- `409` with `SEASON_ALREADY_EXISTS`, `SEASON_LIMIT_REACHED` (12 seasons), `GAME_LIMIT_REACHED` (120 games) or `DUPLICATE_GAME`.

A points total that disagrees with the shooting splits is saved, and the page flags it for the user to check.

### Datasets

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/datasets` | Key or session | Published dataset releases |
| `GET` | `/v1/datasets/{version}` | Key or session | One release's metadata |
| `GET` | `/v1/datasets/{version}/download` | Key or session | The release as CSV |
| `GET` | `/v1/datasets/diff` | Key or session | Compare two releases' metadata and schema (`from`, `to`) |
| `GET` | `/v1/datasets/changes` | Key or session | Releases published after a timestamp (`since`) |
| `POST` | `/v1/datasets/admin/publish` | Admin | Publish a new release |

### Custom statistics

Analysts define a statistic as an expression over per-game fields: points, rebounds, assists, steals, blocks, turnovers and minutes. The expression is parsed by a hand-written recursive-descent parser (no `eval`). Unknown field names and division by zero are rejected before evaluation.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/custom-statistics` | Analyst | The caller's definitions |
| `POST` | `/v1/custom-statistics` | Analyst | Create a definition: `{ "name", "expression" }` |
| `PUT` | `/v1/custom-statistics/{id}` | Analyst | Change the expression. This increments its `version`. |
| `GET` | `/v1/custom-statistics/{id}/value` | Analyst | Evaluate it for one player (`playerId`, optional `seasonType`) |

### Admin

Every `/v1/admin/**` route needs a session with the `ADMIN` role.

**Reference data and users**

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/admin/teams` | Paginated teams for editing |
| `PATCH` | `/v1/admin/teams/{id}` | Edit a team's imported fields |
| `GET` | `/v1/admin/players` | Paginated players for editing (`teamId`, `search`) |
| `PATCH` | `/v1/admin/players/{id}` | Edit a player's imported fields |
| `GET` | `/v1/admin/users` | Paginated users |
| `PATCH` | `/v1/admin/users/{id}/role` | Change a user's role |
| `DELETE` | `/v1/admin/users/{id}` | Delete a user account |

**Ingestion review.** For how batches and the pull worker fit together, see [Data Ingestion](design/ingestion.md#batches).

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/admin/batches` | Paginated ingestion batches (`status`, `search`, `sort`, `order`) |
| `GET` | `/v1/admin/batches/{id}` | One batch in detail |
| `POST` | `/v1/admin/batches/{id}/approve` | Approve a `PENDING_REVIEW` batch. This publishes its games. |
| `POST` | `/v1/admin/batches/{id}/reject` | Reject a batch. Its data stays unpublished. |
| `POST` | `/v1/admin/ingestion/pull` | Start a pull (optional `season`, `fromDate`, `toDate`). The deployed API queues it for a pull worker. |
| `GET` | `/v1/admin/ingestion/requests` | The 10 most recent pulls, with status and output |
| `POST` | `/v1/admin/ingestion/requests/{id}/cancel` | Cancel a pull no worker has claimed |
| `GET` | `/v1/admin/ingestion/schedule` | The pull schedule and when a worker last checked in |
| `PUT` | `/v1/admin/ingestion/schedule` | Set the schedule: `NEVER`, `HOURLY`, `DAILY` or `WEEKLY` |
| `DELETE` | `/v1/admin/ingestion/batches/{id}` | Soft-delete a batch. No game data is deleted. |

**Event corrections.** Only 2025-26 games have stored play-by-play, so only they can be corrected. See [ADR-005](decisions/adr-005-play-by-play-storage.md).

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/admin/games` | Find games by `season`, `teamId` and date window, with event counts |
| `GET` | `/v1/admin/games/{gameId}/events` | A game's header, every event with player names, and the roster |
| `GET` | `/v1/admin/games/{gameId}/anomalies` | Player box-score rows that fail sanity checks |
| `POST` | `/v1/admin/games/{gameId}/events/{sequence}/preview` | Dry run: changed fields and each player's stats before and after |
| `POST` | `/v1/admin/games/{gameId}/events/{sequence}/correct` | Apply a correction (reason required). Recomputes affected stats in one transaction. |
| `POST` | `/v1/admin/corrections/{id}/revert` | Undo a correction by applying its old values as a new correction |
| `POST` | `/v1/admin/games/{gameId}/replay` | Re-derive a game's stats from its current events |
| `GET` | `/v1/admin/games/{gameId}/corrections` | One game's correction history |
| `GET` | `/v1/admin/events/corrections` | Paginated correction history (optional `gameId`) |

**API consumers**

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/admin/consumers` | Consumers with their limits and usage |
| `POST` | `/v1/admin/consumers` | Create a consumer and its first key |
| `PATCH` | `/v1/admin/consumers/{id}` | Change a consumer's name, limits or active flag |
| `DELETE` | `/v1/admin/consumers/{id}` | Delete a consumer with its keys and usage log |
| `POST` | `/v1/admin/consumers/{id}/keys` | Issue another key |
| `DELETE` | `/v1/admin/consumers/{id}/keys/{keyId}` | Revoke a key |
| `DELETE` | `/v1/admin/consumers/{id}/keys/{keyId}/purge` | Permanently delete a key |

### Archetypes

Each uses the latest fitted season unless `season` is given. See [Player Archetypes](player-archetypes/index.md).

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/v1/players/{id}/archetype` | Key or session | Up to three archetypes, the five most similar players and the style-map position. `archetype` is `null` if the player wasn't placed. |
| `GET` | `/v1/archetypes` | Key or session | The season's archetypes and their sizes |
| `GET` | `/v1/archetypes/map` | Key or session | Every placed player's style-map position; `404` if no season has been fitted |

The API also serves BetterAuth's own sign-in routes under `/auth/*`. They appear in Swagger as catch-all `/*splat` entries and are not counted above.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
