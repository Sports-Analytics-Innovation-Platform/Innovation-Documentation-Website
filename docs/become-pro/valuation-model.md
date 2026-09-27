# Valuation Model

How [Become Pro](index.md) turns a user's season line into a projected draft pick and a dollar figure. There is no LLM anywhere in this pipeline: a Python job fits an ordinary least-squares model on real NBA rookie seasons, and the API applies it.

!!! warning "What the figure means, and what it doesn't"
    The model answers **"which pick's rookie year does this line most resemble"**, not "where would this player be drafted". Age, size, athleticism and scouting aren't in the data. The page says the figure is "not an offer and not a market price", and it always shows a wide range beside it. See [Known limitations](#known-limitations) before quoting a figure.

## From a season line to a dollar figure

1. **Adjust for level.** The season's volume stats are multiplied by its [competition-level factor](#competition-level-factors), to put the line on a Division I footing. Rates are left alone.
2. **Project a pick.** Points, rebounds and assists per game, plus true shooting %, go through the fitted linear model to give a draft pick.
3. **Price the pick** on the [rookie salary scale](#the-rookie-salary-scale).
4. **Add a range** around the value, [clamped to the scale](#the-value-range).
5. **Explain it.** The API writes the drivers, finds the three [comparable rookies](#comparable-rookies), and looks up the players actually drafted at that pick.

Steps 1–5 run in the API every time the season changes. Everything the API needs for them (coefficients, scale, factors, interval widths and the comparable index) is trained and stored by Python, in one `ProspectValuationModel` row.

## Training the model

A Python job, `apps/valuation/train_valuation_model.py`, fits a model on real NBA rookie seasons that maps a season line to a projected draft pick.

| Step | Detail |
|---|---|
| Training rows | Every drafted player whose rookie regular season is in the database. The rookie season is worked out from `draftYear`: a 2023 draftee's is 2023-24 |
| Filters | Published games only, and at least 20 games |
| Features | Points, rebounds and assists per game, plus true shooting % |
| Fit | Ordinary least squares via `numpy.linalg.lstsq`, with the pick number as the target |
| Output | One `ProspectValuationModel` row holding the coefficients, the rookie scale, the level factors, the interval settings and the comparable-player index. The fit's error (MAE, in picks) and Spearman rank correlation are stored on the same row |

### The production model

Trained against production on 2026-09-27, from the `train-valuation-model` branch. Nothing else in production was written.

| | |
|---|---|
| Training rows | 140 rookie seasons (the 2023, 2024 and 2025 draft classes) |
| Mean absolute error | 10.9 picks (in-sample) |
| Rank correlation (Spearman) | 0.54 (in-sample) |
| Model id | `c40a1d20-5001-4ec1-8aac-dad196768e61` |

An MAE of 10.9 picks means that, on the 140 rookie seasons it was fitted to, the model's pick is on average about eleven places from where the player was actually drafted. Both figures are **in-sample**: `fit_slot_model` in `draft_slot_model.py` scores the model on its own training rows, and there is no held-out evaluation yet. So 10.9 picks is a best case, not a measured error on unseen players. That's why the page leads with a range rather than a single number.

**Why it's built this way:**

- **Trained on NBA rookies.** No dataset anywhere pairs amateur seasons with draft outcomes. So the model learns "rookie production → pick" and applies it to the user's line in reverse. `apps/valuation/README.md` states this plainly.
- **Four features and OLS.** With about 140 rows, that is about as much as the data supports. The model is interpretable, and every coefficient can be inspected in the model row.
- **Rookie season from `draftYear`.** The earliest season on record is a rookie year only for recent draftees. Using it would have paired veterans' prime seasons with their draft pick. That was a real bug, caught and fixed during the build.
- **Published games only.** Become Pro follows the same visibility rule as every public figure (`apps/api/src/common/game-visibility.ts`).
- **`db.py` reads only `apps/valuation/.env`.** The root `.env` points at production, and a bare `load_dotenv()` would walk up the directory tree and find it.

**Stack:** Python with `psycopg2-binary`, `numpy`, `python-dotenv` and `pytest` (see [Tech Stack](../tech-stack.md#valuation-appsvaluation)).

**Files:** `apps/valuation/train_valuation_model.py` (entry point), `draft_slot_model.py` (fit, Spearman, slot bounds, interval settings), `rookie_scale.py`, `level_factors.py`, `db.py`, `test_valuation.py`, `README.md`

## Valuing a season on every write

The API applies the newest trained model to a season the moment it changes.

- **On every change.** After a game is logged, corrected or removed, or a season's details are edited, `revalueSeason` re-derives the line, applies the model and stores a `ProspectValuation` row.
- **Only when something changed.** A new row is written only if a figure changed, so correcting a typo in an opponent's name doesn't add a point to the history.
- **A new model applies on the next visit.** When a newer model exists, the page's next read re-values the season (`refreshIfStale`). The API spots this by comparing the stored valuation's `modelId` with the latest model's.
- **The model is cached for 60 seconds** (`USER_ACTIVITY_TTL_MS`, through the API's `ResponseCacheService`). Only the model is cached; the user's own data never is (see [Performance](../design/performance.md#what-is-cached-and-what-deliberately-is-not)).

A season is always in one of three states:

| State | Meaning | What the page shows |
|---|---|---|
| `VALUED` | The season has a projected value | Pick, value, range, drivers, comparables |
| `BELOW_GAMES_FLOOR` | Fewer than 10 games logged: the user's to fix | "N more games needed", with no dollar figure |
| `AWAITING_MODEL` | Enough games, but no model has been trained yet: the system's to fix | Says so, rather than hiding it |

**Why:**

- **Value on write.** This was the user's choice ("API values on every write"), so the figure updates the moment a game is logged. Only the always-on API can do that; Python runs as a batch job.
- **Settings travel in the model row.** Slot bounds, interval widths, the scale and the level factors all arrive inside the model row, so each is defined once, in Python, rather than copied into TypeScript where the two copies could drift.
- **The 10-game floor.** No figure appears until the line has settled. This is the same idea as the accuracy leaderboard's minimum number of calls.
- **A missing model reads differently.** A user with 30 games must be able to tell "the model isn't ready" apart from "I haven't logged enough".

**Files:** `apps/api/src/become-pro/prospect-valuation.service.ts`, `valuation-model.ts` (slot, value, interval, comparables, drivers), `valuation-state.ts` (10-game floor and states), `become-pro.service.ts` (read side)

## Competition-level factors

Translates production at a given level to a Division I footing before valuing it.

| Level | Factor |
|---|---|
| NCAA Division I | 1.0 |
| International pro | 0.95 |
| NCAA Division II | 0.62 |
| NAIA | 0.55 |
| Junior college | 0.52 |
| NCAA Division III | 0.42 |
| Semi-pro | 0.40 |
| High school | 0.30 |
| Recreational | 0.15 |

- **Volume is discounted; rates are untouched.** Points, rebounds, assists, steals, blocks, turnovers and makes/attempts are multiplied by the level's factor. Shooting percentages and minutes are left alone.
- **The basis is shown.** Each factor has a one-sentence basis, and the page prints both the factor and the basis.

**Why:**

- **Level matters.** 30 points a game in a recreational league and 30 in Division I are not the same claim.
- **Ordered by judgement, not fitted.** There is no amateur-to-NBA dataset to fit these factors on. So they are presented as an assumption on the page, never folded silently into the dollars.
- **Rates untouched.** Shooting 58% against weaker opposition still means the shots went in.

**Files:** `apps/valuation/level_factors.py` (the constants), `apps/api/src/become-pro/level-adjustment.ts` (applies them)

## The rookie salary scale

Turns a projected pick into dollars.

| Projected pick | Priced at |
|---|---|
| 1–30 | The published 2026-27 first-year rookie scale at 100%: pick 1 $12,290,000, down to pick 30 $2,439,000 |
| 31–60 | The 2026-27 two-way salary, $678,882 |
| Beyond 60 | The "undrafted range", priced at the maximum Exhibit 10 bonus, $91,000 |

The page prints the scale year next to every figure, and each stored valuation records which scale year produced it.

**Why:**

- **No salary data exists in the project.** Nothing in the repo or in `nba_api` has salary or contract data. The rookie scale is small, fixed and public, which makes "what would this player sign for entering the league" answerable.
- **The pick is shown first.** The pick is what the model computes; the dollars follow from it.

**Source and cross-check:**

- The figures come from Hoops Rumors' July 2026 rookie-scale table (its 120% figures divided by 1.2).
- They were cross-checked against 2025-26: every pick rose exactly in line with the salary cap.
- An earlier hand-entered table had pick 1 about 11% too high and was a year out of date. This one replaced it during testing.

**Files:** `apps/valuation/rookie_scale.py`. The table reaches the API inside the model row.

## The value range

A low–high range is always shown with the figure.

- **Width:** ±28% of the value, plus another 15% while fewer than 25 games are logged.
- **Clamped to the scale:** never above what pick 1 is paid, and never below the undrafted figure.

**Why:**

- **Visibly wide.** The honest reading of the model is "somewhere in this neighbourhood".
- **The clamp.** Testing found a pick-1 range of "$7.3M – $18.3M", and no pick is paid $18.3M.

**Files:** `valueInterval` in `apps/api/src/become-pro/valuation-model.ts`; the settings are in `apps/valuation/draft_slot_model.py`.

## Comparable rookies

The three real NBA rookies whose rookie line is closest to the user's (see the [page section](index.md#comparison-with-nba-rookies) for what's shown).

- **Distance:** Euclidean over the standardised four-feature vector (points, rebounds, assists, true shooting %), turned into a similarity from 0 to 1.
- **Candidates:** drawn from the model's own training set.
- **Their lines:** each rookie's rookie regular season, published games only.
- **The user's line:** the level-adjusted one, the same line the model valued.

**Why standardise:** without it, points (a wide spread) would swamp assists (a narrow one).

Separately, **"Drafted at pick N"** lists the three most recent real players taken at the projected pick (`Player.draftNumber`, newest `draftYear` first). The comparable ids and alumni ids are stored on the valuation row.

**Files:** `findComparables` in `apps/api/src/become-pro/valuation-model.ts`; `slotAlumni` in `prospect-valuation.service.ts`

## Known limitations

These are stated here so the figure isn't read as more than it is:

- **What the model answers.** "Which pick's rookie year does this line most resemble", not "where would this player be drafted". Age, size, athleticism and scouting aren't in the data.
- **Only drafted players.** It's fitted on players who were drafted, so it says little about lines far below the weakest rookie season.
- **Minutes follow the pick.** Rookie minutes partly result from draft position, because high picks play more.
- **Players who left the league are missing.** Ingestion keeps only players on a current roster, so draftees who washed out are absent from older classes. That leans the fit optimistic, more so the older the class.
- **Level factors are judgement, not fitted.**
- **Small sample.** About 140 rookie seasons. A strong Division I line projects to pick 1.
- **The accuracy figures are in-sample.** MAE and rank correlation are measured on the same rows the model was fitted to, so they flatter it. A held-out or cross-validated measure hasn't been added.

## Operating it

**Train or retrain:**

```bash
cd apps/valuation
pip install -r requirements.txt
DATABASE_URL=<target> python train_valuation_model.py
```

The API picks up the new model within about a minute (the 60-second model cache), and re-values each season the next time its owner visits.

**When to retrain:**

- When the 2026 draft class has about 20 games of 2026-27, which adds a fourth class.
- Every summer, when the NBA publishes a new rookie scale. First update `FIRST_ROUND_SCALE`, `ROOKIE_SCALE_YEAR`, `SECOND_ROUND_VALUE` and `UNDRAFTED_VALUE` in `apps/valuation/rookie_scale.py`.

**Where it runs:** the deployed API on Render has no Python, so training runs from a machine with the Python environment, the same as the predictor and optimizer.

!!! note "One constant lives in two places"
    `MINIMUM_GAMES_REQUIRED` (10) must match between `apps/valuation/train_valuation_model.py` and `apps/api/src/become-pro/valuation-state.ts`.

!!! note "Python tests don't run in CI"
    `apps/valuation/test_valuation.py` (24 tests, no database needed) runs locally with `pytest`. CI runs no Python tests for any service yet; that was an open item at hand-off.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
