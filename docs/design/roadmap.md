# Roadmap

!!! success "Unblocked"
    Same source as [Feature Tiers](feature-tiers.md) — the team's plain-English answer on what the ML component does. This maps those tiers onto the brief's actual sprint dates.

!!! warning "Draft, not assigned"
    This is a proposed sequencing, not a committed plan — it doesn't assign individual owners (that needs the team's own input, e.g. via `team-questions.md`), and it isn't confirmed against your actual velocity so far in Sprint 1.

## Sprint 1 — infra & foundation (due 2026-08-25)

Already largely covered by what's built: NestJS/Prisma/Postgres backend, React/Vite/Tailwind frontend, BetterAuth migration, TanStack Query + shadcn/ui, Vitest test suites, CI/CD. Remaining for this sprint:

- `nba_api` ingestion path — still the single biggest open item in [Tech Stack](../tech-stack.md) and [Architecture](architecture.md). No ML work depends on this being *finished*, but it does need a plan.
- No ML work is expected yet — the client explicitly said (per the [client meeting notes](../meetings/client/index.md)) a working model is realistic around **Sprint 3**, not before.

## Sprint 2 — real data + first prediction model (due 2026-09-15)

- Get `nba_api` ingestion actually flowing into Postgres (real data, not just mock seed data)
- Build the **Basic-tier** prediction: team-level matchup win-probability, trained on whatever real historical data is available by this point
- Redis/BullMQ, Zod, S3/MinIO — the confirmed-but-unbuilt stack items — likely belong here if the ingestion pipeline needs batch/scheduled processing

## Sprint 3 — model maturity (due 2026-09-29)

- **Intermediate-tier** prediction: player-level predictions, predicted-vs-actual accuracy view
- This is the sprint the client flagged as the realistic target for "a working ML model" — treat this as the sprint where prediction quality actually needs to be defensible in a review, not just present

## Submission (due 2026-10-11)

- **Advanced-tier** recommendation layer, if time allows — this is explicitly the highest-risk, most-optional item; the client's own guidance ("last two weeks are mostly touch-ups if ML isn't ready") suggests treating this as a stretch goal, not a commitment
- Polish, responsiveness/accessibility pass, final documentation pass

## Still open

- Individual ownership of each sprint's work
- Whether the advanced-tier recommendation layer is a real commitment or an explicit stretch goal — worth deciding as a team rather than leaving implicit

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
