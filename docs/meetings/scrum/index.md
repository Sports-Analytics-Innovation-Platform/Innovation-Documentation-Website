---
hide:
  - toc
---
# Scrum Meetings

??? note "2026-08-23 — Scrum"

    **Attendees:** Owen, Adrian, Josh, Kiran, Sanele

    ## Agenda


    - Sprint 1 rubric walkthrough and readiness check
    - Code coverage status in CI
    - API structure and how to call it
    - Google Auth login issue (404 on hosted site)
    - Methodology documentation and project plan
    - Sprint Log as sprint reflection replacement
    - Auth requirements gap (sign-up, password reset, account deletion)
    - Documentation updates needed
    - AI transcript collection


    ## Decisions


    - The team agreed the methodology will be updated to describe the process as a **dynamic, initiative-based Scrum adaptation** rather than a planned sprint with pre-assigned tasks. Team members communicate which features they intend to work on and pull work based on capacity.
    - The **Sprint Review ceremony** is replaced with a **Sprint Reflection** using the Sprint Log — a week-by-week tabulated record of what each person completed, serving as evidence of individual contribution.
    - A **project plan** will be maintained showing task allocation per sprint, generated from the Sprint Log and roadmap to demonstrate forward planning.
    - The **Gitea project board** is confirmed as the work tracker evidence. Sanele set it up and will assign tasks to the team members who did the closest work.
    - **Two critical tasks** remain before Sprint 1 deadline: (1) fix Google Auth login on the hosted site, (2) implement credential-based sign-up, password reset, and account deletion. Daniel volunteered for auth; Sanele offered as backup.
    - Adrian will handle documentation updates to ensure the docs site accurately reflects the current state of the project.
    - Auth is called via **session cookies** — the NestJS API runs on port 4000, and the frontend proxies `/api/*` to it. Users authenticate through Google OAuth and the session cookie is sent with subsequent requests.
    - The team will **not** remove the docs folder from the project repo until all AI transcripts are confirmed moved to the docs site.


    ## Actions


    | Action | Owner | Due |
    |---|---|---|
    | Fix Google Auth login (404 on hosted site) | Daniel / Team | 2026-08-25 |
    | Implement credential-based sign-up, password reset, and account deletion | Daniel (Sanele backup) | 2026-08-25 |
    | Update documentation site to reflect current project state | Adrian | 2026-08-25 |
    | Replace/update diagrams on the docs site | Adrian | 2026-08-25 |
    | Assign Gitea project board tasks to closest matching team members | Sanele | 2026-08-25 |
    | Add remaining AI transcripts to docs site transcripts folder | Owen, Josh, Kiran, Sanele | 2026-08-25 |
    | Ensure all team members are familiar with the tech stack for marking questions | All | 2026-08-25 |


    ## Notes


    - The team walked through every Sprint 1 rubric criterion against the actual project state. Most criteria scored at Advanced level. The two gaps identified were: (1) auth features (sign-up, password reset, account deletion) not yet implemented, and (2) project plan documentation needed to show forward planning.
    - Code coverage in CI covers `apps/api` and `apps/web` only — the Python services (ingestion, predictor, optimizer) have no test files. The team accepted this as not critical for Sprint 1 but noted it for Sprint 2.
    - The team discussed how to call the API: it's a NestJS API listening on port 4000, endpoints are accessed with a session cookie from Google OAuth. The API is hosted on Render and the frontend on Cloudflare Pages.
    - Google Auth was returning a 404 on the hosted site during the meeting. The team suspects it may be a cold-start delay on Render (free tier). This needs to be verified and fixed.
    - The brief's auth requirements (sign up, sign in, reset passwords, delete accounts) were flagged as a risk — currently only Google OAuth sign-in exists. The team agreed this must be addressed before Sprint 1 marking.
    - The Sprint Log was generated during the meeting using AI from the git commit history, producing a week-by-week breakdown of Sprint 1 tasks with owners. The team was satisfied with the result.
    - The team discussed the distinction between the methodology documentation on the docs site vs. the project repo — both exist and serve different purposes (docs site is for markers, project repo is for developers).
    - Josh and Owen handed off additional AI transcript files to Adrian for inclusion in the docs site.
    - The team noted that Supabase is explicitly listed in the brief as a disallowed auto-API system — the docs should make it explicit that Supabase's auto-generated APIs are deliberately unused.


    ??? note "Raw transcript (Craig)"
    [2026-08-23-team-standup.txt](../../transcripts/meeting-transcripts/2026-08-23-team-standup.txt)


