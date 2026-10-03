# ADR-004: Caching Strategy

**Status:** Accepted, in use since 13 Sep (PR #124).

## Decision

Cache public API reads **in memory, inside the NestJS API**, with short lifetimes, instead of adding Redis or a hosted cache. Separately, let **BetterAuth trust its session cookie for 5 minutes**, so signed-in requests stop re-reading the `Session` and `User` tables.

Nothing under `/v1/me` is cached.

## Context

- The API is **one Render free-plan instance** reaching Supabase through a connection pooler, so every database round trip is slow, and large reads use up a free plan's compute.
- Before this, **nothing was cached**: every page view, tab switch and signed-in request reached Postgres ([Performance](../design/performance.md#what-the-audit-found)).
- The NBA data is **almost read-only.** It changes only when the Python jobs run, so a short cache lifetime costs nothing in correctness.
- The instance has about 512 MB of memory and sleeps when idle, so the cache must be bounded and must cope with being emptied.

## Alternatives considered

| Option | Why not |
|---|---|
| **Redis or Upstash** | A shared cache solves sharing between processes, and there is only one. It would add a network hop to every cache read and another service to run. Losing the cache on a cold start is fine: the first request refills it. |
| **HTTP caching through a CDN, or Postgres materialised views** | Reasonable, but not evaluated. |

## Consequences

- **Public data can be up to 5 minutes old** after a data job (1 hour for teams and seasons). Nothing a user owns is ever stale, and a new pick clears the leaderboard at once.
- **A revoked session can last up to 5 minutes.** A session revoked from another device, or a role changed in the database, takes effect when the cookie is next checked. Sign-out and account deletion are immediate ([Security](../security.md#session-cookie-cache)).
- **The cache is off in tests,** because the end-to-end suite reseeds one shared database between specs.
- **Cached values are shared,** so callers must not modify them.
- **It only works for one instance.** With several, each would have its own cache, users could see different data, and clearing the leaderboard would clear only one. That is when to reconsider Redis.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5], Claude-Code[Claude Opus 5.5]*
