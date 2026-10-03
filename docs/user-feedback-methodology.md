# User Feedback Methodology

This page documents the feedback collection methodology for the NBA Analytics Tool — why we chose this approach, how it was distributed, what we learned, and how the feedback was integrated into development. This is the "evidence of integration" the rubric requires for the User Feedback criterion.

---

## Methodology

### Why Google Forms + WhatsApp

We chose **Google Forms** distributed via **WhatsApp** for the following reasons:

- **Low friction**: respondents can complete the survey on their phone in 5–7 minutes without installing anything or creating accounts
- **Familiar tooling**: Google Forms is widely understood, and WhatsApp is the team's primary communication channel
- **Anonymous by default**: respondents can answer without identifying themselves, encouraging honest feedback
- **Easy analysis**: Google Forms exports directly to CSV/Excel for quantitative analysis
- **No backend work required**: avoided building a custom feedback system in the app during the final sprint weeks

The alternative considered was an in-app feedback form with a NestJS API endpoint and Prisma model. That approach was deferred to Sprint 3 to avoid competing with the model-maturity and feature work in Sprint 2.

### Target Audience

The survey targeted:
- **Basketball fans** (casual to hardcore)
- **Fantasy sports players** (the primary user persona)
- **Sports analytics enthusiasts**

The ideal sample was ~12 responses (~2 per team member × 6 team members), distributed through personal WhatsApp groups and team sharing.

### Survey Design

The survey instrument is a **25-question Google Form** covering:

1. **Respondent profile** (Q1–Q3): basketball familiarity, fantasy experience, device
2. **First impressions** (Q4–Q6): landing page comprehension, visual appeal, clarity
3. **Dashboard usefulness** (Q6b–Q6c): personalised features, Beat the Model engagement
4. **Player browsing** (Q7–Q9): ease of finding players, stats usefulness, additional info requests
5. **Team browsing** (Q10–Q11): ease of browsing teams, desire for more details
6. **Game predictions** (Q12–Q14): confidence in predictions, trust-builders, intended use
7. **Fantasy optimiser** (Q15–Q17): usefulness, output clarity, improvement requests
8. **Overall experience** (Q18–Q22): satisfaction, likelihood to return, most useful feature, missing features, bugs
9. **Final thoughts** (Q23–Q25): feature requests, comments, follow-up interview opt-in

The full survey instrument with all questions is on the [User Feedback Survey](feedback-survey.md) page.

---

## Distribution

### Timeline

- **Survey drafted**: early September 2026 (25 questions + screenshot placeholders)
- **Google Form built**: mid-September 2026
- **Distributed via WhatsApp**: final week of Sprint 2 (2026-09-08 to 2026-09-14)
- **Responses collected**: 11 responses by 2026-09-15 — 7 by the Sprint 2 deadline (2026-09-14), 4 more the next day while the form stayed open
- **Form remains open**: for a second wave in Sprint 3 after improvements
- **First follow-up interview**: 2026-09-27 — hands-on session with one of the survey volunteers ([User Interviews](user-interviews.md))

### Response Count

**11 responses** were collected against a target of ~12 — 7 by the Sprint 2 deadline, 4 more the following day while the form stayed open. The target is effectively met; the analysis below covers the full set.

### Respondent Profile

| Measure | Result |
|---|---|
| Basketball familiarity | Not at all familiar — 7 · Casual fan — 1 · Regular fan — 2 · Hardcore fan — 1 |
| Fantasy basketball experience | Never — 8 · Played a few seasons — 3 |
| Device | Phone — 9 · Desktop — 2 |

The sample still skews toward **basketball novices on mobile**, but the four late responses include the tool's first hardcore daily follower with fantasy experience — the target persona — plus a second regular fan. The set now brackets the audience: the novice experience was stress-tested *and* at least one insider's read was captured.

---

## Key Findings

### Quantitative Highlights

