# Valuation Model

How [Become Pro](index.md) turns a user's season line into a projected draft pick and a dollar figure. There is no LLM in this pipeline: a Python job fits an ordinary least-squares model on real NBA rookie seasons, and the API applies it.

The figure answers "which pick's rookie year does this line most resemble", not "where would this player be drafted" ([Known limitations](#known-limitations)). The page calls it "not an offer and not a market price" and always shows a range.

## From a season line to a dollar figure

1. **Adjust for level.** Volume stats are multiplied by the [competition-level factor](#competition-level-factors).
2. **Project a pick** from points, rebounds and assists per game plus true shooting %, through the fitted model.
3. **Price the pick** on the [rookie salary scale](#the-rookie-salary-scale).
4. **Add a range** around the value ([The value range](#the-value-range)).
5. **Explain it:** short sentences on what drives the value, the three [comparable rookies](#comparable-rookies), and the players actually drafted at that pick.

The API runs these steps whenever the season changes. Everything it needs (coefficients, scale, level factors, range settings, comparable index) is trained by Python and stored in one `ProspectValuationModel` row, so each setting is defined once.

## Training the model

`apps/valuation/train_valuation_model.py` fits the model on real NBA rookie seasons.

| Step | Detail |
|---|---|
| Training rows | Every drafted player whose rookie regular season is in the database, with at least 20 published games. The rookie season comes from `draftYear`: a 2023 draftee's is 2023-24. |
| Features | Points, rebounds and assists per game, and true shooting % |
| Fit | Ordinary least squares (`numpy.linalg.lstsq`), with the pick number as the target |

No dataset pairs amateur seasons with draft outcomes, so the model learns "rookie production → pick" and applies it to the user's line. With about 140 rows, four features is about as much as the data supports, and every coefficient can be inspected.

### The production model

Trained against production on 27 Sep.

| | |
|---|---|
| Training rows | 140 rookie seasons (the 2023, 2024 and 2025 draft classes) |
| Mean absolute error | 10.9 picks |
| Rank correlation (Spearman) | 0.54 |

Both figures are **in-sample**: they are measured on the rows the model was fitted to, so they are a best case, not the error on unseen players. That's why the page leads with a range.

## Valuing a season on every write

- **On every change.** After a game is logged, edited or removed, or the season's details change, the API re-derives the line, applies the model and stores a `ProspectValuation` row, but only if a figure changed.
- **A new model applies on the next visit.** If the stored valuation came from an older model, the next page load re-values it.
- **The model is cached for 60 seconds.** Only the model; the user's own data never is ([Performance](../design/performance.md#what-is-cached-and-what-deliberately-is-not)).

| State | Meaning | The page shows |
|---|---|---|
| `VALUED` | The season has a value | Pick, value, range, drivers, comparables |
| `BELOW_GAMES_FLOOR` | Fewer than 10 games: the user's to fix | "N more games needed", no figure |
| `AWAITING_MODEL` | Enough games, but no trained model yet: the system's to fix | Says so |

The two "no figure" states read differently so a user with 30 games can tell "the model isn't ready" from "I haven't logged enough".

## Competition-level factors

Production at each level is scaled to a Division I footing before it is valued.

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

Counting stats are multiplied by the factor; shooting percentages and minutes are left alone, because shooting 58% against weaker opposition still means the shots went in. **The factors are judgement, not fitted,** since there is no amateur-to-NBA data to fit them on. The page shows the factor and a one-sentence basis for it, so the assumption is never hidden in the dollars.

## The rookie salary scale

| Projected pick | Priced at |
|---|---|
| 1–30 | The 2026-27 first-year rookie scale: pick 1 $12,290,000 down to pick 30 $2,439,000 |
| 31–60 | The 2026-27 two-way salary, $678,882 |
| Beyond 60 | The maximum Exhibit 10 bonus, $91,000 ("undrafted range") |

The project has no salary or contract data, and the rookie scale is small, fixed and public. The figures are Hoops Rumors' July 2026 table (its 120% figures divided by 1.2), cross-checked against 2025-26. The page prints the scale year beside every figure.

## The value range

The range is ±28% of the value, plus another 15% while fewer than 25 games are logged. It is clamped so it never goes above what pick 1 is paid or below the undrafted figure; testing had found a range of "$7.3M – $18.3M", and no pick is paid $18.3M.

## Comparable rookies

The three real NBA rookies closest to the user's level-adjusted line, measured as distance across the four standardised features and shown as a similarity from 0 to 1. Standardising stops points (a wide spread) from swamping assists (a narrow one). Candidates come from the model's own training set, so every comparable is a real rookie season the model learned from.

## Known limitations

- **It answers "resembles", not "would be drafted".** Age, size, athleticism and scouting aren't in the data.
- **Only drafted players.** It says little about lines far below the weakest rookie season.
- **Minutes follow the pick.** High picks play more, so rookie production partly reflects draft position.
- **Players who left the league are missing.** Ingestion keeps only current rosters, so draftees who washed out are absent. That makes the fit optimistic, more so for older classes.
- **Small sample.** About 140 seasons. A strong Division I line projects to pick 1.
- **In-sample accuracy.** There is no held-out test yet.

## Operating it

```bash
cd apps/valuation
pip install -r requirements.txt
DATABASE_URL=<target> python train_valuation_model.py
```

The script reads only `apps/valuation/.env`, never the repo's root `.env`, which points at production. The API picks up a new model within a minute. Render has no Python, so training runs from a team member's machine, like the predictor and optimizer.

**Retrain** when the 2026 draft class has about 20 games of 2026-27, and every summer when a new rookie scale is published (update `apps/valuation/rookie_scale.py` first).

The 24 Python tests (`test_valuation.py`) run locally with `pytest`; CI doesn't run Python tests yet. `MINIMUM_GAMES_REQUIRED` (10) is defined in both `train_valuation_model.py` and the API's `valuation-state.ts`, and the two must match.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