??? note "2026-08-12 — Scrum"

    **Attendees:** Adrian, Daniel, Owen, Sanele, Kieran, Josh

    ## Agenda

    - Sprint 1 requirements, documentation, and rubric
    - Git methodology, versioning, and coding conventions
    - Documentation site and required project documents
    - Sprint task allocation and individual responsibilities
    - CI/CD, testing, and work tracker
    - Stakeholder/tutor interaction and weekly check-ins
    - Project methodology and team workflow
    - Feature implementation strategy and Sprint 1 scope
    - Game project ideas and constraints

    ## Decisions

    - The team will use date-based versioning because it makes changes easier to track historically. The team noted that GitHub actions, pushes, and documentation already provide additional information about when changes were made.
    - The team agreed to follow the Git methodology discussed in class and document it. The team considered this important because the methodology may be checked against the rubric.
    - The project methodology will describe the team's process as a Scrum adaptation with three sprints, including sprint planning and a review/retrospective at the end of each milestone.
    - Gitea Projects and Gitea Issues will be used as the work tracker. Issues will be used for tasks and assignments.
    - The team will meet weekly and have a weekly tutor check-in to validate work and stay synchronized.
    - Work will be organized feature-by-feature, with the team aiming to build features vertically rather than working separately on isolated horizontal layers.
    - Team members should announce in the WhatsApp group which feature they are working on so that multiple people do not implement conflicting features simultaneously.
    - The team agreed that the first sprint should establish the project foundations and remove the largest blockers, while also beginning the core features.
    - The main Sprint 1 goal is to get the core pipeline, a solid prediction model, the optimisation component, the mobile core, and a coherent project in place. The project scope has also been changed to include all teams.
    - The main feature work is expected to begin the following week.
    - For the game component, the team established that the project must be 3D. Non-turn-based boss combat was considered too difficult for the project scope, so boss-fighting mechanics were ruled out.

    ## Actions

    | Action | Owner | Due |
    |---|---|---|
    | Maintain the documentation site and incorporate project documents as they are completed | Adrian | TBD |
    | Assist Adrian with documentation diagrams | Owen and Josh | TBD |
    | Keep the work tracker accurate, help allocate tasks, and ensure completed work is recorded correctly | Sanele | Ongoing |
    | Implement the CI/CD workflow, including YAML workflows, test coverage, linting, and testing | Kieran | TBD |
    | Maintain repository hygiene and follow the agreed Git/coding conventions | Josh | Ongoing |
    | Push the three additional documents and the project overview to the repository | Owen | TBD |
    | Add the pushed documentation to the documentation site | Adrian | TBD |
    | Arrange the tutor meeting and handle the booking | Daniel | TBD |
    | Prepare/write notes from the tutor meeting | Daniel | TBD |
    | Prepare questions for the stakeholder/tutor meeting | Team | TBD |
    | Tell the team which feature is being worked on before implementation | All team members | Ongoing |
    | Implement one or two project features in addition to documentation responsibilities | Adrian | TBD |

    ## Notes

    - The documentation site currently exists as a template and is deployed through GitHub Pages using MkDocs Material.
    - The team identified a substantial amount of documentation required by the rubric, including project overview, Git methodology, project methodology, tech stack, getting-started/developer guides, work tracker, coding conventions, stakeholder interaction, development plan, architecture/design documentation, and supporting documents.
    - Several documents already exist in the project codebase and can be moved or incorporated into the documentation site.
    - The team noted that the technology choices are partly constrained by the project requirements, but a document explaining the technology choices will still be needed.
    - Adrian is expected to handle the majority of the documentation, while Owen and Josh will assist with diagrams. The team explicitly discussed that assigning all documentation to Adrian would be unfair.
    - CI/CD appeared to be partially in place, but the team had not confirmed whether it was running automatically.
    - The team did not fully allocate individual project features during this meeting. The group agreed that the detailed feature discussion could happen later.
    - The tutor is reportedly available on Friday before 15:00. If team members cannot attend in person, questions can be collected and asked on their behalf, with Discord as an alternative.
    - Daniel expressed uncertainty about his role; the team clarified that everyone is expected to document their own feature work and contribute to the actual project.
    - The team expressed some uncertainty about exactly what the project currently entails. The immediate approach is to establish the core system first and build on it.
    - For the game component, possible ideas discussed included a house/area-based concept, a card-based concept, and a medieval/dungeon-crawler concept. The card idea was questioned because the project requires three levels. A dungeon-crawler was discussed, but boss AI was identified as too difficult for the scope.

    ??? note "Raw transcript (Craig)"
    [2026-08-12-scrum.txt](../../transcripts/meeting-transcripts/2026-08-12-scrum.txt)

---

*AI Declaration: The preceding document was generated with the assistance of the following: ChatGPT[GPT-5.6 Luna], Qoder[Qoder Lite]*