| Measure | Average (n=11) |
|---|---|
| Landing page visual appeal | **4.73** / 5 |
| Personalised dashboard usefulness | **4.64** / 5 |
| Overall satisfaction | **4.55** / 5 |
| Ease of browsing teams | **4.36** / 5 |
| Fantasy optimiser usefulness | **4.18** / 5 |
| Optimiser output clarity | **4.18** / 5 |
| "Beat the Model" engagement | **4.09** / 5 |
| Likelihood to use again | **3.91** / 5 |
| **Confidence in predictions** | **3.73** / 5 (lowest) |

### What Worked Well

1. **The landing page sells the product** — all 11 respondents correctly inferred what the tool does; visual appeal averaged 4.73/5
2. **The optimiser ties for the headline feature** — shares the most-useful-feature vote with player browsing (4 votes each of 11), including the hardcore fantasy player's vote; all 11 respondents chose an improvement they'd like (100% answer rate on Q17)
3. **Zero reported bugs** — 11 of 11 encountered no bugs, errors, or confusing elements (the first hands-on interview later surfaced two real bugs the screenshot-based survey could not — see [User Interviews](user-interviews.md))
4. **Personalisation lands** — the dashboard (watchlist, followed teams, Beat the Model) averaged 4.64/5 usefulness

### What Needs Work

1. **Prediction trust is the biggest gap** — confidence in using predictions for fantasy decisions is the lowest-rated measure (3.73/5, ten of eleven at 4 or below); respondents want evidence (historical accuracy, methodology, comparison to other sources, player-level detail)
2. **Novices can't read the stats** — unexplained stat abbreviations (RPG, APG, TS%), labels unreadable without zooming, "wordy… LLM-esque language"
3. **Retention tracks audience fit, not product quality** — satisfaction 4.55 but likelihood-to-return only 3.91; one respondent rated satisfaction 5/5 and return-likelihood 1/5 (not a basketball person)
4. **Beat the Model is polarised** — 4.09/5 with five 5s and four 3s; the 3s no longer come only from novices (the hardcore fantasy player also rated it 3), and the first interview found the name itself opaque until explained

