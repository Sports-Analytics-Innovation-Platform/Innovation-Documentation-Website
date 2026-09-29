---
hide:
  - toc
---
# Scrum Meetings

??? note "2026-09-28 — Scrum"

    **Attendees:** Kiran, Owen, Daniel, Josh, Sanele
    **Absent:** Adrian

    ## Agenda

    - Become Pro valuation model and NBA rookie comparison
    - Rubric compliance review and fixes
    - API testing and POPIA implementation timing
    - Database RLS (Row Level Security) planning
    - Documentation status and remaining updates

    ## Decisions

    - **Become Pro valuation** uses the same system as the training model — needs a minimum of 10 games to determine a user's value in R and dollars. No extra backend work required.
    - **POPIA compliance** deferred to Sprint 4 — Daniel investigated but team agreed to implement it next sprint to avoid breaking existing database connections or services.
    - **RLS (Row Level Security)** Josh planning to enable on the database.
    - **Player profile types** accidentally added by Josh — needs to be checked and merged into main.

    ## Actions

    | Action | Owner | Due |
    |---|---|---|
    | Enable RLS on database | Josh | Sprint 4 |
    | Check and merge player profile types into main | Josh | ASAP |
    | Implement POPIA compliance | Daniel | Sprint 4 |
    | Update diagrams | Sanele | Ongoing |
    | Finalize meeting notes | Sanele | Ongoing |

    ## Notes

    - **Kiran** completed the NBA rookies comparison for Become Pro — the valuation model trains on rookie seasons and prices users against the published rookie scale.
    - **Owen** reviewed the full rubric and made fixes to address any shortfalls — several pushes landed to ensure all requirements are met.
    - **Daniel** conducted API testing and investigated POPIA requirements. Team decided to defer POPIA implementation to avoid disrupting current database/service connections.
    - **Josh** is planning to enable RLS on the database. Accidentally added player profile types in a recent change — needs review and merge to main.
    - **Sanele** updated all documentation. Remaining work: diagrams and final meeting notes.

??? note "2026-09-14 — Scrum"

    **Attendees:** Owen, Kiran, Josh, Daniel, Sanele, Adrian

    ## Agenda

    - Sprint 2 submission review and rubric compliance check
    - Supabase egress limit crisis (278% over the 5 GB free tier)
    - Database caching and query optimization strategy
    - Individual sprint contributions review
    - Survey distribution and response tracking
    - Bug tracker usage and Gitea issue management
    - UI animation inconsistencies across devices
    - Password reset approach discussion

    ## Decisions

    - **Supabase egress** is 278% over the 5 GB free tier (13.88 GB used). Team agreed to implement caching and query optimization in Sprint 3 rather than upgrading the plan — the project only needs one more month of hosting.
    - **Caching strategy**: implement backend response caching (recalculate stats hourly instead of on every page load) and add React Query `staleTime` to stop refetching all 41 queries on every mount/tab switch.
    - **Password reset** deferred — the effort (email verification flow, new database fields) is too high for the remaining time. Team will ask the client if it's a hard requirement given Google OAuth is the only sign-in method.
    - **Bug tracker**: team needs to file real bugs on Gitea — the tracker exists but had minimal active usage. Each member to file issues for known problems.
    - **UI animations** not rendering on Owen's device only — works on all other team members' devices and phones. Accepted as a device-specific issue, not a code bug.
    - **Sprint 2 is essentially complete** — focus shifts to documentation, survey distribution, and Sprint 3 planning.

    ## Actions

    | Action | Owner | Due |
    |---|---|---|
    | Implement backend caching layer (response cache, staleTime, query consolidation) | Kiran | Sprint 3 |
    | Ask client whether password reset is a hard requirement | Owen | Next class |
    | File known bugs as Gitea issues | All | Sprint 3 |
    | Distribute survey via WhatsApp status and share with contacts | All | Ongoing |
    | Update individual Gitea project board cards (move to Done, assign owners) | All | ASAP |
    | Update documentation site with user feedback findings | Adrian | Sprint 3 |

    ## Notes

    - The team walked through the live app together: home dashboard with Beat the Model, player profiles with animations, team pages, admin page, and predictions page. UI looks significantly improved from Sprint 1.
    - Owen flagged the Supabase egress crisis — the database is at 13.88 GB of 5 GB allowed. Root cause: every page load refetches all data from scratch with no caching, and some queries pull excessive data due to missing primary keys.
    - Kiran proposed the caching approach: recalculate stats on an hourly interval rather than per-request, and add React Query `staleTime` on the frontend. Team agreed this should bring usage within limits.
    - Individual contributions review: Josh did the prediction model overhaul, landing page redesign, predictions page, profile page, and all UI implementation from Kiran's designs. Kiran built the home dashboard, Beat the Model game, personalization layer, and caching. Owen built the admin page, Compare/Teams restyle, and fixed login. Daniel fixed auth flows and added accessibility tests. Sanele added advanced stats, bug tracker, and postseason data. Adrian built the documentation site, survey instrument, and Swagger docs.
    - The team reviewed the survey responses (3 so far) and discussed distributing more widely. Owen and Kiran will share on their WhatsApp statuses.
    - Josh noted the Supabase egress might reset at the end of the month — team will monitor.

    ??? note "Raw transcript (Craig)"
        [2026-09-14-team-meeting-6.txt](../../transcripts/meeting-transcripts/2026-09-14-team-meeting-6.txt)


