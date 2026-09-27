# Become Pro

A signed-in user logs their own games. The app works out their season line the same way it does for an NBA player, projects the NBA draft pick that line most resembles, prices that pick on the published NBA rookie salary scale, and shows which real NBA rookies the line resembles. **It is private to each user.**

!!! success "Merged and deployed"
    Merged to `main` in PR #192 (feature commit `3374a85`, branch `BecomeProFeat`) and deployed; Render applied the migration on deploy. The production valuation model was trained on 2026-09-27. Every fact on these pages was checked against the code on 2026-09-27.

This page covers what the user sees and why each part works the way it does. How the number itself is produced (the model, the rookie scale, the level factors, and the limits of what the figure can claim) is on [Valuation Model](valuation-model.md). The full history of how the feature was designed, built, re-scoped and tested (22–27 September 2026) is in the AI chat export [kiran-2026-09-27-become-pro.txt](../transcripts/ai_transcripts/kiran-2026-09-27-become-pro.txt).

## How it fits together

Become Pro spans three parts of the monorepo, and the split between them is deliberate:

| Part | Role | Runs |
|---|---|---|
| `apps/web` | The `/become-pro` page, plus a small card on Home and Profile | In the browser |
| `apps/api/src/become-pro` | Session-guarded routes; derives the season line and values it on every write | Always on (Render) |
| `apps/valuation` | Trains the model on real NBA rookie seasons and stores it as one database row | By hand, as a batch job |

What happens when a user saves one game:

1. The browser posts the box score to `POST /v1/me/become-pro/seasons/:seasonId/games`.
2. The API rejects arithmetic that can't be true (see [Box-score validation](#box-score-validation)), then stores a `ProspectGame` row.
3. `revalueSeason` re-derives the season line with the same code the NBA player pages use, applies the newest trained model, and stores a `ProspectValuation` row if any figure changed.
4. The browser refreshes the page and the Home/Profile card. The page then loads everything it shows in one request, `GET /v1/me/become-pro`.

Training happens separately. Fitting the model needs the whole NBA rookie dataset and only has to happen when NBA data changes, so Python does it. Applying a fitted linear model is a dot product, and it has to happen the moment a user's season changes, which only the always-on API can do. See [Valuation Model](valuation-model.md#valuing-a-season-on-every-write).

## Privacy

A user's Become Pro data is visible **only to that user**. There is no leaderboard, no public profile, and no comparison between users.

- **Guarded routes.** Every route sits under `/v1/me/become-pro`, behind `SessionAuthGuard`.
- **Other users' data returns `404`.** Reading or changing another user's season or game returns `404`, exactly as for an id that doesn't exist, so nothing leaks. This follows the rule the rest of `/v1/me/*` uses (see [API Design](../design/api-design.md#auth-model)).
- **Signed out.** The page shows the app's usual "Sign in required" prompt.

**Why:**

- **It was the user's decision.** On 2026-09-26 the feature was re-scoped to "just comparing your stats to nba players/rookies" (see [Built, then removed](#built-then-removed)).
- **No verification is needed.** A self-reported figure reaches only the person who reported it, which is why nothing a user enters is verified.

**Files:** `apps/api/src/become-pro/me-become-pro.controller.ts`, `me-become-pro.service.ts`, `become-pro.module.ts`

## The Become Pro page

The user's own page at `/become-pro`. It holds their seasons, logged games, season line, projected value and the comparison with NBA rookies. Only the signed-in owner can see it.

- **One request.** `GET /v1/me/become-pro?seasonId=…` returns everything the page shows: the seasons, the active season, its derived line, its game log, the valuation and its history.
- **No season yet.** The page opens straight onto the "Start your first season" form.
- **Layout.** On phones the value card sits directly under the header. From the `lg` breakpoint up it moves to the top of the right-hand rail.
- **Built on what exists.** React 19, TanStack Query, React Router 7 and Tailwind v4, reusing `StatTile`, `PointsTrendChart`, `PlayerTraitsRadar`, `ComparisonTraitsRadar`, `LockerSection`, `Reveal`, `ErrorState` and `PageLoading`.

**Why:**

