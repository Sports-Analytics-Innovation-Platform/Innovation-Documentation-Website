# Demo Guide

A step-by-step walkthrough of the [live web app](https://sportsanalytics.pages.dev/). It is the fastest way to see everything the platform does.

It takes about 10 minutes. Steps 1–8 are public; steps 9–13 need a Google sign-in. The admin tools need a role the team grants, so they are [described at the end](#what-needs-an-admin-account) instead.

- **Live web app:** [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/)
- **Live API:** [sportsanalytics-api.onrender.com/v1/health](https://sportsanalytics-api.onrender.com/v1/health). A pinger keeps it warm.

Every page has a **Read aloud** button in the bottom-right corner.

---

## Part 1: Public features (no login)

### 1. Landing page

Open [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/).

- The navbar has Home, Players, Compare, Teams, Datasets, Optimizer, Predictions, Become Pro, Live and All-Time, plus **Sign in with Google**. On a phone it collapses into a menu.
- Scroll for "How we predict" (Elo win probability, Four Factors margin) and "How we stand out" (dashboard, watchlist, saved comparisons, lineup planning).
- The footer links to the [privacy notice](https://sportsanalytics.pages.dev/privacy).
- Press Tab on page load to see the **Skip to content** link.

### 2. Players

Click **Players**.

- **League leaders** for PPG, RPG, APG and true shooting, with a minimum-games floor. The band follows whatever you filter and search for, so it renames itself ("Leaders · Guards") and shows the leaders among the players on screen.
- A sortable table: player, team, position, number, PPG, RPG, APG, TS%, and a sparkline of the last 8 games.
- Filter by team, position and minimum games; search by name; sort by any stat.
- The **Regular / Play-In / Playoffs / Finals** control switches every figure to that part of the season.

### 3. Player profile

Click any player.

- Headshot, team, position and 12 stat tiles, including usage, plus-minus, and offensive and defensive rating.
- **Points trend by season**, including a projected next season, and a **Player Traits** radar.
- **Edit Stats:** change a counting stat to try a "what-if" and watch the derived figures update. Nothing is saved.
- **Style & similar players:** the player's playing-style archetypes, a league style map, and the five players who play most like them ([Player Archetypes](player-archetypes/index.md)).
- **Compare** opens the comparison with this player already added.

### 4. Compare

Click **Compare**. Add up to four players and compare their season lines side by side for the chosen segment.

### 5. Teams

Click **Teams**.

- A card per team with its logo, record, win %, Elo rating and recent form. Search and pagination are server-side. A line under the heading explains Elo and win %.
- If you are signed in and have picked a favourite team, it appears above the list.
- Click a team for its profile and roster, and a panel of its **injured players** with ESPN's status and estimated return date. Each player links to their profile.

### 6. Datasets

Click **Datasets**.

- Versioned **releases** of season data, each with a description, player and game counts and a publish date. The page explains what a release is and what the download does; a season only appears once some of its games have been played and reviewed.
- Expand a release to see its field **schema**.
- **Download** the CSV. The page checks the file's SHA-256 checksum against the published one, and says whether the file was served as published or rebuilt from current data.

### 7. Live

Click **Live**.

- Games in progress, starting in the next 24 hours, and finished in the last 18 hours, straight from the NBA's live feed. Preseason games count.
- Click a game for its box score and, while it is on, the last five minutes of plays. The page refreshes itself every 15 seconds while a game is live.
- With no NBA games on, all three sections can be empty. That is expected, not an error.

### 8. All-Time leaders

Click **All-Time**, the last link in the navbar.

- Eleven category chips, from points and assists to games played, and a **Regular season / Playoffs** switch.
- Each leader card shows the rank, the career total and a bio: position, height, weight, birth date, school and country, draft and career span.
- Where the NBA only began recording a category partway through its history, the card says when: offensive and defensive rebounds, steals and blocks from 1973-74, and threes from 1979-80.
- The totals are the NBA's career figures, so the list includes players who retired long before the seasons in the app.

---

## Part 2: Signed-in features (Google sign-in)

### 9. Sign in and Home

Click **Sign in with Google**. Sign-in is BetterAuth with Google OAuth. First-time users pick a username, then land on **Home**:

- **Watchlist** of followed players with their current form.
- **Your teams** with recent results.
- **Beat the Model:** call the winner of a finished game whose score is hidden, then compare your record with the model's. A **leaderboard** ranks users.
- **Saved** comparisons and lineups, and your **Become Pro** card.

### 10. Predictions and game detail

Click **Predictions**.

- Games with an **Elo win probability** and a **Four Factors** predicted margin.
- Open a game for its detail page. It shows the predicted top scorers on a **court view**, and the bookmakers' win probability from The Odds API beside the model's.

### 11. Optimizer

Click **Optimizer**. It shows the latest fantasy lineup. `apps/optimizer` predicts each player's fantasy points and picks five under a salary cap with MILP (PuLP/CBC). Lock a player to force them into every lineup, or exclude one to keep them out, then solve again. The page also lists three alternative lineups under the same rules. You can save the lineup to your account.

### 12. Become Pro

Click **Become Pro**.

1. Start a season. Pick a league year, a position and a competition level, for example NCAA Division II.
2. Log a few games. Enter saves from any field, and **Copy last game** fills the next row.
3. Try an impossible line, such as more makes than attempts. It is refused with a "Fix" note.
4. Keep going to 10 games. The card then shows a projected **draft pick**, a rookie-scale value with a wide range, and the three NBA rookies whose first season was closest.
5. Change the competition level and watch the value move.

Become Pro data is private to you. [Valuation Model](become-pro/valuation-model.md) explains how the figure is produced.

### 13. Your profile, API key and data

Open your account menu, then **Profile**. The page carries your favourite team, followed players, avatar, saved items, Become Pro card, API keys, a **Download my data** button (everything stored about your account, as JSON) and account deletion.

Under **API Keys**:

- Create a key. The page shows your usage against the per-minute limit and the daily quota, broken down by endpoint and by day.
- Try it from a terminal:

```bash
curl -H "X-API-Key: <your key>" https://sportsanalytics-api.onrender.com/v1/teams
curl https://sportsanalytics-api.onrender.com/v1/teams   # 401 API_KEY_REQUIRED
```

### What needs an admin account

These are built and live but need the `ADMIN` or `ANALYST` role. Ask the team for access or a screen-share.

- **Batch review:** an ingestion pull can land as `PENDING_REVIEW`. An admin approves or rejects it before its games are published.
- **Event corrections:** find a game, preview a corrected play, then apply it with a reason. Only the affected players' stats are recomputed. Every change is audited and can be reverted. A game with no play-by-play can be submitted by hand as a new batch for review.
- **API consumers:** issue, revoke and monitor keys for external consumers, with usage broken down by endpoint and by day.
- **Custom statistics** (`ANALYST`): define a statistic as an expression such as `points + assists - turnovers` and evaluate it for any player. Every expression a definition has had is kept as a version.

---

## Part 3: The API

- Check that it is up: [/v1/health](https://sportsanalytics-api.onrender.com/v1/health).
- Browse every endpoint in [Swagger UI](https://sportsanalytics-api.onrender.com/api/docs).
- The [API Reference](api-reference.md) lists all 120 operations with their auth rules, including injuries, all-time leaders, queued CSV exports and the on-demand optimizer solve.

---

## What to look for against the rubric

| Rubric criterion | Where to see it |
|---|---|
| **Non-monolithic** | The web app (Cloudflare Pages) and the API (Render) are deployed separately and talk only over HTTP |
| **Hand-written API** | Every route is a hand-written NestJS controller. See the [API Reference](api-reference.md). |
| **Authentication** | Google OAuth through BetterAuth (step 9) |
| **External APIs** | Game data from `nba_api` (stats.nba.com). Sportsbook lines from The Odds API (step 10). Live scores from the NBA's live feed (step 7). Injury status from ESPN (step 5). |
| **Responsiveness** | Resize the window, or open the site on a phone |
| **Accessibility** | Skip link, labelled navigation, keyboard focus, Read aloud. Lighthouse Accessibility is 100. |
| **Optimisation and models** | MILP lineup (step 11), Elo plus Four Factors (step 10), Become Pro's fitted model (step 12) |
| **Event-derived statistics** | Every stat is built from play-by-play events, not copied totals |
| **Versioned dataset releases** | Step 6 |
| **API keys, rate limits, quotas** | Step 13 |
| **Review, corrections, audit trail** | [Admin features](#what-needs-an-admin-account) |
| **CI/CD** | Every push runs lint, typecheck and tests. See [CI/CD Pipeline](ci-cd.md). |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