The full quantitative analysis, categorical breakdowns, and qualitative themes are on the [Survey Results and Analysis](feedback-survey.md#survey-results-and-analysis) section.

---

## Integration Evidence

Every distinct piece of feedback was mapped to an action: what was already shipped, what's in the backlog, or what's planned for Sprint 3. This is the **feedback-to-action traceability table** — the "integration" half of the rubric's user-testing requirement.

### Summary

| ID | Feedback | Action | Status |
|---|---|---|---|
| F1 | Show historical prediction accuracy | Model Accuracy Ledger + predicted-vs-actual view shipped (PR #94); deepen in Sprint 3 | **Shipped** |
| F2 | Explain how predictions are calculated | "How it works" explainer shipped (PR #95); expand in Sprint 3 | **Shipped** |
| F3 | Compare predictions with other sources (ESPN/CBS) | Evaluate for Sprint 3 | Backlog |
| F4 | Optimiser: allow custom constraints ("must include LeBron") | Add hard constraints to MILP solver | Backlog |
| F5 | Optimiser: show projected points per player | Per-slot predictions shipped (PR #111); verify + extend | **Shipped** |
| F6 | Optimiser: show alternative lineups | "Next-best 3 lineups" from solver | Backlog |
| F7 | Stat abbreviations unexplained; labels unreadable | Hover-over tooltips + minimum font size. Partial fix shipped: abbreviation explainer in Player Traits Radar | **Shipped** (partial) |
| F8 | Landing-page copy is "wordy" with "LLM-esque language" | Copy-simplification pass | Backlog |
| F9 | Beginner's guide — "what am I looking at" | In-app guide or glossary page. Player Archetypes & Style Map with tooltip explainer addresses this on player profiles | Pending merge |
| F10 | Tolerant player search ("manon" → "Chris Mañón") | Fuzzy/diacritic-insensitive matching | Backlog |
| F11 | All-time NBA stat leaders | Evaluate against Sprint 3 scope | Backlog |
| F12 | Show player position in player stats | Verify if surfaced; file if missing | Verify |
| F13 | More team details (recent games with outcomes) | Team profile pages shipped (PR #123); extend if needed | **Shipped** |
| F14 | Beat the Model: random daily/weekly games | Evaluate for Sprint 3 | Backlog |
| F15 | One respondent left contact details for an interview | 1:1 follow-up interview conducted 2026-09-27 | **Complete** |
| F16 | Injuries and expected return dates | Evaluate adding an injury data source | Backlog |
| F17 | Player-level predictions, not just game-level | Matchup projections already exist (PR #105); surface them next to game predictions | Shipped |
| F18 | Some pages carry too much information | Progressive-disclosure pass alongside F7–F9 | Backlog |
| F19 | Player height | Verify the bio fields render on the profile page; file if missing | Verify |

The full traceability table with sources, categories, and detailed actions is on the [Survey Results and Analysis](feedback-survey.md#feedback-to-action-traceability) page.

### Follow-up interviews

The first 1:1 follow-up interview — a hands-on walkthrough of the live app with the one volunteer — was conducted on **2026-09-27**. It confirmed the survey's novice-onboarding and readability findings, surfaced two bugs a screenshot-based survey could not, and produced twelve new feedback items (F20–F31). Session notes: [User Interviews](user-interviews.md).

### Integration Process

Per the [Testing — How feedback is integrated](testing.md#how-feedback-is-integrated) process:

1. ✅ Feedback collected via the survey during the testing window
2. ✅ Responses reviewed by the team and categorised (trust/ML, feature, UX/accessibility, UX/copy, UX/docs, data/feature, process)
3. 🔶 Actionable items to be converted into Gitea issues and prioritised in the sprint backlog (pending)
4. ✅ Changes made in response to feedback documented in the [Sprint Log](sprint-log.md) with links back to originating feedback (F1–F31)
5. ✅ Full feedback methodology, questions asked, distribution plan, findings, and integration actions documented on this page and the [User Feedback Survey](feedback-survey.md) page

---

## Limitations

- **Sample size**: 11 responses against a ~12 target (7 by the Sprint 2 deadline, 4 more on 2026-09-15 while the form stayed open); the form remains open for a second wave
- **Sample profile**: 7 of 11 respondents are not at all familiar with basketball; the sample still under-represents the target audience (fantasy players), though it now includes one hardcore fantasy player and two regular fans
- **Not hands-on**: respondents answered while viewing the landing page and app screens (screenshots or live site); several requests concern features completed in the same final-week window as the survey (PR #94, #95, #111, #123), so whether each respondent actually saw them is uncertain
- **Screenshot-based**: the survey was not a hands-on testing session — the first hands-on session (the 2026-09-27 interview) has started closing this gap; more are planned in Sprint 3

---

## Next Steps (Sprint 3)

1. **File the backlog items** (F3–F14, F16, F18, plus the interview items F20–F31) as Gitea issues and prioritise them in the sprint backlog
2. **Run hands-on testing sessions** on the live app rather than screenshots — the first was combined with the 2026-09-27 interview; more are planned
3. **Re-run the survey** after the Sprint 3 model and readability work — a before/after comparison against the n=11 baseline doubles as evidence for the Milestone 3 *Improvement* criterion (5% weight). See [Improvements Made](improvements.md) for the shipped before/after comparisons. The follow-up interview is complete; additional sessions are contingent on more volunteers coming forward

---

## Raw Data

Two artifacts capture the raw responses:

- **Live Google Sheets response sheet** — [NBA Analytics Tool: User Feedback Survey (Responses)](https://docs.google.com/spreadsheets/d/1LTA5ckTegY2CqBV-1OsBeTl-OzeHkn6OoSdLVkws8X8/edit?usp=sharing) — the form's native destination; the email column is redacted in the shared copy
- **Archived CSV export** (with respondent emails redacted) — [docs/assets/survey-responses/user-feedback-survey-responses-2026-09-14.csv](assets/survey-responses/user-feedback-survey-responses-2026-09-14.csv) — a version-controlled snapshot of the same 11 responses

---

*AI Declaration: This methodology page was created with the assistance of Qoder[Qoder Lite].*
