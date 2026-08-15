# ADR-001: Database

**Status:** Accepted (in use since initial scaffold, 2026-08-06)

## Decision

Use **PostgreSQL** as the relational database, accessed through **Prisma** as the ORM/migration tool, from the NestJS (`apps/api`) backend.

## Context

- The data is naturally relational: players belong to teams, games involve two teams, stats are tied to a player and a game. Foreign-key relationships and joins are a better fit than a document store.
- The brief requires statistics to be derived from individual event records rather than stored as pre-typed totals (§2 project descriptions) — a relational schema makes that derivation (event rows → aggregated season averages) straightforward to model and query.
- Prisma generates TypeScript types directly from the schema, which matches the NestJS/TypeScript backend and keeps the frontend's manually-defined types (`nba.ts`) in sync with what the API actually returns, rather than drifting apart.
- Migrations are file-based and reviewable in PRs, consistent with [Git Methodology](../git-methodology.md)'s PR-review requirement.

## Alternatives considered

**Not recorded.** This decision was made during the initial scaffold and no record exists of what else was weighed (e.g. MongoDB, a Supabase-managed Postgres instance, or something else) or why it was rejected. If the team wants this ADR to be complete for the group report, that reasoning needs to be reconstructed from memory and added here — otherwise this ADR is documenting *what* was decided without demonstrating the *judgement* behind it, which is what the rubric actually rewards.

## Consequences

- Schema changes go through `prisma migrate`, reviewed in PRs.
- The actual current schema (entities, fields, relationships) isn't documented anywhere yet — see the open item in [Requirements Traceability](../requirements.md) and the questions raised for the team.
- No decision has been recorded on production database hosting (separate from this ADR — see the not-yet-written ADR-003).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
