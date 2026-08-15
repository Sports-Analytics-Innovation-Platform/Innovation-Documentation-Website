# Security

## Authentication

Sign in is implemented via **BetterAuth** with **Google OAuth** (see [Tech Stack](tech-stack.md) and [ADR-002](decisions/adr-002-auth.md)) rather than a hand-rolled auth system, per the brief's requirement to rely on established practices and libraries (§2.1). This replaces the initial scaffold's Passport.js local-strategy + bcrypt implementation.

!!! danger "Open compliance question — needs a team decision"
    The brief requires sign up, sign in, **password reset**, and account deletion (§2.1, verbatim). Google-OAuth-only sign-in has no password on our side to reset. This needs to be resolved — either confirmed with the client/tutor as satisfied by "reset via your Google account," or a second sign-in path added. See [ADR-002](decisions/adr-002-auth.md) for the full detail. Don't let this surface for the first time in a Milestone 4 demo.

- Account deletion cascades through `Session` and `Account` rows tied to a `User`, satisfying the brief's requirement that users can delete their account, not just deactivate it.
- Session tokens, IP address, and user agent are tracked per `Session` row (BetterAuth's default schema) — useful for a "sign out of all devices" feature if the team wants one, not yet built.

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
- All secrets live in Gitea Actions secrets or host environment variables, never in `.env` files that are tracked in git (`.env` is gitignored; `.env.example` documents required variables without values).
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
