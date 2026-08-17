# Definition of Done

A story or task is not "done" when the code runs on someone's laptop. It's done when every item below is true. This applies to every ticket on the board, regardless of who owns it.

A story is **Done** when:

1. **Code is reviewed and merged** via a GitLab merge request against `main`, with at least one team-member approval (see [Git Methodology](git-methodology.md)).
2. **Tests are written and passing in CI** — unit tests at minimum; integration or E2E tests where the change touches an API route or a critical user journey (auth, import, optimiser run).
3. **Documentation is updated** — the relevant page on this docs site, the OpenAPI spec, or an ADR, as appropriate to the change. A feature isn't done if the docs site doesn't reflect it.
4. **Accessibility is checked**, if the change touches UI — keyboard navigation works, and `axe-core` reports no new critical/serious violations.
5. **Deployed to staging and manually verified** — not just "CI is green." Someone has clicked through the actual behaviour.
6. **The issue is linked to the commit/PR** and moved to Done on the board. Cards don't sit in "In Progress" after the code has shipped.
7. **AI usage is logged** in [`ai-usage.md`](ai-usage.md) if any AI tool was used anywhere in producing the change — code, docs, or review — per the course [AI policy](ai-usage.md).
8. **Coding conventions are followed** — naming, function size, and SOLID structure per [Coding Conventions](coding-conventions.md). Reviewers should block a PR that violates them, not wave it through and file a follow-up.
9. **No secrets or credentials committed** — confirmed as part of the pre-commit check in [Git Methodology](git-methodology.md), not assumed.

## What "Done" is not

- **Not** "works on my machine." If it isn't verified on staging, it isn't done.
- **Not** "I'll add tests later." Tests are part of the same PR, not a follow-up ticket.
- **Not** "I'll document it at the end of the sprint." Docs are updated in the same PR as the change they describe — a docs backlog at milestone time reads as neglect.
- **Not** a judgement call made alone. If a reviewer disagrees a story meets this bar, it goes back to In Progress, not into Done with a caveat.

## Where this gets checked

- **Every PR**, against items 1–3, 8, 9 above, by the reviewer before approving.
- **Every weekly client/standup meeting**, spot-checking a few Done cards against items 4–7.
- **End of each sprint**, as an explicit pass over the board before the milestone deadline — nothing should move to Done in the last 24 hours without having actually gone through the steps above earlier in the sprint.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
