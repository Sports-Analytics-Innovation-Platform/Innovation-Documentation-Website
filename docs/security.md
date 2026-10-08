# Security

How the platform protects accounts, user data and secrets, and the risks that remain.

## Authentication

Sign-in is **BetterAuth** with **Google OAuth** ([ADR-002](decisions/adr-002-auth.md)), an established library as the brief requires (§2.1). It replaced the starter code's hand-written Passport and bcrypt login.

- **No passwords are stored.** Google holds the credential, so a user resets their password with Google. The brief asks for password reset (§2.1); Brendan said it isn't needed ([ADR-002](decisions/adr-002-auth.md)).
- **Account deletion is real deletion.** Deleting an account removes the user's sessions, linked accounts and all their data, not just a flag (§2.1).
- **The session cookie is first-party.** The browser reaches the API through the site's own domain ([ADR-003](decisions/adr-003-hosting-topology.md#how-the-browser-reaches-the-api)), which fixed sign-in on Safari, Firefox and Brave.
- **Sign-in can expire on a cold start.** BetterAuth's sign-in state expires after 10 minutes, a fixed library value. If the free API host is asleep, waking it uses part of that window, so every page pings the API when it loads to wake it early.

### Session cookie cache

BetterAuth trusts a signed session cookie for up to 5 minutes instead of reading the `Session` and `User` tables on every request ([ADR-004](decisions/adr-004-caching-strategy.md), [Performance](design/performance.md)). So a session revoked from another device, or a role changed in the database, takes effect when the cookie is next checked. Sign-out and account deletion clear the cookie at once.

## Authorization

NestJS guards check every route. Roles are `PUBLIC`, `USER` (the default), `ANALYST` and `ADMIN`. Sign-up and Google profile updates can't change `role`, so nobody can grant themselves access.

| Caller | Can do | Enforced by |
|---|---|---|
| Signed out, with an API key | Read players, teams, games, analytics and datasets | `ApiKeyGuard`; a request with no key and no session gets `401` |
| `USER` | The above, plus predictions, the optimizer, picks, follows, saved items, API keys and Become Pro | `SessionAuthGuard`; every `/v1/me/*` query is scoped to the session user |
| `ANALYST` | Define and evaluate custom statistics | `@Roles(ANALYST, ADMIN)` |
| `ADMIN` | Edit teams, players and users; review batches; correct plays; manage API consumers and keys | `@Roles(ADMIN)` on all of `/v1/admin/*` |

**Another user's data returns `404`,** exactly like an id that doesn't exist, so a response never confirms that someone else's row exists ([API Design](design/api-design.md#auth-model)).

## API hardening

The headers were checked on the live API on 3 Oct.

| Control | Detail |
|---|---|
| **HTTPS everywhere** | Cloudflare and Render provide TLS; the database connection uses TLS. |
| **Security headers** | `helmet` sets Content-Security-Policy, Strict-Transport-Security, X-Frame-Options, X-Content-Type-Options and Referrer-Policy. |
| **Cross-site request protection** | `OriginCheckGuard` refuses `POST`, `PUT`, `PATCH` and `DELETE` requests from origins other than the site's own. CORS alone isn't enough: the browser blocks the reply, not the write. |
| **Rate limits** | Each API key has a per-minute limit and a daily quota, counted in the `ApiUsageLog` table so they survive a restart. Over the limit returns `429`. |
| **Hashed keys** | An API key is shown once; only its SHA-256 hash is stored. |
| **Validated input** | Write routes check their body with Zod. Play corrections have their own rules: a reason is required, and a correction that credits the wrong player is refused. |
| **Private pictures** | Profile pictures sit in a private bucket and are served through links that expire after an hour. |

## Self-reported data: Become Pro

[Become Pro](become-pro/index.md) is the only feature where users enter statistics about themselves. **That data is private to its owner:** there is no leaderboard and no comparison between users, only with real NBA players.

- **It isn't verified, on purpose.** A false figure only misleads the person who entered it. If the feature ever becomes public or comparative, verification must come first.
- **Impossible lines are still rejected,** by the same anomaly checks the admin correction tools use, so a typo can't produce a nonsense valuation.
- **Writes are bounded:** 12 seasons per user, 120 games per season, counts from 0 to 200, text up to 120 characters, and no duplicate games.

## Secrets

- **Nothing secret is committed.** `.env` is ignored by git, and `.env.example` lists the variables without values. Production secrets live in the Render, Cloudflare Pages and Gitea Actions settings. Authors check for secrets before committing ([Git Methodology](git-methodology.md)); CI has no secret scanner yet.
- **A leaked secret is rotated,** not just deleted, because it stays in git history.
- **Scripts can't hit production by accident.** The repo's root `.env` points at production, so the valuation job reads only its own `.env` and fails if it is missing.

### Incident, 11 Sep: a runner token on a branch

A Gitea runner registration file (`ci-runner/data/.runner`, runner `kiran-backup`) was committed on the `LandingPageUpdates` branch. Review caught it and it never reached `main`, but the token is still in that branch's history on the Gitea server. The runner hasn't been removed from Gitea yet; removing it there makes the token useless.

## Third-party data

The NBA data is public, so the platform holds no third-party credentials. `nba_api` is an unofficial client and stats.nba.com can block it without warning. The platform copies what it needs into its own database, so users never trigger calls to stats.nba.com, and ingestion waits between calls (`throttle.py`).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
