# Git Methodology

We use a lightweight, PR-based branching workflow on **Gitea**, which hosts the project's `apps/api` and `apps/web` repos per the university's requirement to use university-provided version control. This documentation site is a separate case — it's built with MkDocs and deployed via **GitHub Pages** for public static hosting (see [Documentation Site](getting-started.md)), but the actual codebase and issue tracking live on Gitea, not GitHub.

Nothing reaches `main` without review.

## Branches

- **`main`** — the stable, always-deployable version of the code. Nothing goes here until it's reviewed and the team agrees it's done. Direct pushes to `main` are disabled; all changes land via pull request.
- **Feature branches** — one per feature or fix, branched off `main`. Keep names short, lowercase, and hyphenated, e.g. `applicant-profile-page`, `nba-boxscore-import`, `auth-password-reset`.

## Committing

- **Ask before committing.** Proposed commit message(s) are shown and agreed before `git commit` / `git push` runs — even when the work is clearly finished.
- **One commit per independent unit of work.** A single feature, a single bug fix, a single refactor. If a change touches two unrelated things, split it into two commits rather than bundling them (e.g. don't combine "add CV upload" with "fix typo in nav").
- **Commit format:**

  ```
  tag: short plain-English description
  ```

  - Tag is one lowercase word, no scopes — `feat`, `fix`, `refactor`, `chore`, `docs`, `style`, `test`.
  - Description is imperative, no trailing period: `feat: add CV upload button`, not `feat: Added CV upload button.`
  - Keep detail out of the subject line; use the commit body if more context is needed.

  Examples:

  ```
  feat: add NBA boxscore endpoint
  fix: correct FDR lookup validation
  refactor: simplify optimiser request state
  chore: bump prisma client version
  ```

- **AI-generated or AI-assisted commits** must include an `Assisted-by:` trailer naming the tool and model, per the course [AI policy](ai-usage.md):

  ```
  feat: add player search endpoint

  Assisted-by: Claude-Code[Claude Sonnet 5]
  ```

## Checking for sensitive data before committing

Before any commit is proposed, staged changes are scanned for secrets — `.gitignore` alone is not trusted:

- API keys, tokens, or credentials (e.g. database service keys, JWT secrets)
- `.env` files or hardcoded connection strings/passwords
- Private keys or certificates (`.pem`, `.key`, etc.)
- Any personal data that shouldn't be in the repo

Practically:

1. Run `git diff --staged` (or check `git status` for files about to be added) before writing the commit message.
2. If something sensitive shows up, stop — don't commit it. Move it to a `.env` file (gitignored) or an untracked config file instead.
3. If a secret was already committed in an earlier commit, flag it separately: removing it from the latest commit isn't enough, since it's still in git history, and the key must be rotated.

CI also runs a secret scanner (`gitleaks`/`trufflehog`) on every PR as a backstop — see [Definition of Done](definition-of-done.md).

## Opening and merging PRs

1. Branch off `main` for the feature.
2. Do the work, committing in the small units described above.
3. Push the feature branch and open a pull request on Gitea against `main`.
4. **At least one team member reviews and approves.** Only merge once the team has reviewed the code and agrees it's good and finished — don't assume; confirm it.
5. If there's no confirmation everyone agrees, don't merge — keep the PR open for review instead.
6. Gitea Actions CI (lint, typecheck, tests, build) must pass before merge is allowed.

## Repo hygiene

- Every repo (`apps/api`, `apps/web`) keeps its own README with setup instructions, verified periodically by a teammate who hasn't touched that repo doing a clean install.
- Commit history should stay clean going into each milestone — no dead code, no commented-out blocks, consistent formatting.

!!! note "Why not push straight to `main`?"
    Milestone rubrics grade both **git methodology being documented** and **being used** — direct pushes and unreviewed merges undercut both, even if the code itself is fine.

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*