- **One page, one request.** The page is a single view of one person's season. Splitting it across requests would only add loading states.
- **Reused components.** The season line has exactly the same shape as `SeasonAverages`, so the existing NBA charts work on it with no new chart code, and the page matches the rest of the app's "locker" design language.

**Files:** `apps/web/src/pages/BecomeProPage.tsx` and its spec. The route is in `apps/web/src/App.tsx`, wrapped in `ProtectedRoute` and `ProfileGate`.

## Seasons

A season is one league year at one competition level, for example "2025-26 · NCAA Division II · G · Riverside College". A user can have several.

- **Picking the year.** The league-year picker lists the 8 most recent league years, starting at the new year from July onwards. Years the user already has are left out.
- **Fields.** Position (`G`, `F`, `C`, `G-F`, `F-C`) and competition level are required; team name is optional.
- **Header actions.** "Edit details", "Add a season" and "Delete season". Delete needs a second click, because it removes every game in the season. With more than one season, a radio-group season picker appears.
- **Limits.** 12 seasons per user, and one season per league year (`SEASON_ALREADY_EXISTS`).
- **Validation.** zod on the API. The competition-level enum is built from the Prisma enum.

**Why:**

- **A picker, not free text.** It can't produce "2025/26" or "25-26", and it matches `Game.season`'s "2025-26" format, which lets a user's season sit beside an NBA one.
- **Enum built from Prisma.** There is no second hand-written list to drift out of sync with the database.
- **Level chosen with the season.** Competition level is the biggest single input to the value, so it is required and explained on the form.

**Files:** `apps/web/src/components/becomepro/SeasonSetupForm.tsx`, `apps/web/src/lib/prospectValue.ts` (`recentLeagueYears`, labels, positions), `apps/api/src/become-pro/me-become-pro.controller.ts`, `me-become-pro.service.ts`, `competition-level.ts`

## Logging games

The user enters one box score per game: date, opponent, MIN, PTS, REB, AST, STL, BLK, TOV, FGM, FGA, 3PM, 3PA, FTM and FTA.

- **Fast entry.** Enter saves from any field. Focus returns to the first field with the date carried forward, and "Copy last game" pre-fills the previous row. The 13 number fields use a numeric keypad on phones and sit on one row on desktop.
- **The games table.** Newest first. Each row has **Edit**, which opens the game in place of the add form, and **Remove**, which needs a second click.
- **Limits.** 120 games per season. A second game with the same date and opponent is refused (`DUPLICATE_GAME`).
- **After every save, edit or removal**, `invalidateBecomeProQueries` refreshes both the page and the Home/Profile summary.

**Why:**

- **Per game, not a typed season line.** Every figure the app publishes traces back to per-game records, and Become Pro keeps that rule.
- **Built for speed.** Someone back-filling a season enters around 20 rows in one sitting. A generic form would be correct but too slow to finish.
- **No derived preview in the browser.** The server re-derives and re-values the season after every write, and a second derivation in the browser could only disagree with it.

**Files:** `apps/web/src/components/becomepro/GameEntryRow.tsx`, `SeasonEntryPanel.tsx`, `styles.ts`, `apps/web/src/lib/becomeProApi.ts`

## Box-score validation

Each game is checked for arithmetic that can't be true.

| Blocks the save ("Fix") | Warns but still saves ("Check") |
|---|---|
| A negative stat | Points that disagree with the shooting splits (`POINTS_MISMATCH`): points should equal 2×FGM + 3PM + FTM |
| More makes than attempts (FG, 3P, FT) | |
| More threes made than field goals made | |
| More threes attempted than field goals attempted | |
| Minutes over 65 | |
| A date in the future | |

The server calls `findStatAnomalies`, the same checker the admin correction tools run over ingested NBA data, and adds severity on top. The browser mirrors the same rules and codes, labels each issue "Fix" or "Check", and shows them only after a save is attempted.

**Why:**

- **One shared checker.** "What counts as an impossible line" has one definition in the codebase.
- **Mismatched points only warn.** Real scoresheets sometimes carry totals that don't add up. Refusing the user's own sheet is worse than saving it with the mismatch flagged.
- **The browser mirrors the server.** A row the browser lets you save is always one the API accepts.

