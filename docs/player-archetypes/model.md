# Archetype Model

How `apps/similarity` turns one season of box scores into archetypes, membership weights, a style map and similar players; what each number means; and what the model can't claim. What users see, the API routes and how to run it are on [Player Archetypes](index.md). Like that page, this describes branch `player-archetypes`, which is in review and not yet merged.

## The unit: one player, one season

The model describes **one player in one season**, and each season gets its own fit.

- **Seasons are never mixed.** Averaging a player across seasons would describe someone who never existed: a bench guard who became a starter two years later would land halfway between the two roles.
- **A season's segments are combined.** The regular season, play-in, playoffs and Finals all count together, because an archetype describes how someone played that year. A player whose team went deep into the playoffs just has more games behind their figures. Fitting the segments separately was considered and rejected: the 2025-26 postseason has 61 eligible players against the regular season's 341, which is far too few to place nine groups, and separate fits would produce groups that mean different things under the same names.
- **This differs from the rest of the site on purpose.** The per-segment statistics pages never mix segments, because they answer a different question ("what were this player's playoff numbers?"). The predictor and optimizer use the regular season only, because they project forward and weight recent games. This model projects nothing; it describes a completed season.
- **A season without box scores is ignored.** Every read starts from `PlayerGameStat`, so a scheduled but unplayed season contributes nothing rather than several hundred players with zero minutes.

Only 2025-26 can currently be fitted. The older seasons are missing two of the features, because they were loaded without play-by-play ([ADR-005](../decisions/adr-005-play-by-play-storage.md)).

**Files:** `apps/similarity/player_seasons.py`

## Who gets placed

A player is placed only if:

1. they played **at least 8 games**, and
2. averaged **at least 12 minutes a game**, and
3. **every feature below can be calculated** for them.

Below the minutes floor, rate statistics describe the sample rather than the player. A nine-minute cameo with two three-point attempts reads as a 100% three-point attempt rate. A player who clears the floor but is missing a value, such as a recorded weight, is left out as incomplete data rather than given an invented one: a missing height is a gap to fix in ingestion, not an average-height player. Both kinds of player get the "not enough minutes" message on their profile.

In 2025-26 about 330 players are placed.

**Files:** `apps/similarity/features.py` (`is_eligible_for_archetype`, `build_feature_row`)

## The 15 features

Every feature is a **rate, not a total**, so two players with the same style but different minutes land in the same place. Totals would group players by playing time, which measures job security, not style.

| Feature | What it captures |
|---|---|
| Points per 36 minutes | Scoring volume |
| Offensive rebounds per 36 | Crashing the offensive glass |
| Defensive rebounds per 36 | Defensive rebounding |
| Assists per 36 | Playmaking |
| Steals per 36 | Ball-hawking |
| Blocks per 36 | Shot-blocking |
| Turnovers per 36 | How much a player handles the ball, and the risk that comes with it |
| Three-point attempts per 36 | Three-point volume |
| Three-point attempt rate | The share of field-goal attempts taken from three. This separates a stretch big from a traditional big better than three-point *percentage* does, because it measures what a player is willing to shoot rather than how often it happened to go in. |
| Free-throw rate | Free-throw attempts per field-goal attempt, a stand-in for attacking the rim and drawing contact |
| True shooting % | Scoring efficiency, counting each trip to the line as 0.44 of a shooting possession (the same coefficient the API uses, so the profile page and the model agree) |
| Assist-to-turnover ratio | How safely a player handles their playmaking load |
| Usage % | The NBA's own figure for the share of team plays a player uses, weighted by minutes across games |
| Height | Inches |
| Weight | Pounds |

Two details:

- **The rebound split is divided by the minutes of the games that recorded it**, not by all minutes. Otherwise every game without the split would count as a game with no rebounds.
- **Every column is then standardised** (z-scored) across the placed players, so that each has an average of 0 and a spread of 1. Without this, distance would be dominated by whichever column has the biggest units. Height (around 78 inches) would outweigh three-point attempt rate (around 0.4), and the model would effectively sort players by height. Standardising happens after the eligibility filter, so deep-bench players don't drag the averages down.

