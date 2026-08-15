# UI Overview

!!! note "Not wireframes"
    This project didn't go through a formal low-fidelity wireframing stage — the team found reference dashboards for inspiration and used AI to help generate the initial React implementation directly. This page documents that initial build, not a planning artifact. If you'd rather have this nav entry read as "Wireframes," rename it back in `mkdocs.yml`; otherwise it's titled to match what it actually is.

## Screens

### Home (`/`)

Landing page. States the product's purpose ("Browse players and see season stats derived from game event data.") and links to the players list. Minimal by design — this is the entry point, not a dashboard.

### Players list (`/players`)

Table view: Name, Team, Position, Jersey number. Player name links through to their profile. Data comes from `GET /v1/players`, paginated.

### Player profile (`/players/:playerId`)

The most developed screen. Three-column grid layout (`xl:grid-cols-3`):

- **Header + season stat tiles** (2/3 width) — player identity (initials avatar, name, team, position, jersey, height), then a row of stat tiles (PPG, RPG, APG, Games), followed by a points-trend line chart across recent games.
- **Player traits radar** (1/3 width) — a five-axis radar chart (Scoring, Rebounding, Playmaking, Defense, Efficiency) normalising raw per-game stats onto a shared 0–100 scale so different units can share one chart.
- **Shooting splits** (full width) — FG%, 3P%, FT% as stat tiles.

Data comes from `GET /v1/players/:id` and `GET /v1/players/:id/stats` together.

## Navigation

Sidebar with three links: Home, Players, Teams. **Teams is not yet implemented** — the link exists in `Sidebar.tsx` but there's no corresponding route or page yet.

## Visual design

Dark theme, defined as Tailwind CSS custom properties in `index.css`:

| Token | Value | Use |
|---|---|---|
| `--color-surface-base` | `#0b0e14` | Page background |
| `--color-surface-raised` | `#12161f` | Sidebar background |
| `--color-surface-card` | `#171c27` | Card backgrounds |
| `--color-border-subtle` | `#232939` | Borders/dividers |
| `--color-brand-accent` | `#3b82f6` | Active nav, links, chart accents |
| `--color-text-primary` / `-secondary` / `-muted` | `#f3f5f8` / `#9aa4b8` / `#616d82` | Text hierarchy |

Charts (Recharts — `RadarChart`, `LineChart`) are themed against these same CSS variables rather than hardcoded colours, so a future light-theme toggle wouldn't require touching chart code.

## What's not built yet

- Teams list/profile pages (nav link exists, no route)
- Auth screens (sign up/in, password reset) — not present in `App.tsx`'s routes yet
- Any optimisation/analytics UI beyond the player profile — see [Feature Tiers](feature-tiers.md) (not yet written, open question below)

## Open questions for the team

- Should the accessibility pass (keyboard nav, contrast, `axe-core`) happen against this build directly, or is it expected to change enough before Sprint 2 that it's premature?
- Is `/teams` planned for Sprint 1, or later?

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
