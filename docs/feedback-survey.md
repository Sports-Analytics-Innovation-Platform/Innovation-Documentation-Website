# User Feedback Survey

How the team collected user feedback, what users said, and what was done about it. The hands-on follow-up is on [User Interviews](user-interviews.md), and the shipped changes, with before and after, are on [Improvements Made](improvements.md).

| Method | When | Result |
|---|---|---|
| Survey (Google Forms, shared on WhatsApp) | 8–15 Sep 2026 | 11 responses, findings F1–F19 |
| Hands-on interview with a survey volunteer | 27 Sep 2026 | Two bugs and 12 more findings, F20–F31 |
| Client meetings | Every sprint | Recorded on [Stakeholder Interactions](stakeholder-interactions.md) |

## How feedback is integrated

1. Collect responses during the testing window.
2. Review and categorise each one: trust, feature, UX, copy, data, bug or process.
3. Give each actionable item an ID (F1, F2, …) and turn it into a backlog item.
4. Record the change that answers it on [Improvements Made](improvements.md) and in the [Sprint Log](sprint-log.md).

## Method

Google Forms on WhatsApp was the lowest-friction option: five to seven minutes on a phone, anonymous by default, no account, and a CSV export for analysis. An in-app feedback form was considered and dropped so it wouldn't compete with Sprint 2 feature work.

