# ADR-004: Caching Strategy

**Status:** Accepted — implemented on branch `DBCallRate` (PR #124), 13 September 2026

## Decision

Cache public API reads in an **in-process, in-memory cache inside the NestJS API**, with short TTLs, rather than adding Redis or any hosted cache service. Separately, enable **BetterAuth's session cookie cache** (5 minutes) so signed-in requests stop re-reading the `Session` and `User` tables on every request.

Per-user data is explicitly excluded: nothing under `/v1/me` is cached.

## Context

- The API runs as a **single Render free-tier instance** and reaches Supabase Postgres through the transaction pooler. Each round trip pays real latency, and large row pulls consume Supabase compute and egress on a free plan.
- Before this work, **nothing was cached anywhere** — every page view, tab refocus and signed-in request reached Postgres. See [Performance](../design/performance.md) for the full audit and the measured before/after.
- The NBA data is **almost read-only**: games, stats, predictions and lineups change only when the Python batch jobs run. Between runs the same query returns the same answer, which is what makes a short TTL free in correctness terms.
- The free instance has roughly 512 MB of memory and spins down when idle, so any in-process cache has to be bounded and has to tolerate being emptied without warning.

## Alternatives considered

**Redis or Upstash** was the one alternative genuinely weighed, and it was rejected. A network-attached cache solves the problem of sharing cached state *across processes*, and there is only one API process. Adding it would put a network hop in front of every cache read and introduce a service to run, monitor and pay for, in exchange for a guarantee this deployment does not need. The fact that the cache empties when the free instance spins down is acceptable: the first request after a cold start repopulates it, and that request was going to hit Postgres under the old behaviour anyway.

**Other options were not formally compared.** HTTP-level caching (`Cache-Control` headers with a CDN in front of the API) and materialised views in Postgres would both be reasonable things to evaluate, and neither was. If this ADR needs to demonstrate a weighed comparison rather than a reasoned default, that evaluation still has to happen — it is recorded here as not done rather than implied.

## Consequences

- **A staleness contract now exists, and it is documented rather than incidental.** Public NBA data can be up to 5 minutes stale after an ingestion or predictor run (1 hour for teams and seasons). Nothing a user owns is ever stale, and a new pick invalidates the leaderboard immediately.
- **⚠️ A revoked session stays valid for up to 5 minutes.** This is the real cost of the cookie cache: a session revoked from another device, or a role changed directly in the database, does not take effect until the cookie is re-verified. Sign-out and account deletion clear the cookie immediately, so the common paths are unaffected — but "sign out of all devices", if it is ever built, will not be instant without also shortening or bypassing this window. Flagged in [Security](../security.md#session-cookie-cache) as well, because it is a security property and not only a performance one.
- **The cache must stay off in tests.** The e2e suite truncates and reseeds one shared database, so a cache surviving between specs would serve a previous test's data. It disables itself under Vitest rather than relying on every spec to remember.
- **Cached values are shared references.** Callers must treat them as read-only; a caller that mutated one would corrupt every later reader. Every current caller builds new objects instead, and the service carries a comment saying so.
- **This decision is scoped to a single instance and does not survive scaling.** If the API is ever run with more than one replica, the cache becomes per-replica: two users can sit in different TTL windows and see different data for the same public request, and a targeted invalidation (a pick clearing the leaderboard) would only clear the replica that served it. That is the point at which the Redis option above should be reconsidered rather than assumed still-rejected.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5]*
