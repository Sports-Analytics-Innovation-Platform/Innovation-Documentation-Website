# Coding Conventions

These conventions apply to all code in this project, across both `apps/api` and `apps/web`, regardless of language. They are enforced through code review, not tooling alone — reviewers should block a PR that violates them.

## Design

We follow **SOLID** object-oriented design principles where applicable:

- **S**ingle Responsibility — a class or module should have one reason to change.
- **O**pen/Closed — open for extension, closed for modification.
- **L**iskov Substitution — subtypes must be substitutable for their base types.
- **I**nterface Segregation — prefer several small interfaces over one large one.
- **D**ependency Inversion — depend on abstractions, not concrete implementations.

## Naming

- **Variables are nouns.** Prefer `elapsed_time_in_days = 0` over `d = 0`.
- **Functions are verbs.** Prefer `update_item_price()` over `item_updater()`.
- **Avoid disinformation.** Don't name something in a way that misrepresents what it is — e.g. don't call a `list` an "array", and don't name a variable `accountList` if it's actually a `Map`.
- **Include units of measurement in variable names** when the variable measures something. Prefer `time_in_milliseconds` over `time`.
- **Use consistent casing per language convention** — `snake_case` for Python, `camelCase` for TypeScript/JavaScript — matching the idiomatic style of the language in use.

## Functions

- **Keep functions small.** Each function should do one thing, contain few lines, and prefer a single level of indentation. Treat this as a strong preference, not a hard rule — don't over-extract trivial one-line helpers just to satisfy it.
- **Avoid magic numbers and strings.** Use named constants instead of unexplained literals.

## CRUD methodology

Adhere to Create / Read / Update / Delete separation when designing functions and data-handling code. Each function should map clearly to one operation rather than mixing several — e.g. keep `create_user()`, `get_user()`, `update_user()`, and `delete_user()` as separate functions instead of one function that both creates and updates records.

## Documentation

- **Document code as it's written, not after.** Add docstrings/comments in the same pass as writing the function, not as a cleanup step later.
- **Be descriptive about how functions work** — explain what the function does, its parameters, return value, and any non-obvious behavior or edge cases, not just a restatement of the function name.

## Enforcement

- Linting and formatting are run in CI on every PR (see [Git Methodology](git-methodology.md)) and must pass before merge.
- Naming and structure issues that tooling can't catch are a normal part of code review — call them out on the PR rather than after merge.

!!! note "AI-assisted code"
    If any code on a PR was generated, edited, or reviewed with AI assistance, it must be declared per the [AI Usage Ledger](ai-usage.md) and the course [AI policy](ai-usage.md) — this applies regardless of how small the AI contribution was.

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*