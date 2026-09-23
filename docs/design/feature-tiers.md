# Feature Tiers

!!! note "Rewritten 2026-09-23 against the brief's own tier language"
    The previous version of this page described a different framing (an ML-prediction-and-recommendation product) that predates the direction the codebase actually took. The brief for this project (COMS3011A Project 3, "Sport Analytics Tool") is specific: every tier is built around **event-derived statistics** — submission, schema validation, review-before-publication, correction propagation, versioned dataset releases, and (at the advanced tier) analyst-defined custom statistics over the event schema. That's what actually got built, in detail, across Sprints 1–3. The Elo/Four Factors predictor and the fantasy-lineup optimizer are real, working, additional functionality — kept below as bonus work, not the core tier structure, because the brief doesn't ask for them.

Status legend: ✅ done · ⚠️ partially done, with the gap named · ❌ not done · 🔀 implemented, sitting in an open PR not yet merged to `main`

## Basic tier (MVP)

> "Every statistic it publishes should be derived from a record of the individual events... Only approved submitters should be able to submit... A submission... should be checked against the platform's event schema before it is accepted... Every published statistic should be traceable back to the events and the submission behind it." — brief §1.1.1

- ✅ **Event-derived statistics.** `GameEvent` (real per-play NBA data) is the only source `PlayerGameStat` is computed from — `apps/ingestion/derive_player_game_stats.py` and its TypeScript mirror `apps/api/src/admin/derive-player-game-stats.ts`. Nothing is typed in as a total.
- ✅ **Correcting an event brings dependent statistics back in line automatically.** `AdminEventsService.correctEvent` re-derives exactly the affected player(s)' stats in the same transaction as the edit — no manual re-entry (`apps/api/src/admin/{admin-events.service.ts, plan-stat-recompute.ts}`).
- ✅ **Schema-validated submissions with an explanatory rejection.** `apps/ingestion/event_validation.py` checks every incoming action against the platform's event schema and collects every problem (not just the first) into `IngestionBatch.rejectionSummary`, surfaced to admins rather than failing silently.
- ✅ **Traceability.** Every `GameEvent` carries the `IngestionBatch` that wrote it; every manual edit is recorded on `EventCorrection` with who/when/why.
- ✅ **API reads fixtures, events, and derived statistics**, narrowed and paginated: `GET /v1/games`, `GET /v1/games/:id/events`, `GET /v1/players/:id/stats`, `GET /v1/players/leaders`, `GET /v1/players/aggregates`, all filterable, all paginated (`{ data, page, pageSize, total }`).
- ✅ **Stable identifiers.** Every public id is a UUID (`@default(uuid())`), never reused.
- ✅ **Export a filtered slice as a file.** `GET /v1/players/export`, `GET /v1/games/export`, `GET /v1/datasets/:version/download` — all CSV.

**Basic tier: essentially complete.**

## Intermediate tier

> "A batch should be staged and validated before it lands... Resubmitting a batch should not double-count anything, and a batch that fails part way through should resume rather than restart. Submissions should pass a review before publication... a change to the event data should only cause the figures that depend on it to be recomputed... Queries should meet a stated response time... The API should be versioned, should issue keys... and hold them to rate limits and quotas... with repeated reads served from cache... datasets should become releases... versioned snapshots published with their schema, a description of every field, and a checksum." — brief §1.1.2

