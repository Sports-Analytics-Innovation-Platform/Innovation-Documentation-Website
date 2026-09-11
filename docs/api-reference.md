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

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite]*
