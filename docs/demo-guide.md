# Demo Guide

A step-by-step walkthrough of the [live web app](https://sportsanalytics.pages.dev/). It is the fastest way to see everything the platform does.

!!! tip "For the marking tutor"
    About 10 minutes. Steps 1–6 are public and need no login. Steps 7–11 need a Google sign-in. The admin tools need a role the team has to grant, so they are described [at the end](#what-needs-an-admin-account) instead of walked through.

- **Live web app:** [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/)
- **Live API:** [sportsanalytics-api.onrender.com/v1/health](https://sportsanalytics-api.onrender.com/v1/health). A pinger keeps it warm.

Every page has a **Read aloud** button in the bottom-right corner.

---

## Part 1: Public features (no login)

### 1. Landing page

Open [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/).

- The navbar has Home, Players, Compare, Teams, Datasets, Optimizer, Predictions and Become Pro, plus **Sign in with Google**. On a phone it collapses into a menu.
- Scroll for "How we predict" (Elo win probability, Four Factors margin) and "How we stand out" (dashboard, watchlist, saved comparisons, lineup planning).
- Press Tab on page load to see the **Skip to content** link.

### 2. Players

Click **Players**.

- **League leaders** for PPG, RPG, APG and true shooting, with a minimum-games floor.
- A sortable table: player, team, position, number, PPG, RPG, APG, TS%, and a sparkline of the last 8 games.
- Filter by team, position and minimum games; search by name; sort by any stat.
- The **Regular / Play-In / Playoffs / Finals** control switches every figure to that part of the season.

### 3. Player profile

Click any player.

- Headshot, team, position and 12 stat tiles, including advanced stats (TS%, eFG%, usage, offensive and defensive rating).
- **Points trend by season**, including a projected next season, and a **Player Traits** radar.
- **Edit Stats:** change a counting stat to try a "what-if" and watch the derived figures update. Nothing is saved.
- **Compare** opens the comparison with this player already added.

### 4. Compare

Click **Compare**. Add up to four players and compare their season lines side by side for the chosen segment.

### 5. Teams

Click **Teams**.

- A card per team with its logo, record, win %, Elo rating and recent form. Search and pagination are server-side.
- Click a team for its profile and roster. Each player links to their profile.

### 6. Datasets

Click **Datasets**.

- Versioned **releases** of season data, each with a description, player and game counts and a publish date.
- Expand a release to see its field **schema**.
- **Download** the CSV. The page checks the file's SHA-256 checksum against the published one.

---

## Part 2: Signed-in features (Google sign-in)

### 7. Sign in and Home

Click **Sign in with Google**. Sign-in is BetterAuth with Google OAuth. First-time users pick a username, then land on **Home**:

- **Watchlist** of followed players with their current form.
- **Your teams** with recent results.
- **Beat the Model:** call the winner of a finished game whose score is hidden, then compare your record with the model's. A **leaderboard** ranks users.
- **Saved** comparisons and lineups, and your **Become Pro** card.

### 8. Predictions and game detail

Click **Predictions**.

- Games with an **Elo win probability** and a **Four Factors** predicted margin.
- Open a game for its detail page. It shows the predicted top scorers on a **court view**, and the bookmakers' win probability from The Odds API beside the model's.

### 9. Optimizer

Click **Optimizer**. It shows the latest fantasy lineup. `apps/optimizer` predicts each player's fantasy points and picks five under a salary cap with MILP (PuLP/CBC). You can save the lineup to your account.

### 10. Become Pro

Click **Become Pro**.

1. Start a season. Pick a league year, a position and a competition level, for example NCAA Division II.
2. Log a few games. Enter saves from any field, and **Copy last game** fills the next row.
3. Try an impossible line, such as more makes than attempts. It is refused with a "Fix" note.
4. Keep going to 10 games. The card then shows a projected **draft pick**, a rookie-scale value with a wide range, and the three NBA rookies whose first season was closest.
5. Change the competition level and watch the value move.

Become Pro data is private to you. [Valuation Model](become-pro/valuation-model.md) explains how the figure is produced.

### 11. Your API key

Open your account menu, then **Profile**, then **API Keys**.

- Create a key. The page shows your usage against the per-minute limit and the daily quota.
- Try it from a terminal:

```bash
curl -H "X-API-Key: <your key>" https://sportsanalytics-api.onrender.com/v1/teams
curl https://sportsanalytics-api.onrender.com/v1/teams   # 401 API_KEY_REQUIRED
```

### What needs an admin account

These are built and live but need the `ADMIN` or `ANALYST` role. Ask the team for access or a screen-share.

- **Batch review:** an ingestion pull can land as `PENDING_REVIEW`. An admin approves or rejects it before its games are published.
- **Event corrections:** find a game, preview a corrected play, then apply it with a reason. Only the affected players' stats are recomputed. Every change is audited and can be reverted.
- **API consumers:** issue, revoke and monitor keys for external consumers.
- **Custom statistics** (`ANALYST`): define a statistic as an expression such as `points + assists - turnovers` and evaluate it for any player. Definitions are versioned.

---

## Part 3: The API

- Check that it is up: [/v1/health](https://sportsanalytics-api.onrender.com/v1/health).
- Browse every endpoint in [Swagger UI](https://sportsanalytics-api.onrender.com/api/docs).
- The [API Reference](api-reference.md) lists all 103 operations with their auth rules.

---

## What to look for against the rubric

| Rubric criterion | Where to see it |
|---|---|
| **Non-monolithic** | The web app (Cloudflare Pages) and the API (Render) are deployed separately and talk only over HTTP |
| **Hand-written API** | Every route is a hand-written NestJS controller. See the [API Reference](api-reference.md). |
| **Authentication** | Google OAuth through BetterAuth (step 7) |
| **External APIs** | Game data from `nba_api` (stats.nba.com). Sportsbook lines from The Odds API (step 8). |
| **Responsiveness** | Resize the window, or open the site on a phone |
| **Accessibility** | Skip link, labelled navigation, keyboard focus, Read aloud. Lighthouse Accessibility is 100. |
| **Optimisation and models** | MILP lineup (step 9), Elo plus Four Factors (step 8), Become Pro's fitted model (step 10) |
| **Event-derived statistics** | Every stat is built from play-by-play events, not copied totals |
| **Versioned dataset releases** | Step 6 |
| **API keys, rate limits, quotas** | Step 11 |
| **Review, corrections, audit trail** | [Admin features](#what-needs-an-admin-account) |
| **CI/CD** | Every push runs lint, typecheck and tests. See [CI/CD Pipeline](ci-cd.md). |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
