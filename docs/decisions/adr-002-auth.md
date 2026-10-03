# ADR-002: Auth

**Status:** Accepted, in use since 15 Aug. It replaced the starter code's Passport.js login.

## Decision

Use **BetterAuth** with **Google OAuth** as the only sign-in method, inside the NestJS API. BetterAuth stores users, sessions and linked accounts in Postgres through its Prisma adapter, and serves its own routes under `/auth/*`.

## Context

- **The brief bans hand-written auth** (§2.1). The starter code used Passport.js with a local strategy and bcrypt, which left password hashing and session storage in the team's own code.
- **The team's stack had named BetterAuth with Google from the start.** The starter's Passport login was a mismatch, flagged and replaced between 8 and 15 Aug.
- **BetterAuth fits the stack:** it has a Prisma adapter, so accounts share the database and the migration history ([ADR-001](adr-001-database.md)), and Google sign-in needs only two settings.

## How it is set up

- **Roles are the platform's own.** `role` (`PUBLIC`, `USER`, `ANALYST`, `ADMIN`) is added to BetterAuth's `User` table and marked `input: false`, so neither sign-up nor a Google profile update can set it. NestJS guards enforce roles ([Security](../security.md#authorization)).
- **Account deletion** uses BetterAuth's `deleteUser`, and the database deletes the user's sessions, accounts and data with it.
- **Sessions** are trusted from a signed cookie for 5 minutes, saving a database read per request ([ADR-004](adr-004-caching-strategy.md)).
- **Mounting it took one fix.** NestJS registers its router before other middleware, which hid BetterAuth's routes. The API now creates the Express app itself and mounts `/auth/*splat` on it before NestJS starts.
- **Cross-site sign-in took four.** With the web app and API on different domains, Safari, Firefox and Brave dropped the sign-in cookies. The team set the cookies to `SameSite=None`, lengthened the sign-in state cookie to 10 minutes, turned off BetterAuth's extra state-cookie check (`skipStateCookieCheck`; Google's `state` value is still checked against the database), and finally moved the browser onto one origin with the Cloudflare proxy ([ADR-003](adr-003-hosting-topology.md#how-the-browser-reaches-the-api)).

## Alternatives considered

| Option | Why not |
|---|---|
| **Keep Passport.js** | Leaves password hashing, sessions and reset emails in the team's own code, close to the hand-written auth the brief bans. |
| **Firebase Auth or Supabase Auth** | The brief bans both platforms' generated backends (§2.1), and either would have put users outside our own database. |
| **Email and password alongside Google** | Would meet the brief's password-reset requirement directly, but needs email delivery and verification. Deferred on 14 Sep for lack of time (see below). |

No other library was formally compared.

## Consequences

- **Nobody's password is stored.** A database leak exposes no passwords.
- **Every user needs a Google account.**
- **Password reset is not built.** The brief asks that users can "reset their passwords" (§2.1). With Google-only sign-in, the password is Google's to reset. The client estimated about 80% for the authentication criterion without it (7 Sep), and the team deferred an email-and-password option on 14 Sep because of the effort ([Stakeholder log](../stakeholder-interactions.md)).
- **A revoked session can last up to 5 minutes** because of the cookie cache ([Security](../security.md#session-cookie-cache)).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