??? note "2026-09-10 — Scrum"

    **Attendees:** Owen, Kiran, Josh, Daniel, Sanele
    **Absent:** Adrian

    ## Agenda

    - Front-end UI design review and page implementation progress
    - Profile page concept and additional features
    - Prediction model accuracy and database seeding status
    - RLS (Row Level Security) deferred to end of sprint
    - Swagger documentation setup
    - User feedback survey distribution plan
    - Sprint 2 remaining tasks and next meeting schedule

    ## Decisions

    - **Profile page** will be added — Josh volunteered to help with the UI. It will include account settings, password reset, and data-deletion request options.
    - **Player stats self-upload** ("coach mode" evolution) — Kiran proposed letting users upload their own stats and get evaluated against pro players instead of only coaches uploading. Team agreed it's a good differentiator feature.
    - **Password reset** will be a simple on-page change for now; email-based reset deferred if time permits. Daniel will handle the backend mechanics.
    - **RLS (Row Level Security)** on Supabase tables deferred to near the end of the sprint — team agreed it caused major dev friction last semester and should wait until features are stable.
    - **Prediction report** should show only the most recent 10 games, not all 211 — Josh flagged this as a UX fix.
    - **Next meeting** scheduled for Monday (2026-09-14) to review everything before the Sprint 2 submission on Tuesday.

    ## Actions

    | Action | Owner | Due |
    |---|---|---|
    | Finish remaining UI pages (1–2 per day) | Kiran | 2026-09-14 (Sun) |
    | Fix login button regression | Daniel | 2026-09-11 |
    | Build profile page UI | Josh / Daniel | Sprint 2 |
    | Finish predictions page UI and styling | Josh | 2026-09-11 |
    | Share homepage colour codes with Josh | Kiran | 2026-09-11 |
    | Continue seeding additional seasons (1–2 more) | Josh | Ongoing |
    | Distribute user feedback survey via WhatsApp / Instagram | All | Sprint 2 |
    | Merge Swagger documentation PRs | Sanele / Team | ASAP |
    | Add postseason data for earlier seasons | Sanele | Sprint 2–3 |

    ## Notes

    - Kiran demoed the new Figma-based UI designs for all pages. Team was very positive — designs look great on laptop, acceptable on mobile. Pages are modular so sections can be rearranged or removed.
    - Kiran connected the backend to the homepage, so it now fetches real data and personalises for logged-in users.
    - Josh seeded two additional NBA seasons (now three total), improving prediction model accuracy to ~65%. Seeding is extremely slow (~12 hours for 12,000 games) due to the API's 1-request-per-second rate limit.
    - Josh set up Swagger within the API — it auto-generates documentation from the code. Two PRs were open (one by Kiran, one by Josh); one had a merge conflict Kiran was fixing.
    - Adrian created a testing document and bug tracker document; Daniel created a survey brief with screenshots and plans to build the Google Forms survey.
    - Daniel planned to distribute the survey to coworkers for quality responses; team will also post on WhatsApp statuses.
    - Sanele finished editing postseason data for the most recent season but noted that postseason data for the other two added seasons is missing. Josh said it's not critical for now.
    - The team discussed whether the current sprint deliverables are sufficient. Consensus was that most core requirements are met and the focus should be on polishing UI, adding the profile page, and completing documentation.
    - Kiran estimated all UI pages would be done by Sunday; Josh aimed to finish the predictions page that day.
    - Team agreed to announce tasks in the group chat to avoid duplicate work.

    ??? note "Raw transcript (Craig)"
        [2026-09-10-team-meeting-5.txt](../../transcripts/meeting-transcripts/2026-09-10-team-meeting-5.txt)


