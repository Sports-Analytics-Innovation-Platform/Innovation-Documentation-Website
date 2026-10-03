# API Design

The rules every route follows, and why. The full list of routes, with parameters, auth and error codes, is on [API Reference](../api-reference.md) and in the live Swagger UI.

## Principles

| Rule | How it works | Why |
|---|---|---|
| **The API is the only way to the data** | Hand-written NestJS controllers; the database and Supabase are never reachable from the browser. | The brief bans auto-generated backends (§2.1). |
| **Resources, not actions** | Nouns under `/v1/`: `/v1/players/:id`, `/v1/players/:id/stats`, `/v1/me/picks`. `GET` reads, `POST`/`PUT`/`PATCH`/`DELETE` change. | Predictable URLs; an external consumer can guess a route from its neighbours. |
| **Versioned from day one** | Every route sits under `/v1/`. `ApiVersionGuard` rejects any other `Accept-Version` with `406`. A retired route sends `Deprecation`, `Sunset` and `Link` headers before it is removed (`GET /health` → `GET /v1/health`, sunset 31 Mar 2027). | Consumers can't be broken silently. |
| **One error shape** | `{ "error": { "code", "message" } }` for every error, applied by one global filter. Unexpected errors become `500 INTERNAL_ERROR` and are logged. | A client handles errors in one place; stack traces never leak. |
| **Lists are paged** | `page` and `pageSize` (default 25, maximum 100), returning `{ data, page, pageSize, total }`. | No route can return the whole table. |
| **Statistics are derived on request** | Season averages and percentages are calculated from per-game rows when asked for, never stored. | The brief requires derived statistics (§2.4); see [ADR-001](../decisions/adr-001-database.md#design-rules-in-the-schema). |
| **Regular season and playoffs never mix** | Stat routes take `seasonType` (`REGULAR` by default) and echo it back. | A chart can't be labelled with the wrong segment. |
| **Request bodies are validated** | The main write routes check their body with a Zod schema and return `400` with the reason. | Bad input is rejected at the edge, not in the database. |
| **Public reads are cached briefly** | In memory, for up to 5 minutes (1 hour for teams and seasons). Nothing under `/v1/me` is cached ([ADR-004](../decisions/adr-004-caching-strategy.md)). | NBA data changes only when the batch jobs run, and every trip from Render to the database is slow. |
| **Everything is synchronous** | Every response, including dataset publishing, completes within the request. Long NBA pulls are queued and run by the pull worker ([Data Ingestion](ingestion.md)). | Simpler for consumers at this scale. |

## Auth model

Each route needs one of three things ([API Reference: Authentication](../api-reference.md#authentication)):

- **A session or an API key** for public reads (players, teams, games, analytics, datasets). Keys are rate-limited per minute and per day. Signed-out visitors on the site use the site's own key, which the proxy adds ([ADR-003](../decisions/adr-003-hosting-topology.md#how-the-browser-reaches-the-api)).
- **A session** for anything about the signed-in user (`/v1/me/*`), predictions and the optimizer.
- **A role**: `ADMIN` for `/v1/admin/*`, `ANALYST` or `ADMIN` for custom statistics.

Two further rules protect user data:

- **Every `/v1/me/*` query includes the session's user id** in the same `where` clause as the resource id. Another user's row returns `404`, exactly like a row that doesn't exist, because a `403` would confirm the row exists.
- **State-changing requests must come from the site's own origin.** `OriginCheckGuard` runs on every route and refuses `POST`, `PUT`, `PATCH` and `DELETE` requests from other origins, because a session cookie alone doesn't stop cross-site requests ([Security](../security.md)).

The request flow for an authenticated route, `GET /v1/games/:id/prediction`, is drawn on [Architecture](architecture.md#sequence-diagram-get-v1gamesidprediction).

## Not built

- **Asynchronous jobs for consumers.** A large request runs to completion within the request; there is no "submit, then poll" pattern.
- **A live event feed.** Games are ingested after they finish, one batch per game. The platform has one automated data source, not many competing submitters, so the brief's late-arriving-event rules apply only to re-pulls and corrections ([Feature Tiers](feature-tiers.md)).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
