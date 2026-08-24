# Demo Guide

A step-by-step walkthrough of the [live webapp](https://sportsanalytics.pages.dev/) for the marking tutor. This is the fastest way to see everything the platform does today.

!!! tip "For the marking tutor"
    This guide takes ~10 minutes to walk through. It covers every built feature mapped to the rubric. If you only have 5 minutes, do steps 1–4 (public features, no login needed). Steps 5–7 require Google sign-in.

## Before you start

- **Live webapp**: [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/)
- **Live API**: [sportsanalytics-api.onrender.com/health](https://sportsanalytics-api.onrender.com/health)
- The API is kept warm by a pinger service, so it should respond immediately.

---

## Part 1: Public features (no login required)

### 1. Home page

Navigate to [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/).

**What to look for:**
- Dark-themed, responsive landing page with hero section stating the product's purpose
- Recent-result widget in the navbar (hidden on mobile)
- Top navbar with five navigation links: Home, Players, Teams, Optimizer, Predictions
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
- All charts are themed against the same CSS custom properties as the rest of the UI

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

| Endpoint | Auth? | What it returns |
|---|---|---|
| `GET /health` | No | Health check |
| `GET /v1/players` | No | Paginated player list |
| `GET /v1/players/:id` | No | Player detail |
| `GET /v1/players/:id/stats` | No | Player season stats |
| `GET /v1/teams` | No | Paginated team list |
| `GET /v1/teams/:id` | No | Team detail with roster |
| `GET /v1/games` | Yes | Game list with predictions joined in |
| `GET /v1/games/:id` | Yes | Single game detail |
| `GET /v1/games/:id/prediction` | Yes | Win probability + predicted margin |
| `GET /v1/optimizer/lineup` | Yes | Latest MILP-solved fantasy lineup |

Full API documentation: [API Design](design/api-design.md)

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

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder [Qoder Lite]*
