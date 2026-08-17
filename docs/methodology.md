# Methodology

We follow a **Scrum adaptation across three sprints**, agreed at the first team standup (2026-08-12), plus a fourth milestone for final submission and polish.

## Why Scrum, not Kanban or XP

- **Kanban** suits continuous flow with no fixed deadlines. Our project has three hard milestone dates set by the course, so working in timeboxed sprints with a planning and review cycle around each one maps directly onto how we're actually graded — rather than a continuous board with no natural checkpoint for a retro.
- **Extreme Programming (XP)** practices like pair programming and test-driven development are heavier than a six-person, part-time, three-sprint student project can sustain alongside the rest of the coursework. We adopt some XP-adjacent habits (small commits, code review as a gate) without adopting the full XP process.
- **Scrum**, adapted down from its usual weekly/two-weekly sprint cadence to match the course's three milestone windows, gives us sprint planning, a working increment at the end of each sprint, and a retrospective to actually adjust before the next one — which is what the milestone-based rubric rewards.

Reference: [Scrum Guide (scrumguides.org)](https://scrumguides.org) — our ceremonies below are a lightweight adaptation of the roles/events it defines, not a literal implementation (we don't run a dedicated Scrum Master role, and sprint length is set by the course rather than chosen by the team).

## Ceremonies

| Ceremony | Cadence | What happens |
|---|---|---|
| **Standup** | Weekly | Quick sync in person (Tuesday/Friday labs) or on WhatsApp: what's done, what's next, what's blocking. Whoever's starting a new feature announces it so two people don't build the same thing. |
| **Sprint planning** | Start of each sprint | Team reviews the sprint's rubric weighting, agrees the scope for the sprint, and turns it into sized issues on the work tracker. |
| **Client/tutor check-in** | Weekly | Meeting with the client tutor to validate direction and surface blockers — minuted in [Client Meetings](meetings/client/index.md). |
| **Sprint review** | End of each sprint | Demo of what was actually built against what sprint planning agreed, before the milestone deadline. |
| **Retrospective** | End of each sprint | What worked, what didn't, one or two concrete changes for the next sprint. Minuted alongside the sprint review. |

## Work tracking

**Gitea Issues and Milestones** are the work tracker. This was a change from the initial plan at the first standup (2026-08-12), which proposed GitHub Projects/Issues — the team moved to Gitea's built-in tracker once it was clear the codebase itself would live on Gitea (`sdp.ms.wits.ac.za`), per the university's version-control requirement, to keep code and issue tracking on the same platform. Every task is an issue with an owner; issues are assigned and closed throughout the sprint rather than in a batch immediately before the deadline.

## How we split work

Rather than horizontal layers (one person "does the backend," another "does the frontend"), we build **feature-by-feature, vertically** — one person takes a feature from schema/API through to UI where practical. Sprint 1 is the exception: because the foundational work (schema, docs site, CI/CD, methodology, auth) has to exist before vertical feature ownership makes sense, Sprint 1 is split by the six things it's actually marked on instead — see [Coding Conventions](coding-conventions.md) and [Git Methodology](git-methodology.md) for the process pieces that came out of that split.

## Versioning

We use **date-based versioning** rather than semantic version numbers, since Gitea's commit history and Actions runs already provide a detailed record of when and what changed, and a semantic version adds overhead without a corresponding release process to justify it.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
