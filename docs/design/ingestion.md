# Data Ingestion

How NBA data gets from stats.nba.com into the database: what a **pull** does, how each game's run is recorded as a **batch** and reviewed before it is published, and why pulls requested on the live site are run by a **pull worker** on a team member's computer. The tables are on the [ERD](erd.md#ingestion-and-review).

## How a pull reaches the database

```
  Admin clicks "Pull Data"          The pull schedule is due
  on the live site                  (checked every hour)
            │                                │
            └───────────────┬────────────────┘
                            ▼
    API on Render: can't reach stats.nba.com, so it saves a
    queued pull (IngestionRequest, QUEUED)
                            │
                            ▼
    Pull worker on a team member's computer
    checks every minute, claims the pull (RUNNING)
    and runs:  ingest.py --review [--season] [--from-date] [--to-date]
                            │  calls stats.nba.com
                            ▼
    For each game: box score + play-by-play → plays, statistics
    and one batch, left as PENDING_REVIEW
                            │
                            ▼
    Admin → Batches: an admin approves each batch,
    and only then does the game appear on the public site
```

A developer can also run `ingest.py` by hand, and a locally running API starts `ingest.py` itself. Batches work the same either way.

## What a pull does

`apps/ingestion/ingest.py` waits a second between calls, because stats.nba.com publishes no rate limit and blocks clients that call too fast.

1. **Teams, rosters and player biographies.** A pull for a *past* season leaves rosters alone, so traded or retired players aren't moved back to old teams.
2. **Which games.** By default, each team's 15 most recent games plus the whole postseason; with `--from-date` and `--to-date`, only that window.
3. **Each game:** box score and play-by-play. Plays are translated into the platform's event format (`feed_translation.py`), validated (`event_validation.py`), stored as `GameEvent` rows, and summed into each player's box-score line (`derive_player_game_stats.py`).
4. **Plus-minus, usage and ratings,** from a few league-wide calls.

| Option | Default | Effect |
|---|---|---|
| `--review` | off | Batches finish as `PENDING_REVIEW`, so an admin must approve them. Pulls from the site always use it. |
| `--season` | `2025-26` | Which season to pull |
| `--from-date`, `--to-date` | none | Only games in that window (`YYYY-MM-DD`) |

A full pull takes 35–45 minutes. Every write is an upsert on the NBA's own IDs, so re-running a pull refreshes data instead of duplicating it.

!!! warning "Don't pull an older season into production"
    Play-by-play is stored for 2025-26 only, because of the 500 MB database limit. Pulling 2023-24 or 2024-25 would add at least 120 MB. Older postseasons have their own route: see [ADR-005](../decisions/adr-005-play-by-play-storage.md#loading-older-postseasons).

## Batches

### What batches provide

A batch is one game's ingestion run, stored as an `IngestionBatch`. It is what makes each published statistic traceable, and it meets four requirements in the brief ([Feature Tiers](feature-tiers.md)):

| The brief asks for | How batches provide it |
|---|---|
| Every statistic traceable to its events and submission | Every stored play points to the batch that wrote it; the batch records its source (`nba_api:playbyplayv3`) and time. |
| Validation, with a reason for each rejection | Actions that fail validation are left out and counted by reason, such as `UNKNOWN_ACTION_TYPE` or `UNKNOWN_PLAYER`. |
| Review before publication | A batch can be held as `PENDING_REVIEW`; its game stays hidden until an admin approves it. |
| Resume, not restart, and no double counting | Each batch keeps a checkpoint (the last play saved), and each game is committed as soon as it finishes. Plays are keyed on game and sequence number, so a re-run updates them. |

### How one game's batch runs

1. **Open** the batch as `RUNNING`. If the game has a `FAILED` batch from the same source, reopen that one and continue after its checkpoint.
2. **Download** the play-by-play, translate it and validate each action.
3. **Save** each accepted play, linked to the batch, moving the checkpoint forward. Count rejections by reason.
4. **Derive** each player's box-score line from the accepted plays; minutes and plus-minus come from the NBA's box-score feeds.
5. **Finish** as `COMPLETED`, or `PENDING_REVIEW` with `--review`, and commit.

If the download fails, the batch is committed as `FAILED` and the next pull of that game resumes it. A re-pull opens a **new** batch and re-links the game's plays to it; the old batch stays, so the history of runs and rejections is kept.

```
RUNNING ──┬── finished, no --review ──► COMPLETED        published at once
          ├── finished, --review ─────► PENDING_REVIEW ──┬─ Approve ─► COMPLETED   published
          │                                              └─ Reject ──► REJECTED    stays hidden
          └── download failed ────────► FAILED            the next run of the game resumes it
```

### Review and publication

**The rule:** a game is left out of every public page and API response while **any** of its batches is `PENDING_REVIEW`, `RUNNING`, `FAILED` or `REJECTED`. The rule is one shared query filter, `PUBLISHED_GAME_FILTER` in `apps/api/src/common/game-visibility.ts`, so no route can forget it. Games loaded before batches existed have either no batch or a `COMPLETED` placeholder (source `backfill:pre-batch-tracking`).

- **Approve** marks a pending batch `COMPLETED`, records the reviewer, time and notes, and clears the API cache so the game appears at once.
- **Reject** marks it `REJECTED`, and the game stays hidden. To publish it later, pull it again, approve the new batch and delete the rejected one.
- **Every batch counts.** An old pending or rejected batch keeps a game hidden even after a newer batch is approved. The team hit this after re-pulling 2025-26 on 23 Sep.
- **Delete** is a soft delete: it hides the batch and stops it counting, but leaves the game's plays and statistics. So deleting a pending or rejected batch *publishes* the game unless another batch holds it back. (The confirmation dialog wrongly says "The game data will be removed from the system.")

### The Batches tab

**Admin → Batches** has a **Pull Data** button with optional Season, From and To fields; a **Schedule** menu (never, hourly, daily or weekly); and, on the live site, a **Pull queue** panel. The list opens filtered to *Pending Review* and can be filtered by status, searched by game ID or team, and sorted. Each pending row has Notes, **Approve** and **Reject**; every row has **Correct plays**, which opens the corrections tool, and **Delete**. The routes are in the [API Reference](../api-reference.md#admin).

## The pull worker

### Why it is needed

stats.nba.com **blocks cloud providers' networks** (well documented in the `nba_api` issue tracker), so the API on Render can never download NBA data. The pull worker works like a print queue: the site adds a job, and a computer on a home connection picks it up. **Nothing runs a worker automatically.** If nobody has one running, queued pulls wait, and the admin page says so.

### How it works

`apps/ingestion/pull_worker.py` connects to the database directly, not through the API.

1. **On start,** it prints which database it is using and marks as `FAILED` any pull it claimed earlier but never finished.
2. **Every minute,** it checks in (so the admin page shows it online) and claims the oldest queued pull with `FOR UPDATE SKIP LOCKED`, so two workers never take the same pull.
3. **It runs** `ingest.py --review` with the pull's options, checking in every minute while it runs.
4. **It records** `SUCCEEDED` or `FAILED`, with the last 15 lines of output.

The API picks the mode itself: if `ingest.py` and its virtual environment sit next to it (local development), it runs the pull directly; otherwise (Render) it queues it. `INGESTION_MODE="queue"` forces the queue locally.

**Queue rules:** one pull at a time; a queued pull can be cancelled, a running one can't; a pull still running after 3 hours counts as abandoned; the hourly schedule never queues a second pull while one waits.

### Running it

Setup is in the "Pull worker" section of `apps/ingestion/README.md`. In short, from `apps\ingestion` with its virtual environment and an `.env` whose `DATABASE_URL` is the Supabase **session pooler** (port 5432):

```powershell
.venv\Scripts\python.exe pull_worker.py --once   # run what's queued, then exit
.venv\Scripts\python.exe pull_worker.py          # keep checking every minute
```

**Always read the first line it prints:** it names the database it is about to write to. Afterwards, re-run the predictor and optimizer (`apps/predictor/predict_games.py`, then `apps/optimizer/predict.py` and `optimize.py`) so predictions use the new data.

The worker bypasses the site's login, so the production `DATABASE_URL` must stay private. Never run the sample-data seed against production: it deletes every game.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
