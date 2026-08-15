# ADR-002: Auth

**Status:** Accepted (in use since initial scaffold, 2026-08-06)

## Decision

Use **Passport.js** as the authentication library within the NestJS backend, rather than a hand-rolled auth implementation.

## Context

- The brief explicitly requires established practices and libraries for auth, and explicitly disallows writing your own authentication system (§2.1).
- Passport.js integrates natively with NestJS via `@nestjs/passport`, fitting the guard/module structure already adopted (see [ADR-001](adr-001-database.md) context and [Tech Stack](../tech-stack.md)).
- The frontend's `apiClient.ts` calls `fetch` with `credentials: "include"`, implying a cookie/session-based auth flow rather than a bearer-token flow stored in frontend state — consistent with a Passport session or JWT-in-cookie strategy.

## Alternatives considered

**Not recorded**, and this one has a genuine open question attached to it: which Passport **strategy** is actually in use (local + session, JWT, OAuth) isn't documented anywhere I've seen, and the frontend's cookie-based fetch pattern only narrows it down, it doesn't confirm it. This needs to come from whoever built the auth module.

## Consequences

- Password reset and account deletion flows sit on top of whatever Passport strategy is chosen — not yet documented (see [Security](../security.md), which lists these as implemented per the brief but doesn't detail the mechanism).
- Role-based access (`public`/`user`/`analyst`/`admin`, proposed in [Security](../security.md)) is *not yet confirmed as in scope* — this ADR shouldn't be read as implying RBAC exists yet.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
