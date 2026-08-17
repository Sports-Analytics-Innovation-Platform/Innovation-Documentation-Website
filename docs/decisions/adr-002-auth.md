# ADR-002: Auth

**Status:** Accepted (superseded Passport.js on 2026-08-06; in use since)

## Decision

Use **[BetterAuth](https://better-auth.com)**, with its Prisma adapter and Google OAuth provider, as the authentication library within the NestJS backend, rather than a hand-rolled auth implementation.

## Context

- The brief explicitly requires established practices and libraries for auth, and explicitly disallows writing your own authentication system (§2.1).
- The team's initial scaffold (2026-08-06) used a Passport.js local-strategy prototype; it was replaced the same day with BetterAuth once the team settled on Google OAuth as the sign-in method — Passport's local strategy is built for username/password, which the project doesn't use.
- BetterAuth's Prisma adapter maps directly onto the project's existing Prisma/Postgres setup (see [ADR-001](adr-001-database.md)), and its `basePath` config lets the handler mount at `/auth` to match the frontend's existing routing convention.
- The frontend's `apiClient.ts` calls `fetch` with `credentials: "include"` — a cookie/session-based flow, which is exactly what BetterAuth manages: it owns session cookies and the OAuth redirect dance end-to-end.
- `main.ts` mounts BetterAuth's handler on a plain Express instance *before* `NestFactory.create()` hands that instance to Nest — Nest's own router (including its catch-all 404 controller) attaches during `NestFactory.create()`, so anything mounted afterward would never be reached.

## Alternatives considered

- **Passport.js** (the original choice) — dropped because its strength is pluggable strategies for username/password and third-party providers via separate strategy packages, which added complexity the project didn't need once Google OAuth was the only sign-in method decided on.
- **Hand-rolled OAuth flow** — explicitly disallowed by the brief (§2.1).

## Consequences

- Sign-in is **Google OAuth only** — no username/password exists, so there's no password to hash, reset, or leak.
- **Password reset does not apply** under this model. **Self-service account deletion is not yet built** — a real gap against the brief's requirement that users can delete their account. See [Security](../security.md) for the open item.
- Role-based access (`PUBLIC`/`USER`/`ANALYST`/`ADMIN`) exists as a custom BetterAuth user field with guards (`SessionAuthGuard`, `RolesGuard`) — both are implemented and unit-tested, but **no route currently uses `@Roles(...)`**, so RBAC isn't enforced anywhere yet in practice.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