**Files:** `apps/similarity/features.py` (`FEATURE_NAMES` fixes the column order)

## Grouping players into archetypes

Players are grouped with **K-Means clustering** into **nine groups** over the 15 standardised features. K-Means repeatedly assigns each player to the nearest group centre and moves each centre to the middle of its players. It uses a fixed random seed (42) and 10 restarts, keeping the best, so the same data always gives the same groups.

- **Why nine?** The silhouette score (see below) is nearly flat from 4 to 9 groups, so it doesn't pick a number. Nine was chosen because it produces groups a basketball fan would recognise as real roles. `--k` overrides it.
- **Too little data is refused.** A fit needs at least 10 eligible players per group (90 for nine groups). K-Means would otherwise happily split a dozen players into nine groups without any error.

**Files:** `apps/similarity/clustering.py` (`fit_cluster_assignments`), `apps/similarity/build_archetypes.py`

## How well the groups separate

The **silhouette score** measures how cleanly groups separate, from 0 (no separation) to 1 (completely separate). These groups score **about 0.14**. They are boundaries drawn through a continuum of playing styles, not separate populations, and most players sit near an edge.

That shapes the rest of the design:

- each player gets **up to three archetypes**, not a single label;
- the percentages show **closeness, not confidence**;
- **similar players are found without using the groups**, so they stay useful where the boundaries are debatable.

## Membership weights

For each player, the model measures the distance to each of the nine group centres and turns those distances into weights that add up to 1:

```
weight for a group ∝ exp(−(distance to that group − distance to the nearest group) ÷ 1.0)
```

The nearest centre always gets the largest weight, so a player's main archetype and their bars can never disagree.

Up to three weights are stored per player, strongest first. After the first, any weight at or below 0.10 is dropped as rounding noise. The strongest is always kept, even when it is small: a player whose best weight is low is genuinely between styles, not a player without one.

The 1.0 in the formula (`MEMBERSHIP_TEMPERATURE`) controls how quickly weight falls away with distance, and it was tuned on real data. At 1.0, the median player's strongest archetype carries about 0.43 and about three archetypes are shown. At 0.75 it would be about 0.54, with two or three shown. Both are defensible; 1.0 shows more of the blend, which a silhouette of 0.14 says is the more honest picture.

**These are not probabilities.** They say how close a player is to each group relative to the others, so the site shows them as bars and whole percentages.

### Why not a Gaussian mixture model

A Gaussian mixture, which gives each player a probability of belonging to each group, was the obvious choice and was tried first. It failed in two measured ways:

1. **Over 15 features, it collapsed to hard labels.** The median player's top weight was 0.999, and only 11% of players had a second archetype above 10%.
2. **Fitting it on 3–5 principal components fixed that but broke something worse.** Its strongest group then disagreed with the player's K-Means group for 18–24% of players, so the badge and the bars would have contradicted each other for about one player in five.

Two leftovers from that first design still mention a mixture: the model version written on every row (`kmeans-gmm-k9`) and the schema comment on `PlayerArchetypeMembership.weight`. Both are labels only. The weights come from `calculate_membership_weights` in `clustering.py`.

**Files:** `apps/similarity/clustering.py` (`calculate_membership_weights`, `select_top_memberships`)

## The style map

Each player's map position is the **first two principal components** of the same standardised features. Principal component analysis finds the two directions along which players differ most. Together, those two directions carry **about 56%** of the variation between players.

| Axis | What drives it | Label on the map |
|---|---|---|
| Horizontal | Offensive and defensive rebounds, height, weight and blocks, with three-point volume pulling the other way | Perimeter shooting → size & interior play |
| Vertical | Usage, points, turnovers and assists | Off-ball role → on-ball creation |

