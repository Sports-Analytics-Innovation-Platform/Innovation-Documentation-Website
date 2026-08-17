# Security

## Authentication

Sign-in is implemented via **[BetterAuth](https://better-auth.com)** (see [Tech Stack](tech-stack.md)) rather than a hand-rolled auth system, per the brief's requirement to rely on established practices and libraries (§2.1). This replaced an earlier Passport.js local-strategy prototype on 2026-08-06 — BetterAuth's Prisma adapter and session handling fit the NestJS/Prisma stack more directly, and Google OAuth avoids the project ever handling or storing a password itself.

- Sign-in is **Google OAuth only** — there is no password to hash, reset, or leak, since BetterAuth owns the OAuth redirect flow and session cookies end-to-end.
- `role` (`PUBLIC`/`USER`/`ANALYST`/`ADMIN`) is a custom BetterAuth user field that only a direct database update can set — never settable via sign-up or the Google profile sync.

!!! warning "Known gap against the brief's key requirements (§2.1)"
    The brief requires users be able to "sign up, sign in, reset their passwords, and delete their accounts." Sign-up/sign-in are covered by Google OAuth; **password reset doesn't apply** (no password exists to reset under OAuth-only sign-in), but **self-service account deletion is not yet built** — no route currently lets a signed-in user delete their account and associated data. This needs to land before Milestone 4 at the latest; see [Roadmap](design/roadmap.md).

## Authorization

If the project needs write access for approved users to submit or manage statistics (per the brief's Sport Analytics Tool description), role-based access should be enforced **in our own API middleware/guards**, not left to the database or a third-party service to decide. A proposed role set:

| Role | Can do |
|---|---|
| `public` | Read-only access to player/team/stat endpoints, no account needed |
| `user` | Same as public, plus saved preferences/favourites if we build that |
| `analyst` / `admin` | Submit or correct statistics, manage data quality flags |

!!! note "Not yet confirmed"
    This role table is a proposed design, not a decided one — confirm with the team whether write-access roles are actually in scope before treating this as final, and update this page once they are.

## Third-party data and credentials

Unlike a project that imports a user's own account from a third-party service (e.g. an FPL-style "enter your team" flow, which would mean storing that service's session cookie), our data source (`nba_api`) pulls **public league data** — there's no user credential from a third party to store, and no equivalent liability. Worth stating explicitly in the group report as a deliberate design point, not an oversight: we don't hold third-party credentials because the data source doesn't require them.

## Secrets management

- No secret (API keys, database credentials, tokens) is ever committed — enforced by a pre-commit check and a CI secret scanner (`gitleaks`/`trufflehog`) on every PR, per [Git Methodology](git-methodology.md).
- All secrets live in GitLab CI/CD variables or host environment variables, never in `.env` files that are tracked in git (`.env` is gitignored; `.env.example` documents required variables without values).
- If a secret is ever committed by mistake, the fix is **rotate the credential**, not just remove it from the latest commit — it remains in git history otherwise.

## Transport and API hardening

Not yet implemented — tracked here so it isn't forgotten before Milestone 4:

- **HTTPS/TLS** on whatever production host is chosen (see [Tech Stack](tech-stack.md) — hosting isn't decided yet).
- **Rate limiting** on public API endpoints, both to protect our own DB and because `nba_api` itself depends on stats.nba.com not banning our IP for excessive scraping — see the ingestion note below.
- **Security headers** (e.g. `helmet` middleware in NestJS) — CSP, HSTS, X-Frame-Options, etc.
- **Input validation** on every endpoint (NestJS `ValidationPipe`/DTOs), especially any endpoint that accepts analyst/admin-submitted stat corrections.

## Data ingestion risk

`nba_api` is an **unofficial** client for `stats.nba.com` — it can break or get rate-limited without warning. This isn't a security hole in our own system, but it is an availability risk worth documenting here since a scraping ban would look identical to an attack from the outside if it isn't understood:

- Pull and cache the data we need locally/in our own DB early, rather than hitting `stats.nba.com` live on every user request.
- Any scheduled ingestion job should throttle its own request rate rather than assuming the upstream API will do it for us.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