The target was about 12 responses, two per team member, from basketball fans, fantasy players and analytics enthusiasts. The form showed screenshots of each part of the app, then asked 25 questions across nine sections ([full list below](#the-questionnaire)).

The form was shared in the final week of Sprint 2 (8–14 Sep). Seven responses came in by the Sprint 2 deadline and four more the next day.

## Respondents

| Measure | Result |
|---|---|
| Basketball familiarity (Q1) | Not at all 7 · Casual 1 · Regular 2 · Hardcore 1 |
| Fantasy basketball (Q2) | Never 8 · A few seasons 3 |
| Device (Q3) | Phone 9 · Desktop 2 |

The sample leans towards basketball novices on phones, but it includes one hardcore fan with fantasy experience: the target user.

## Results

| Question | Average (n=11) | Distribution |
|---|---|---|
| Q5: Landing page visual appeal | **4.73** / 5 | eight 5s, three 4s |
| Q6b: Personalised dashboard usefulness | **4.64** / 5 | seven 5s, four 4s |
| Q18: Overall satisfaction | **4.55** / 5 | six 5s, five 4s |
| Q10: Ease of browsing teams | **4.36** / 5 | six 5s, three 4s, two 3s |
| Q7: Ease of finding a player | **4.27** / 5 | five 5s, four 4s, two 3s |
| Q15: Optimiser usefulness | **4.18** / 5 | four 5s, five 4s, two 3s |
| Q16: Optimiser output clarity | **4.18** / 5 | three 5s, seven 4s, one 3 |
| Q6c: Beat the Model engagement | **4.09** / 5 | five 5s, two 4s, four 3s |
| Q19: Likely to use again | **3.91** / 5 | six 5s, four 3s, one 1 |
| Q12: Confidence in predictions | **3.73** / 5 | one 5, six 4s, four 3s |

- **Purpose clear (Q6):** 6 very clear, 5 somewhat clear.
- **Most useful feature (Q20):** optimiser 4, player browsing 4, predictions 3.
- **What would build trust (Q13):** historical accuracy 5 (plus 2 similar answers), other sources 2, methodology 1, player-level detail 1.
- **Bugs met (Q22):** none, 11 of 11.

## Findings

**What worked:**

- **The landing page explains the product.** All 11 respondents described correctly what the tool does.
- **The optimiser and player browsing are the headline features.** The hardcore fantasy player voted for the optimiser.
- **Personalisation is valued.** The dashboard scored 4.64.

**What needs work:**

- **Trust in predictions is the weakest score (3.73).** Respondents want evidence: past accuracy, the method, other sources and player-level detail.
- **Novices can't read the stats.** They asked what "RPG, APG, TS%" mean, couldn't read labels without zooming, and found the copy "wordy… LLM-esque". One asked for a "beginners guide to what Im looking at".
- **Returning depends on interest in basketball, not on quality.** One respondent rated satisfaction 5 and returning 1 because they don't follow basketball.
- **Beat the Model splits opinion.** Five rated it 5 and four rated it 3, including the hardcore fan.

## Limitations

- **Small sample.** 11 responses, and 7 of the respondents don't follow basketball.
- **Not hands-on.** People answered from screenshots, and some features (PRs #94, #95, #111, #123) landed in the same week, so not every respondent saw them. The [interview](user-interviews.md) was hands-on and found two bugs the survey could not.

## Feedback to action

| ID | Feedback (question) | Action | Status |
|---|---|---|---|
| F1 | Show past prediction accuracy (Q13: 5 + 2) | Model accuracy card on Home (PR #94) | **Shipped** |
| F2 | Explain how predictions are made (Q13: 1) | "How it works" on Predictions (PR #95) | **Shipped** |
| F3 | Compare with ESPN or CBS (Q13: 2) | Evaluate the cost | Backlog |
| F4 | Optimiser: "must include LeBron" constraints (Q17: 3) | Hard constraints in the MILP | Backlog |
| F5 | Optimiser: projected points per player (Q17: 3) | Per-slot projections, also saved with lineups (PR #111) | **Shipped** |
| F6 | Optimiser: alternative lineups (Q17: 3) | Next-best lineups from the solver | Backlog |
| F7 | Stat abbreviations unexplained; small labels (Q9, Q21) | Explanations beside the traits radar; the stat tiles still have none | **Partly shipped** |
| F8 | Landing copy is "wordy… LLM-esque" (Q6) | Plain-language pass | Backlog |
| F9 | A beginner's guide (Q21, Q23) | Player Archetypes explains each player's style in plain words | **Shipped** |
| F10 | Search "manon" should find "Mañón" (Q24) | Accent-insensitive search | Backlog |
| F11 | All-time stat leaders (Q9) | Needs seasons beyond the three ingested | Backlog |
| F12 | Show player position (Q21) | Already in the profile header | **Already shown** |
| F13 | More team details (Q11: 3 yes, 6 maybe) | Team pages with record, Elo, last five and roster (PR #123) | **Shipped** |
| F14 | Daily or weekly Beat the Model games (Q23) | Evaluate | Backlog |
| F15 | A volunteer for an interview (Q25) | Interviewed on 27 Sep | **Done** |
| F16 | Injuries and return dates (Q9) | Needs an injury data source | Backlog |
| F17 | Player-level predictions (Q13) | Matchup projection on each player profile (PR #105) | **Shipped** |
| F18 | Some pages carry too much (Q21) | Lead with the headline numbers, keep detail one click away | Backlog |
| F19 | Player height (Q9) | Already in the profile's Bio card | **Already shown** |

F20–F31 come from the interview and are listed on [User Interviews](user-interviews.md#new-feedback-items-f20f31).

## Raw data

- The [response sheet](https://docs.google.com/spreadsheets/d/1LTA5ckTegY2CqBV-1OsBeTl-OzeHkn6OoSdLVkws8X8/edit?usp=sharing) on Google Sheets, with the email column removed.
- A [CSV snapshot](assets/survey-responses/user-feedback-survey-responses-2026-09-14.csv) of the same 11 responses, with emails removed.

## The questionnaire

| Section | Questions |
|---|---|
| About you | Q1 basketball familiarity · Q2 fantasy experience · Q3 device |
| First impressions (landing page) | Q4 what does the tool do? (text) · Q5 visual appeal (1–5) · Q6 is it clear what you can do? |
| Home dashboard | Q6b dashboard usefulness (1–5) · Q6c Beat the Model engagement (1–5) |
| Players | Q7 finding a player (1–5) · Q8 stats usefulness · Q9 what else would you like to see? (text) |
| Teams | Q10 browsing teams (1–5) · Q11 want more team details? |
| Predictions | Q12 confidence for fantasy decisions (1–5) · Q13 what would build trust? · Q14 what would you use them for? |
| Optimiser | Q15 usefulness (1–5) · Q16 output clarity (1–5) · Q17 what would improve it? |
| Overall | Q18 satisfaction (1–5) · Q19 use again (1–5) · Q20 most useful feature · Q21 what's missing? (text) · Q22 any bugs? |
| Final thoughts | Q23 one feature to add (text) · Q24 other comments (text) · Q25 follow-up interview? (optional email) |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Qoder[Qoder Lite], Claude-Code[Claude Opus 5.5]*
