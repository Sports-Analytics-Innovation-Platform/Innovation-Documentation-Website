# Demo Guide

A step-by-step walkthrough of the [live webapp](https://sportsanalytics.pages.dev/) for the marking tutor. This is the fastest way to see everything the platform does today.

!!! tip "For the marking tutor"
    This guide takes ~10 minutes to walk through. It covers every built feature mapped to the rubric. If you only have 5 minutes, do steps 1–4 (public features, no login needed). Steps 5–7 require Google sign-in.

!!! warning "Updated 2026-09-23 — this guide was missing a week of Sprint 3 work"
    Steps 9b (Datasets) and 9c (self-service API keys) were added, since neither existed anywhere on this page before, despite both being built and live. The Admin corrections/review workflow (event corrections, batch review, API-consumer management, custom statistics) is real and substantial but requires the `ADMIN`/`ANALYST` role — not walkable by an anonymous marker without one being granted, so it's described rather than given step numbers. Ask the team for temporary access or a screen-share if you want to see it directly, rather than assuming it doesn't exist because it isn't in this walkthrough.

## Before you start

- **Live webapp**: [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/)
- **Live API**: [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health)
- The API is kept warm by a pinger service, so it should respond immediately.

---

## Part 1: Public features (no login required)

### 1. Landing page

Navigate to [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/).

**What to look for:**
- Marketing page with alternating light/dark sections (hero, developer credits, tech-stack marquee) — distinct from the dark app shell every other page uses
- Recent-result widget in the navbar (hidden on mobile)
- Top navbar with four links here: Home, Players, Compare, Teams (Optimizer/Predictions appear once you navigate into the app shell)
- "Get Started" button — signs in with Google and lands on `/home`, a separate personalised dashboard backed by real endpoints since PR #94. Its data is per-user, so it needs a working Google sign-in — see [UI Overview](design/wireframes.md) for what the page shows and for the current sign-in caveat
- Skip-to-content link (try tabbing to see the focus indicator)

### 2. Players list

Click **Players** in the navbar.

**What to look for:**
- Paginated table of NBA players: Name, Team, Position, Jersey number
- **Filter bar** — filter by team and position using the dropdowns
- **Search** — type a player name in the search field to do server-side search across the full dataset
- Player names are clickable links to their profile page

### 3. Player profile

Click any player name.

**What to look for:**
- **Header + stat tiles**: Player headshot (from nba.com CDN), name, team, position, jersey number, height, followed by season stat tiles (PPG, RPG, APG, Games)
- **Points trend chart**: Line chart showing recent game scoring trends (Recharts, themed against CSS variables)
- **Player traits radar**: Five-axis radar chart (Scoring, Rebounding, Playmaking, Defense, Efficiency) normalised onto a 0–100 scale
- **Shooting splits**: FG%, 3P%, FT% as stat tiles
- **Season segment control**: switch between Regular, Play-In, Playoffs, and Finals — stats, splits, and the compare link all follow the selected segment
- **Advanced stats**: true shooting%, effective FG%, assist-to-turnover, plus-minus, usage%, offensive/defensive rating
- **Local stat editing**: click into any counting stat to try a "what-if" value and see it ripple into the derived figures — resets on refresh, segment change, or leaving the page (never saved)
- All charts are themed against the same CSS custom properties as the rest of the UI

### 3b. Player comparison

Click **Compare** in the navbar, or the compare link from a player profile.

**What to look for:**
- Side-by-side comparison of 2–4 players' derived season lines for the same segment picked on the profile page
- Same season-segment control as the profile page — comparing from inside a postseason view compares postseason lines, not regular-season ones

### 4. Teams

Click **Teams** in the navbar.

**What to look for:**
- Table of all 30 NBA teams with **real team logos**
- Paginated with server-side search
- Click any team to see their roster

### 5. Team profile

Click any team.

**What to look for:**
- Team information with logo
- Full roster of players linked to their profiles

---

## Part 2: Authenticated features (Google sign-in required)

### 6. Sign in

Click the **Sign in** button in the navbar (top-right).

**What to look for:**
- Google OAuth redirect (BetterAuth — not hand-rolled auth, per the brief's requirement)
- After signing in, the navbar shows an auth status button with sign-out option
- The Optimizer and Predictions nav links are now accessible

### 7. Predictions

Click **Predictions** in the navbar.

**What to look for:**
- List of games with **Elo-based win probabilities** and **Four Factors predicted score margins**
- Each game shows the predicted winner and margin
- Click a game to see the detail page

### 8. Game detail

Click any game from the Predictions page.

**What to look for:**
- Win probability display
- Predicted score margin
- **Court view** — a basketball court visualisation showing predicted top scorers from both teams positioned by their location on the court. This is a signature visualisation unique to the platform.

### 9. Optimizer

Click **Optimizer** in the navbar.

**What to look for:**
- Fantasy-lineup optimizer showing the latest **MILP-solved** lineup
- Five players selected under a salary cap with their predicted fantasy points
- This demonstrates the optimisation engine: `apps/optimizer` predicts per-player fantasy points and solves a 5-player lineup via MILP (PuLP/CBC)

### 9b. Datasets (added 2026-09-23)

Click **Datasets** in the navbar.

**What to look for:**
- A list of published dataset **releases** — versioned snapshots of season statistics, each with a publish date and row count
- Click a release to see its **schema**: every column's name, type, and description
- **Download** a release's CSV — the response carries an `X-Checksum-SHA256` header; the page compares it against the release's published checksum and shows whether it matches, so you can verify the file you got is byte-identical to what was published
- This is the brief's "datasets should become releases... versioned snapshots published with their schema, a description of every field, and a checksum" requirement, built directly (§1.1.2)

### 9c. Self-service API keys (added 2026-09-23)

Click your account menu → **Profile**, then the **API Keys** section.

**What to look for:**
- Issue your own API key from the UI
- Every keyed request is checked against a per-key rate limit (requests/minute) and daily quota — the page shows your current usage against both
- Try it: `curl -H "X-API-Key: <your key>" https://sportsanalytics-api.onrender.com/v1/players` from a terminal, then again with no header at all (401 `API_KEY_REQUIRED` — every public read now needs either a session or a key, not open access)

### 9d. What you won't see without an admin account

Not walkable in this guide, but real and substantial — ask the team for access if you want to see it directly:

- **Admin event corrections** — an admin can look up any game's full play-by-play, preview the effect of correcting one event (e.g. reassigning an assist to the right player), apply it with a required reason, and undo it later. Corrections recompute exactly the affected player's stats, mark any dataset release covering that data as stale, and leave a full audit trail.
- **Batch review** — an ingestion pull can land as `PENDING_REVIEW` rather than publishing immediately; an admin approves or rejects it before its data appears anywhere public.
- **API consumer management** — issuing/revoking keys for external consumers, viewing usage.
- **Custom statistics** (`ANALYST`/`ADMIN` role) — define a new statistic as an expression over a player's per-game fields (e.g. `points + assists - turnovers`), evaluated for any player/segment. Validated and sandboxed (no arbitrary code execution), versioned so a figure stays reproducible after the definition changes.

---

## Part 3: API verification

### 10. API health check

Visit [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health).

**Expected response:**
```json
{ "status": "ok" }
```

This proves the NestJS backend is live and reachable. The API is hand-written (no auto-generated endpoints), versioned under `/v1/`, and uses BetterAuth for session cookies.

### 11. API endpoints (for reference)

!!! note "Auth column corrected 2026-09-23"
    This table previously marked games/predictions as requiring auth. Since PR #172, **every** row below needs either a signed-in session or an `X-API-Key` header (mandatory, not optional) — "No" below means "no *extra* role beyond that baseline," not "truly open." Optimizer specifically also needs a session (no API-key path).

| Endpoint | Extra auth beyond session-or-key? | What it returns |
|---|---|---|
| `GET /health` | No auth at all | Health check (deprecated in favour of `GET /v1/health`) |
| `GET /v1/players` | No | Paginated player list |
| `GET /v1/players/:id` | No | Player detail |
| `GET /v1/players/:id/stats` | No | Player season stats for one segment (`?seasonType=`, `?asOf=` for a point-in-time cutoff) |
| `GET /v1/players/:id/stats/splits` | No | The same stats for every segment at once |
| `GET /v1/players/leaders` | No | Season leaders by category |
| `GET /v1/players/league-averages` | No | Competition-wide averages |
| `GET /v1/players/aggregates` | No | Group-by aggregate (team/position) over a chosen metric |
| `GET /v1/players/compare` | No | Side-by-side stats for 2–4 players |
| `GET /v1/players/export` | No | Filtered player slice as CSV |
| `GET /v1/teams` | No | Paginated team list |
| `GET /v1/teams/:id` | No | Team detail with roster |
| `GET /v1/games` | No | Game list with predictions joined in (`?seasonType=` to filter) |
| `GET /v1/games/:id` | No | Single game detail |
| `GET /v1/games/:id/events` | No | Paginated raw play-by-play for one game |
| `GET /v1/games/:id/prediction` | No | Win probability + predicted margin |
| `GET /v1/games/export` | No | Filtered game slice as CSV |
| `GET /v1/datasets` | No | Paginated dataset releases |
| `GET /v1/datasets/:version` | No | One release's schema/metadata/checksum |
| `GET /v1/datasets/:version/download` | No | The release's CSV |
| `GET /v1/datasets/diff`, `/changes` | No | Diff/changes-since between releases |
| `GET /v1/optimizer/lineup` | Session required (no API-key path) | Latest MILP-solved fantasy lineup |
| `GET/POST/PUT /v1/custom-statistics` | `ANALYST`/`ADMIN` role | Define/evaluate a custom statistic |
| `GET/POST/DELETE /v1/me/api-keys` | Session required | Self-service API key management |
| `/v1/admin/*` | `ADMIN` role | Batch review, event corrections, consumer management — see step 9d |

Full API documentation: [API Design](design/api-design.md) and the live Swagger UI at `/api/docs`.

---

## What to look for against the rubric

| Rubric criterion | Where to see it in this demo |
|---|---|
| **Non-monolithic** | Frontend (Cloudflare Pages) and API (Render) are separate, independently deployed apps that only communicate over HTTP |
| **Hand-written API** | Every endpoint in the table above is a manually written NestJS controller — no auto-generated CRUD |
| **Authentication** | Google OAuth via BetterAuth (steps 6–9 are auth-gated) |
| **External API integration** | All player/team/game data comes from `nba_api` (stats.nba.com) via the ingestion service |
| **CI/CD** | Every push triggers lint, typecheck, and test on Gitea Actions; deploys to Cloudflare Pages and Render are automatic via GitHub mirror |
| **Responsiveness** | Try resizing your browser window — the layout adapts at mobile/tablet/desktop breakpoints |
| **Accessibility** | Skip-to-content link (tab from page load), `aria-label` on navigation, keyboard-navigable |
| **Optimisation** | The Optimizer page (step 9) demonstrates MILP-based lineup optimisation; predictions use Elo + Four Factors |
| **Event-derived statistics** | Every player stat traces back to `GameEvent` rows, not a typed-in total — see a game's raw play-by-play at `GET /v1/games/:id/events` |
| **Versioned dataset releases** | Step 9b — schema, checksum, diff/changes-since between releases |
| **API keys, rate limits, quotas** | Step 9c — issue a key, watch it get rate-limited |
| **Submission review, corrections, audit trail** | Step 9d — requires admin access to walk through directly |
| **Analyst-defined custom statistics** | Step 9d — requires `ANALYST`/`ADMIN` role |
| **Second external API integration** | A real sportsbook win-probability line (The Odds API) shown alongside the model's own prediction on the game detail page |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5] (2026-09-23: added Datasets/API-keys/Admin coverage, corrected the endpoint auth table, added Sprint 3 rubric rows)*
