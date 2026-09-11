# UI Overview

!!! success "Confirmed: UI Overview, not Wireframes"
    This project didn't go through a formal low-fidelity wireframing stage — the team found reference dashboards for inspiration and used AI to help generate the initial React implementation directly. This page documents the current build, not a planning artifact.

## Navigation

Header component (`LandingHeader`, renamed from the original `Navbar` when it was unified across routes — PR #50) with two link sets: the landing page shows just **Home**, **Players**, **Compare**, **Teams** (`overlaysContent` mode, since it sits over a full-bleed hero); every other route adds **Optimizer** and **Predictions**. The navbar also includes a recent-result widget (hidden on small screens) and an auth status button (sign in/sign out). A skip-to-content link is available for keyboard navigation.

The sidebar from the initial scaffold was replaced with the top navbar during Sprint 1 (week of 11 Aug) to accommodate the growing number of pages.

## Screens

### Landing (`/`)

The public marketing page: hero section with a screenshot cascade, a developer-credits and tech-stack marquee, and a "Get Started" call to action that signs in with Google and lands on `/home`. The first cascade panel shows a real screenshot of the signed-in home page (PR #88); the other two (player profile, optimizer) are still placeholder blocks awaiting their own screenshots.

### Home (`/home`)

The signed-in dashboard ("The Locker"). Built on the landing page's light palette rather than the dark app shell, deliberately, so signing in reads as walking further into the same building.

Every figure on the page is now live (PR #94, merged 2026-09-11), replacing the `components/home/placeholderData.ts` shell that PR #87 shipped. The premise the page is built on is that **an account has to be necessary, not decorative**: the NBA data is identical for every visitor, but which players you follow, what you wrote about them, and how well you call games are yours alone. Nothing derived is stored, so a newly ingested game shows up here immediately.

| Section | What it shows | Backed by |
|---|---|---|
| **Beat the Model** | A completed game with its final score withheld, for you to call. Submitting grades your call against both the real result and what the Elo model predicted, and only then reveals the score | `GET /v1/me/challenge/next`, `POST /v1/me/picks`, `GET /v1/me/picks/record` |
| **Your Watchlist** | Followed players with points, rebounds and assists per game and a five-game scoring trend, plus your own scouting note per player (editable, 500 characters) | `GET /v1/me/watchlist` |
| **Your Teams** | Recent results for the teams you follow, oriented to your side — your team, the opponent, whether you won — rather than home/away | `GET /v1/me/teams/results` |
| **Model accuracy ledger** | The model's real accuracy against an always-pick-home baseline, its Brier score, and per-band calibration | `GET /v1/analytics/model-accuracy` |
| **Accuracy leaderboard** | Callers ranked by hit rate, with the Elo model on the board as a benchmark row rather than a rival | `GET /v1/analytics/leaderboard` |
| **Saved shelf** | Saved player comparisons, and saved optimizer lineups showing how far each slot's salary and predicted points have drifted since you saved it | `GET /v1/me/saved/comparisons`, `GET /v1/me/saved/lineups` |

Beat the Model is the page's focal action, and the clearest illustration of why the account is required at all: the server can only hide a completed game's score from you *and still score you on it* if it knows who you are.

Two sections from the original shell — **"Add to Locker"** and **"Jump Back In"** — were removed. Both were layout with nothing behind them: one searched nothing, the other listed views nobody had recorded. Removing "Add to Locker" left no way to *start* a follow, so an "Add to watchlist" control was added to the [player profile](#player-profile-playersplayerid) instead — that is now the entry point into the watchlist.

!!! warning "Known gap: these flows are not yet clickable in a browser"
    Google OAuth credentials are not configured on the deployed environment, so every `/v1/me/*` route returns `401` to a signed-out browser — which is currently every browser. The routes themselves are proven end to end by the API's e2e suite against a real Postgres database, including cross-user isolation (see [Testing](../testing.md)); what has *not* happened is a human clicking through the flow in production. Cite the e2e suite as the evidence here, not a live demo, until the OAuth client is configured.

### Players list (`/players`)

Table view: Name, Team, Position, Jersey number. Player name links through to their profile. Data comes from `GET /v1/players`, paginated. Includes a filter bar for team and position filtering, plus a search field for server-side search across the full player dataset.

### Player profile (`/players/:playerId`)

The most developed screen. Three-column grid layout (`xl:grid-cols-3`):

- **Header + season stat tiles** (2/3 width) — player identity (headshot from nba.com CDN, name, team, position, jersey, height), then a row of stat tiles (PPG, RPG, APG, Games), followed by a points-trend line chart across recent games.
- **Player traits radar** (1/3 width) — a five-axis radar chart (Scoring, Rebounding, Playmaking, Defense, Efficiency) normalising raw per-game stats onto a shared 0–100 scale so different units can share one chart.
- **Shooting splits** (full width) — FG%, 3P%, FT% as stat tiles.
- **Season segment control** — a control to switch the whole page between `REGULAR`, `PLAY_IN`, `PLAYOFFS`, and `FINALS`, backed by `GET /v1/players/:id/stats/splits` so every segment is available without a re-fetch per click. Carries through to the profile's compare link so a comparison started from a postseason view compares postseason lines, not regular-season ones.
- **Advanced stats table** — true shooting%, effective FG%, assist-to-turnover, plus-minus, usage%, and offensive/defensive rating, alongside the basic per-game stats. Local stat editing lets a visitor temporarily edit a displayed stat to see how it ripples into the derived figures, then reset back to the real value — a "what-if" exploration, not a persisted change.

- **Add to watchlist** — a follow/unfollow control (PR #94), added here because removing the home page's "Add to Locker" section left no other way to start a follow. It renders its own state from `GET /v1/me/watchlist/ids`, which returns just the followed player ids so the button costs one request rather than a full watchlist fetch.

Data comes from `GET /v1/players/:id`, `GET /v1/players/:id/stats`, and `GET /v1/players/:id/stats/splits` together, plus `GET /v1/me/watchlist/ids` when signed in.

### Player comparison (`/compare`)

Side-by-side comparison of 2–4 players' derived season lines for a chosen season segment, reachable from a player profile's compare link or by picking players directly on the page. Data comes from `GET /v1/players/compare`.

### Teams list (`/teams`)

Table view of all NBA teams with real team logos. Data comes from `GET /v1/teams`, paginated with server-side search. Built with TanStack Query and shadcn/ui components.

### Team profile (`/teams/:teamId`)

Team detail page showing team information and its roster of players. Data comes from `GET /v1/teams/:id`.

### Predictions (`/predictions`) — auth-gated

Lists games with their Elo-based win probabilities and Four Factors predicted margins. Links through to the game detail page. Data comes from `GET /v1/games` (with predictions joined in).

### Game detail (`/games/:gameId`) — auth-gated

Single game view showing win probability, predicted score margin, and a **court view** visualising predicted top scorers from both teams by position on a basketball court. Data comes from `GET /v1/games/:id` and `GET /v1/games/:id/prediction`.

### Optimizer (`/optimizer`) — auth-gated

Fantasy-lineup optimizer page showing the latest MILP-solved lineup: five players selected under a salary cap with their predicted fantasy points. Data comes from `GET /v1/optimizer/lineup`.

## Visual design

Dark theme, defined as Tailwind CSS custom properties in `index.css`:

| Token | Value | Use |
|---|---|---|
| `--color-surface-base` | `#0b0e14` | Page background |
| `--color-surface-raised` | `#12161f` | Navbar background |
| `--color-surface-card` | `#171c27` | Card backgrounds |
| `--color-border-subtle` | `#232939` | Borders/divider |
| `--color-brand-accent` | `#3b82f6` | Active nav, links, chart accents |
| `--color-text-primary` / `-secondary` / `-muted` | `#f3f5f8` / `#9aa4b8` / `#616d82` | Text hierarchy |

Charts (Recharts — `RadarChart`, `LineChart`) are themed against these same CSS variables rather than hardcoded colours, so a future light-theme toggle wouldn't require touching chart code.

## Accessibility

- Skip-to-content link for keyboard navigation
- `tabIndex={-1}` on `<main>` for focus management
- `aria-label` on primary navigation
- Responsive layouts with mobile/tablet/desktop breakpoints
- `axe-core` automated accessibility checks run in component tests (`src/test/accessibility.ts`) across the players list, home page, optimizer, and predictions pages — merged into `main` 2026-09-11 (PR #92)

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5]*
