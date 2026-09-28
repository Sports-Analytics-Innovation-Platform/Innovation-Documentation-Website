# Player Archetypes

Every NBA player with enough minutes is placed into **playing-style archetypes**, such as "Stretch big" or "Pass-first point guard", and matched with the **five players who play most like them**. Both are worked out from box-score data, and both appear on the player's profile page in the "Style & similar players" section.

!!! warning "In review, not yet merged"
    Built on branch `player-archetypes`, which was pushed for review on 2026-09-27. It adds the `apps/similarity` service, migration `20260922200000_add_player_archetypes` (four new tables), three API routes and the profile card. Until the model has been written to production, every profile shows "No season has been analysed for playing style yet." and nothing else changes. Every fact on these pages was checked against the branch on 2026-09-28. Update this box when the pull request is merged.

This page covers what users see, how the parts fit together, and how to run and deploy it. How the archetypes and similarity scores are calculated, and what they can't claim, is on [Archetype Model](model.md).

## What users see

The **"Style & similar players"** section of a player's profile has three parts.

- **Playing style.** Up to three archetypes, strongest first, each with a bar and a whole-number percentage. The player's listed position is shown above them ("Listed G."), followed by either "Sits between archetypes — closest first." or "Sits clearly inside one archetype."
- **Style map.** A scatter plot of every placed player in the season. Players close together play alike. The player being viewed is a large ringed point; players in the same archetype are drawn in the accent colour and everyone else in grey. The horizontal axis runs from perimeter shooting to size and interior play, and the vertical axis from an off-ball role to on-ball creation. It has no tick marks, because the values are not meaningful as numbers.
- **Similar players.** Five tiles, each with a headshot, name, team badge and a similarity score from 0 to 100, labelled "Closest in playing style, not in quality." Each tile links to that player's profile.

An info button beside the heading explains, in plain language, that players are grouped by how they play rather than how well; that listed position is not an input; that the percentages are closeness, not confidence; and that the model can't see defence beyond steals and blocks.

The section handles three situations, and all three are normal:

