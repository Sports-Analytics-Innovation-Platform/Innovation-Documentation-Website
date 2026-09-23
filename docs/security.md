# Security

## Authentication

Sign in is implemented via **BetterAuth** with **Google OAuth** (see [Tech Stack](tech-stack.md) and [ADR-002](decisions/adr-002-auth.md)) rather than a hand-rolled auth system, per the brief's requirement to rely on established practices and libraries (§2.1). This replaces the initial scaffold's Passport.js local-strategy + bcrypt implementation.

!!! danger "Open compliance question — needs a team decision"
    The brief requires sign up, sign in, **password reset**, and account deletion (§2.1, verbatim). Google-OAuth-only sign-in has no password on our side to reset. This needs to be resolved — either confirmed with the client/tutor as satisfied by "reset via your Google account," or a second sign-in path added. See [ADR-002](decisions/adr-002-auth.md) for the full detail. Don't let this surface for the first time in a Milestone 4 demo.

- Account deletion cascades through `Session` and `Account` rows tied to a `User`, satisfying the brief's requirement that users can delete their account, not just deactivate it.
- Session tokens, IP address, and user agent are tracked per `Session` row (BetterAuth's default schema) — useful for a "sign out of all devices" feature if the team wants one.

!!! warning "Known issue: intermittent 'sign-in link expired' — mitigated, not eliminated"
    BetterAuth's OAuth state row hard-expires 10 minutes after sign-in starts, hardcoded in the library with no config option (checked against the latest release, `better-auth@1.7.4`). A normal Google sign-in finishes in seconds, so hitting that ceiling often points at Render's free-tier cold start eating into the window between the pinger's hits, not the 10-minute cap itself being too short. Mitigated 2026-09-11 by pinging `/health` on every page's header mount to give the API a head start waking up before a visitor reaches the sign-in button — this reduces exposure but can't guarantee the API is warm, since a ping doesn't block the click that follows it.

### Session cookie cache

Since PR #124 (13 September 2026), BetterAuth verifies a **signed session cookie** for up to 5 minutes instead of reading the `Session` and `User` tables on every authenticated request. That removed a database round trip from every signed-in page load — see [Performance](design/performance.md) and [ADR-004](decisions/adr-004-caching-strategy.md) for the reasoning.

!!! warning "A revoked session stays valid for up to 5 minutes"
    This is a security property, not only a performance one, so it is recorded here as well as on the performance page. A session revoked from another device, or a `role` changed directly in the database, does not take effect until the cookie is next re-verified — a window of up to five minutes.

    **Not affected:** sign-out and account deletion clear the cookie immediately, so the paths a user actually controls stay instant. **Would be affected:** the "sign out of all devices" feature suggested above would no longer be immediate under this setting without shortening the window or bypassing the cookie cache for that one action. Worth deciding deliberately if that feature gets built, rather than discovering it during testing.

## Authorization

Role-based access control is implemented via NestJS guards. The schema defines four roles (`PUBLIC`, `USER`, `ANALYST`, `ADMIN`) with `role` defaulting to `USER` and marked non-writable in the BetterAuth config so a Google profile can't grant itself elevated access.

| Role | Can do | Status |
|---|---|---|
| `public` (no session) | Read-only access to player/team/game/dataset endpoints — **but no longer without a credential.** Since PR #172, a request needs either a signed-in session or a valid `X-API-Key`; a truly anonymous request gets `401 API_KEY_REQUIRED` | Implemented |
| `user` | The above, plus predictions and optimizer endpoints (`SessionAuthGuard`), self-service API keys, watchlists/picks/saved comparisons | Implemented |
| `analyst` | Define and evaluate custom statistics over event-derived fields | Implemented — `/v1/custom-statistics/*` behind `@Roles(ANALYST, ADMIN)` |
| `admin` | Edit `Team`/`Player`/`User` rows, review and approve/reject ingestion batches, correct individual play-by-play events (preview/apply/undo), manage external API consumers and keys | Implemented and merged — `/v1/admin/*`, all behind `SessionAuthGuard` + `@Roles(ADMIN)` |

## Third-party data and credentials

Unlike a project that imports a user's own account from a third-party service (e.g. an FPL-style "enter your team" flow, which would mean storing that service's session cookie), our data source (`nba_api`) pulls **public league data** — there's no user credential from a third party to store, and no equivalent liability. Worth stating explicitly in the group report as a deliberate design point, not an oversight: we don't hold third-party credentials because the data source doesn't require them.