- **Calculated once in Python and stored** (`plotX`, `plotY`), never in the browser, so the map can't drift from the groups.
- **The direction of each axis is fixed.** Principal component analysis can flip an axis between fits. The model makes each component's largest loading positive, so a re-fit doesn't mirror the whole map and make it look reshuffled.
- **No tick marks.** The units are component scores, which mean nothing on their own. The map is for reading position and neighbourhood, not values.

**Files:** `apps/similarity/clustering.py` (`project_to_plot_coordinates`, `canonicalize_component_signs`)

## Similar players

A player's similar players are the **five placed players closest to them** in the same standardised space, measured as a straight-line (Euclidean) distance. A player is never their own neighbour.

This doesn't use the groups at all. It answers "who plays like this player?" directly, without first having to agree how many groups the league divides into.

The distance is turned into a **score from 0 to 100**:

```
score = 100 × exp(−distance ÷ average distance between any two placed players)
```

- 100 means identical, and the score never reaches 0.
- Scaling by the season's average distance keeps scores comparable between seasons and stops a single extreme player from setting the scale.

**Style, never quality.** Two players can have a very high score and be far apart in ability, because the features describe shot selection, playmaking load, size and rebounding, not how often shots go in. Nothing should rank players by this score. The scores also bunch into a narrow band, and their order says more than the numbers, so the tiles show them quietly.

**Files:** `apps/similarity/similarity.py`

## Naming the archetypes

Clustering produces unnamed groups; **a person names them**, in `apps/similarity/archetype_labels.py`.

**The problem.** K-Means numbers its groups arbitrarily and renumbers them whenever the fit changes. On this project's own data, the rim-running bigs were group 0 with 6 groups, group 6 with 8, and group 0 again with 9. A names file keyed on group numbers would silently relabel half the league the first time anyone re-ran the model.

**The fix.** Each name is stored with the **centre it was given to, in real units** (points per 36, inches and so on). On a re-fit:

1. the stored centres are converted into the new fit's standardised space;
2. each new group is matched to a stored name one-to-one, minimising the total distance across all pairs at once (the Hungarian algorithm), so no name is used twice;
3. a new group further than 8.0 (in standardised units) from every stored centre is labelled **"Unnamed archetype"**, visibly, so that a person names it.

Real units are used because standardised values shift whenever the pool of players changes, while points per 36 and inches don't. Names are stored on the `Archetype` row, so renaming one is a single-row update.

### The committed names

From the reference fit (2025-26, whole season, nine groups), made before 2025-26 was re-ingested on 23 September:

| Name | Players | Closest to the centre |
|---|---|---|
| Scoring wing | 66 | Desmond Bane, Kyle Kuzma, RJ Barrett, Cedric Coward |
| Catch-and-shoot wing | 50 | Devin Vassell, Harrison Barnes, Vít Krejčí, Svi Mykhailiuk |
| Low-usage forward | 42 | Jamir Watkins, Zaccharie Risacher, Ryan Dunn, Jake LaRavia |
| Pass-first point guard | 41 | De'Aaron Fox, Pat Spencer, Javon Small, Walter Clayton Jr. |
| Traditional big | 36 | Neemias Queta, Marvin Bagley III, Mark Williams, Jakob Poeltl |
| Stretch big | 34 | Onyeka Okongwu, Santi Aldama, Sandro Mamukelashvili, Jabari Smith Jr. |
| Scoring guard | 30 | Keyonte George, Donovan Mitchell, James Harden, Devin Booker |
| Point forward | 20 | Alperen Sengun, Paolo Banchero, Julius Randle, LeBron James |
| Defensive guard | 12 | Kris Dunn, Cason Wallace, Bez Mbeng, Alex Caruso |

