# Git Methodology

We use a lightweight, PR-based branching workflow on **Gitea**, which hosts the project's monorepo (`apps/api`, `apps/web` and the Python services) per the university's requirement to use university-provided version control. This documentation site is a separate case — it's built with MkDocs and deployed via **GitHub Pages** for public static hosting (see [Documentation Site](getting-started.md)), but the actual codebase and issue tracking live on Gitea, not GitHub.

Changes reach `main` through pull requests that a teammate reviews.

## Branches

- **`main`** — the stable, always-deployable version of the code. Nothing goes here until it's reviewed and the team agrees it's done, and every change lands via pull request.

  In practice the rule had exceptions: `main` wasn't protected on Gitea, and 21 commits went to it directly, mostly in the first weeks (two of them reverts). An 8 Oct audit found that one of them had carried a CI runner's registration file onto `main` ([Security](security.md#incident-11-sep-a-runner-token-committed)). Of the 107 pull requests merged by 8 Oct, 79 were merged by a teammate other than their author; reviews mostly happened in the lab and on WhatsApp, so only a few carry a recorded Gitea approval.

  Since 8 Oct `main` is protected on Gitea: direct pushes are refused, and a pull request can merge only once its CI checks pass. Approvals aren't enforced, so review stays the team's rule rather than Gitea's.
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

The intent is for CI to also run a secret scanner (`gitleaks`/`trufflehog`) on every PR as a backstop — see [Definition of Done](definition-of-done.md). **This is not yet implemented**: the current pipeline runs lint, typecheck, and test only (see [CI/CD Pipeline](ci-cd.md)), so the manual check above is the only line of defence today.

## Opening and merging PRs

1. Branch off `main` for the feature.
2. Do the work, committing in the small units described above.
3. Push the feature branch and open a pull request on Gitea against `main`.
4. **At least one team member reviews and approves.** Only merge once the team has reviewed the code and agrees it's good and finished — don't assume; confirm it.
5. If there's no confirmation everyone agrees, don't merge — keep the PR open for review instead.
6. CI must pass before merging; since 8 Oct Gitea enforces this on `main`. The pipeline lints, typechecks and tests `apps/api` and `apps/web` on every push and pull request — see [CI/CD Pipeline](ci-cd.md) for its jobs and what they don't cover. Newer pushes cancel in-flight runs on the same ref, so only the latest commit's result counts.

## Repo hygiene

- Every app (`apps/api`, `apps/web` and each Python service) keeps its own README with setup instructions, verified periodically by a teammate who hasn't touched that app doing a clean install.
- Commit history should stay clean going into each milestone — no dead code, no commented-out blocks, consistent formatting.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5], Claude-Code[Claude Opus 5.5]*