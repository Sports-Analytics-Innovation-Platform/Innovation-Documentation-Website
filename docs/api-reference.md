# API Reference (OpenAPI / Swagger)

This page is the canonical API reference for the NBA Analytics API. It documents every endpoint, request/response shape, authentication requirement, and error format — and describes how Swagger UI is served from the running API (see [Setup](#swagger-setup) below).

The API is hosted at **[sportsanalytics-api.onrender.com](https://sportsanalytics-api.onrender.com)**. Swagger UI is live at:

- **Swagger UI:** `https://sportsanalytics-api.onrender.com/api/docs`
- **OpenAPI JSON:** `https://sportsanalytics-api.onrender.com/api-json`
- **OpenAPI YAML:** `https://sportsanalytics-api.onrender.com/api-yaml`

---

## Swagger setup

The NestJS API uses `@nestjs/swagger` to auto-generate an OpenAPI 3.0 specification from decorator metadata on controllers and DTOs.

### Installation

```bash
cd apps/api
npm install @nestjs/swagger
```

### Bootstrap changes (`src/main.ts`)

> **Status: Done.** `@nestjs/swagger@^7` is installed and configured. The Swagger setup below is active in production.

Swagger setup in `src/main.ts`, after the Nest app is created and before `app.listen()`:

```typescript
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";

// ... after app creation and global filters ...

const swaggerConfig = new DocumentBuilder()
  .setTitle("NBA Analytics API")
  .setDescription(
    "REST API for the NBA Analytics & Optimisation Engine. " +
    "Provides player/team/game data ingested from nba_api, " +
    "Elo-based game predictions, Four Factors analysis, and " +
    "MILP fantasy lineup optimisation."
  )
  .setVersion("1.0")
  .addCookieAuth("better-auth.session_token", {
    type: "apiKey",
    in: "cookie",
    name: "better-auth.session_token",
    description: "BetterAuth session cookie. Required for auth-gated endpoints.",
  })
  .addTag("health", "Service health check")
  .addTag("players", "Player data and statistics (public)")
  .addTag("teams", "Team data (public)")
  .addTag("games", "Game data and predictions (auth required)")
  .addTag("optimizer", "Fantasy lineup optimiser (auth required)")
  .build();

const document = SwaggerModule.createDocument(app, swaggerConfig);
SwaggerModule.setup("api/docs", app, document, {
  swaggerOptions: { persistAuthorization: true },
});
```

### Controller decorators

Each controller gets `@ApiTags()` and each method gets `@ApiOperation()`, `@ApiResponse()`, and `@ApiQuery()` decorators. Example for the players controller:

```typescript
import { ApiTags, ApiOperation, ApiResponse, ApiQuery, ApiParam } from "@nestjs/swagger";

@ApiTags("players")
@Controller("v1/players")
export class PlayersController {

  @Get()
  @ApiOperation({ summary: "List players (paginated)" })
  @ApiQuery({ name: "teamId", required: false, description: "Filter by team ID" })
  @ApiQuery({ name: "position", required: false, description: "Filter by position (PG, SG, SF, PF, C)" })
  @ApiQuery({ name: "search", required: false, description: "Search by player name" })
  @ApiQuery({ name: "page", required: false, type: Number, description: "Page number (default: 1)" })
  @ApiQuery({ name: "pageSize", required: false, type: Number, description: "Items per page (default: 25, max: 100)" })
  @ApiResponse({ status: 200, description: "Paginated player list" })
  listPlayers(@Query() query: Record<string, unknown>) { ... }

  @Get(":id")
  @ApiOperation({ summary: "Get player by ID" })
  @ApiParam({ name: "id", description: "Player UUID" })
  @ApiResponse({ status: 200, description: "Player details with team" })
  @ApiResponse({ status: 404, description: "Player not found" })
  async getPlayer(@Param("id") id: string) { ... }
}
```

### DTOs with `@ApiProperty()`

For richer schema documentation, create DTO classes with `@ApiProperty()` decorators. This is optional — the auto-generated spec works from controller metadata alone — but produces better Swagger UI descriptions:

```typescript
import { ApiProperty, ApiPropertyOptional } from "@nestjs/swagger";

class PagedResponseDto {
  @ApiProperty({ description: "Array of result items" })
  data: unknown[];

  @ApiProperty({ description: "Current page number", example: 1 })
  page: number;

  @ApiProperty({ description: "Items per page", example: 25 })
  pageSize: number;

  @ApiProperty({ description: "Total number of matching items", example: 540 })
  total: number;
}

class ErrorResponseDto {
  @ApiProperty({ description: "Error envelope" })
  error: {
    @ApiProperty({ description: "Machine-readable error code", example: "NOT_FOUND" })
    code: string;

    @ApiProperty({ description: "Human-readable error message" })
    message: string;
  };
}
```

---

## Endpoint reference

### Health

#### `GET /health`

Service health check. No authentication required. Used by the topology pinger to verify the API is running.

**Response `200`:**

```json
{ "status": "ok" }
```

---

### Players

All player endpoints are **public** — no authentication required.

#### `GET /v1/players`

Paginated list of players, optionally filtered by team, position, or name search. Results ordered by last name ascending.

**Query parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `teamId` | string | No | — | Filter by team UUID |
| `position` | string | No | — | Filter by position (`PG`, `SG`, `SF`, `PF`, `C`) |
| `search` | string | No | — | Search by first or last name (case-insensitive, space-separated terms) |
| `seasonType` | string | No | `REGULAR` | Season segment (`REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS`). Only takes effect when combined with `participated=true` |
| `participated` | boolean | No | `false` | When `true`, restrict the list to players who appeared in at least one game of `seasonType`. Lets the players list swap to a postseason-only roster |
| `page` | integer | No | `1` | Page number (minimum 1) |
| `pageSize` | integer | No | `25` | Items per page (1–100) |

**Response `200`** — `PagedResult<PlayerWithTeam>`:

```json
{
  "data": [
    {
      "id": "uuid",
      "nbaPlayerId": 2544,
      "firstName": "LeBron",
      "lastName": "James",
      "position": "SF",
      "heightInches": 81,
      "weightLbs": 250,
      "jerseyNumber": "23",
      "headshotUrl": "https://cdn.nba.com/headshots/nba/latest/260x190/2544.png",
      "teamId": "uuid",
      "team": { "id": "uuid", "nbaTeamId": 1610612747, "name": "Los Angeles Lakers", "abbreviation": "LAL", "city": "Los Angeles", "conference": "West", "division": "Pacific", "logoUrl": "..." }
    }
  ],
  "page": 1,
  "pageSize": 25,
  "total": 540
}
```

---

#### `GET /v1/players/:id`

Single player by UUID, with team relationship included.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `id` | string (UUID) | Player ID |

**Response `200`:** Player object (same shape as array items above).

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Player not found" } }
```

---

#### `GET /v1/players/:id/stats`

Season averages and per-game scoring log for a player, both derived at request time from `PlayerGameStat` rows for one season segment.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `id` | string (UUID) | Player ID |

**Query parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `seasonType` | string | No | `REGULAR` | Season segment (`REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS`) |

**Response `200`:** the response echoes back the resolved `seasonType` so a caller can't mislabel a chart it already rendered.

```json
{
  "playerId": "uuid",
  "seasonType": "REGULAR",
  "seasonAverages": {
    "gamesPlayed": 71,
    "minutesPerGame": 35.2,
    "pointsPerGame": 25.7,
    "reboundsPerGame": 7.3,
    "assistsPerGame": 8.0,
    "stealsPerGame": 1.2,
    "blocksPerGame": 0.6,
    "turnoversPerGame": 3.4,
    "fieldGoalsMadePerGame": 9.8,
    "fieldGoalsAttemptedPerGame": 19.4,
    "fieldGoalPercentage": 0.505,
    "threesMadePerGame": 2.1,
    "threesAttemptedPerGame": 6.2,
    "threePointPercentage": 0.341,
    "freeThrowsMadePerGame": 4.0,
    "freeThrowsAttemptedPerGame": 5.3,
    "freeThrowPercentage": 0.756,
    "trueShootingPercentage": 0.598,
    "effectiveFieldGoalPercentage": 0.559,
    "assistToTurnoverRatio": 2.35,
    "plusMinusPerGame": 4.1,
    "usagePercentage": 31.2,
    "offensiveRating": 118.4,
    "defensiveRating": 109.7
  },
  "gameLog": [
    {
      "gameId": "uuid",
      "gameDate": "2025-03-15T00:00:00.000Z",
      "points": 30
    }
  ]
}
```

`assistToTurnoverRatio` is `null` rather than `0` when a player recorded no turnovers (a zero-denominator ratio is undefined, and `0.0` would read as the worst possible ratio, not the best). `plusMinusPerGame`, `usagePercentage`, `offensiveRating`, and `defensiveRating` are `null` for games predating the advanced-boxscore columns, not `0` — a real measurement of an even plus-minus or a 0% usage rate is different from a missing one, and the frontend renders `null` as "—".

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Player not found" } }
```

---

#### `GET /v1/players/:id/stats/splits`

The same derived season line as `/:id/stats` above, but for every season segment at once (`REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS`) in a single request — used by the postseason comparison view so it doesn't have to make four separate calls.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `id` | string (UUID) | Player ID |

**Response `200`:**

```json
{
  "playerId": "uuid",
  "splits": {
    "REGULAR": { "gamesPlayed": 71, "pointsPerGame": 25.7, "...": "..." },
    "PLAY_IN": { "gamesPlayed": 1, "pointsPerGame": 30.0, "...": "..." },
    "PLAYOFFS": { "gamesPlayed": 12, "pointsPerGame": 28.4, "...": "..." },
    "FINALS": { "gamesPlayed": 0, "pointsPerGame": 0, "...": "..." }
  }
}
```

Each value under `splits` has the same `DerivedSeasonAverages` shape as `seasonAverages` above. A segment the player never played in still gets an entry — with zeroed/null stats — rather than being omitted, so the frontend can render every segment tab without a presence check.

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Player not found" } }
```

---

#### `GET /v1/players/compare`

Compare 2–4 players side by side for one season segment. Returns each player's identity plus their derived season line for that segment. **Public** — no authentication required.

**Query parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `ids` | string (comma-separated UUIDs) | Yes | — | Comma-separated list of 2–4 player IDs to compare |
| `seasonType` | string | No | `REGULAR` | Season segment to compare (`REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS`) |

**Example request:**

```
GET /v1/players/compare?ids=a1b2c3d4-e5f6-7890-abcd-ef1234567890,f0e9d8c7-b6a5-4321-0987-654321fedcba&seasonType=PLAYOFFS
```

**Response `200`:** the response echoes back the resolved `seasonType`, same reasoning as `/:id/stats` — comparing two players from inside a postseason view has to compare their postseason lines, or the comparison silently answers a different question than the one on screen.

```json
{
  "seasonType": "REGULAR",
  "players": [
    {
      "player": {
        "id": "uuid",
        "firstName": "LeBron",
        "lastName": "James",
        "position": "SF",
        "team": { "id": "uuid", "name": "Los Angeles Lakers", "abbreviation": "LAL" }
      },
      "seasonAverages": { "gamesPlayed": 71, "pointsPerGame": 25.7, "...": "..." }
    }
  ]
}
```

**Response `400`:**

```json
{ "error": { "code": "BAD_REQUEST", "message": "A comparison needs between 2 and 4 player ids" } }
```

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Player {id} not found" } }
```

---

### Teams

All team endpoints are **public** — no authentication required.

#### `GET /v1/teams`

Paginated list of all NBA teams.

**Query parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `search` | string | No | — | Search by team name, city, or abbreviation |
| `page` | integer | No | `1` | Page number |
| `pageSize` | integer | No | `25` | Items per page (1–100) |

**Response `200`** — `PagedResult<Team>`:

```json
{
  "data": [
    {
      "id": "uuid",
      "nbaTeamId": 1610612747,
      "name": "Los Angeles Lakers",
      "abbreviation": "LAL",
      "city": "Los Angeles",
      "conference": "West",
      "division": "Pacific",
      "logoUrl": "https://cdn.nba.com/logos/nba/1610612747/global/L/logo.svg"
    }
  ],
  "page": 1,
  "pageSize": 25,
  "total": 30
}
```

---

#### `GET /v1/teams/:id`

Single team by UUID.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `id` | string (UUID) | Team ID |

**Response `200`:** Team object (same shape as array items above).

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Team not found" } }
```

---

### Games

All game endpoints require **authentication** via a BetterAuth session cookie (`better-auth.session_token`).

#### `GET /v1/games`

Paginated list of games, most recent first. Each game includes both teams and its prediction (if generated).

**Query parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `seasonType` | string | No | — | Filter to one season segment (`REGULAR`, `PLAY_IN`, `PLAYOFFS`, `FINALS`). Omitted means no filter — all segments returned |
| `page` | integer | No | `1` | Page number |
| `pageSize` | integer | No | `25` | Items per page (1–100) |

Postseason games are excluded from the prediction and optimizer models regardless of this filter — a playoff matchup doesn't behave like a regular-season one statistically, so it's never used as training or projection input.

**Response `200`** — `PagedResult<GameWithTeamsAndPrediction>`:

```json
{
  "data": [
    {
      "id": "uuid",
      "nbaGameId": "0022400001",
      "gameDate": "2025-03-15T00:00:00.000Z",
      "season": "2024-25",
      "homeTeamId": "uuid",
      "awayTeamId": "uuid",
      "homeScore": 112,
      "awayScore": 105,
      "homeTeam": { "id": "uuid", "name": "...", "abbreviation": "LAL", "..." : "..." },
      "awayTeam": { "id": "uuid", "name": "...", "abbreviation": "BOS", "..." : "..." },
      "prediction": {
        "id": "uuid",
        "gameId": "uuid",
        "homeWinProbability": 0.62,
        "homeTeamEloPre": 1580.0,
        "awayTeamEloPre": 1520.0,
        "predictedMarginHome": 5.3,
        "marginMethod": "regression"
      }
    }
  ],
  "page": 1,
  "pageSize": 25,
  "total": 1230
}
```

**Response `401`:** Unauthenticated — no valid session cookie.

---

#### `GET /v1/games/:id`

Full game detail including win probability, predicted margin, and predicted top scorers from both rosters. Everything the game detail page needs in one request.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `id` | string (UUID) | Game ID |

**Response `200`** — `GameDetail`:

```json
{
  "id": "uuid",
  "nbaGameId": "0022400001",
  "gameDate": "2025-03-15T00:00:00.000Z",
  "season": "2024-25",
  "homeTeamId": "uuid",
  "awayTeamId": "uuid",
  "homeScore": 112,
  "awayScore": 105,
  "homeTeam": { "..." : "..." },
  "awayTeam": { "..." : "..." },
  "prediction": { "..." : "..." },
  "predictedScorers": [
    {
      "player": { "id": "uuid", "firstName": "LeBron", "lastName": "James", "position": "SF", "team": { "..." : "..." } },
      "predictedPoints": 27.3,
      "gamesConsidered": 10
    }
  ]
}
```

The `predictedScorers` array contains the top 5 predicted scorers per team (up to 10 total), computed using recency-weighted scoring averages from prior games.

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Game not found" } }
```

---

#### `GET /v1/games/:id/prediction`

Elo win probability and Four Factors predicted margin for a specific game.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `id` | string (UUID) | Game ID |

**Response `200`** — `GamePrediction`:

```json
{
  "id": "uuid",
  "gameId": "uuid",
  "homeWinProbability": 0.62,
  "homeTeamEloPre": 1580.0,
  "awayTeamEloPre": 1520.0,
  "predictedMarginHome": 5.3,
  "marginMethod": "regression",
  "createdAt": "2025-03-14T10:00:00.000Z"
}
```

| Field | Type | Description |
|---|---|---|
| `homeWinProbability` | float | Elo-based win probability for home team, in [0, 1] |
| `homeTeamEloPre` | float | Home team's Elo rating before this game |
| `awayTeamEloPre` | float | Away team's Elo rating before this game |
| `predictedMarginHome` | float? | Four Factors predicted margin (home − away, in points). Null if either team has insufficient history. |
| `marginMethod` | string? | `"regression"` (fitted OLS) or `"heuristic"` (fixed weights, used when < `MINIMUM_GAMES_FOR_REGRESSION` completed games) |

**Response `404`** (game not found):

```json
{ "error": { "code": "NOT_FOUND", "message": "Game not found" } }
```

**Response `404`** (game found but no prediction yet):

```json
{ "error": { "code": "NOT_FOUND", "message": "No prediction has been generated for this game yet — run predict_games.py in apps/predictor." } }
```

---

### Optimizer

Auth required.

#### `GET /v1/optimizer/lineup`

Returns the most recently generated fantasy lineup. The lineup is produced by `apps/optimizer/predict.py` (player fantasy point predictions) and `apps/optimizer/optimize.py` (MILP solve under a salary cap).

**Response `200`:**

```json
{
  "id": "uuid",
  "totalPredictedPoints": 245.7,
  "totalSalary": 48000,
  "budget": 50000,
  "createdAt": "2025-03-14T10:00:00.000Z",
  "slots": [
    {
      "id": "uuid",
      "lineupId": "uuid",
      "playerId": "uuid",
      "player": { "id": "uuid", "firstName": "...", "lastName": "...", "team": { "..." : "..." } },
      "predictedFantasyPoints": 42.5,
      "salary": 12000
    }
  ]
}
```

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "No lineup has been generated yet — run predict.py then optimize.py in apps/optimizer." } }
```

---

#### `GET /v1/optimizer/predictions/:playerId`

Predicted fantasy points for a single player by NBA player ID, as computed by `apps/optimizer/predict.py`.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `playerId` | string | NBA player ID (`nbaPlayerId`, not the internal UUID) |

**Response `200`:** player prediction data (predicted fantasy points and the inputs behind it).

**Response `404`:**

```json
{ "error": { "code": "NOT_FOUND", "message": "Prediction not found" } }
```

---

### Analytics

Added in PR #94 (merged 2026-09-11). Both analytics endpoints are **public** — no authentication required. They describe the model and the leaderboard, not the caller, so every visitor gets the same response and gating them would add nothing.

!!! note "Exact key names"
    The endpoints below were documented from their behaviour and their design notes. The quantities each returns are accurate; for the literal JSON key names and types, read the generated spec at [`/api-json`](https://sportsanalytics-api.onrender.com/api-json), which is produced from the controllers themselves and cannot drift from them.

#### `GET /v1/analytics/model-accuracy`

The prediction model's measured accuracy, deliberately published *against a baseline* rather than on its own — an accuracy figure with nothing to compare it to is not a meaningful number.

**Response `200`** — returns:

- **Model accuracy** over completed games that had a prediction
- **Always-pick-home baseline** accuracy over the same games — the number the model has to beat to be worth anything
- **Brier score** — scores the probability itself, not just the called side, so a confident wrong call costs more than a hedged one
- **Games evaluated** — the denominator, so the accuracy figure can be weighed
- **Forward-prediction count** — how many predictions exist for games that had *not* yet been played when the prediction was made
- **Calibration bands** — predicted probability bucketed against observed hit rate

As of 2026-09-11, over **231** completed games: **64.1%** accuracy against a **58.9%** always-pick-home baseline, Brier score **0.2163**, with **0** forward predictions.

Two honesty mechanisms are part of the endpoint's contract, not just its presentation:

- The **forward-prediction count is reported even when it is zero**, so a backtest is never quietly presented as a live track record. At present, every prediction was made against a game that had already been played.
- A calibration band with too few games reports **"n too small"** rather than a percentage. A hit rate over four games is not a fact, and printing `75%` next to the real bands would read as though it were one.

---

#### `GET /v1/analytics/leaderboard`

Ranks callers by hit rate, **with the Elo model on the board as a benchmark row** rather than as a rival.

**Response `200`** — a ranking of qualifying users (each with `id`, `name`, and their record) plus one row for the model.

Three fairness mechanisms are worth knowing before reading the board:

- **A minimum of 5 calls to qualify.** One lucky call cannot top the board. The model is exempt from the threshold — it is the benchmark, not a competitor for the top spot.
- **Users and the model are scored through the same code path.** Both figures come from one shared summariser rather than two parallel implementations, so they cannot drift apart as either side changes.
- **The comparison is not like-for-like, and the card says so in words.** A user's figure covers only the games they chose to call; the model's covers every game it predicted. The strictly comparable number is the same-subset head-to-head record from `GET /v1/me/picks/record`, which scores both sides over exactly the games that user called.

Only `id` and `name` are read for any user on this board. Email addresses are never selected.

---

### Me

Added in PR #94. Every route in this section **requires authentication** via a BetterAuth session cookie, and every query is **scoped to the session user's id in the same `where` clause as the resource id** — so a valid session plus a guessed resource id still cannot reach another user's rows.

Where a resource exists but belongs to a different user, the response is **`404`, not `403`**. A `403` would confirm that the row exists, which is itself a disclosure.

These are the API's first state-changing routes, so they also sit behind `OriginCheckGuard`, registered globally as an `APP_GUARD` — see [Security](security.md) for why CORS alone does not cover this.

#### `GET /v1/me/challenge/next`

Serves one **completed** game for the user to call, **with the final score withheld**, excluding every game they have already picked.

Three correctness details:

- The score is removed by an explicit **allow-list serializer** — the response is built up from named fields, rather than taking a full game row and deleting the score from it. A deny-list breaks silently the first time a new scoring field is added to the model; an allow-list fails closed.
- **Games that ended in a tie are excluded.** There is no correct call to make on one, and including them deadlocked the "next game" query.
- **Only games that actually have a prediction are served**, since a pick with nothing to grade against the model is not a Beat the Model round.

**Response `200`:** the game, both teams, and the date — without `homeScore` or `awayScore`.

**Response `401`:** no valid session cookie.

---

#### `POST /v1/me/picks`

Submits the user's call on a game. The server grades it against both the real result and the model's prediction, and **only then reveals the score**.

This is the clearest illustration of why the platform requires an account at all: the server can hide a completed game's result from you and still score you on it only if it knows who you are.

The model's numbers are **frozen into the pick row** at this moment (`modelHomeWinProbabilityAtPick`, `modelPredictedMarginAtPick`, `homeTeamEloAtPick`, `awayTeamEloAtPick`) rather than joined at read time — see [ERD](design/erd.md#personalisation-entities) for why.

**Response `200`:** the graded outcome (`CORRECT` / `MISSED`), what the model called, and the now-revealed final score.

**Response `400`:** the body failed Zod validation via `parseBody`.

---

#### `GET /v1/me/picks/record`

The caller's head-to-head record against the model **on the same games** — the strictly like-for-like comparison that the leaderboard's ranking deliberately is not.

---

#### `GET /v1/me/watchlist`

The caller's followed players, each with points, rebounds and assists per game, a five-game scoring trend, and the caller's own scouting note.

Every figure is **derived at request time** from existing `PlayerGameStat` rows. Nothing is stored, so a newly ingested game is reflected on the next load.

**Query parameters:** `page`, `pageSize` — the standard [pagination](#pagination) envelope.

!!! success "Three queries regardless of how many players are followed"
    The whole board costs **three queries**: one page of follows, one grouped aggregate for the averages, and one ordered scan for recent points. It is deliberately *not* a loop through the per-player stats service, which would be an N+1 on a page that loads on every visit to the signed-in home page. Following twenty players costs the same three queries as following two.

---

#### `GET /v1/me/watchlist/ids`

Just the followed player ids, nothing else. This exists so a follow button on a player profile can render its own state in **one** request, instead of fetching the full derived watchlist to answer a yes/no question.

---

#### `POST /v1/me/follows/players/:playerId`

Follow a player. **Idempotent** — following an already-followed player succeeds rather than erroring or creating a duplicate.

---

#### `PATCH /v1/me/follows/players/:playerId`

Replace the scouting note on an existing follow (free text, 500 characters). Clearing the note goes through the same route.

**Response `404`:** the caller does not follow this player. This route deliberately **does not create the follow** — a note written about a player you are not following is more likely a stale client than an intent to follow.

---

#### `DELETE /v1/me/follows/players/:playerId`

Unfollow a player.

**Response `200`:**

```json
{ "playerId": "uuid", "removed": true }
```

---

#### `PUT /v1/me/follows/teams/:teamId`

Follow a team, optionally as the caller's **primary** team. At most one followed team can be primary; setting a new one clears the previous.

`PUT` rather than `POST` because the call is idempotent and fully describes the desired end state of that one follow.

---

#### `DELETE /v1/me/follows/teams/:teamId`

Unfollow a team.

---

#### `GET /v1/me/teams/results`

Recent results for the teams the caller follows, **oriented to the caller's side**: each result names `yourTeam` and the `opponent` and says whether you `won`, rather than reporting home and away and leaving the frontend to work out which side the user cares about.

---

#### `GET` `POST` `DELETE` `/v1/me/saved/comparisons`

Saved player comparisons — a named set of players saved from the Compare tab, so a comparison worth returning to does not have to be rebuilt by hand.

---

#### `GET` `POST` `DELETE` `/v1/me/saved/lineups`

Saved optimizer lineups. Each slot's `salaryAtSave` and `predictedPointsAtSave` are **frozen at save time**, which is what makes the **drift since you saved this** figure computable: the optimizer's predictions are append-and-take-latest, so without a stored baseline there is nothing to have drifted from. See [ERD](design/erd.md#personalisation-entities).

---

## Error format

Every error response uses a structured envelope, applied globally by `AllExceptionsFilter`:

```json
{ "error": { "code": "string", "message": "string" } }
```

| HTTP status | Code | Meaning |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Invalid request body or query parameters |
| `401` | `UNAUTHORIZED` | No valid session cookie |
| `403` | `FORBIDDEN` | Insufficient role permissions |
| `404` | `NOT_FOUND` | Resource not found |
| `500` | `INTERNAL_ERROR` | Unexpected server error (logged server-side) |

---

## Authentication

The API uses **BetterAuth** for session-based authentication. Authenticated endpoints require a valid session cookie:

```
Cookie: better-auth.session_token=<token>
```

Sessions are created via:
- **Google OAuth:** `GET /auth/sign-in/social` → redirect to Google → callback sets session cookie
- **Credential sign-up:** `POST /auth/sign-up/email`
- **Credential sign-in:** `POST /auth/sign-in/email`

See [ADR-002: Auth](decisions/adr-002-auth.md) for the full auth architecture.

---

## Pagination

All list endpoints use the same pagination envelope:

| Parameter | Default | Max | Description |
|---|---|---|---|
| `page` | `1` | — | Page number (1-indexed) |
| `pageSize` | `25` | `100` | Items per page |

Response envelope:

```json
{ "data": [...], "page": 1, "pageSize": 25, "total": 540 }
```

---

## Data sources

All NBA data is ingested from [nba_api](https://github.com/swar/nba_api) (Python client for stats.nba.com) by the `apps/ingestion` service and stored in PostgreSQL (Supabase). The NestJS API reads from the same database — it does not call `nba_api` directly.

Derived statistics (offensive rating, PIE, usage%) are calculated by the team from event-level data, not taken from `nba_api`'s precomputed stats. See [Tech Stack](tech-stack.md) for the full data pipeline description.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5]*