**Files:** `apps/api/src/become-pro/prospect-box-score.ts`, `apps/api/src/admin/stat-anomalies.ts`, `apps/web/src/lib/boxScoreValidation.ts`

## The season line

- **Stat grid:** Games, PPG, RPG, APG, SPG, BPG, MPG, TOV, FG%, 3P%, FT% and TS%.
- **"Scoring by game" chart:** points in each game across the season.
- **Traits radar:** the season's profile across the five traits.

How it's derived:

- **On the server, from the logged games,** by `deriveSeasonAverages`, which was extracted from the NBA player stats service so both use identical code.
- **Percentages come from season totals.** 6-for-21 is 28.6%, not the average of each game's percentage.
- **No attempts shows "—".** A percentage with no attempts behind it shows "—" instead of 0%.
- **Advanced figures are always null.** Plus-minus, usage and the two ratings need the possession context of a tracked game, which an amateur box score doesn't carry.
- **Short-sample warning** under 4 games.

**Why:**

- **One derivation.** A user's line is computed exactly the way an NBA player's is, so the comparison is like for like.
- **The existing shape.** Keeping the `SeasonAverages` type unchanged avoided touching every NBA page that depends on it.

**Files:** `apps/api/src/players/season-averages.ts`, `apps/api/src/players/stats.service.ts` (now delegates to it), `apps/api/src/become-pro/prospect-games.ts`; `SeasonStatGrid` and `toTrendData` in `apps/web/src/pages/BecomeProPage.tsx`

## The projected value card

The headline card.

