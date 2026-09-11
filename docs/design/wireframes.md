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

The signed-in dashboard shell ("The Locker") — a challenge-of-the-day card, a watchlist board, a followed-teams list, a "jump back in" rail, and a saved-comparisons/lineups shelf. Built on the landing page's light palette rather than the dark app shell, deliberately, so signing in reads as walking further into the same building.

!!! warning "Not wired to the API yet"
    Every figure on this page currently comes from `components/home/placeholderData.ts`, not a live query — the component itself documents this as the single seam to replace with a `GET /v1/me/dashboard` endpoint once it exists. Don't cite this page as evidence of live personalisation; it's a built shell awaiting a backend.

### Players list (`/players`)

Table view: Name, Team, Position, Jersey number. Player name links through to their profile. Data comes from `GET /v1/players`, paginated. Includes a filter bar for team and position filtering, plus a search field for server-side search across the full player dataset.

### Player profile (`/players/:playerId`)

The most developed screen. Three-column grid layout (`xl:grid-cols-3`):

- **Header + season stat tiles** (2/3 width) — player identity (headshot from nba.com CDN, name, team, position, jersey, height), then a row of stat tiles (PPG, RPG, APG, Games), followed by a points-trend line chart across recent games.
- **Player traits radar** (1/3 width) — a five-axis radar chart (Scoring, Rebounding, Playmaking, Defense, Efficiency) normalising raw per-game stats onto a shared 0–100 scale so different units can share one chart.
- **Shooting splits** (full width) — FG%, 3P%, FT% as stat tiles.
- **Season segment control** — a control to switch the whole page between `REGULAR`, `PLAY_IN`, `PLAYOFFS`, and `FINALS`, backed by `GET /v1/players/:id/stats/splits` so every segment is available without a re-fetch per click. Carries through to the profile's compare link so a comparison started from a postseason view compares postseason lines, not regular-season ones.
- **Advanced stats table** — true shooting%, effective FG%, assist-to-turnover, plus-minus, usage%, and offensive/defensive rating, alongside the basic per-game stats. Local stat editing lets a visitor temporarily edit a displayed stat to see how it ripples into the derived figures, then reset back to the real value — a "what-if" exploration, not a persisted change.

Data comes from `GET /v1/players/:id`, `GET /v1/players/:id/stats`, and `GET /v1/players/:id/stats/splits` together.

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

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
