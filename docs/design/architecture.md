# Architecture Overview

## System shape

```
┌─────────────────────┐        HTTPS/fetch         ┌──────────────────────┐
│   apps/web           │ ───── /api/v1/... ───────▶ │   apps/api            │
│   React + Vite       │ ◀──── JSON, cookies ─────  │   NestJS              │
│   (Tailwind, Recharts│                             │   (BetterAuth, Prisma)│
│   React Router)      │                             │                       │
└─────────────────────┘                             └──────────┬────────────┘
                                                                  │
                                                                  ▼
                                                        ┌──────────────────┐
                                                        │   PostgreSQL       │
                                                        └──────────────────┘

                                              ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
                                              │  nba_api (Python) ingestion  │
                                              │  — integration path with the │
                                              │  TypeScript backend not yet  │
                                              │  confirmed (see Tech Stack)  │
                                              └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

This satisfies the brief's non-monolithic requirement (§2.1): `apps/web` and `apps/api` are separate, independently buildable applications that only communicate over HTTP — confirmed directly from the code (`apiClient.ts` calls `fetch` against a base URL, nothing shares in-process state).

## Frontend (`apps/web`)

- **React + Vite**, **Tailwind CSS v4** (via `@theme` custom properties in `index.css`, not the older `tailwind.config.js` token approach).
- **React Router** — client-side routing, three routes currently: `/`, `/players`, `/players/:playerId`.
- **Recharts** — `RadarChart` (player traits) and `LineChart` (points trend), both themed against the same CSS variables as the rest of the UI.
- All backend calls go through `lib/apiClient.ts`, a single `fetchJson<T>` wrapper — one place controls the base URL and request options, which is a solid pattern for API changes later.
- `credentials: "include"` on every request implies cookie-based auth is expected once auth-gated routes exist, even though the three current endpoints appear to be public reads.
- **TanStack Query** and **shadcn/ui** are confirmed added (Teams pages, filters/sort/pagination on Players) — see [Tech Stack](../tech-stack.md).

## Backend (`apps/api`)

- **NestJS** on top of **Prisma** and **PostgreSQL** — see [ADR-001](../decisions/adr-001-database.md).
- **BetterAuth** (Google OAuth) for auth — see [ADR-002](../decisions/adr-002-auth.md), including an open compliance question this introduces.
- Routes are versioned under `/v1/` (`/v1/players`, `/v1/players/:id`, `/v1/players/:id/stats`), served under an `/api` base path the frontend proxies to — see [API Design](api-design.md) for the full picture of what's confirmed vs. open.

## Local development

**Docker Compose** runs Postgres locally; both apps run with their own dev server (`npm run start:dev` / `npm run dev`) against it — see [Getting Started](../getting-started.md).

## Open questions

- **`nba_api` (Python) integration path** — still unresolved (also flagged in [Tech Stack](../tech-stack.md)). Is there a separate scheduled ingestion service, a one-off import script, or something else that populates Postgres? This is the single biggest gap in this diagram — everything left of the database is confirmed from code, everything ingesting *into* it is not.
- **Production hosting topology** — not decided (see the not-yet-written ADR-003).
- **Whether any endpoint is actually auth-gated yet** — the three confirmed routes don't appear to require a logged-in user, based on `PlayersListPage.tsx` and `PlayerProfilePage.tsx` calling them unconditionally on mount.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