## Secrets management

- No secret (API keys, database credentials, tokens) should be committed — enforced by a pre-commit check per [Git Methodology](git-methodology.md). A CI secret scanner (`gitleaks`/`trufflehog`) on every PR is the intended backstop but is **not yet in the pipeline** ([CI/CD Pipeline](ci-cd.md)), so the manual check is currently the only control — see the incident below for exactly the kind of thing that control is supposed to catch.
- All secrets live in Gitea Actions secrets, Render environment variables, or Cloudflare Pages environment variables — never in `.env` files that are tracked in git (`.env` is gitignored; `.env.example` documents required variables without values).
- If a secret is ever committed by mistake, the fix is **rotate the credential**, not just remove it from the latest commit — it remains in git history otherwise.

!!! danger "Incident: a live runner token reached a PR branch (2026-09-11)"
    `ci-runner/data/.runner` — an `act_runner` registration file (name `kiran-backup`, self-hosted, a working registration token for `sdp.ms.wits.ac.za`) — was committed on the `LandingPageUpdates` branch, almost certainly local runner state committed by accident rather than application code. It was caught during review before merging and excluded from the merge into `main`, so it never reached `main`'s history. **The token itself is still live and still needs rotating** — removing the file from the merge doesn't invalidate it, per the policy above; it remains valid on the branch's own history on the Gitea server regardless. Whoever administers the runner registrations needs to reset or delete the `kiran-backup` runner entry. Filed here rather than only in the AI usage ledger since this is exactly the class of incident the secrets-management policy above exists to prevent, and the manual pre-commit check is what should have caught it on that branch.

## Transport and API hardening

Implemented:

- **HTTPS/TLS** — Cloudflare Pages provides managed TLS for the frontend. Render provides managed TLS for the API. Supabase connections use TLS via the pooled connection string.
- **CORS** — configured in `apps/api/src/main.ts` with `credentials: true` and `origin: process.env.WEB_ORIGIN`, restricting cross-origin requests to the known frontend domain(s).
- **Request body validation** — a shared `parseBody` helper runs a Zod schema over every request body on the write routes added in PR #94, rejecting anything that doesn't match the schema before it reaches a handler.

**Rate limiting: implemented, not still open.** Every API-key-authenticated request is checked against a DB-backed sliding-window limit (requests/minute) and a daily quota, per `ApiConsumer` (`apps/api/src/common/api-key.guard.ts`, backed by `ApiUsageLog`) — hand-rolled rather than `@nestjs/throttler`, so the limit survives a server restart. Exceeding either returns `429` (`RATE_LIMIT_EXCEEDED` / `QUOTA_EXCEEDED`). This also protects `nba_api`/`stats.nba.com` indirectly, since a rate-limited public consumer can't drive proportional ingestion load.

Still not yet implemented — tracked here so it isn't forgotten before Milestone 4:

- **Security headers** (e.g. `helmet` middleware in NestJS) — CSP, HSTS, X-Frame-Options, etc. Note: `helmet` is already imported in `main.ts` but the full header suite should be verified in production.
- **A project-wide `ValidationPipe`/DTO layer.** Request bodies are validated (see `parseBody` above), but query and path parameters are still parsed per-controller rather than through one uniform pipe. Worth noting this caveat's original context has changed: analyst/admin-submitted stat corrections are now live (the admin event-corrections workflow, `AdminEventsService.correctEvent`) and have their own dedicated validation (`event-correction-rules.ts` — rejects a correction that leaves a play's credit on the wrong player, requires a reason) rather than routing through `parseBody`. The uniform-pipe gap is still real, just no longer blocking on that specific feature.

## Data ingestion risk

`nba_api` is an **unofficial** client for `stats.nba.com` — it can break or get rate-limited without warning. This isn't a security hole in our own system, but it is an availability risk worth documenting here since a scraping ban would look identical to an attack from the outside if it isn't understood:

- Pull and cache the data we need locally/in our own DB early, rather than hitting `stats.nba.com` live on every user request. The ingestion service (`apps/ingestion`) already does this — data is fetched into Postgres, and the API reads from Postgres.
- Any scheduled ingestion job should throttle its own request rate rather than assuming the upstream API will do it for us. The ingestion service includes a `throttle.py` module for this purpose.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5] (2026-09-23: rate limiting and role-authorization status corrected)*