- **It leads with the pick,** for example "Pick 14", then the dollar figure, then the [range](valuation-model.md#the-value-range).
- **Value-over-time sparkline** (see [Value history](#value-history)).
- **A provenance sentence:** games logged, level, level factor, scale year, and "not an offer and not a market price".
- **The level's basis:** the one sentence explaining where its factor comes from.
- **Drivers:** server-written sentences about scoring, efficiency and playmaking. Playmaking appears only above 4 assists a game.
- **Below the 10-game floor there is no dollar figure at all.** The card reads "N more games needed".

**Why:**

- **Drivers are written on the server.** An explanation written in the browser about a server-side model would be invention.
- **Never "$0".** An absent value is never shown as zero, because "not valued yet" is not "valued at nothing".
- **No separate "Level" driver.** It repeated the basis sentence already on the card, and was removed during testing.

**Files:** `apps/web/src/components/becomepro/ProspectValueCard.tsx`; `formatProjectedValue`, `formatValueRange`, `formatDraftSlot` and `describeValuationState` in `apps/web/src/lib/prospectValue.ts`

## Comparison with NBA rookies

The three real NBA rookies whose rookie line is most like the user's. Each shows a percentage similarity and links to that player's page.

The comparison radar (`ComparisonTraitsRadar`) plots each rookie's rookie regular-season line (published games only) against the user's line. The user's line is drawn first, in the leather accent, and labelled "You (level-adjusted)" whenever the level factor isn't 1.

**Why:**

- **Candidates come from the model's own training set,** so every comparable is a real player whose real rookie season helped train the model.
- **The radar plots the level-adjusted line,** because that's the line the similarity was measured on. Drawing the raw line would contradict the percentage printed beside it.
- **Widened prop, no fake player.** `ComparisonTraitsRadar`'s prop was widened to `TraitsComparisonEntry`, so the user is never given a fabricated NBA `Player` record.

How similarity is measured is on [Valuation Model](valuation-model.md#comparable-rookies).

**Files:** `findComparables` in `apps/api/src/become-pro/valuation-model.ts`, `resolveComparables` in `become-pro.service.ts`, `ComparablesPanel` in `apps/web/src/pages/BecomeProPage.tsx`, `apps/web/src/components/ComparisonTraitsRadar.tsx`

## Players drafted at the projected pick

"Drafted at pick N": the three most recent real players taken at the projected pick, each linking to their page. It queries `Player` by `draftNumber`, newest `draftYear` first, and the result is stored with the valuation.

**Why:** it turns an abstract pick number into real players.

**Files:** `slotAlumni` in `apps/api/src/become-pro/prospect-valuation.service.ts`

## Value history

A sparkline of the projected value over time, on the value card and on the Home/Profile card. It reads the season's valuation rows in date order. A run of identical values collapses into one point, and a single valuation draws no line.

**Why:** a valuation row is written whenever *any* part of it changes, and the explanation changes with nearly every game. Without collapsing runs, a flat value showed as a string of repeated points.

**Files:** `valueHistory` in `apps/api/src/become-pro/become-pro.service.ts`

## Home and Profile card

A small "Become Pro" card on `/home` (right-hand rail) and `/profile`. It shows the season, level, value, pick and sparkline, links to the page, and invites the user to "Start a season" if they haven't.

It reads `GET /v1/me/become-pro/summary`, a lighter endpoint than the full page.

**Why:** these are two of the most-visited pages, so they shouldn't pull the NBA comparables just to show four lines.

**Files:** `apps/web/src/components/becomepro/MyProspectCard.tsx`, `apps/web/src/pages/HomePage.tsx`, `apps/web/src/pages/ProfilePage.tsx`

## Navigation link

"Become Pro" is the eighth link in the app header, after Home, Players, Compare, Teams, Datasets, Optimizer and Predictions. It was appended last, so no existing link moved.

To make room, the in-row nav breakpoint moved from `lg` to `xl`, and the landing banner from `xl` to `2xl`. **Why:** eight links were measured overflowing the header at 1024px wide.

**Files:** `apps/web/src/components/landing/LandingHeader.tsx`

## Built, then removed

The first design (22–25 September) had:

- a public value leaderboard, with a "#N" rank badge next to the user's name
- public prospect profiles and a directory
- evidence uploads (scorecards) with a reliability score
- an admin evidence-review queue

**All of it was removed on 26 September 2026**, when Become Pro was re-scoped to private, you-versus-NBA only. With no comparison between users there is nothing to verify, and the Supabase storage bucket for evidence is no longer needed. Any documentation that mentions these features is out of date.

## Testing

| Suite | Result at hand-off |
|---|---|
| API | 974 tests passing, including 31 Become Pro end-to-end tests against a real Postgres (`apps/api/test/become-pro.e2e-spec.ts`) |
| Web | 717 tests passing, with 90.2% line and 84.2% branch coverage against the 80% threshold |
| Python (`apps/valuation`) | 24 tests, none needing a database |
| Live browser test (26–27 Sep) | 65 of 65 checks passed, with no browser console errors |

The live browser test drove the real web app, API, a Postgres database filled by the `nba_api` pull, and a trained model through headless Edge, with two real signed-in users. It covered starting a season, the 10-game floor, the value appearing on the 10th game, editing and removing games, validation, duplicates, changing level, switching and deleting seasons, comparables linking to player pages, Home and Profile agreeing, phone width, and privacy between the two users.

Spec-by-spec detail is on [Testing](../testing.md).

## Where the code lives

| Area | Files |
|---|---|
| Page and route | `apps/web/src/pages/BecomeProPage.tsx`, `apps/web/src/App.tsx` |
| Web components | `apps/web/src/components/becomepro/` (`SeasonSetupForm`, `SeasonEntryPanel`, `GameEntryRow`, `ProspectValueCard`, `MyProspectCard`, `styles.ts`) |
| Web helpers | `apps/web/src/lib/becomeProApi.ts` (query keys `["myBecomePro"]` and `["myBecomeProSummary"]`), `prospectValue.ts`, `boxScoreValidation.ts` |
| Web types | The Become Pro section of `apps/web/src/types/nba.ts` (`ProspectSeason`, `ProspectGame`, `ProspectValuation`, `MyBecomePro`, `MyBecomeProSummary`, `TraitsComparisonEntry` and others) |
| Test fixtures | `apps/web/src/test/becomeProFixtures.ts` |
| API | `apps/api/src/become-pro/`, plus `apps/api/src/players/season-averages.ts` |
| Schema | `apps/api/prisma/schema.prisma`, migration `20260923000000_add_become_pro` (see [ERD](../design/erd.md#become-pro-entities)) |
| Model training | `apps/valuation/` (see [Valuation Model](valuation-model.md)) |
| Routes | [API Reference — Become Pro](../api-reference.md#become-pro) |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