- ✅ **Idempotent resubmission.** Ingestion upserts `GameEvent` on `(gameId, sequence)` — a re-run overwrites the same rows, never double-counts (`apps/ingestion/play_by_play.py`).
- 🔀 **Batch resume survives a real crash, not just a graceful one.** The resume mechanism itself (`IngestionBatch.resumeAfterSequence`) was real, but only committed once per ~40-minute phase — a crash could silently roll back every already-successful game and the failing game's own resume marker along with it. Fixed to commit per game and before re-raising on failure; **open PR, not yet on `main`** (`fix-ingestion-resume-durability`).
- 🔀 **Review actually gates publication**, not just labels a batch. Until this fix, a `PENDING_REVIEW`/`REJECTED` batch's events and derived stats were already live on the public API the moment ingestion wrote them — the status was an audit label, not a gate. Now `PUBLISHED_GAME_FILTER` excludes a game from every public read until its latest batch is `COMPLETED`. **Open PR, not yet on `main`** (`fix-batch-review-gating`).
- ✅ **Validation catches impossible/conflicting data**, and **corrections leave a history.** `event-correction-rules.ts` validates a correction (e.g. rejects one that leaves a play's credit on the wrong player); `EventCorrection` is an append-only audit trail, and undo is a *new* correction reverting a prior one, never a delete.
- ✅ **Incremental recompute, not full recompute.** `plan-stat-recompute.ts` only recomputes the players who actually appear in the corrected game — never the whole roster, never other games.
- ⚠️ **Figures checked against reference results.** Exact-value unit tests exist and pin real formulas (e.g. `stats.service.spec.ts` against a real Finals boxscore, matched to 3 decimal places against `BoxScoreAdvancedV3`). What's missing: a committed, automated "golden" regression test replaying one real game's full play-by-play against that game's own externally-published box score.
- 🔀 **Performance at the brief's stated scale.** Indexing is real and deliberately reasoned (see `Game`/`PlayerGameStat`/`GameEvent` index comments in `schema.prisma`), but nothing measured response time under load until now. `apps/api/scripts/load-test.mjs` (`npm run load-test`) benchmarks the hot read paths against a stated target (p95 < 300ms, p99 < 800ms). **Open PR, not yet run against a real at-scale database** (`add-api-load-test`) — this closes the "stated target" half of the requirement; someone still needs to run it and record the number.
- ✅ **API versioning is real machinery, not just a URL prefix.** `/v1/` enforced by `api-version.guard.ts`; `@DeprecateEndpoint` + `deprecation.interceptor.ts` emit real `Deprecation`/`Sunset`/`Link` headers (one live example: `GET /health` deprecated in favour of `GET /v1/health`).
- ✅ **API keys, rate limits, quotas — correctly separate from session auth.** `apps/api/src/common/api-key.guard.ts` enforces a DB-backed sliding-window rate limit and daily quota per `ApiConsumer`; self-service keys are issued from the profile page (`ApiKeysSection.tsx`), admin-issued keys from the admin Consumers tab. A signed-in session and an API key are two independent ways to authenticate the same public read routes.
- ✅ **Repeated reads served from cache.** `apps/api/src/cache/response-cache.service.ts` — single-flight, TTL-tiered by data volatility, invalidated on writes. See [Performance](performance.md) for the query-count side of this (a related but separate optimisation pass).
- ✅ **Dataset releases: versioned, schema-documented, checksummed, reproducible.** `DatasetRelease` snapshots a CSV at publish time with a per-field schema description and a SHA-256 checksum; a later correction marks the affected release stale rather than silently rewriting it under the same version name.

**Intermediate tier: substantially complete.** The two real gaps (review-gating, resume durability) are fixed and sitting in open PRs as of 2026-09-23 — see the repo's [main app README](https://github.com/Sports-Analytics-Innovation-Platform) or ask Owen for the PR links. Load testing has tooling but no recorded result yet.

## Advanced tier

> "An analyst should be able to define a new statistic over the event schema itself... validated before they run, contained... and versioned... the platform should also accept a feed from a fixture in progress, and should cope with events that arrive late or out of order... say what a statistic was as of a given date... let a consumer see what changed between two dataset releases... hand large requests off as jobs... offer a feed of changes... retiring versions along a published deprecation path, testing its own contracts, and showing each consumer what it has used... flagging events that look wrong against the history, reconciling submitters that disagree." — brief §1.1.3

- ✅ **Analyst-defined custom statistics.** `apps/api/src/custom-statistics/` — a hand-rolled expression parser (no `eval`, division-by-zero rejected), versioned per edit, role-gated to `ANALYST`/`ADMIN`. Evaluates over 7 event-derived fields (points, rebounds, assists, steals, blocks, turnovers, minutes) — "over the event schema" holds transitively, since those fields are themselves event-derived, rather than exposing arbitrary `GameEvent` field predicates directly.
- ✅ **Point-in-time queries.** `GET /v1/players/:id/stats?asOf=` — only games completed by that timestamp contribute.
- ✅ **Diff / changes-since feed between dataset releases.** `GET /v1/datasets/diff?from=&to=` and `GET /v1/datasets/changes?since=` — a real cursor feed, not a stub.
- ✅ **Deprecation path with a lightweight self-contract test.** `apps/api/test/openapi-contract.e2e-spec.ts` snapshot-tests the API's public surface against its own generated OpenAPI document. Worth being precise: this guards against the team's *own* API silently changing shape — it is not Pact-style consumer-driven contract testing (no external client's own expectations are verified against).
- ⚠️ **Per-consumer usage.** The data exists (`ApiUsageLog`, every keyed request logged with endpoint + timestamp) but the UI only ever shows a raw lifetime count, not a per-endpoint or time-series breakdown.
- ⚠️ **Flagging events that look wrong.** `apps/api/src/admin/stat-anomalies.ts` catches internally-impossible lines (negative stats, made-3s exceeding made-FGs, an invalid rebound split) — genuinely useful, but explicitly not statistical outlier detection against historical baselines, which is what "against the history" implies.
- ✅ **Corrections propagate through downstream aggregates and releases.** Game-level and season aggregates update immediately; a `DatasetRelease` is marked stale in the same transaction as the correction, and a stale release without a stored file refuses to silently rebuild under the old version name.
- ❌ **Live/in-progress feed, late/out-of-order events.** Ingestion is still batch-per-game, run after the fact — not a feed from a fixture in progress. `event_validation.py` actively *rejects* an out-of-order or duplicate action rather than merging it in. Explicitly out of scope per the schema's own doc comment: this project has one automated "submitter" (the pipeline itself), not a multi-human-submitter workflow, so there is nothing to reconcile between disagreeing submitters either.
- ❌ **Async jobs for large consumer requests.** No 202-Accepted/job-id pattern exists for any endpoint — every response, including dataset publication, runs synchronously. The only job/claim-style pattern in the codebase (`IngestionRequest`/`pull_worker.py`) is internal to the ingestion pipeline, not reachable by an external API consumer.

**Advanced tier: strong but genuinely partial.** Custom statistics, point-in-time queries, and dataset diffing are real, working advanced-tier features — a meaningfully large share of this tier is done. The live-feed/multi-submitter reconciliation piece is a deliberate scope cut (documented in the schema itself), not an oversight, and async jobs for consumers is the one clean gap with no groundwork laid yet.

## Bonus, beyond the brief

Not required by the brief, but real, working, and worth presenting:

- **Prediction**: Elo-based home win probability + Four Factors-based predicted margin (`apps/predictor`), with model versioning (`GamePredictionRun`) so a past prediction stays reproducible after the model changes, and a real accuracy ledger (`GET /v1/analytics/model-accuracy`) measured against an always-pick-home baseline.
- **Fantasy-lineup optimizer**: `apps/optimizer` solves a 5-player salary-capped lineup via MILP (PuLP/CBC).
- **Market odds**: a second external API integration (The Odds API), a real de-vigged, bookmaker-averaged win probability shown alongside the model's own prediction — a genuinely demanding baseline ("does our model beat the market") rather than a coin flip.
- **Personalisation layer**: watchlists, followed teams/players, saved comparisons/lineups, a "Beat the Model" pick game with a public leaderboard.

## Still open

- Run `npm run load-test` against a database populated at the brief's stated scale and record the actual number here.
- Merge the two open PRs above (review-gating, resume durability) — until then, `main` still has the bugs they fix.
- Decide whether to invest further in the two Advanced-tier gaps (async consumer jobs, per-consumer usage dashboard) given remaining sprint time, or treat them as explicit stretch goals for the submission milestone.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5]*
