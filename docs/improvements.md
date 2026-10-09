# Improvements Made

Changes made in answer to the [user survey](feedback-survey.md) and the [follow-up interview](user-interviews.md), each linked to the feedback item that asked for it.

F1, F2, F5 and F13 were already being built when the survey ran and landed the same week (PRs #94, #95, #111, #123), so the survey confirmed them rather than started them. F7, F9 and F17 came after it. From the interview, the page tutorials (PRs #202, #203) partly answer F20, F24, F27 and F29.

## Shipped

| ID | Asked for | Before | After | PR |
|---|---|---|---|---|
| F1 | Past prediction accuracy | Nothing showed whether the model was any good | A **Model accuracy** card on Home: hit rate, Brier score, calibration and recent calls | #94 |
| F2 | How predictions are made | A bare "GSW 68%" | A **How it works** section on Predictions: Elo, the Four Factors, in plain English | #95 |
| F5 | Projected points per optimiser pick | Names and salaries only | Projected fantasy points per slot and in total, also saved with each lineup | #111 |
| F7 (part) | Explain stat abbreviations | Tiles showed "RPG", "TS%" with no explanation | Click a trait on the radar to see each stat behind it, explained in a sentence | — |
| F13 | More team detail | Name, logo, conference and division | Team pages with record, Elo, last five and roster; the list filters, searches and sorts by Elo, win % or name | #123 |
| F17 | Player-level predictions | Game-level only | A matchup projection for the player's next game on every profile | #105 |
| F9 | A beginner's guide | Stats with no sense of what kind of player someone is | **Player Archetypes** on every profile: up to three playing styles in plain words, a league style map and the five most similar players ([Player Archetypes](player-archetypes/index.md)) | — |
| F15 | A follow-up interview | Screenshot-based answers only | A hands-on interview on 27 Sep that found two bugs and 12 new items | — |
| F20, F24, F27, F29 (part) | Explain Beat the Model, Datasets, the Optimizer and Become Pro | The interviewee needed each of these pages explained live | A **page tutorial** on every page except the landing page and Live. It opens the first time a signed-in user reaches the page, walks through each section on a sketch of it, and replays from a **?** button ([UI Overview](design/wireframes.md#page-tutorials)). The labels on these pages are unchanged. | #202, #203 |

F7 example, from the radar: *"TS%: True shooting percentage. Scoring efficiency that counts threes and free throws, so volume chuckers and efficient scorers are not lumped together."* The interviewee singled these explanations out as helpful. The stat tiles above the radar still have none.

Source files for the shipped items:

- F1: [ModelAccuracyLedger.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/components/home/ModelAccuracyLedger.tsx)
- F2: [PredictionsPage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/PredictionsPage.tsx#L350-L374)
- F5: [OptimizerPage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/OptimizerPage.tsx)
- F7: [PlayerTraitsRadar.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/components/PlayerTraitsRadar.tsx#L45-L160)
- F13: [TeamProfilePage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/TeamProfilePage.tsx), [TeamsListPage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/TeamsListPage.tsx)
- F17: [PlayerProfilePage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/PlayerProfilePage.tsx)
- F20, F24, F27, F29: [tutorial definitions](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/components/tutorial/definitions), one file per page

## Measured improvement

One improvement came from measuring, not from feedback. Before PR #124 nothing was cached. Afterwards, most public endpoints dropped from 1–6 database queries to 0 on a repeat call ([Performance](design/performance.md)).

## Still in the backlog

- **Survey:** F3 (compare with other sources), F4 and F6 (optimiser constraints and alternative lineups), F7 (tooltips on the stat tiles), F8 (simpler copy), F10 (accent-insensitive search), F11 (all-time leaders), F14 (daily Beat the Model), F16 (injuries), F18 (less on each page).
- **Interview:** F21–F23, F25, F26, F28, F30 and F31, and the rest of F20, F24, F27 and F29. See [User Interviews](user-interviews.md#new-feedback-items-f20f31).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5.5]*
