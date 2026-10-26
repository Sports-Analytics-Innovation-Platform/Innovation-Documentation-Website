# Improvements Made

Changes made in answer to the [user survey](feedback-survey.md) and the [follow-up interview](user-interviews.md), each linked to the feedback item that asked for it.

F1, F2, F5 and F13 were already being built when the survey ran and landed the same week (PRs #94, #95, #111, #123), so the survey confirmed them rather than started them. F7, F9 and F17 came after it, and the page tutorials (PRs #202, #203) partly answered F20, F24, F27 and F29. The submission week then closed almost all of what was left: F4, F6, F7, F8, F10, F11, F16 and F20–F26, F28–F31, in PRs #219–#227.

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

F7 example, from the radar: *"TS%: True shooting percentage. Scoring efficiency that counts threes and free throws, so volume chuckers and efficient scorers are not lumped together."* The interviewee singled these explanations out as helpful. Every stat tile, the radar, Compare and the season-splits tables now read from one file (`apps/web/src/lib/statGlossary.ts`), so the wording can't drift again and the tiles are no longer the exception.

Source files for the shipped items:

- F1: [ModelAccuracyLedger.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/components/home/ModelAccuracyLedger.tsx)
- F2: [PredictionsPage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/PredictionsPage.tsx#L350-L374)
- F5: [OptimizerPage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/OptimizerPage.tsx)
- F7: [PlayerTraitsRadar.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/components/PlayerTraitsRadar.tsx#L45-L160)
- F13: [TeamProfilePage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/TeamProfilePage.tsx), [TeamsListPage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/TeamsListPage.tsx)
- F17: [PlayerProfilePage.tsx](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/pages/PlayerProfilePage.tsx)
- F20, F24, F27, F29: [tutorial definitions](https://sdp.ms.wits.ac.za/innovation/sportsanalytics/src/branch/main/apps/web/src/components/tutorial/definitions), one file per page

## Shipped in submission week (PRs #219–#227)

| ID | Asked for | What changed | PR |
|---|---|---|---|
| F4 | Optimiser: force a player in | Lock a player into every lineup, or exclude one, then solve again | #221 |
| F6 | Alternative lineups | Three alternatives under the same rules, each a different set of players | #221 |
| F7 | Explain the stat abbreviations | One glossary behind every stat tile, the radar, Compare and the splits tables | #219 |
| F8 | Landing copy is "wordy… LLM-esque" | The landing page rewritten in plain language | #223 |
| F10 | Search "manon" should find "Mañón" | Accent-insensitive search on the players list and in the API | #224 |
| F11 | All-time stat leaders | `/all-time`: eleven categories, regular season or playoffs, a bio on every card | #226 |
| F16 | Injuries and return dates | ESPN's injury report on team and player pages | #225 |
| F20 | "Beat the Model" and "model GSW 64%" opaque | A one-line explainer on the card, and the probability named for what it is | #219 |
| F21 | Headshots too small on desktop | Every headshot size steps up at `md`/`lg`; phone sizes unchanged | #219 |
| F22 | Leaders don't follow the filters | The leaders band follows the search, filters and min-games floor | #224 |
| F23 | Favourite team first, and what Elo means | The favourite team sits above the list, with a line explaining Elo and win % | #227 |
| F24 | Datasets page purpose unclear | The page says what a release is, when a season is released, and what a download does | #227 |
| F25 | Unstarted season listed as ingested | A season appears once some of its games have been played and reviewed | #222 |
| F26 | Season download fails after an edit | Downloads are rebuilt from current data, with the checksum verdict spelled out | #222 |
| F28 | Predictions controls hard to read | Contrast and readability pass on the form controls | #219 |
| F29 | Become Pro purpose unclear | A first-visit **How it works** explainer on the page itself | #219 |
| F30 | Player profiles undiscoverable | A named "Full player profile" link on every top-scorer card | #219 |
| F31 | Public data behind a login | A public API guide and a route-by-route audit of what needs a key | #220 |

## Measured improvement

One improvement came from measuring, not from feedback. Before PR #124 nothing was cached. Afterwards, most public endpoints dropped from 1–6 database queries to 0 on a repeat call ([Performance](design/performance.md)).

## Still in the backlog

- **Survey:** F3 (compare with other sources), F14 (daily Beat the Model) and F18 (less on each page).
- **Interview:** F27, the optimiser's scope — the controls now work, but the page still doesn't model a real fantasy format, so the reframe-or-rebuild decision is open. See [User Interviews](user-interviews.md#new-feedback-items-f20f31).
- **F31 follow-through:** the access model itself is unchanged. Public reads still need an API key or a signed-in session, and closing that gap is a product decision recorded in the guide's own "what needs a key" audit.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5.5]*
