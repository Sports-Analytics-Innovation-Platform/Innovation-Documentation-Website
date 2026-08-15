# API Design

## Confirmed endpoints

These are the only endpoints actually called in the frontend code (`lib/nbaApi.ts`), so they're the only ones this page can state with confidence:

| Method | Path | Called from | Returns |
|---|---|---|---|
| `GET` | `/v1/players` | `PlayersListPage.tsx` | `PagedResult<Player>` |
| `GET` | `/v1/players/:id` | `PlayerProfilePage.tsx` | `Player` |
| `GET` | `/v1/players/:id/stats` | `PlayerProfilePage.tsx` | `PlayerStatsResponse` |

All requests go through a single wrapper (`apiClient.ts`):

```ts
const API_BASE_URL = "/api";
fetch(`${API_BASE_URL}${path}`, { credentials: "include" })
```

So the full path hit on the wire is `/api/v1/players`, etc. — `/api` is a base/proxy prefix in front of the versioned `/v1/` routes. `credentials: "include"` means cookies are sent on every request, which matters for [ADR-002](../decisions/adr-002-auth.md) even though none of these three routes appear to require a logged-in user yet.

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

**Current behaviour, confirmed from code:** `apiClient.ts` throws a generic JS `Error` with a templated message (`Request to ${path} failed with status ${response.status}`) on any non-OK response. There is **no structured error envelope** yet — no error code, no consistent JSON error shape from the backend that the frontend parses.

This is a gap against the brief/rubric's expectation of "a consistent error envelope" (see [Requirements Traceability](../requirements.md) and [Definition of Done](../definition-of-done.md)). Worth deciding the shape early — e.g. `{ error: { code, message, details? } }` — since retrofitting it later means touching every existing endpoint and the frontend's error handling in `PlayersListPage.tsx`/`PlayerProfilePage.tsx`, both of which currently just show a flat string.

## Not yet confirmed / not built

- **Versioning beyond `/v1/`** — no `/v2/` or deprecation policy exists yet, which is fine at this stage but worth deciding before it matters.
- **Teams endpoints** — `Sidebar.tsx` links to `/teams` in the frontend, but there's no corresponding API call anywhere yet.
- **Write endpoints** — nothing in the current code writes data (no POST/PUT/PATCH/DELETE calls exist). If the proposed analyst/admin roles in [Security](../security.md) are real, these don't exist yet.
- **Auth endpoints** — sign up/in/reset/delete aren't called from any page shown so far (no login screen exists in `App.tsx`'s routes).
- **OpenAPI/Swagger spec** — not confirmed to exist. The brief and [Requirements Traceability](../requirements.md) expect the hand-written API to be documented; nothing in the frontend code confirms whether this exists on the backend.
- **Rate limiting / pagination parameters** — `PagedResult` shows the response shape has `page`/`pageSize`, but nothing confirms what request parameters actually control them (query string? `?page=2`? unconfirmed).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
