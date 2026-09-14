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
- **Responses collected**: 7 responses by 2026-09-14 (Sprint 2 deadline)
- **Form remains open**: for a second wave in Sprint 3 after improvements

### Response Count

**7 responses** were collected against a target of ~12. The shortfall is acknowledged as a limitation (see below).

### Respondent Profile

| Measure | Result |
|---|---|
| Basketball familiarity | Not at all familiar — 5 · Casual fan — 1 · Regular fan — 1 |
| Fantasy basketball experience | Never — 6 · Played a few seasons — 1 |
| Device | Phone — 5 · Desktop — 2 |

The sample skews heavily toward **basketball novices on mobile** — not the hardcore fantasy players the tool ultimately targets. This is a stated limitation, but a useful one: it stress-tested the novice experience, and several of the sharpest findings came from exactly that group.

---

## Key Findings

### Quantitative Highlights

| Measure | Average (n=7) |
|---|---|
| Landing page visual appeal | **4.86** / 5 |
| Personalised dashboard usefulness | **4.57** / 5 |
| Overall satisfaction | **4.43** / 5 |
| Fantasy optimiser usefulness | **4.00** / 5 |
| Optimiser output clarity | **4.00** / 5 |
| "Beat the Model" engagement | **3.86** / 5 |
| **Confidence in predictions** | **3.71** / 5 (lowest) |
| Likelihood to use again | **3.57** / 5 |

### What Worked Well

1. **The landing page sells the product** — all 7 respondents correctly inferred what the tool does; visual appeal averaged 4.86/5
2. **The optimiser is the headline feature** — won the most-useful-feature vote (3 of 7); every respondent chose an improvement they'd like (100% answer rate on Q17)
3. **Zero reported bugs** — 7 of 7 encountered no bugs, errors, or confusing elements
4. **Personalisation lands** — the dashboard (watchlist, followed teams, Beat the Model) averaged 4.57/5 usefulness

### What Needs Work

1. **Prediction trust is the biggest gap** — confidence in using predictions for fantasy decisions is the lowest-rated measure (3.71/5); respondents want evidence (historical accuracy, methodology, comparison to other sources)
2. **Novices can't read the stats** — unexplained stat abbreviations (RPG, APG, TS%), labels unreadable without zooming, "wordy… LLM-esque language"
3. **Retention tracks audience fit, not product quality** — satisfaction 4.43 but likelihood-to-return only 3.57; one respondent rated satisfaction 5/5 and return-likelihood 1/5 (not a basketball person)
4. **Beat the Model engagement is middling** — 3.86/5; all three 3s came from respondents not at all familiar with basketball

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
| F7 | Stat abbreviations unexplained; labels unreadable | Hover-over tooltips + minimum font size | Backlog |
| F8 | Landing-page copy is "wordy" with "LLM-esque language" | Copy-simplification pass | Backlog |
| F9 | Beginner's guide — "what am I looking at" | In-app guide or glossary page | Backlog |
| F10 | Tolerant player search ("manon" → "Chris Mañón") | Fuzzy/diacritic-insensitive matching | Backlog |
| F11 | All-time NBA stat leaders | Evaluate against Sprint 3 scope | Backlog |
| F12 | Show player position in player stats | Verify if surfaced; file if missing | Verify |
| F13 | More team details (recent games with outcomes) | Team profile pages shipped (PR #123); extend if needed | **Shipped** |
| F14 | Beat the Model: random daily/weekly games | Evaluate for Sprint 3 | Backlog |
| F15 | Two respondents left contact details for interviews | 1:1 follow-up interviews in Sprint 3 | **Planned** |

The full traceability table with sources, categories, and detailed actions is on the [Survey Results and Analysis](feedback-survey.md#feedback-to-action-traceability) page.

### Integration Process

Per the [Testing — How feedback is integrated](testing.md#how-feedback-is-integrated) process:

1. ✅ Feedback collected via the survey during the testing window
2. ✅ Responses reviewed by the team and categorised (trust/ML, feature, UX/accessibility, UX/copy, UX/docs, data/feature, process)
3. 🔶 Actionable items to be converted into Gitea issues and prioritised in the sprint backlog (pending)
4. ✅ Changes made in response to feedback documented in the [Sprint Log](sprint-log.md) with links back to originating feedback (F1–F15)
5. ✅ Full feedback methodology, questions asked, distribution plan, findings, and integration actions documented on this page and the [User Feedback Survey](feedback-survey.md) page

---

## Limitations

- **Sample size**: 7 responses against a ~12 target; the form remains open for a second wave
- **Sample profile**: 5 of 7 respondents are not at all familiar with basketball; the sample does not represent the target audience (fantasy players)
- **Not hands-on**: respondents answered while viewing the landing page and app screens (screenshots or live site); several requests concern features completed in the same final-week window as the survey (PR #94, #95, #111, #123), so whether each respondent actually saw them is uncertain
- **Screenshot-based**: the survey was not a hands-on testing session; Sprint 3 adds interviews and hands-on sessions to close this gap

---

## Next Steps (Sprint 3)

1. **File the backlog items** (F3–F14) as Gitea issues and prioritise them in the sprint backlog
2. **Interview the two follow-up volunteers** — disambiguate the discoverability questions raised above
3. **Run hands-on testing sessions** on the live app rather than screenshots
4. **Re-run the survey** after the Sprint 3 model and readability work — a before/after comparison of these scores doubles as evidence for the Milestone 3 *Improvement* criterion (5% weight)

---

## Raw Data

The raw response export (with respondent emails redacted) is archived at [docs/assets/survey-responses/user-feedback-survey-responses-2026-09-14.csv](assets/survey-responses/user-feedback-survey-responses-2026-09-14.csv).

---

*AI Declaration: This methodology page was created with the assistance of Qoder[Qoder Lite].*
