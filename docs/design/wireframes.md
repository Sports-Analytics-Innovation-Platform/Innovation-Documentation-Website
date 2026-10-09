# UI Overview

The web app as it stands at submission. The team skipped low-fidelity wireframes: it worked from reference dashboards and built the React pages directly, so this page documents the build itself. Screenshots were taken from the live site on 2026-10-03.

![Landing page](../assets/ui/landing-desktop.jpg)

## Navigation

One header (`LandingHeader`) on every page:

- Links: **Home**, **Players**, **Compare**, **Teams**, **Datasets**, **Optimizer**, **Predictions**, **Become Pro** and **Live**, plus **Admin** for an admin.
- Sign in or out on the right.
- Below the `xl` breakpoint the links move into a drawer behind a menu button.
- A skip-to-content link comes first in the tab order.
- A **Read aloud** button, bottom right, reads the page with the browser's speech synthesis.

## Pages

| Route | Access | What it shows | Main endpoints |
|---|---|---|---|
| `/` | Public | Landing: what the product does, how predictions work, developer credits, and Get Started | — |
| `/players` | Public | League leaders, then a sortable, filterable table with a last-8-games sparkline per player | `GET /v1/players`, `/v1/players/leaders` |
| `/players/:playerId` | Public | 12 stat tiles, points trend by season with a projected next season, traits radar with stat definitions, Edit Stats ("what-if", never saved), Follow, Compare | `GET /v1/players/{id}`, `/stats`, `/stats/splits`, `/stats/career` |
| `/compare` | Public | Up to four players side by side for one season segment | `GET /v1/players/compare` |
| `/teams` | Public | A card per team: record, win %, Elo and last five results. Filter by conference or division, sort by Elo. | `GET /v1/teams`, `/v1/teams/records` |
| `/teams/:teamId` | Public | Record, win %, Elo and roster | `GET /v1/teams/{id}` |
| `/datasets` | Public | Versioned releases with schema, CSV download and a checksum check | `GET /v1/datasets`, `/{version}/download` |
| `/live` | Public | Live, upcoming (next 24 hours) and recent (last 18 hours) NBA games | `GET /v1/live/games` |
| `/live/:gameId` | Public | One live or recent game's box score, and its last five minutes of plays while it is live | `GET /v1/live/games/{gameId}` |
| `/home` | Signed in | Dashboard (see below) | `/v1/me/*`, `/v1/analytics/*` |
| `/predictions` | Signed in | Games with Elo win probability and Four Factors margin | `GET /v1/games` |
| `/games/:gameId` | Signed in | Prediction, bookmaker probability from The Odds API, and predicted top scorers on a court view | `GET /v1/games/{id}` |
| `/optimizer` | Signed in | The latest MILP fantasy lineup under a salary cap, with projected points per slot. It can be saved. | `GET /v1/optimizer/lineup`, `POST /v1/me/lineups` |
| `/become-pro` | Signed in, private | The user's own seasons, games, derived line, projected pick and value, and NBA comparables | `GET /v1/me/become-pro` |
| `/profile` | Signed in | Favourite team, followed players, avatar, API keys with usage, Become Pro card, account deletion. `/api-keys` redirects here. | `GET /v1/me`, `/v1/me/api-keys` |
| `/onboarding` | Signed in | First run: pick a username, a favourite team and players to follow | `PATCH /v1/me` |
| `/admin` | Admin | Batch review, event corrections, ingestion schedule, API consumers, reference data and users | `/v1/admin/*` |

The [API Reference](../api-reference.md) lists every endpoint.

### Players

![Players list](../assets/ui/players-desktop.jpg)

Every figure is derived from the app's own box-score rows. The **Regular / Play-In / Playoffs / Finals** control switches the page to that part of the season.

### Player profile

![Player profile](../assets/ui/player-profile-desktop.jpg)

The radar puts five traits (Scoring, Rebounding, Playmaking, Defense, Efficiency) on one 0–100 scale. Clicking a trait lists the stats behind it in plain English, which came from user feedback ([F7](../improvements.md)).

### Compare

![Compare](../assets/ui/compare-desktop.jpg)

Each stat is a pair of bars, and a glossary under them explains every term.

### Teams

![Teams](../assets/ui/teams-desktop.jpg)

The Elo shown is the same pre-game rating stored on each prediction, so it is what the model saw.

### Datasets

![Datasets](../assets/ui/datasets-desktop.jpg)

### Home (signed in)

![Home, signed in](../assets/ui/home-signed-in-desktop.jpg)

The dashboard is built around one idea: **an account has to be necessary, not decorative**. The NBA data is the same for everyone; what you follow and how well you call games is yours.

