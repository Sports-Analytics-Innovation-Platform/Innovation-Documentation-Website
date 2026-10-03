# Improvements Made Based on User Feedback

This page documents the improvements shipped based on user feedback from the [Sprint 2 survey](feedback-survey.md) and [follow-up interviews](user-interviews.md). Each improvement is mapped to the originating feedback item (F1–F31) and shows the before/after state — the evidence required for the Milestone 3 **Improvement** criterion (5% weight).

**Note on causality:** F1, F2, F5, and F13 were already in development or landed during the survey window (PRs #94, #95, #111, #123). The survey confirmed the need for these changes rather than originating them. F7 (partial) and F17 were shipped after the survey. F15 (interviews) and F9 (pending merge) are direct responses to survey and interview feedback. No interview-sourced changes have shipped yet.

---

## Shipped Improvements

### F1 — Historical Prediction Accuracy → Model Accuracy Ledger

**Feedback:** "Show historical accuracy (e.g., '65% of past predictions were correct')" — Q13 (5 respondents + 2 near-variants)

**Before:** No visibility into whether the model was actually accurate. Users had to take the predictions on faith.

**After:** The **Model Accuracy Ledger** on the home dashboard shows:
- Overall accuracy percentage (e.g., "62%")
- Brier score (lower is better — measures prediction quality, not just hit/miss)
- Calibration bands (how close the predicted probabilities were to actual outcomes)
- Recent hit/miss history

**Where:** [ModelAccuracyLedger.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/components/home/ModelAccuracyLedger.tsx) — shipped in PR #94

**Impact:** Directly addresses the #1 trust gap. Confidence in predictions was the lowest-rated measure in the survey (3.73/5). The ledger is now the first thing users see on the home page.

---

### F2 — Explain How Predictions Work → "How It Works" Explainer

**Feedback:** "Explain how predictions are calculated (methodology)" — Q13 (1 respondent, but echoed in interviews)

**Before:** Predictions showed win probabilities and margins with no explanation of where they came from. Users saw "GSW 68%" but didn't know what drove it.

**After:** The Predictions page now has a **"How it works"** section that explains:
- Two separate models: Elo ratings (team strength over time) and Four Factors (offensive/defensive efficiency)
- What each factor measures (effective shooting, turnover rate, free-throw rate, rebounding rate)
- Why two models are better than one (Elo captures momentum, Four Factors capture style matchups)
- Plain-English descriptions — no math formulas, just "here's what the numbers mean"

**Where:** [PredictionsPage.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/pages/PredictionsPage.tsx#L350-L374) — shipped in PR #95

**Impact:** Addresses the "black box" problem. Users can now understand *why* the model predicts what it does, which builds trust even when the prediction is wrong.

---

### F5 — Show Projected Points Per Player in Optimizer → Per-Slot Predictions

**Feedback:** "Add projected points for each player" — Q17 (3 respondents)

**Before:** The optimizer showed the generated lineup with player names and salaries, but no projection of how many fantasy points each player would score.

**After:** Each slot in the optimized lineup now shows:
- Player name and team
- Salary
- **Projected fantasy points** for that slot (based on the matchup projection)
- Total projected points for the entire lineup

The projections are also snapshotted into saved lineups, so users can compare their saved lineups' projected vs. actual performance later.

**Where:** [OptimizerPage.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/pages/OptimizerPage.tsx) — shipped in PR #111

**Impact:** Makes the optimizer's output actionable. Users can see *why* a player was chosen (high projection) and make informed decisions about whether to accept the optimized lineup or manually adjust.

---

### F7 (partial) — Stat Abbreviations Unexplained → Abbreviation Explainer in Player Traits Radar

**Feedback:** "What RPG, APG, TS%, etc mean" — Q9/Q21 (1 respondent); "The fix F7 asks for already exists in-app. The compare page's abbreviation explanations were praised — extend the pattern rather than invent a new one" — Interview 1 (2026-09-27)

**Before:** Player profiles showed stat tiles labeled "PPG", "RPG", "APG", "MPG", "STL", "BLK", "TOV" with no explanation. Novice users (7 of 11 survey respondents) didn't know what these meant.

**After:** The **Player Traits Radar** on the player profile page now has an interactive explainer:
- Click any trait axis (Scoring, Rebounding, Playmaking, Defense, Efficiency) to expand it
- Each stat abbreviation is shown with its full name and a plain-English explanation:
  - **PTS/G**: "Points per game. The headline scoring output — simply how many points they average a night."
  - **FGA/G**: "Field goal attempts per game — how many shots they take. Volume is half of what fills the scoring column."
  - **TS%**: "True shooting percentage. Scoring efficiency that counts threes and free throws, so volume chuckers and efficient scorers are not lumped together."
  - **REB/G**: "Rebounds per game — missed shots they collect. Offensive boards keep a possession alive; defensive ones end the other team's."
  - **AST/G**: "Assists per game — baskets they directly set up for teammates. The raw measure of a player's passing output."
  - **AST:TO**: "Assist-to-turnover ratio. Playmaking weighed against mistakes — above 2.0 means they create twice as often as they cough it up."
  - And 10+ more

The radar itself is also interactive — clicking a trait axis highlights that trait and shows the raw season figures behind the normalized shape.

**Where:** [PlayerTraitsRadar.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/components/PlayerTraitsRadar.tsx#L45-L160) — the `explain` field on every `TraitStatLine`

**Impact:** Directly praised in the first follow-up interview: "The trait radar 'is cool', and the abbreviation explanations beside it were appreciated." This is the exact pattern the interview recommended extending site-wide. The stat tiles on the player profile (PPG, RPG, etc.) still lack tooltips, but the radar's explainer panel covers the most important advanced stats.

---

### F13 — More Team Details → Team Profile Pages with Records, Recent Form, Elo Ratings

**Feedback:** "More team details — recent games with outcomes" — Q11 (3 yes + 6 maybe)

**Before:** Team pages showed team name, logo, conference, and division. No record, no recent games, no Elo rating.

**After:** Team profile pages now show:
- Win/loss record
- Elo rating (team strength metric)
- Recent form (last 5 games with results)
- Follow/unfollow button (adds to your watchlist on the home dashboard)
- Roster view (players on the team)

The teams list page also gained:
- Filtering by conference/division
- Sorting by Elo, win percentage, or name
- Search by team name
- Elo ratings displayed on each team card

**Where:** [TeamProfilePage.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/pages/TeamProfilePage.tsx) and [TeamsListPage.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/pages/TeamsListPage.tsx) — shipped in PR #123

**Impact:** Addresses the "more team details" request and the interview finding that "teams list sorted by Elo confuses — expected favourite team first" (F23). Users can now see a team's record and recent performance at a glance.

---

### F17 — Player-Level Predictions → Matchup Projections Per Player

**Feedback:** "Show more detail (player-level predictions, not just game-level)" — Q13 (1 respondent, the hardcore-fantasy player)

**Before:** Predictions were game-level only: "LAL vs BOS, LAL 58% win probability." No per-player breakdown.

**After:** Each player profile page now shows a **Matchup Projection** section:
- Projected points for the player's next game
- Opponent team and whether it's home/away
- How the projection was calculated (shrunk toward the player's overall season rate, based on recent performance and opponent strength)

The projection is computed by `GET /v1/players/:id/matchup-projection`, which blends the player's recent form with their season average, adjusted for the upcoming opponent.

**Where:** [PlayerProfilePage.tsx](https://github.com/Sports-Analytics-Innovation-Platform/sportsanalytics/blob/main/apps/web/src/pages/PlayerProfilePage.tsx) — shipped in PR #105

**Impact:** Directly serves the hardcore fantasy player persona (the target audience). Fantasy players need player-level projections to make lineup decisions, not just game-level win probabilities.

---

### F15 — Follow-Up Interviews → First Hands-On Session Conducted

**Feedback:** "One respondent left contact details for a follow-up interview" — Q25 (1 respondent)

**Before:** Survey was screenshot-based (not hands-on). Respondents answered while viewing static screenshots, not using the live app.

**After:** The first follow-up interview was conducted on **2026-09-27** — a hands-on walkthrough of the live app with a survey volunteer. The participant drove the app while thinking aloud, which surfaced:
- Two real bugs (F25, F26) that the screenshot-based survey structurally could not find
- Comprehension gaps on Beat the Model (F20), the optimizer (F27), and Become Pro (F29)
- Confirmation that the abbreviation explainer pattern (F7) was the right fix
- Twelve new feedback items (F20–F31) that are now queued for triage

**Where:** [User Interviews](user-interviews.md) — Interview 1 (2026-09-27)

**Impact:** Validates the survey findings and uncovers issues the survey format couldn't catch. The interview confirmed that "the novice-onboarding gap (F8/F9) is real and page-specific" and that "hands-on testing finds what screenshots cannot."

---

### F9 — Beginner's Guide → Player Archetypes & Style Map (Pending Merge)

**Feedback:** "Beginner's guide — 'what am I looking at and how to use the tool'" — Q23/Q21 (1 respondent); "The novice-onboarding gap (F8/F9) is real and page-specific" — Interview 1 (2026-09-27)

**Before:** Player profiles showed raw stats with no context about *what kind of player* this is. Novices couldn't tell if a player was a scorer, playmaker, defender, or role player — just numbers without meaning.

**After:** The **Style & Similar Players** section on the player profile page shows:
- **Playing Style archetypes** — e.g., "Scoring guard 69%, Point forward 14%" — plain-English labels describing how the player plays
- **Style Map** — a scatter plot placing every NBA player in a 2D space (off-ball role vs. on-ball creation, perimeter shooting) so you can see where this player sits relative to the league
- **Tooltip explainer** — an "i" icon that opens a panel explaining: how archetypes are computed (shot diet, playmaking load, rebounding, size — not by position), that position is not part of it, how similar players are found (closest in that space), and the model's limitations (box-score data only, can't see defence beyond steals/blocks)
- **Similar players** — listed by proximity in the style space, so you can find comparable players at a glance

**Where:** `player-archetypes` branch (not yet merged to `main` as of 2026-09-29)

**Impact:** Directly addresses the novice-onboarding gap (F9). Instead of a separate "beginner's guide" page, the explanation is embedded right where the user needs it — on the player profile, next to the chart. The tooltip pattern is the same approach the interview praised for the abbreviation explainer (F7): explain inline, don't make users go find a glossary.

---

## Summary Table

| Feedback ID | What was requested | What shipped | PR | Status |
|---|---|---|---|---|
| F1 | Show historical prediction accuracy | Model Accuracy Ledger on home dashboard | #94 | **Shipped** |
| F2 | Explain how predictions are calculated | "How it works" explainer on Predictions page | #95 | **Shipped** |
| F5 | Show projected points per player in optimizer | Per-slot projections on optimizer board + saved lineups | #111 | **Shipped** |
| F7 (partial) | Stat abbreviations unexplained | Abbreviation explainer in Player Traits Radar (click any trait axis) | — | **Shipped** (pattern praised in interview) |
| F13 | More team details (recent games, records) | Team profile pages with record, Elo, recent form, roster | #123 | **Shipped** |
| F17 | Player-level predictions, not just game-level | Matchup projections per player on profile page | #105 | **Shipped** |
| F15 | Follow-up interview with survey volunteer | Hands-on interview conducted 2026-09-27, 12 new feedback items (F20–F31) | — | **Complete** |
| F9 | Beginner's guide — "what am I looking at" | Player Archetypes & Style Map with tooltip explainer on player profile | — | **Pending merge** (`player-archetypes` branch) |

---

## What's Still Backlog

The following feedback items have **not yet been addressed** and remain in the backlog for Sprint 4:

- **F3**: Compare predictions with other sources (ESPN/CBS)
- **F4**: Optimizer: allow custom constraints ("must include LeBron")
- **F6**: Optimizer: show alternative lineups (not just the best)
- **F7 (remaining)**: Extend abbreviation explainer to stat tiles on player profile (PPG, RPG, etc.) — the radar covers the advanced stats, but the basic tiles still lack tooltips
- **F8**: Landing-page copy is "wordy" with "LLM-esque language" — copy-simplification pass
- **F10**: Tolerant player search ("manon" should find "Chris Mañón")
- **F11**: All-time NBA stat leaders (requires historical data beyond 3 seasons)
- **F12**: Show player position in player stats (verify if surfaced; file if missing)
- **F14**: Beat the Model: random daily/weekly games
- **F16**: Injuries and expected return dates for players
- **F18**: Progressive-disclosure pass — some pages carry too much information
- **F19**: Player height (verify if bio fields render on profile page; file if missing)
- **F20–F31**: All twelve interview-sourced items (see [User Interviews](user-interviews.md))

---

## Next Steps

1. **Re-run the survey** after Sprint 4 improvements — compare against the n=11 baseline (2026-09-14/15) to measure the before/after delta. This is the Milestone 3 Improvement criterion evidence. The follow-up interview is complete; additional sessions are contingent on more volunteers coming forward
2. **File the backlog items** (F3–F14, F16, F18, F20–F31) as Gitea issues and prioritize them in the Sprint 4 backlog
3. **Extend the abbreviation explainer pattern** (F7 remaining) to the stat tiles on the player profile page — the interview confirmed this is the right approach

---

*AI Declaration: This page was created with the assistance of Qoder[Qoder Lite].*
