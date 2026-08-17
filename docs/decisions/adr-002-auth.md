# ADR-002: Auth

**Status:** Accepted — supersedes the initial Passport.js scaffold (in use 2026-08-06 → migrated to BetterAuth the same day)

## Decision

Use **[BetterAuth](https://better-auth.com)**, with its Prisma adapter and Google OAuth provider, as the sole sign-in method within the NestJS backend, replacing the initial scaffold's Passport.js local-strategy + bcrypt + server-side sessions.

## Context

- The brief requires established practices/libraries for auth and disallows a hand-rolled system (§2.1) — BetterAuth satisfies this the same way Passport.js did.
- The team's confirmed target stack always specified `Auth: BetterAuth (Google OAuth)`. The initial scaffold shipped with Passport.js first (matching the rest of the Express-era scaffold), and BetterAuth was migrated in the same day once the mismatch between the scaffold and the target stack was flagged — Passport's local strategy is built for username/password, which the project doesn't use.
- BetterAuth's Prisma adapter maps directly onto the project's existing Prisma/Postgres setup (see [ADR-001](adr-001-database.md)), and its `basePath` config lets the handler mount at `/auth` to match the frontend's existing routing convention.
- The frontend's `apiClient.ts` calls `fetch` with `credentials: "include"` — a cookie/session-based flow, which is exactly what BetterAuth manages: it owns session cookies and the OAuth redirect dance end-to-end.
- **Implementation note** (worth keeping for the group report's "challenges faced" section): `main.ts` mounts BetterAuth's handler on a plain Express instance *before* `NestFactory.create()` hands that instance to Nest — NestJS attaches its own router (including its catch-all 404 controller) during `NestFactory.create()`, so anything mounted afterward would silently never be reached. This wasn't a config typo, it was a real integration bug caught during the migration.
- The Prisma schema now includes `User`, `Session`, `Account`, and `Verification` models matching BetterAuth's required core schema, with `role` (`PUBLIC`/`USER`/`ANALYST`/`ADMIN`, the project's own RBAC field) layered on top and marked non-writable in the BetterAuth config — so a Google profile sync can't grant itself elevated access.

## Alternatives considered

- **Passport.js** (the original choice) — dropped because its strength is pluggable strategies for username/password and third-party providers via separate strategy packages, which added complexity the project didn't need once Google OAuth was the only sign-in method decided on.
- **Auth.js / Firebase Auth** — not formally compared anywhere; the switch to BetterAuth was migrating an existing scaffold to match a stack decision the team had already made elsewhere, not a weighed evaluation. If this ADR needs to demonstrate judgement (not just the decision) for the group report, that reasoning needs to come from whoever chose BetterAuth over alternatives in the first place.
- **Hand-rolled OAuth flow** — explicitly disallowed by the brief (§2.1).

## Consequences

- **⚠️ Compliance risk worth flagging to the team directly:** the brief requires, verbatim, that *"Users must be able to sign up, sign in, reset their passwords, and delete their accounts"* (§2.1). With Google OAuth as the only sign-in method, there is no password on our side to reset — BetterAuth's `Verification` table exists in the schema but is explicitly unused while Google OAuth is the only provider. This needs an answer before Milestone 4 marking: either the brief's "reset your password" requirement is interpreted as not applicable to OAuth-only auth (worth confirming with the client/tutor explicitly, not assuming), or a second credential-based sign-in path needs adding. Surface this to the team now rather than let it surface in a demo.
- **Self-service account deletion is not yet built** — checked directly against `apps/api/src`, no route triggers it. The Prisma schema *models* cascading deletes on `Session`/`Account` rows tied to a `User` (`onDelete: Cascade`), which would clean up correctly if a `User` row were ever deleted — but nothing in the API currently exposes a way for a signed-in user to actually delete their own account. Schema-level cascade support isn't the same as the feature existing; don't conflate the two in the group report.
- Role-based access (`PUBLIC`/`USER`/`ANALYST`/`ADMIN`) is re-expressed as NestJS guards under BetterAuth (`SessionAuthGuard`, `RolesGuard`), same enforcement approach as the original Passport-based guard — both are implemented and unit-tested, but **no route currently uses `@Roles(...)`**, so RBAC isn't enforced anywhere yet in practice. Whether analyst/admin write-access roles are actually needed (e.g. a submissions or review feature) is still unconfirmed.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