| Section | What it shows |
|---|---|
| **Beat the Model** | A finished game with the score hidden. Call it, then see the result beside the model's call. |
| **Watchlist** | Followed players with their averages, recent scoring and your own scouting note |
| **Your Teams** | Recent results for your teams, shown from your team's side |
| **Model accuracy** | The model's hit rate against an always-pick-home baseline, its Brier score and calibration |
| **Leaderboard** | Users ranked by hit rate, with the model as a benchmark row |
| **Saved** | Saved comparisons and lineups |
| **Become Pro** | Your projected pick and value, or an invitation to start a season |

### Predictions

![Predictions](../assets/ui/predictions-desktop.jpg)

Played games are marked as a model hit or miss. **Your Matchups** shows the next game for your team and the players you follow.

### Optimizer

![Optimizer](../assets/ui/optimizer-desktop.jpg)

Under the lineup, **Solver checks** shows each constraint and whether the lineup meets it.

### Become Pro

![Become Pro](../assets/ui/become-pro-desktop.jpg)

Only the signed-in user sees this page. [Become Pro](../become-pro/index.md) explains the valuation.

### Profile

![Profile](../assets/ui/profile-desktop.jpg)

### Live

Added on 8 Oct (PR #201). Three sections: **Live**, **Upcoming** and **Recent**, each with an empty message when there are no games. A game card opens `/live/:gameId`, which has each team's box score and, while the game is on, the plays from its last five minutes.

The data comes straight from the NBA's public live feed (`cdn.nba.com`), including preseason, and none of it is stored. The list refreshes every 15 seconds while a game is live or past its start time, and every minute otherwise. A game page refreshes every 15 seconds while its game is live. If the feed is down, the page says so (`503 LIVE_DATA_UNAVAILABLE`).

The live routes need no session or API key. Each request would otherwise cost a key check and a usage-log row, and viewers polling every 15 seconds would use up the site proxy's shared key quota. The API caches each feed file instead, and only ever requests games that are on the schedule.

## Page tutorials

Added on 8 Oct (PRs #202, #203). Every page except the landing page and Live has a short walkthrough. It opens by itself the first time a signed-in user reaches the page. The dialog shows a sketch of the page with numbered steps, and has **Next**, **Back**, **Skip** (this page), **Skip all** (every page) and **Exit**. A **?** button on the page replays it.

Seen tutorials are stored on the account (`UserSeenTutorial`, `PUT /v1/me/seen-tutorials/{tutorialId}`), so a tutorial doesn't reopen on another browser. **Skip all** sets `User.autoOpenTutorials` to false. Each tutorial is plain data in `apps/web/src/components/tutorial/definitions/`, so giving a page a tutorial needs no migration.

## Visual design

Colours are Tailwind `@theme` custom properties in `index.css`. Charts (Recharts) use the same variables, so they follow the theme.

The "locker" palette is a light, warm grey with a leather accent. It was introduced on the landing page and now covers every page, so signing in feels like going further into the same building rather than into another product.

| Token | Value | Use |
|---|---|---|
| `--color-locker-surface` | `#e3e0dc` | Page and card background |
| `--color-locker-leather` | `#a4441c` | Accent and active state. 4.66:1 contrast on the surface, so it is safe for small text. |
| `--color-locker-ink-muted` | `#4a423b` | Secondary text |
| `--color-locker-you` / `-model` | `#c2410c` / `#1f6f9c` | Your pick against the model's, checked as a colour-blind-safe pair |
| `--color-locker-good` / `-bad` | `#15733f` / `#a8202c` | Outcomes. Always shown with a glyph and a word, never colour alone. |

The header stays dark (`--color-landing-ink`, `#14100c`) with the orange `#f97316` accent from the original palette.

## Responsive design

Layouts are mobile-first Tailwind with `sm`, `md`, `lg` and `xl` breakpoints. On a phone, tables keep their key columns and the navbar becomes a menu. The pages were checked at 390 px wide with no horizontal scrolling.

<div class="grid" markdown>

![Players on a phone](../assets/ui/players-phone.jpg){ width="300" }

![Player profile on a phone](../assets/ui/player-profile-phone.jpg){ width="300" }

</div>

## Accessibility

- **Lighthouse Accessibility 100** on Home, Teams, Players and Admin (mobile, 2026-09-29). See [Performance](performance.md).
- `axe-core` checks run in the component tests (`src/test/accessibility.ts`) for Players, Home, Optimizer and Predictions.
- Skip-to-content link, with `tabIndex={-1}` on `<main>` so focus lands there.
- Labelled navigation, with visible focus states.
- Text contrast is measured, and colour is never the only signal.
- Read aloud on every page.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5], Claude-Code[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*