??? note "2026-09-07 — Scrum"

    **Attendees:** Owen, Adrian, Josh, Daniel, Sanele

    ## Agenda


    - Sprint 2 progress review and rubric status walkthrough
    - CI test fixes and new loading animation
    - User feedback survey approach (Google Forms vs in-app)
    - Bug tracker demo (custom Gitea issue form)
    - Scheduling client meetings with Kovendan
    - Remaining Sprint 2 tasks and documentation status


    ## Decisions


    - Cards on the board stay in review/testing until the day before the deadline — don't move to Done early, so the team can track what's actually sprint-complete vs carry-over.
    - User feedback will use **Google Forms** distributed via WhatsApp rather than an in-app survey — simpler, no database management needed.
    - Daniel offered to distribute the survey to coworkers for responses. Each team member should get ~2 people to fill it in.
    - **axe-core accessibility** scanning deferred to Sprint 3, not Sprint 2.
    - Owen fixed CI tests (weren't passing previously) and switched to the correct Gitea-provided runners.
    - Sanele demoed the custom Gitea issue form for bug tracking — team satisfied.
    - Adrian to message Kovendan to schedule 2–3 client meetings before the Sprint 2 deadline.
    - Team confirmed the app uses **real NBA data** (not mock).
    - Sanele wants to add **postseason data** with separate regular/postseason views — Owen said go for it if he can do it right.


    ## Actions


    | Action | Owner | Due |
    |---|---|---|
    | Create and distribute Google Forms user feedback survey | Adrian / Team | Sprint 2 |
    | Message Kovendan to schedule 2–3 client meetings | Adrian | ASAP |
    | Add postseason data with separate views | Sanele | Sprint 2–3 |
    | Merge Owen's CI test fix PR | Owen / Team | ASAP |


    ## Notes


    - The team walked through the Sprint 2 rubric checklist. Most items are in a solid position: core features done, tests fixed, API exists (needs expansion), bug tracker built, database documentation current. The main gaps are the user feedback survey and scheduling more client meetings.
    - Owen added a basketball bouncing loading animation and fixed CI tests that hadn't been passing.
    - Josh mentioned wanting to focus on user personalization features and more ML model testing with backtesting documentation.
    - Adrian was reviewing the feature tiers document, thinking about what's needed to move from basic to intermediate/advanced tier.
    - Kiran is working on a Figma-based UI redesign (mentioned post-client-meeting).
    - The team discussed a "coach mode" idea (from Kiran, post-client-meeting) where users could submit their own stats and get compared to pro players with an estimated draft number — team responded positively.
    - Testing documentation, third-party docs, and Swagger/OpenAPI docs were noted as needing updates but not considered difficult.


    ??? note "Raw transcript (Craig)"
    [2026-09-07-scrum.txt](../../transcripts/meeting-transcripts/2026-09-07-scrum.txt)


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
