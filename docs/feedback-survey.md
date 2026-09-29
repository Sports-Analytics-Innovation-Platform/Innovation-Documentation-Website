# NBA Analytics Tool — User Feedback Survey

**Target audience:** Basketball fans, fantasy sports players, sports analytics enthusiasts  
**Distribution:** WhatsApp groups, team sharing (~2 respondents per team member = ~12 total)  
**Estimated completion time:** 5–7 minutes  
**Status:** Fielded via Google Forms — **11 responses collected (2026-09-14 – 2026-09-15)**; see [Survey Results and Analysis](#survey-results-and-analysis)

---

## Survey Results and Analysis

**Collected:** 2026-09-14 – 2026-09-15 · **Responses:** 11 of the ~12 targeted — 7 by the Sprint 2 deadline, 4 more the next day while the form stayed open · **Raw data:** [redacted response export](assets/survey-responses/user-feedback-survey-responses-2026-09-14.csv) — respondent emails removed · **Live results:** [Google Sheets response sheet](https://docs.google.com/spreadsheets/d/1LTA5ckTegY2CqBV-1OsBeTl-OzeHkn6OoSdLVkws8X8/edit?usp=sharing) (email column redacted)

### Respondent profile

| Measure | Result |
|---|---|
| Basketball familiarity (Q1) | Not at all familiar — 7 · Casual fan — 1 · Regular fan — 2 · Hardcore fan — 1 |
| Fantasy basketball experience (Q2) | Never — 8 · Played a few seasons — 3 |
| Device (Q3) | Phone — 9 · Desktop — 2 |

The sample still skews toward basketball novices on mobile — but the four late responses include the tool's first hardcore daily follower with fantasy experience (the actual target persona) and a second regular fan. The full set now brackets the audience: it stress-tested the novice experience *and* captured at least one insider's read.

### Quantitative results

| Question | Average (n=11) | Range | Distribution |
|---|---|---|---|
| Q5 — Landing page visual appeal | **4.73** / 5 | 4–5 | eight 5s, three 4s |
| Q6b — Personalised dashboard usefulness | **4.64** / 5 | 4–5 | seven 5s, four 4s |
| Q18 — Overall satisfaction | **4.55** / 5 | 4–5 | six 5s, five 4s |
| Q10 — Ease of browsing teams | **4.36** / 5 | 3–5 | six 5s, three 4s, two 3s |
| Q7 — Ease of finding a specific player | **4.27** / 5 | 3–5 | five 5s, four 4s, two 3s |
| Q15 — Fantasy optimiser usefulness | **4.18** / 5 | 3–5 | four 5s, five 4s, two 3s |
| Q16 — Optimiser output clarity | **4.18** / 5 | 3–5 | three 5s, seven 4s, one 3 |
| Q6c — "Beat the Model" engagement | **4.09** / 5 | 3–5 | five 5s, two 4s, four 3s |
| Q19 — Likelihood of using the tool again | **3.91** / 5 | 1–5 | six 5s, four 3s, one 1 |
| Q12 — Confidence in predictions for fantasy decisions | **3.73** / 5 | 3–5 | one 5, six 4s, four 3s |

Categorical picks:

- **Clarity of the site's purpose (Q6):** 6 very clear, 5 somewhat clear — nobody below
- **Player stats usefulness (Q8):** 5 extremely useful, 6 very useful
- **More team details wanted (Q11):** 3 yes-definitely, 6 maybe, 2 no
- **What would build trust in predictions (Q13):** show historical accuracy — 5 (plus 2 near-variants: "all of the above except methodology", "show past correct predictions/results"), compare with other sources — 2, explain methodology — 1, player-level detail — 1
- **Intended use of predictions (Q14):** fantasy decisions — 4, sports betting — 3, just for fun — 3, wouldn't use them — 1
- **Optimiser improvements (Q17):** custom constraints — 3, projected points per player — 3, alternative lineups — 3, explain why chosen — 2 — *all 11 respondents answered*
- **Most useful feature (Q20):** fantasy lineup optimiser — 4, player browsing and stats — 4 (tied at the top), game predictions — 3
- **Bugs encountered (Q22):** none — 11 of 11

### What worked well

**The landing page sells the product.** All eleven respondents correctly inferred what the tool does from the landing page — "takes a lot of data about basketball games and players, and presents them. Main focus being predicting results/stats", "analyses win probability for teams", "helps make fantasy related decisions in order to select the best lineup possible". Visual appeal averaged 4.73/5 (eight maximum scores) — a strong return on the Sprint 2 landing-page redesign (PR #103).

**The optimiser and player browsing share the headline.** At n=7 the optimiser was the sole most-useful-feature winner; at n=11 it ties with player browsing and stats at 4 votes each — and the vote that tied it came from the hardcore fantasy player, the persona the optimiser was built for. Its usefulness and output clarity both averaged 4.18/5, and all 11 respondents chose an improvement they would like (Q17 had a 100% answer rate) — which reads as engagement rather than dissatisfaction.

**Zero reported bugs.** 11 of 11 encountered no bugs, errors, or confusing elements. One readability issue surfaced through Q9 instead (see F7 below) — though the first hands-on session (the 2026-09-27 interview) later surfaced two real bugs that a screenshot-based survey structurally could not; see [User Interviews](user-interviews.md).

**Personalisation lands.** The personalised dashboard (watchlist, followed teams, Beat the Model) averaged 4.64/5 usefulness — validating the PR #94 personalisation layer.

### What needs work

**Prediction trust is still the biggest gap.** Confidence in using predictions for fantasy decisions remains the lowest-rated measure (3.73/5, ten of eleven respondents at 4 or below). Respondents want *evidence*: historical accuracy (still the top request), methodology, comparison against other sources — and, from the hardcore fantasy player, player-level predictions rather than game-level only (F17). The Model Accuracy Ledger and predicted-vs-actual view (PR #94) and the "How it works" explainer (PR #95) were completed in the same final-week window as the survey — the repeated request suggests a discoverability or presentation gap (or a desire for more depth), which the Sprint 3 model-maturity work on the [Roadmap](design/roadmap.md) (target: 75–80% accuracy) directly addresses.

**Novices can't read the stats.** The clearest theme from the novice majority: unexplained stat abbreviations ("what RPG, APG, TS%, etc mean"), labels unreadable without zooming in, and "wordy… LLM-esque language" in the copy. One respondent's single feature wish was a "beginners guide to what Im looking at, and how to make use of the tool to its full extent".

**Retention tracks audience fit, not product quality.** Satisfaction averaged 4.55 and likelihood-to-return 3.91 — the gap narrowed as the late respondents arrived (it was 4.43 vs 3.57 at n=7) but persists. The sharpest example is unchanged: a respondent rating satisfaction 5/5 and return-likelihood 1/5 — not at all familiar with basketball, never played fantasy, wouldn't use the predictions. The tool impresses even outside its target audience; making the novice experience navigable (F7, F9 below) is what converts that goodwill into return visits.

**Beat the Model is polarised rather than middling (4.09/5).** Five respondents rated it 5 and four rated it 3. The 3s no longer come only from novices — the hardcore fantasy player also gave it 3, which points at the game's depth and framing rather than just basketball knowledge. The one regular-fan respondent engaged (4/5) and volunteered ideas for it (F14), and the first user interview (2026-09-27) found the name itself opaque until explained live (F20).

**Information density cuts both ways.** One respondent found some pages carry "a little bit too much info… could be tweaked to be read and understood quicker" (F18), while the novices above ask for more guidance (F7–F9). The answer is not more content or less content everywhere — it is progressive disclosure: lead with the headline numbers, keep the depth one click away, and explain the vocabulary inline.

### Limitations

- **Sample size and profile.** 11 responses against a ~12 target (7 by the Sprint 2 deadline, 4 more on 2026-09-15 while the form stayed open), and 7 of 11 respondents are not at all familiar with basketball. The form remains open, and a second wave follows the Sprint 3 improvements.
- **Not hands-on.** Respondents answered while viewing the landing page and app screens; several requests concern features completed in the same final-week window as the survey (PR #94, #95, #111, #123), so whether each respondent actually saw them is uncertain. The first hands-on session (the 2026-09-27 interview) has since started closing this gap and confirmed several findings — see [User Interviews](user-interviews.md).
- **Contactable respondents.** One respondent left contact details for a follow-up interview (address removed from the published export and withheld from this page). The interview was conducted 2026-09-27.

### Feedback-to-action traceability

Every distinct piece of feedback, mapped to what happened or will happen with it — the "integration" half of the rubric's user-testing requirement.

| ID | Feedback (source) | Category | Action | Status |
|---|---|---|---|---|
| F1 | Show historical prediction accuracy (Q13: 5 + 2 near-variants) | Trust / ML | Model Accuracy Ledger and predicted-vs-actual accuracy view already shipped (PR #94); make accuracy more prominent next to predictions and keep expanding it with the Sprint 3 model-maturity work | Shipped (PR #94) — deepen in Sprint 3 |
| F2 | Explain how predictions are calculated (Q13: 1) | Trust / UX | "How it works" explainer shipped on the Predictions page (PR #95); expand into a fuller methodology section | Shipped (PR #95) — expand in Sprint 3 |
| F3 | Compare predictions with other sources, e.g. ESPN/CBS (Q13: 2) | Trust | Evaluate cost/benefit for Sprint 3 | Backlog — file as Gitea issue |
| F4 | Optimiser: allow custom constraints, e.g. "must include LeBron" (Q17: 3) | Feature | Add hard constraints to the MILP solver (must-include / must-exclude players) | Backlog — file as Gitea issue |
| F5 | Optimiser: show projected points per player (Q17: 3) | Feature | Per-slot predictions already render on the optimizer board and are snapshotted into saved lineups (PR #111); verify visibility and consider explicit fantasy-points projections | Shipped (PR #111) — verify + extend |
| F6 | Optimiser: show alternative lineups, not just the best (Q17: 3) | Feature | e.g. "next-best 3 lineups" from the solver | Backlog — file as Gitea issue |
| F7 | Stat abbreviations unexplained; labels unreadable without zooming (Q9/Q21: 1) | UX / Accessibility | Hover-over tooltips or a glossary for stat abbreviations (RPG, APG, TS%, eFG%…) plus a minimum label font size; extends the existing axe-core accessibility checks. Partial fix shipped: abbreviation explainer in Player Traits Radar (click any trait axis) | Shipped (partial) — radar explainer done, stat tiles still need tooltips |
| F8 | Landing-page copy is "wordy" with "LLM-esque language" (Q6: 1) | UX / Copy | Copy-simplification pass on landing page and in-app text | Backlog — file as Gitea issue |
| F9 | Beginner's guide — "what am I looking at and how to use the tool" (Q23/Q21: 1) | UX / Docs | In-app beginner's guide or glossary page; directly serves the novice segment that dominated this sample. Player Archetypes & Style Map with tooltip explainer addresses this on player profiles | Pending merge (`player-archetypes` branch) |
| F10 | Tolerant player search — "manon" should find "Chris Mañón" (Q24: 1) | Feature | Fuzzy/diacritic-insensitive matching on the search endpoints | Backlog — file as Gitea issue |
| F11 | All-time NBA stat leaders (Q9: 1) | Data / Feature | Requires historical data beyond the three ingested seasons — evaluate against Sprint 3 scope | Backlog — evaluate |
| F12 | Show player position in player stats (Q21: 1) | Feature | Position is available from the CommonPlayerInfo endpoint the bio fields already come from — verify it is surfaced on the profile page; file if missing | Verify, then file if needed |
| F13 | More team details — recent games with outcomes (Q11: 3 yes + 6 maybe) | Feature | Team profile pages with records and recent form landed in PR #123 in the final Sprint 2 week; confirm discoverability and consider a standings view | Shipped (PR #123) — extend if needed |
| F14 | Beat the Model: random daily/weekly games (Q23: 1) | Feature / Engagement | Idea for lifting the game's polarised engagement score (five 5s, four 3s) — evaluate for Sprint 3 | Backlog — evaluate |
| F15 | Three respondents left contact details for interviews | Process | 1:1 follow-up interviews in Sprint 3 for deeper qualitative feedback — first session conducted 2026-09-27 ([User Interviews](user-interviews.md)); two volunteers remain | In progress — 1 of 3 done |
| F16 | Injuries and expected return dates for players (Q9: 1) | Data / Feature | Injury data is not part of the ingested NBA dataset — evaluate adding an injury source in Sprint 3 | Backlog — evaluate |
| F17 | Player-level predictions, not just game-level (Q13: 1 — the hardcore-fantasy respondent) | Trust / Feature | Per-player matchup projections already exist (`GET /v1/players/:id/matchup-projection`, PR #105); surface player-level predictions alongside the game-level ones | Shipped (PR #105) — surface + extend |
| F18 | Some pages carry too much information (Q21: 1) | UX / Copy | Progressive-disclosure pass — lead with the headline numbers and keep depth one click away; serves this and the novice-guidance asks (F7–F9) together | Backlog — file as Gitea issue |
| F19 | Player height (Q9: 1) | Data | Height and weight already come from the CommonPlayerInfo bio fields; verify they render on the player profile page | Verify, then file if needed |

### Next steps (Sprint 3)

1. **File the backlog items** (F3–F14, F16, F18 and the interview items F20–F31) as Gitea issues and prioritise them in the sprint backlog, per the [integration process](testing.md#how-feedback-is-integrated)
2. **Run hands-on testing sessions** on the live app rather than screenshots — the first was combined with the 2026-09-27 interview; more are planned
3. **Re-run the survey** after Sprint 4 improvements — compare against the n=11 baseline (2026-09-14/15) to measure the before/after delta. The follow-up interview is complete; additional sessions are contingent on more volunteers coming forward

---

## Survey Introduction

> **Welcome to the NBA Analytics Tool survey!**
> 
> We're building a web platform that helps basketball fans and fantasy players explore NBA data, predict game outcomes, and optimise fantasy lineups. Your feedback will directly shape the next version of this tool.
> 
> **What you'll see:** Screenshots of the current prototype (some UI elements are still being polished).  
> **What we need:** Your honest opinions on usability, usefulness, and what's missing.
> 
> This survey takes about 5–7 minutes. All responses are anonymous unless you choose to provide your email at the end.
> 
> *Thank you for your time!*

---

## Section 1: About You

### Q1. How familiar are you with basketball?
- [ ] Not at all familiar
- [ ] Casual fan (follow occasionally)
- [ ] Regular fan (watch games weekly)
- [ ] Hardcore fan (follow daily, know stats)

### Q2. Have you ever played fantasy basketball (NBA Fantasy, ESPN, Yahoo, etc.)?
- [ ] Never
- [ ] Tried it once or twice
- [ ] Played a few seasons
- [ ] Play regularly every season

### Q3. What device are you using to answer this survey?
- [ ] Phone
- [ ] Tablet
- [ ] Desktop/Laptop

---

## Section 2: First Impressions

*[SCREENSHOT: Landing page — full-screen basketball hero, "NBA Fantasy League Optimizer" heading, "Court Vision" tagline, Get Started button]*

### Q4. Looking at the landing page, what do you think this tool does?
- [Open text]

### Q5. How visually appealing is the homepage?
- [ ] 1 — Not appealing at all
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very appealing

### Q6. Is it clear what you can do on this site?
- [ ] Very clear
- [ ] Somewhat clear
- [ ] Not very clear
- [ ] Not clear at all

**If not clear, what's confusing?**  
- [Open text]

---

## Section 3: Your Home Dashboard

*[SCREENSHOT: Signed-in home page ("The Locker") — Beat the Model card, Watchlist board, Your Teams, Leaderboard, Saved shelf, Model Accuracy Ledger]*

### Q6b. How useful is having a personalised dashboard with your followed players and teams?
- [ ] 1 — Not useful at all
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Extremely useful

### Q6c. The "Beat the Model" game (predict game outcomes and compare your accuracy against the algorithm) — how engaging is it?
- [ ] 1 — Not engaging at all
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very engaging

---

## Section 4: Player Browsing

*[SCREENSHOT: Players list page with search bar, filters, and player cards]*

*[SCREENSHOT: Individual player profile page showing stats, headshot, team info]*

### Q7. How easy is it to find a specific player?
- [ ] 1 — Very difficult
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very easy

### Q8. The player stats (points, rebounds, assists, etc.) — how useful are they?
- [ ] Not useful at all
- [ ] Slightly useful
- [ ] Moderately useful
- [ ] Very useful
- [ ] Extremely useful

### Q9. What additional player information would you like to see?
- [Open text]

---

## Section 5: Team Browsing

*[SCREENSHOT: Teams list page showing team cards with logos]*

*[SCREENSHOT: Individual team profile page]*

### Q10. How easy is it to browse teams?
- [ ] 1 — Very difficult
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very easy

### Q11. Would you like to see more team details (roster, recent games, standings)?
- [ ] Yes, definitely
- [ ] Maybe
- [ ] No, current info is enough

**If yes, what's most important?**  
- [Open text]

---

## Section 6: Game Predictions

*[SCREENSHOT: Games list page showing upcoming/past games with win probabilities]*

*[SCREENSHOT: Individual game detail page showing prediction, Elo ratings, predicted top scorers]*

### Q12. How confident would you feel using these predictions for fantasy basketball decisions?
- [ ] 1 — Not confident at all
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very confident

### Q13. What would make the predictions more trustworthy?
- [ ] Show historical accuracy (e.g., "65% of past predictions were correct")
- [ ] Explain how predictions are calculated (methodology)
- [ ] Show more detail (player-level predictions, not just game-level)
- [ ] Compare predictions to other sources (ESPN, CBS, etc.)
- [ ] Other: [Open text]

### Q14. Would you use these predictions for:
- [ ] Fantasy basketball lineup decisions
- [ ] Sports betting
- [ ] Just for fun / curiosity
- [ ] Nothing — I wouldn't use them
- [ ] Other: [Open text]

---

## Section 7: Fantasy Optimiser

*[SCREENSHOT: Optimizer page showing salary cap, player selection, and generated lineup]*

### Q15. How useful is the fantasy lineup optimiser?
- [ ] 1 — Not useful at all
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Extremely useful

### Q16. How easy is it to understand the optimiser's output?
- [ ] 1 — Very confusing
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very clear

### Q17. What would improve the optimiser?
- [ ] Allow custom constraints (e.g., "must include LeBron")
- [ ] Show alternative lineups (not just the top one)
- [ ] Explain why certain players were chosen
- [ ] Add projected points for each player
- [ ] Other: [Open text]

---

## Section 8: Overall Experience

### Q18. Overall, how satisfied are you with the NBA Analytics Tool?
- [ ] 1 — Very dissatisfied
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very satisfied

### Q19. How likely are you to use this tool again in the future?
- [ ] 1 — Not likely at all
- [ ] 2
- [ ] 3
- [ ] 4
- [ ] 5 — Very likely

### Q20. What's the **most useful** feature?
- [ ] Player browsing and stats
- [ ] Player comparison
- [ ] Team browsing
- [ ] Game predictions and model accuracy
- [ ] Beat the Model challenge
- [ ] Player watchlist
- [ ] Fantasy lineup optimiser
- [ ] Saved comparisons and lineups
- [ ] Other: [Open text]

### Q21. What's **missing** or could be improved?
- [Open text]

### Q22. Did you encounter any bugs, errors, or confusing elements?
- [ ] Yes
- [ ] No

**If yes, please describe:**  
- [Open text]

---

## Section 9: Final Thoughts

### Q23. If you could add one feature to this tool, what would it be?
- [Open text]

### Q24. Any other comments or suggestions?
- [Open text]

### Q25. (Optional) Would you like to be contacted for a follow-up interview?
- [ ] Yes — email: [Short answer]
- [ ] No, I prefer to stay anonymous

---

## Thank You Message

> **Thank you for your feedback!**
> 
> Your responses will directly influence the next version of the NBA Analytics Tool. We're working on:
> - Expanded prediction coverage (more games, deeper backtesting)
> - Coach mode (upload your own stats and get evaluated against pro players)
> - More detailed team analytics and standings
> - Mobile-optimized layouts
> 
> If you'd like to see the final product or have more ideas, feel free to reach out!
> 
> — The NBA Analytics Team

---

## Screenshot Placeholders (for Kiran to fill in)

| Screenshot | Description | Status |
|---|---|---|
| Landing page | Full-screen basketball hero, "NBA Fantasy League Optimizer" heading, Court Vision tagline | ⚠️ Placeholder needed |
| Home dashboard | "The Locker" — Beat the Model card, Watchlist, Your Teams, Leaderboard, Saved shelf, Accuracy Ledger | ⚠️ Placeholder needed |
| Players list | Player cards with search, filters, and postseason filter | ⚠️ Placeholder needed |
| Player profile | Player page with stats, season splits, headshot, team info | ⚠️ Placeholder needed |
| Compare | Side-by-side player comparison with radar charts | ⚠️ Placeholder needed |
| Teams list | Team cards with logos | ⚠️ Placeholder needed |
| Team profile | Individual team page with roster | ⚠️ Placeholder needed |
| Predictions | Game predictions with model accuracy and calibration bands | ⚠️ Placeholder needed |
| Game detail | Game detail page with prediction, Elo ratings, predicted top scorers | ⚠️ Placeholder needed |
| Optimizer | Fantasy lineup generator with salary cap and saved lineups | ⚠️ Placeholder needed |

**Note for Kiran:** Take screenshots at 1920×1080 resolution. Crop to show the key UI element in each shot. The live deployed version is at [sportsanalytics.pages.dev](https://sportsanalytics.pages.dev/) — some features (Beat the Model, watchlist, saved comparisons) require signing in with Google.

---

## Google Forms Setup Instructions

1. Go to [forms.google.com](https://forms.google.com)
2. Create a new form titled "NBA Analytics Tool — User Feedback Survey"
3. Copy the introduction text into the form description
4. Add each question in order:
   - **Multiple choice** for single-select questions
   - **Checkboxes** for multi-select questions
   - **Short answer** for open-ended questions
   - **Linear scale** for 1–5 rating questions
5. Add section breaks between each "Section" heading
6. Insert screenshots between sections (use the image icon in the right toolbar)
7. Enable "Collect email addresses" only for Q25 (optional follow-up)
8. Set the form to anonymous by default
9. Test the form yourself before sharing
10. Generate the share link and distribute via WhatsApp

The form's responses land in its linked Google Sheet — the [live response sheet](https://docs.google.com/spreadsheets/d/1LTA5ckTegY2CqBV-1OsBeTl-OzeHkn6OoSdLVkws8X8/edit?usp=sharing) for this survey (with the email column redacted before sharing).

---

*AI Declaration: This survey was drafted and its responses analysed with the assistance of Qoder[Qoder Lite].*
