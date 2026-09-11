# 2026-09-10 — Scrum

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
