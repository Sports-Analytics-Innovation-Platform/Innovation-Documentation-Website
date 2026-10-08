# Become Pro

A signed-in user logs their own games. The app works out their season line the same way it does for an NBA player, projects the NBA draft pick that line most resembles, prices that pick on the NBA rookie salary scale, and shows the real NBA rookies it most resembles. **It is private to each user.**

Merged in PR #192 and live since 27 Sep. How the number is produced is on [Valuation Model](valuation-model.md).

## How it works

| Part | Role | Runs |
|---|---|---|
| `apps/web` | The `/become-pro` page, plus a small card on Home and Profile | In the browser |
| `apps/api/src/become-pro` | Signed-in routes; derives the season line and values it on every write | Always on (Render) |
| `apps/valuation` | Trains the model on real NBA rookie seasons and stores it as one database row | By hand, as a batch job |

When a user saves a game, the API checks it (see [Box-score validation](#box-score-validation)), stores it, re-derives the season line with the code the NBA player pages use, applies the newest model, and stores a new valuation if anything changed. The page then reloads everything in one request, `GET /v1/me/become-pro`.

Training is separate because it needs the whole NBA rookie dataset and only has to run when NBA data changes. Applying the trained model is a quick calculation, so the API does it on every write ([Valuation Model](valuation-model.md#valuing-a-season-on-every-write)).

## Privacy

A user's Become Pro data is visible **only to that user**. There is no leaderboard, no public profile and no comparison between users.

- Every route sits under `/v1/me/become-pro`, behind the session guard.
- Another user's season or game returns `404`, exactly like an id that doesn't exist ([API Design](../design/api-design.md#auth-model)).
- Signed out, the page shows the usual "Sign in required" prompt.

Because a self-reported figure reaches only the person who entered it, nothing a user enters needs verifying ([Security](../security.md#self-reported-data-become-pro)).

## What the user does

**Seasons.** A season is one league year at one competition level, for example "2025-26 · NCAA Division II · G · Riverside College". The year comes from a picker of the 8 most recent league years, so it always matches the NBA's "2025-26" format. Position and level are required, because level is the biggest single input to the value. Limits: 12 seasons per user, one per league year.

**Games.** One box score per game: date, opponent, MIN, PTS, REB, AST, STL, BLK, TOV, FGM, FGA, 3PM, 3PA, FTM and FTA. Entry is built for back-filling a season in one sitting: Enter saves from any field, the date carries forward, and "Copy last game" pre-fills the previous row. Each game can be edited or removed. Limits: 120 games per season, and no two games with the same date and opponent.

Games are logged one by one, not as a typed season line, because every figure the app publishes traces back to per-game records.

### Box-score validation

Each game is checked for arithmetic that can't be true, using `findStatAnomalies`, the same checker the admin tools run over NBA data.

| Blocks the save | Warns but still saves |
|---|---|
| A negative stat; more makes than attempts; more threes than field goals (made or attempted); over 65 minutes; a date in the future | Points that don't equal 2×FGM + 3PM + FTM |

Mismatched points only warn, because real scoresheets sometimes don't add up. The browser runs the same rules, so a row it lets you save is always one the API accepts.

## What the user sees

- **The season line:** games, per-game averages, shooting percentages and TS%, a points-by-game chart and a traits radar. Percentages come from season totals (6-for-21 is 28.6%), and a percentage with no attempts shows "—". A warning appears under 4 games.
- **The projected value card:** the projected pick (for example "Pick 14"), the dollar figure and its [range](valuation-model.md#the-value-range), a value-over-time sparkline, and short server-written sentences on what drives the value. **Below 10 games there is no figure at all**, only "N more games needed", because "not valued yet" isn't "valued at nothing".
- **Players drafted at that pick:** the three most recent real players taken there, each linking to their page.
- **Home and Profile card:** the season, value, pick and sparkline, from a lighter `/summary` route.

### Comparison with NBA rookies

The three real NBA rookies whose rookie line is most like the user's, each with a percentage similarity and a link to their page. Candidates come from the model's own training set. The radar plots the user's **level-adjusted** line, because that is the line the similarity was measured on ([Valuation Model](valuation-model.md#comparable-rookies)).

## Built, then removed

The first design (22–25 Sep) had a public value leaderboard, public prospect profiles, uploaded scorecards as evidence, and an admin review queue. **All of it was removed on 26 Sep**, when the feature was re-scoped to "just comparing your stats to nba players/rookies". With no comparison between users, there was nothing to verify.

## Testing

| Suite | Result at hand-off |
|---|---|
| API | 974 tests, including 31 Become Pro end-to-end tests against a real Postgres |
| Web | 717 tests, 90.2% line coverage |
| Python (`apps/valuation`) | 24 tests |
| Live browser test (26–27 Sep) | 65 of 65 checks passed with two real signed-in users, including privacy between them |

The full design and build history is in the AI chat export [kiran-2026-09-27-become-pro.txt](../transcripts/ai_transcripts/kiran-2026-09-27-become-pro.txt). Routes are in the [API Reference](../api-reference.md#become-pro); tables in the [ERD](../design/erd.md#become-pro).

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
