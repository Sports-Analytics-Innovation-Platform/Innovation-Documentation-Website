# Methodology

The team ran a **Scrum adaptation with three sprints**, agreed at the first standup (12 Aug), each ending on a course milestone. A fourth period, **Milestone 4**, ran from the end of Sprint 3 (29 Sep) to submission (11 Oct); the meeting minutes call it "Sprint 4". Work wasn't assigned at planning: team members picked up unassigned issues from the backlog as they had capacity.

## Why Scrum

- **Kanban** suits continuous work with no fixed deadlines. The course sets three hard milestone dates, so timeboxed sprints with planning around each one match how the project is marked.
- **Extreme Programming (XP):** pair programming and test-driven development are more than a six-person, part-time team could sustain alongside other courses. We kept two XP habits: small commits, and code review as a gate.
- **Scrum**, with sprints stretched to the course's milestone windows, gives sprint planning and a working increment at the end of each sprint.

This is a lightweight version of the [Scrum Guide](https://scrumguides.org): the course sets the sprint length, and Adrian Draxl is the Scrum Master in a lighter role than the guide describes.

## Ceremonies

| Ceremony | When | What happens | Record |
|---|---|---|---|
| **Standup** | Weekly, in the Tuesday or Friday lab or on WhatsApp | What's done, what's next, what's blocking. Whoever starts a feature says so, so two people don't build the same thing. | [Scrum Meetings](meetings/scrum/index.md) |
| **Sprint planning** | Start of each sprint | The team reads the sprint's rubric weights, agrees the scope and turns it into issues on the board. | [Roadmap](design/roadmap.md), [project board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8) |
| **Client check-in** | Weekly | The client tutor checks the direction and raises blockers. | [Client Meetings](meetings/client/index.md), [Stakeholder Interactions](stakeholder-interactions.md) |
| **Rubric check** | Before each milestone | The team walks through the milestone's rubric against the project, for example on 23 Aug and 7 Sep. | [Scrum Meetings](meetings/scrum/index.md) |
| **Sprint Log** | End of each sprint | Every completed task, by week, type (`feat`, `fix`, `docs` and so on) and person. | [Sprint Log](sprint-log.md) |

The team held no separate retrospectives. Process changes were agreed in standups when a problem came up: on 7 Sep, for example, the team agreed that cards stay in review until the day before a deadline, so the board shows what is really finished.

## Work tracking

Work is tracked in **Gitea Issues** on the [project board](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/projects/8). The first standup planned to use GitHub Projects; the team moved to Gitea's tracker once the code was on Gitea, the university's required host, to keep code and issues together. Issues are closed as the work finishes, not in a batch before the deadline.

## How work is split

Features are built **vertically**: where practical, one person takes a feature from the schema and API through to the UI, instead of one person owning the backend and another the frontend. Sprint 1 was the exception. The foundations (schema, docs site, CI/CD, methodology, auth) had to exist first, so Sprint 1 was split by the six things it was marked on. [Git Methodology](git-methodology.md) and [Coding Conventions](coding-conventions.md) cover how code reaches `main`.

## Versioning

Versions are identified by date, not semantic version numbers. The commit history and CI runs already record what changed and when, and there is no release process that a version number would serve.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Qoder[Qoder Lite], Claude-Code[Claude Opus 5.5]*
