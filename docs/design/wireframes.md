# UI Overview

!!! success "Confirmed: UI Overview, not Wireframes"
    This project didn't go through a formal low-fidelity wireframing stage — the team found reference dashboards for inspiration and used AI to help generate the initial React implementation directly. This page documents the current build, not a planning artifact.

## Navigation

Top navbar with five links: **Home**, **Players**, **Teams**, **Optimizer** (auth-gated), **Predictions** (auth-gated). The navbar also includes a recent-result widget (hidden on small screens) and an auth status button (sign in/sign out). A skip-to-content link is available for keyboard navigation.

The sidebar from the initial scaffold was replaced with the top navbar during Sprint 1 (week of 11 Aug) to accommodate the growing number of pages.

## Screens

### Home (`/`)

Landing page with a hero section stating the product's purpose and linking to the players list. Includes a recent-result widget. Entry point, not a full dashboard.

### Players list (`/players`)

Table view: Name, Team, Position, Jersey number. Player name links through to their profile. Data comes from `GET /v1/players`, paginated. Includes a filter bar for team and position filtering, plus a search field for server-side search across the full player dataset.

### Player profile (`/players/:playerId`)

The most developed screen. Three-column grid layout (`xl:grid-cols-3`):

- **Header + season stat tiles** (2/3 width) — player identity (headshot from nba.com CDN, name, team, position, jersey, height), then a row of stat tiles (PPG, RPG, APG, Games), followed by a points-trend line chart across recent games.
- **Player traits radar** (1/3 width) — a five-axis radar chart (Scoring, Rebounding, Playmaking, Defense, Efficiency) normalising raw per-game stats onto a shared 0–100 scale so different units can share one chart.
- **Shooting splits** (full width) — FG%, 3P%, FT% as stat tiles.

Data comes from `GET /v1/players/:id` and `GET /v1/players/:id/stats` together.

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
- `axe-core` checks planned for Sprint 2

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