!!! warning "Two names are stale: re-name before the first write to production"
    Re-fitting on the re-ingested 2025-26 data still attaches all nine names, but two no longer describe their group:

    - **"Defensive guard"** described a tight 12-player group stealing 2.48 times per 36 minutes. That group no longer exists (no group now exceeds 1.90), and the name has attached itself to a 36-player combo-guard group (Caldwell-Pope, Edgecombe, Monk).
    - **"Low-usage forward"** now sits on the group with the highest steal rate (1.90), which is closer to what "Defensive guard" was meant to describe.

    Matching guarantees a *stable* assignment, not that a name still fits: when a group disappears, the nearest surviving group inherits its name. A re-fit therefore needs a person to re-read the groups. The steps are under [Deploying to production](index.md#deploying-to-production).

Judgement calls recorded in `archetype_labels.py`:

- **"Catch-and-shoot wing", not "3-and-D wing".** The group's three-point attempt rate (0.67) is measured, but the defence isn't, so the name doesn't claim it.
- **"Defensive guard" does claim a defensive skill,** but on the strength of a measurement: steals, which means ball-hawking specifically, not defence in general.
- **"Low-usage forward" is an invented name.** No common basketball term covers this group.
- **"Combo guard"** was used in an earlier fit and retired, because it sat between the pass-first and scoring guards without a region of its own.
- **Vocabulary.** Seven of the nine names match role names in the [Athlore archetype taxonomy](https://athlore.app/discover/archetypes), used only as a source of names. It has no statistics behind it, so it did not define the groups.

**Files:** `apps/similarity/labeling.py`, `apps/similarity/archetype_labels.py`, `apps/similarity/explore_fit.py` (`--emit-labels` writes a starter file with the centres filled in and the names left blank)

## What the model can't see

- **Defence beyond steals and blocks.** Good perimeter defence produces no box-score event when it works, because the shot never happens. A "3-and-D wing" therefore can't be told apart from a low-usage off-ball shooter, and the names describe a player's offensive role plus their rebounding and shot-blocking.
- **Where shots come from.** Shot-location data isn't loaded, so a midrange scorer and a rim attacker of equal efficiency look the same.
- **Share-of-team rates.** Assist percentage and rebound percentage aren't available, so playmaking and rebounding are measured per 36 minutes rather than as a share of the team's chances while the player was on court.
- **Listed position.** Position isn't an input. About a third of players land in an archetype that doesn't match their listed position, which the profile shows openly.
- **Older seasons.** 2025-26 only ([ADR-005](../decisions/adr-005-play-by-play-storage.md)).
- **Quality.** The model describes style only.

## Tuned constants

| Constant | Value | File | What it controls |
|---|---|---|---|
| `MINIMUM_GAMES_FOR_ARCHETYPE` | 8 | `features.py` | Games needed to be placed |
| `MINIMUM_MINUTES_PER_GAME_FOR_ARCHETYPE` | 12 | `features.py` | Minutes per game needed to be placed |
| `ARCHETYPE_COUNT` | 9 | `build_archetypes.py` | Number of groups (overridable with `--k`) |
| `MINIMUM_PLAYERS_PER_ARCHETYPE` | 10 | `build_archetypes.py` | Eligible players needed per group before a fit runs |
| `MAXIMUM_MEMBERSHIPS_PER_PLAYER` | 3 | `build_archetypes.py` | Archetypes stored per player |
| `MINIMUM_MEMBERSHIP_WEIGHT` | 0.10 | `build_archetypes.py` | Weight below which a secondary archetype is dropped |
| `SIMILAR_PLAYERS_PER_PLAYER` | 5 | `build_archetypes.py` | Similar players stored per player |
| `MEMBERSHIP_TEMPERATURE` | 1.0 | `clustering.py` | How quickly membership weight falls with distance |
| `RANDOM_SEED`, `KMEANS_RESTARTS` | 42, 10 | `clustering.py` | Repeatable K-Means fits |
| `MAXIMUM_MATCH_DISTANCE` | 8.0 | `labeling.py` | How far a new group can be from a stored name before it is left unnamed |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
