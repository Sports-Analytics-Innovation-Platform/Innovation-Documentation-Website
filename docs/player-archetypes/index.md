# Player Archetypes

Every NBA player with enough minutes is placed into **playing-style archetypes**, such as "Stretch big" or "Pass-first point guard", and matched with the **five players who play most like them**. Both are worked out from box-score data and shown in a "Style & similar players" section on the player's profile.

!!! warning "In review, not merged"
    Built on branch `player-archetypes` (last updated 28 Sep). It adds the `apps/similarity` service, four tables, three API routes and the profile section. None of it is in production yet.

## What users see

- **Playing style.** Up to three archetypes, strongest first, each with a bar and a whole-number percentage. The player's listed position is shown beside them, because position isn't a model input and about a third of players land in an archetype that doesn't match it.
- **Style map.** A scatter plot of every placed player in the season, with the viewed player ringed and their archetype highlighted. Players close together play alike. Across: perimeter shooting → size and interior play. Up: off-ball role → on-ball creation.
- **Similar players.** Five tiles with a similarity score from 0 to 100, labelled "Closest in playing style, not in quality."

An info button explains in plain words that players are grouped by how they play, not how well; that the percentages are closeness, not confidence; and that the model can't see defence beyond steals and blocks. A player with too few minutes gets "Not enough minutes in 2025-26 to describe this player's style…".

Archetypes have no colours of their own, because names can change on a re-fit and nine colours would be unreadable at that size.

## How it works

`apps/similarity/build_archetypes.py` fits one season and writes four tables; the API only reads them. This is the same split the predictor and optimizer use.

1. **Who is placed:** players with at least 8 games, averaging at least 12 minutes, with every feature available. About 330 players in 2025-26.
2. **Features:** 15 rates, so playing time doesn't drive the result: points, offensive and defensive rebounds, assists, steals, blocks, turnovers and three-point attempts per 36 minutes; three-point attempt rate; free-throw rate; true shooting %; assist-to-turnover ratio; usage %; height; weight. Each is standardised so height in inches doesn't outweigh a rate like 0.4.
3. **Groups:** K-Means clustering into nine archetypes, with a fixed seed so the same data gives the same groups.
4. **Membership:** each player's distance to every group centre becomes up to three weights. The nearest group always has the largest weight, so the main archetype and the bars agree.
5. **Map:** the first two principal components, about 56% of the variation, stored so the browser draws nothing it has to compute.
6. **Similar players:** the five nearest players in the same 15-feature space, ignoring the groups entirely.

A season's regular season and playoffs are combined, because the playoffs alone have 61 eligible players, too few for nine groups. Only 2025-26 can be fitted: older seasons were loaded without play-by-play, so they lack the rebound split and usage ([ADR-005](../decisions/adr-005-play-by-play-storage.md)).

### Who gets placed

Below 12 minutes a game, rates describe the sample, not the player: a nine-minute cameo with two threes reads as a 100% three-point rate. A player missing a value such as weight is left out rather than given an invented one.

### How well the groups separate

The silhouette score, from 0 (no separation) to 1 (fully separate), is **about 0.14**. Playing styles are a continuum and most players sit near a boundary. So each player gets **up to three archetypes, not one label**; the percentages show **closeness, not confidence**; and similar players are found without the groups.

A Gaussian mixture model was tried first. Over 15 features it put the median player 99.9% in one archetype, and on fewer features its top group disagreed with the K-Means group for about one player in five.

## Naming the archetypes

Clustering produces unnamed groups; a person names them in `archetype_labels.py`. K-Means renumbers its groups on every fit, so each name is stored with the group centre it was given to, in real units. On a re-fit, new groups are matched to stored names one to one, and a group far from every stored name is shown as "Unnamed archetype" for a person to name.

The current names, from the 2025-26 fit of 28 Sep:

| Archetype | Players | Closest to the centre |
|---|---|---|
| Catch-and-shoot wing | 53 | Harrison Barnes, Devin Vassell, Svi Mykhailiuk, Landry Shamet |
| Scoring wing | 53 | Kyle Kuzma, Desmond Bane, RJ Barrett, Cedric Coward |
| Low-usage forward | 42 | Jamir Watkins, Jake LaRavia, Ryan Dunn, Bruce Brown |
| Combo guard | 36 | Kentavious Caldwell-Pope, VJ Edgecombe, Walter Clayton Jr., Malik Monk |
| Stretch big | 34 | Onyeka Okongwu, Sandro Mamukelashvili, Santi Aldama, Jabari Smith Jr. |
| Pass-first point guard | 32 | Ajay Mitchell, De'Aaron Fox, Dylan Harper, Javon Small |
| Scoring guard | 31 | Keyonte George, James Harden, Donovan Mitchell, Devin Booker |
| Traditional big | 31 | Neemias Queta, Mark Williams, Marvin Bagley III, Daniel Gafford |
| Point forward | 19 | Alperen Sengun, Karl-Anthony Towns, Derik Queen, Julius Randle |

"Catch-and-shoot wing" is the group most people would call 3-and-D, but only the shooting is measured, so the name claims only that. An earlier "Defensive guard" group no longer exists after the 23 Sep re-ingestion, and "Combo guard" took its place.

## What the model can't see

- **Defence beyond steals and blocks.** Good perimeter defence leaves no box-score event.
- **Where shots come from.** Shot locations aren't loaded.
- **Quality.** Two players can be very similar in style and far apart in ability. Nothing should rank players by the similarity score.

## API routes

All public, behind the same guards as other player routes. Without `season`, each uses the latest fitted season.

| Route | Returns |
|---|---|
| `GET /v1/players/:id/archetype` | Up to three archetypes, five similar players and the map position. `200` with `"archetype": null` if the player wasn't placed. |
| `GET /v1/archetypes` | The season's archetypes and their sizes |
| `GET /v1/archetypes/map` | Every placed player's map position; `404` if no season has been fitted |

Details are in the [API Reference](../api-reference.md#archetypes); the four tables are in the [ERD](../design/erd.md#player-archetypes).

## Deploying to production

1. **Apply the migration.** It only adds four tables, so the current code ignores them.
2. **Merge the pull request.** Profiles show "No season has been analysed for playing style yet." until step 3.
3. **Write the model:** `python build_archetypes.py --season 2025-26 --apply` with `apps/similarity/.env` pointing at production.

Without `--apply` the script is a dry run on a read-only connection. With it, the season's rows are replaced in one transaction. It refuses to fit fewer than 10 eligible players per group.

## Testing

| Suite | Tests |
|---|---|
| Python (`apps/similarity`) | 73, none needing a database |
| API (`archetypes.service.spec.ts`) | 15 |
| Web (`PlayerArchetypeCard.spec.tsx`) | 18 |

The full write was also run against a scratch copy of production's 2025-26 season.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