| Situation | What the section shows |
|---|---|
| The player was placed | The three parts above |
| The player was left out | "Not enough minutes in 2025-26 to describe this player's style…". This is the usual case for deep-bench players (see [Who gets placed](model.md#who-gets-placed)). A small number of players are left out because a value such as their weight is missing, and they get the same message. |
| No season has been analysed yet | "No season has been analysed for playing style yet." |

**Why:**

- **Bars, not precise percentages.** The groups overlap a great deal (see [How well the groups separate](model.md#how-well-the-groups-separate)), so a figure like "43.2%" would suggest a precision the model doesn't have. Bars are scaled against the player's strongest archetype, so a player spread thinly across three still produces a readable chart.
- **No colour per archetype.** Archetype names change when the model is re-fitted, and a colour per name would silently re-colour players after a rename. Nine colours would also be unreadable at this size.
- **The listed position is shown next to the archetype.** Position is not a model input, and about a third of the league lands in an archetype that doesn't match their listed position. Showing both makes that visible and intentional, rather than looking like a mistake.
- **The style map only loads for placed players.** An unplaced player has no point to highlight, so the whole league's coordinates would be downloaded for nothing. It is cached per season, so moving between profiles reuses it.
- **The section ignores the profile's season-segment selector.** An archetype describes a player's whole season, all segments together, so switching to the playoffs view would give the same answer.

**Files:** `apps/web/src/components/PlayerArchetypeCard.tsx`, `apps/web/src/components/PlayerStyleMap.tsx`, and the "Style & similar players" section of `apps/web/src/pages/PlayerProfilePage.tsx`

## How it fits together

| Part | Role | Runs |
|---|---|---|
| `apps/similarity` | Fits the model from one season's box scores and writes it to four tables | By hand, as a batch job |
| `apps/api/src/players` (`ArchetypesService`, `ArchetypesController`) | Reads the four tables and serves three routes | Always on (Render) |
| `apps/web` | The profile section and the style map | In the browser |

What `build_archetypes.py` does for one season:

1. Reads every `PlayerGameStat` row for the season, with each player's height and weight.
2. Turns each eligible player's season into 15 standardised rate statistics.
3. Groups players into nine archetypes with K-Means clustering, and works out how close each player is to every group.
4. Places every player on a 2D map (principal component analysis) and finds each player's five nearest neighbours.
5. Names the groups by matching them to the names committed in `archetype_labels.py`.
6. Replaces the season's rows in all four tables in a single transaction.

The API only ever reads these tables, and the Python service writes nothing else. This is the same arrangement the predictor and optimizer use: Python writes its output, the API reads it.

**Why a batch job?** The answer only changes when the model is re-fitted. Storing it makes every read a plain indexed lookup, and the browser never has to run a projection or a distance search.

## API routes

All three are public, behind the same guards as the other player routes (`OptionalSessionGuard` and `ApiKeyGuard`). When `season` is omitted, each route uses the most recently fitted season.

| Method | Path | Returns |
|---|---|---|
| `GET` | `/v1/players/:id/archetype?season=` | The player's archetypes (up to three), five similar players with their teams, the player's standardised feature values, their distance from their main archetype's centre, and their map position |
| `GET` | `/v1/archetypes?season=` | The season's archetypes and how many players each holds, largest first |
| `GET` | `/v1/archetypes/map?season=` | Every placed player's map position and main archetype, plus the archetype list for the legend |

"Not placed" is not an error:

- `GET /v1/players/:id/archetype` returns **404 only when the player doesn't exist.** A player who exists but wasn't placed gets `200` with `"archetype": null`. When no season has been fitted, it returns `"season": null`.
- `GET /v1/archetypes` returns an empty list when no season has been fitted.
- `GET /v1/archetypes/map` returns **404** when no season has been fitted, because there is no map to draw.

Full request and response details are in the [API Reference](../api-reference.md#archetypes).

**Files:** `apps/api/src/players/archetypes.controller.ts`, `archetypes.service.ts`, and the `:id/archetype` route in `players.controller.ts`

## Data

The model is stored in four tables. Each is written only by `apps/similarity` and read only by the API.

| Table | One row per |
|---|---|
| `Archetype` | Archetype per season: its name, its centre in real units, and its member count |
| `PlayerArchetype` | Player per season: their standardised feature values, map position and distance from their main archetype's centre |
| `PlayerArchetypeMembership` | One of a player's top archetypes (up to three), with its rank and weight |
| `PlayerSimilarity` | One of a player's five most similar players, with its rank and score |

Every column is documented on the [ERD](../design/erd.md#player-archetypes). Two design choices are worth knowing:

- **Names are stored once, on `Archetype`.** Players point to an archetype by id, so renaming an archetype updates one row and nothing else.
- **A player's main archetype isn't stored separately.** It is their rank-1 membership. Storing it twice would create a second copy that could disagree with the first.

The model is small. One season is about 330 placements, up to 3 memberships each and 5 similar players each. That comes to well under 2 MB with indexes (an estimate), compared with the database's [500 MB limit](../decisions/adr-005-play-by-play-storage.md).

## Running it

`apps/similarity` has its own `.env` containing a single variable, `DATABASE_URL`. It is deliberately separate from the ingestion pipeline's `.env`. Fitting the model needs a fully loaded season, which during development means reading production, while ingestion stays pointed at a local database. A shared file couldn't express both, and repointing it would silently change where ingestion writes. Every script prints which database it is about to use before it does anything.

```
cd apps/similarity
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt

python explore_fit.py --list-seasons              # seasons with box scores
python explore_fit.py --season 2025-26 --k 9      # fit and describe the groups; never writes
python build_archetypes.py --season 2025-26       # dry run: fits and prints what it would write
python build_archetypes.py --season 2025-26 --apply
```

- **Nothing is written without `--apply`.** A dry run opens the connection in read-only mode, so the database server itself rejects any write. Reading production to inspect a fit is therefore safe.
- **Re-running replaces a season.** `--apply` deletes that season's rows and writes the new ones in one transaction, so a half-written model can never be left behind.
- **It refuses to fit on too little data.** Nine archetypes need at least 90 eligible players (10 per archetype). A development database with a few mock players is refused with a message rather than producing a confident, meaningless model.
- **To test `--apply` at full size without touching production,** `copy_season_to_scratch.py` copies one season from production (read-only) into a scratch database on the local development Postgres. Its docstring has the four steps. Use port 55432; the test instance on 55433 keeps its data in memory and runs out.

## Deploying to production

The order matters:

1. **Apply the migration.** It only adds four tables, so the current code ignores them. Render applies pending migrations when the API starts (`npx prisma migrate deploy` in `render.yaml`), but applying it before merging means the new code never starts against a database without its tables.
2. **Merge the pull request.** Until step 4, profiles show "No season has been analysed for playing style yet."
3. **Re-name the archetypes.** The names committed in `archetype_labels.py` come from a fit made before 2025-26 was re-ingested on 23 September, and two of the nine no longer describe their group (see [Naming the archetypes](model.md#naming-the-archetypes)). Delete the file, run `python explore_fit.py --k 9 --emit-labels archetype_labels.py` against production, read each group's description and closest players, type the names, and commit the file.
4. **Write the model:** `python build_archetypes.py --season 2025-26 --apply`, with `DATABASE_URL` pointing at production.

Only 2025-26 can be fitted. The older seasons lack the offensive/defensive rebound split and usage the model needs, because they were loaded without play-by-play; see [ADR-005](../decisions/adr-005-play-by-play-storage.md).

## Testing

Counted from the test files on the branch:

| Suite | Tests |
|---|---|
| Python (`apps/similarity`): features, clustering, similarity, naming and the scratch copy | 73 test functions, none needing a database |
| API: `apps/api/src/players/archetypes.service.spec.ts` | 15 |
| Web: `apps/web/src/components/PlayerArchetypeCard.spec.tsx` | 18 |

Beyond the automated tests, the full write was exercised against a scratch copy of production's 2025-26 season (made with `copy_season_to_scratch.py`), not against production itself.

## Where the code lives

| Area | Files |
|---|---|
| Model (Python) | `apps/similarity/`: `build_archetypes.py` (the only writer), `explore_fit.py`, `features.py`, `player_seasons.py`, `clustering.py`, `similarity.py`, `labeling.py`, `archetype_labels.py`, `copy_season_to_scratch.py`, `db.py` |
| API | `apps/api/src/players/archetypes.controller.ts`, `archetypes.service.ts`, `players.controller.ts` (`GET /v1/players/:id/archetype`), `players.module.ts` |
| Web | `apps/web/src/components/PlayerArchetypeCard.tsx`, `PlayerStyleMap.tsx`, `apps/web/src/pages/PlayerProfilePage.tsx`, `apps/web/src/lib/nbaApi.ts` (`fetchPlayerArchetype`, `fetchStyleMap`), `apps/web/src/types/nba.ts` |
| Schema | `apps/api/prisma/schema.prisma`, migration `20260922200000_add_player_archetypes` (see [ERD](../design/erd.md#player-archetypes)) |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
