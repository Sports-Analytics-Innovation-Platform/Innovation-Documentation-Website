# Methodology

We follow a **dynamic Scrum adaptation across three sprints**, agreed at the first team standup (2026-08-12), plus a fourth milestone for final submission and polish. Rather than assigning work to individuals during sprint planning, team members pull tasks based on initiative and capacity — anyone can pick up any unassigned issue from the backlog and start working on it.

## Why Scrum, not Kanban or XP

- **Kanban** suits continuous flow with no fixed deadlines. Our project has three hard milestone dates set by the course, so working in timeboxed sprints with a planning and review cycle around each one maps directly onto how we're actually graded — rather than a continuous board with no natural checkpoint for a retro.
- **Extreme Programming (XP)** practices like pair programming and test-driven development are heavier than a six-person, part-time, three-sprint student project can sustain alongside the rest of the coursework. We adopt some XP-adjacent habits (small commits, code review as a gate) without adopting the full XP process.
- **Scrum**, adapted down from its usual weekly/two-weekly sprint cadence to match the course's three milestone windows, gives us sprint planning, a working increment at the end of each sprint, and a retrospective to actually adjust before the next one — which is what the milestone-based rubric rewards.

Reference: [Scrum Guide (scrumguides.org)](https://scrumguides.org) — our ceremonies below are a lightweight adaptation of the roles/events it defines, not a literal implementation (we don't run a dedicated Scrum Master role, and sprint length is set by the course rather than chosen by the team).

## Ceremonies

| Ceremony | Cadence | What happens |
|---|---|---|
| **Standup** | Weekly | Quick sync in person (Tuesday/Friday labs) or on WhatsApp: what's done, what's next, what's blocking. Whoever's starting a new feature announces it so two people don't build the same thing. |
| **Sprint planning** | Start of each sprint | Team reviews the sprint's rubric weighting, agrees the scope for the sprint, and turns it into sized issues on the product backlog. Work is not pre-assigned — team members self-select tasks based on initiative and capacity throughout the sprint. |
| **Client/tutor check-in** | Weekly | Meeting with the client tutor to validate direction and surface blockers — minuted in [Client Meetings](meetings/client/index.md). |
| **Sprint reflection (Sprint Log)** | End of each sprint | A tabulated record of every task completed during the sprint, grouped by week and owner, documenting *what was done, by whom, and when*. This supplements the retrospective by providing a concrete performance picture — see [Sprint Log](#sprint-log) below. |
| **Retrospective** | End of each sprint | What worked, what didn't, one or two concrete changes for the next sprint. Informed by the Sprint Log data. Minuted alongside the sprint reflection. |

## Work tracking

**Gitea Projects and Gitea Issues** are the work tracker. This was a change from the initial plan at the first standup (2026-08-12), which proposed GitHub Projects/Issues — the team moved to Gitea's built-in tracker once it was clear the codebase itself would live on Gitea, per the university's version-control requirement, to keep code and issue tracking on the same platform. The **product backlog** holds all issues for the current sprint; tasks are not pre-assigned to individuals during planning. Instead, team members take initiative by self-selecting and claiming issues as they have capacity, which encourages ownership and flexibility. Issues are closed throughout the sprint rather than in a batch immediately before the deadline.

## How we split work

Rather than horizontal layers (one person "does the backend," another "does the frontend"), we build **feature-by-feature, vertically** — one person takes a feature from schema/API through to UI where practical. Sprint 1 is the exception: because the foundational work (schema, docs site, CI/CD, methodology, auth) has to exist before vertical feature ownership makes sense, Sprint 1 is split by the six things it's actually marked on instead — see [Coding Conventions](coding-conventions.md) and [Git Methodology](git-methodology.md) for the process pieces that came out of that split.

## Sprint Log

At the end of each sprint the team produces a **Sprint Log** — a week-by-week, tabulated record of every task that was completed, categorised by type (`feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`) and attributed to the team member who did the work. The Sprint Log serves as a **sprint reflection**: it supplements the retrospective with hard evidence of who contributed what and when, giving a clear picture of individual and team performance across the sprint.

The Sprint Log is maintained at [Sprint Log](sprint-log.md) in this documentation site.

## Versioning

We use **date-based versioning** rather than semantic version numbers, since Gitea's commit history and Actions runs already provide a detailed record of when and what changed, and a semantic version adds overhead without a corresponding release process to justify it.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite]*
