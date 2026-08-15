# ADR-002: Auth

**Status:** Accepted — supersedes the initial Passport.js scaffold (in use 2026-08-06 → migration decided 2026-08-08, merged to `main` by 2026-08-15)

## Decision

Use **BetterAuth** with **Google OAuth** as the sole sign-in method, within the NestJS backend, replacing the initial scaffold's Passport.js local-strategy + bcrypt + server-side sessions.

## Context

- The brief requires established practices/libraries for auth and disallows a hand-rolled system (§2.1) — BetterAuth satisfies this the same way Passport.js did.
- The team's confirmed target stack always specified `Auth: BetterAuth (Google OAuth)`. The initial scaffold shipped with Passport.js first (matching the rest of the Express-era scaffold), and BetterAuth was migrated in afterward once the mismatch between the scaffold and the target stack was flagged.
- Implementation note, in case it matters for the group report's "challenges faced" section: NestJS attaches its own router before other middleware can mount, which was silently swallowing BetterAuth's auth routes. Fixed by handing NestJS a pre-built Express instance instead of letting it construct its own — a genuine integration bug, not a config typo.
- The Prisma schema now includes `User`, `Session`, `Account`, and `Verification` models matching BetterAuth's required core schema, with `role` (the project's own RBAC field, reusing the existing `Role` enum) layered on top and marked non-writable in the BetterAuth config — so a Google profile can't grant itself elevated access.

## Alternatives considered

**Still not formally recorded.** The switch to BetterAuth wasn't a weighed comparison against other options (e.g. Auth.js, Firebase Auth) documented anywhere — it was migrating an existing scaffold to match a stack decision the team had already made elsewhere. If this ADR needs to demonstrate judgement (not just the decision) for the group report, that reasoning needs to come from whoever chose BetterAuth over alternatives in the first place.

## Consequences

- **⚠️ Compliance risk worth flagging to the team directly:** the brief requires, verbatim, that *"Users must be able to sign up, sign in, reset their passwords, and delete their accounts"* (§2.1). With Google OAuth as the only sign-in method, there is no password on our side to reset — BetterAuth's `Verification` table exists in the schema but is explicitly unused while Google OAuth is the only provider. This needs an answer before Milestone 4 marking: either the brief's "reset your password" requirement is being interpreted as not applicable to OAuth-only auth (worth confirming with the client/tutor explicitly, not assuming), or a second credential-based sign-in path needs adding alongside Google OAuth. I'd surface this to the team now rather than let it surface in a demo.
- Account deletion is unaffected — cascading deletes on `Session`/`Account` rows tied to a `User` are already modelled via `onDelete: Cascade`.
- Role-based access (`PUBLIC`/`USER`/`ANALYST`/`ADMIN`) is re-expressed as NestJS guards under BetterAuth, same enforcement approach as the original Passport-based `RolesGuard` — see [Security](../security.md). Whether analyst/admin write-access roles are actually *used* anywhere yet (e.g. a submissions or review feature) is still unconfirmed — no such feature exists in the codebase as of the last snapshot I have.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
