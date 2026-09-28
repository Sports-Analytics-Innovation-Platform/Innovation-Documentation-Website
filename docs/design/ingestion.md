# Data Ingestion

How NBA data gets from stats.nba.com into the database. It covers what a *pull* does, how each game's ingestion run is recorded as a **batch** and reviewed before it is published, and why pulls requested from the live site are run by a **pull worker** on a team member's computer. The last part explains how to run the pull worker.

Checked against `main` in the source repository on 2026-09-28.

## Words used on this page

| Term | Meaning |
|---|---|
| Pull | One run of `apps/ingestion/ingest.py`. It downloads NBA data from stats.nba.com and writes it to the database. |
| Batch | The record of one ingestion run for **one game**: where the data came from, how many plays were accepted and rejected and why, and whether it has been reviewed. Stored as an `IngestionBatch` row. A pull that covers 400 games creates 400 batches. |
| Review | An admin checking a batch on **Admin → Batches** and approving or rejecting it. Until a batch is approved, its game stays off the public site. |
| Queued pull | A pull someone asked for on the live site, waiting for a pull worker. Stored as an `IngestionRequest` row. |
| Pull worker | A small Python program, `apps/ingestion/pull_worker.py`, that a team member runs on their own computer. It carries out queued pulls. |

The tables themselves are described on the [ERD](erd.md#ingestion-and-review).

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
    The queued pull ends as SUCCEEDED or FAILED
                            │
                            ▼
    Admin → Batches: an admin approves each batch,
    and only then does the game appear on the public site
```

A developer can also run `ingest.py` by hand from their own computer, and when the API runs locally it starts `ingest.py` itself. Either way, the batches work the same.

## What a pull does

`ingest.py` works through these steps. It waits a second between calls to stats.nba.com, which publishes no rate limit; calling faster risks a temporary block.

1. **Teams, rosters and player biographies.** Rosters decide each player's current team. A pull for a *past* season leaves rosters and biographies alone, so traded or retired players aren't moved back to old teams.
2. **Which games.** By default, each team's 15 most recent games of the season plus the whole postseason. With `--from-date` and `--to-date`, only the games in that window, found with one league-wide call.
3. **Each game:** its box score and its play-by-play. The plays are translated into the platform's event format (`feed_translation.py`), checked (`event_validation.py`), stored as `GameEvent` rows, and added up into each player's box-score line (`derive_player_game_stats.py`). Each game gets a batch (see below).
4. **Plus-minus, usage and ratings,** from a few league-wide calls rather than one per game.

| Option | Default | Effect |
|---|---|---|
| `--review` | off | Batches finish as `PENDING_REVIEW` instead of `COMPLETED`, so an admin must approve them before the games are published. Pulls started from the site always use it. |
| `--season` | `2025-26` | Which season to pull, for example `2024-25` |
| `--from-date`, `--to-date` | none | Only games on or between these dates (`YYYY-MM-DD`). Either can be used alone. |

**How long it takes.** A full pull takes 35–45 minutes. A date-window pull is much quicker, apart from the first pull into an empty database, which has to fetch about 500 player biographies (a 10-game window took 16.9 minutes that way).

**It is safe to run again.** Every write updates the existing row if there is one, keyed on the NBA's own ids, so re-running a pull refreshes data instead of duplicating it.

!!! warning "Don't pull an older season into production"
    Play-by-play is stored for 2025-26 only, because of the database's 500 MB limit. A pull for 2023-24 or 2024-25, whether run by hand or queued on the site with that season filled in, would store at least 120 MB of plays and take the database past its limit. Older postseasons have their own route. See [ADR-005: Play-by-play storage](../decisions/adr-005-play-by-play-storage.md#loading-older-postseasons).

## Batches

### What a batch is for

A batch is the pipeline's own "submission": the record that every published statistic can be traced back to. Batches exist to meet four requirements in the brief ([Feature Tiers](feature-tiers.md)):

| The brief asks for | How batches provide it |
|---|---|
| Every statistic traceable to the events and the submission behind it | Every stored play points to the batch that wrote it, and the batch records the source (`nba_api:playbyplayv3`) and when it ran. |
| Submissions checked against the event schema, with a reason when they are rejected | Each action that fails the check is left out and counted under its reason in the batch's rejection summary, for example `UNKNOWN_ACTION_TYPE` or `UNKNOWN_PLAYER`, instead of failing silently. |
| Review before publication | A batch can be held as `PENDING_REVIEW`, and its game stays off the public site until an admin approves it. |
| A batch that fails part way through resumes rather than restarts, and resubmitting doesn't double-count | Each batch keeps a checkpoint (the last play saved), and every game is committed as soon as it finishes. Plays are keyed on game and sequence number, so a re-run updates them instead of adding copies. |

### How one game's batch runs

1. **The batch is opened** as `RUNNING`. If the game has a `FAILED` batch from the same source, the most recent one is reopened instead, and the run continues after its checkpoint.
2. **The play-by-play is downloaded,** translated into the platform's event format, and each action is checked against the event schema.
3. **Each accepted play is saved** and linked to the batch, and the checkpoint moves forward. Rejected actions are counted by reason.
4. **Each player's box-score line is derived** from the accepted plays. Minutes and plus-minus come from the NBA's official box-score feeds.
5. **The batch is finished** as `COMPLETED`, or as `PENDING_REVIEW` when the pull used `--review`, with its accepted and rejected counts. The game is committed to the database straight away.

If the download fails, the batch is marked `FAILED` and committed immediately, so the next pull of that game resumes from the checkpoint.

A re-pull of a game opens a **new** batch, and the game's plays are re-linked to it. The old batch stays, so the history of runs and rejections is never lost. The one exception is the reopened `FAILED` batch in step 1.

Older postseasons loaded with `ingest_postseason.py --skip-play-storage` still get a batch with its counts, even though the plays themselves aren't saved ([ADR-005](../decisions/adr-005-play-by-play-storage.md)).

### Statuses

```
RUNNING ──┬── finished, no --review ──► COMPLETED        published at once
          ├── finished, --review ─────► PENDING_REVIEW ──┬─ Approve ─► COMPLETED   published
          │                                              └─ Reject ──► REJECTED    stays hidden
          └── download failed ────────► FAILED            the next run of the game resumes it
```

### Review and publication

**The rule:** a game is left out of every public page and API response while **any** of its batches is `PENDING_REVIEW`, `RUNNING`, `FAILED` or `REJECTED`. Deleted batches don't count. The rule lives in one shared query filter, `PUBLISHED_GAME_FILTER` in `apps/api/src/common/game-visibility.ts`, so no page can forget it. A game with no batch at all is unaffected. Games loaded before batches existed either have none, or a `COMPLETED` placeholder batch (source `backfill:pre-batch-tracking`) added by `backfill_batches.py`.

- **Approve** (a pending batch only) marks it `COMPLETED` and records who reviewed it, when, and any notes. It also clears the API's cache, so the game appears on the site straight away.
- **Reject** (a pending batch only) marks it `REJECTED`. The game stays hidden. A rejected batch can't be approved later: to publish the game, pull it again, approve the new batch, and delete the rejected one.
- **Every batch counts, not just the newest.** If a game has an old batch that is still pending or rejected, approving a newer batch isn't enough; the old one keeps the game hidden until it is approved or deleted. The team ran into this after re-pulling 2025-26 on 23 September: batches left pending by an earlier pull held re-pulled games back until they were cleared.

### Deleting a batch

The **Delete** button on a batch row is a soft delete. It sets the batch's `deletedAt`, which hides the batch from the Batches list and stops it counting towards the publication rule. **Nothing else is deleted:** the game's plays and statistics stay in the database.

So deleting a pending or rejected batch **publishes** its game's data, unless another batch still holds the game back. Delete a rejected batch only once the game has a newer, approved batch.

!!! bug "The confirmation message is wrong"
    Before deleting, the page asks "Delete this batch? The game data will be removed from the system." No game data is removed; only the batch is hidden (`deleteBatch` in `apps/api/src/admin/admin-ingestion.service.ts`).

### The Batches tab

**Admin → Batches**, for admins only:

- **Pull controls:** a **Pull Data** button with optional **Season**, **From** and **To** fields, and a **Schedule** menu: *Never (manual only)*, *Hourly*, *Daily* or *Weekly*. On the live site, a **Pull queue** panel appears under them (see [The pull queue](#the-pull-queue)).
- **The list** opens filtered to *Pending Review*. It can be filtered by status and searched by NBA game id or team name. Its columns are Game, Date, Season, Status, Accepted, Rejected, Ingested and Reviewer, and it can be sorted by game date, season or ingest time.
- **Each row:** a pending batch has a **Notes** box with **Approve** and **Reject** buttons. Every batch has **Correct plays**, which opens the corrections tool for that game, and **Delete**.

The routes behind the tab are in the [API Reference](../api-reference.md#admin).

## The pull worker

### Why it's needed

The NBA data comes from **stats.nba.com**, which **blocks requests from cloud providers' networks**. This is well documented in the `nba_api` library's own issue tracker. The API runs on Render, a cloud host, so it can never download NBA data itself. The Python ingestion code isn't deployed there either.

Before the pull worker existed, a pull could only be started by a team member running `ingest.py` on their own computer, and the **Pull Data** button was disabled on the live site. The pull worker lets an admin ask for a pull from the live site, while the pull itself runs where stats.nba.com allows it: on a home internet connection. It works like a print queue. The site adds a job, and whichever computer is running the worker picks it up.

It doesn't remove the need for a person. **Nothing runs a worker automatically**: if nobody has one running, queued pulls just wait, and the admin page says so.

### What it is

`apps/ingestion/pull_worker.py` is a small Python program. It talks to the database directly, not through the API or a login.

1. **On start,** it prints which database it's using, and marks as `FAILED` any pull that a worker with the same name claimed but never finished (for example because it was stopped mid-pull).
2. **Every minute,** it checks in, which updates its `IngestionWorker` row so the admin page can show it as online, and claims the oldest queued pull. The claim is a single database statement that locks the row (`FOR UPDATE SKIP LOCKED`), so two workers can never take the same pull.
3. **It runs** `ingest.py --review` with the pull's season and dates, as a separate process, printing its output. While the pull runs, which can take 35–45 minutes, it keeps checking in every minute, so the admin page doesn't report it offline.
4. **It records the result:** `SUCCEEDED` or `FAILED`, with the last 15 lines of the pull's output (where any error will be). A successful pull also counts as the schedule's latest run.

Because every pull it runs uses `--review`, its batches always arrive as `PENDING_REVIEW` for an admin to approve.

### Direct and queue mode

The API decides for itself how to carry out **Pull Data**:

| Mode | When | What happens |
|---|---|---|
| Direct | `ingest.py` and its Python virtual environment exist next to the API, which is true in local development | The API starts `ingest.py --review` itself. Only one pull runs at a time. |
| Queue | They don't, which is the case on Render | The API saves a queued pull for a worker. The Pull queue panel appears on the Batches tab. |

To try the queue locally, set `INGESTION_MODE="queue"` in `apps/api/.env` and restart the API.

### The pull queue

- **One pull at a time.** While a pull is queued or running, the site refuses another and says why.
- **Cancelling.** A queued pull can be cancelled from the Pull queue panel. A running one can't, because it is running on someone else's computer.
- **Abandoned pulls.** A pull still marked running after 3 hours is treated as abandoned and stops blocking new ones. The worker also marks its own unfinished pulls as failed when it restarts.
- **Scheduled pulls.** Every hour the API checks the **Schedule** setting. When a pull is due, it queues one with the default options: the current season's recent games and the postseason. It never queues a second while one is waiting, so a worker that is switched off doesn't come back to a backlog.
- **The panel** shows each pull's status, who asked for it (or "Scheduled"), which worker ran it and the end of its output. It says whether a worker is online, meaning it checked in within the last 3 minutes, and refreshes every 15 seconds while a pull is active.

### Running the worker

#### 1. One-time setup

You need the repository and Python 3.11 or newer. From the repository root:

```powershell
git checkout main
git pull

cd apps\ingestion
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
copy .env.example .env
```

Use `.venv\Scripts\python.exe -m pip` rather than a bare `pip`, which can install into a different Python from the project's. If `.venv` already exists, skip those two lines, but do run `git pull` on `main`.

#### 2. Point it at the right database

The worker uses **only** the `DATABASE_URL` in `apps\ingestion\.env`. Setting `DATABASE_URL` in your terminal has no effect, because `db.py` deliberately lets the `.env` file win. Make sure exactly one `DATABASE_URL` line is active:

- **For the live site:** the production database's connection string from the Supabase dashboard (**Connect**). Use the session pooler string on port `5432`, which is what the team's pulls have run on, not the transaction pooler on port `6543`, which is for the API.
- **For testing:** the local Docker database, `postgresql://postgres:postgres@localhost:55432/nba_analytics`. Don't add `?schema=public`; the Python database driver rejects it.

The `.env` file contains the database password. It is ignored by git, and must never be committed.

#### 3. Queue a pull on the site

With an admin account, go to **Admin → Batches**. Optionally fill in **Season**, **From** and **To**, and click **Pull Data**. The Pull queue panel shows the pull as `QUEUED`.

#### 4. Run the worker

From `apps\ingestion`:

```powershell
# Run whatever is queued, then exit
.venv\Scripts\python.exe pull_worker.py --once

# Or keep running, checking every minute
.venv\Scripts\python.exe pull_worker.py
```

| Option | What it does | Default |
|---|---|---|
| `--once` | Runs anything queued, then exits | Keeps running |
| `--name "Home-PC"` | How the worker appears on the admin page | The computer's name |
| `--interval 30` | Seconds to wait between checks when there's nothing to do | `60` |

Press **Ctrl+C** to stop it. Stopping it mid-pull stops that pull too, and the worker marks it `FAILED` the next time it starts under the same `--name`.

#### 5. What you should see

```
Using database aws-0-eu-west-2.pooler.supabase.com:5432/postgres (from apps/ingestion/.env).
Pull worker 'Home-PC' checking for queued pulls once.
Claimed pull 3f2a…: --review --season 2025-26 --from-date 2026-04-14 --to-date 2026-04-18
…the pull's own progress…
Pull 3f2a… succeeded.
```

**Always read the first line.** It names the database the worker is about to write to. If it isn't the one you meant, press Ctrl+C straight away and fix `apps\ingestion\.env`.

On the site, the panel shows the pull as running on your worker, then `SUCCEEDED` or `FAILED` with the end of its output. The new batches appear on the Batches tab as `PENDING_REVIEW`, ready to approve.

#### 6. Optional: run it on a timer

To have queued pulls picked up without starting the worker by hand, use Windows Task Scheduler (or cron on macOS or Linux):

1. **Create Task**, named for example `SportsAnalytics pull worker`.
2. **Trigger:** daily, repeating every 15 minutes, indefinitely.
3. **Action:** start `<repository>\apps\ingestion\.venv\Scripts\python.exe` with the argument `pull_worker.py --once`, starting in `<repository>\apps\ingestion`.

The computer has to be on and online for it to run.

### Troubleshooting

| You see | What it means |
|---|---|
| "No pull worker has checked in yet" on the Batches tab | Nobody has run a worker against this database. Start one. |
| "Pull worker last checked in … and may be offline" | The worker was stopped or the computer is off. Start it again. |
| The worker's first line names the wrong database | The wrong `DATABASE_URL` is active in `apps\ingestion\.env`. |
| `invalid URI query parameter: "schema"` | Remove `?schema=public` from `DATABASE_URL`. |
| `relation "IngestionRequest" does not exist` | That database doesn't have the queue tables; the worker is probably pointed at the wrong one. |
| Many timeouts talking to stats.nba.com | Run from a normal home network. Cloud servers and some VPNs are blocked. |
| A pull says `RUNNING` but no worker is running | The worker was stopped mid-pull. Starting it again marks the pull `FAILED`; after 3 hours it stops blocking new pulls anyway. |
| A pull is `FAILED` | The panel shows the end of the pull's output. The cause is usually on the last line. |
| The games from a finished pull aren't on the site | Their batches are waiting for approval on the Batches tab, or an older batch for the same game is still pending or rejected (see [Review and publication](#review-and-publication)). |

### Good to know

- **The worker bypasses the site's login.** Anyone with the production `DATABASE_URL` can write to the database, so keep it private.
- **Two workers at once are safe,** because a pull is only ever claimed by one. One is enough.
- **Never run the sample-data script against production.** `npx prisma db seed` deletes every game and all game statistics.
- **After a pull,** re-run the predictor and optimizer (`apps/predictor/predict_games.py`, then `apps/optimizer/predict.py` and `optimize.py`) so predictions and lineups use the new data.
- **Source files:** `apps/ingestion/pull_worker.py`, `apps/ingestion/ingest.py`, `apps/ingestion/play_by_play.py`, `apps/api/src/admin/admin-ingestion.service.ts`, `apps/api/src/admin/admin-batches.service.ts`, `apps/web/src/components/PullQueuePanel.tsx`, and the "Pull worker" section of `apps/ingestion/README.md`.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Code[Claude Opus 5.5]*
