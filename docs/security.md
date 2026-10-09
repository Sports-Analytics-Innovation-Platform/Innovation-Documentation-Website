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
| **Rate limits** | Each API key has a per-minute limit and a daily quota. Since 9 Oct (PR #204) they are counted in memory and reloaded from the `ApiUsageLog` table after a restart, so a restart doesn't reset them. Over the limit returns `429`. |
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

### Incident, 11 Sep: a runner token committed

A Gitea runner registration file (`ci-runner/data/.runner`, runner `kiran-backup`) was committed on 11 Sep. Review caught it on the `LandingPageUpdates` branch, but the same file reached `main` the next day in a direct commit (`020f4c6`, 12 Sep) that bypassed review, and an 8 Oct audit found it still there.

Deleting the file is not enough: the token stays in git history. What makes it useless is removing the `kiran-backup` runner on Gitea (and registering a new one if it is still needed). Removing the file from `main`, with `ci-runner/` added to `.gitignore` so a runner's local state can't be committed again, is in review ([PR #207](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/207)). Since 8 Oct `main` also refuses direct pushes, the route this file took.

### Incident, 8 Oct: the Supabase Data API was open

Supabase serves every table in `public` through an auto-generated REST and GraphQL API, authenticated by the project's `anon` key, which isn't a secret. A read-only check of production on 8 Oct found that Supabase's default grants gave the `anon` and `authenticated` roles every privilege on all 41 tables. The 39 row-level security policies, all named `service_role_all`, applied to everyone with `USING (true)`, so they allowed everything rather than restricting it, and two tables had no RLS at all. Anyone with the `anon` key could have read or changed users' emails, session tokens, Google sign-in tokens and API keys.

There is no sign it was used: the `anon` key was never in either repo or in the web app, which doesn't talk to Supabase directly. Nothing in the platform needs those roles, because the API and the Python jobs connect as the tables' owner and Storage uses the service-role key. A migration drops the allow-all policies, enables RLS on every table and revokes the two roles' privileges, now and for future tables ([PR #211](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/211)). Turning off the Data API for the `public` schema in the Supabase dashboard closes it at the source as well.

## Privacy (POPIA)

The platform keeps personal information about signed-in users only: what Google sign-in provides (name, email address, picture link, account ID and sign-in tokens), what they add (username, photo, follows, picks, saved items, Become Pro seasons, custom statistics, API keys), session start and expiry times, and API usage. Browsing without an account stores nothing. The [privacy notice](https://sportsanalytics.pages.dev/privacy) tells users this in plain language. How each POPIA condition is met (changes marked with a PR were in review on 9 Oct and go live when it merges):

| Condition | How |
|---|---|
| Notification (s18) | A public privacy notice at `/privacy`, linked from the landing page, the onboarding username step and the profile ([PR #212](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/212)). Contact: analyticclaritycontact@gmail.com. |
| Minimality (s10) | Sessions no longer store IP addresses or user agents, which nothing read ([PR #212](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/212)). |
| Consent and purpose (s11, s13) | The public Beat the Model leaderboard shows the username a user chose, not the real name from Google ([PR #212](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/212)). Nothing is used for advertising or analytics. |
| Retention (s14) | A daily job deletes expired sessions and verification codes and API usage rows older than 90 days. Deleting an account cascades to everything the user added, and deletes their photo from Storage ([PR #212](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/212)). |
| Security safeguards (s19) | TLS everywhere, photos in a private bucket, API keys stored only as SHA-256 hashes, the database closed to Supabase's public roles ([PR #211](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/211)), and role checks on admin routes. |
| Access, correction and deletion (s23, s24) | **Download my data** on the profile returns everything stored about the user as JSON, minus credentials; the profile edits the user's details; **Delete account** removes everything ([PR #212](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/pulls/212)). |
| Cross-border transfer (s72) | Data is stored in Supabase's London region; the API runs on Render in the US and the site is served through Cloudflare. The notice says so. |

Not covered: registering an Information Officer with the Information Regulator, and reviewing the hosting providers' data-processing terms as operator agreements. Those are paperwork rather than code, and fall to whoever runs the platform beyond the course.

## Third-party data

The NBA data is public, so the platform holds no third-party credentials. `nba_api` is an unofficial client and stats.nba.com can block it without warning. The platform copies what it needs into its own database, so users never trigger calls to stats.nba.com, and ingestion waits between calls (`throttle.py`).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
