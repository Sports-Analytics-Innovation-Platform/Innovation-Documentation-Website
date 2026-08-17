# Feature Tiers

!!! success "Unblocked"
    Previously blocked on "what does the optimisation engine actually optimise?" The team's answer: compile team/player analytics from real game data, use ML to **predict** outcomes, then (as an advanced-tier feature) **recommend** actions that improve a team's win chances. Tiers below follow the brief's own definitions (§1.1): Basic = MVP, Intermediate = functional and useful, Advanced = market-ready.

!!! warning "What's still a proposal, not a decision"
    The tier *shape* comes directly from the team's answers. The specifics inside each tier (which model type, which exact features go into it, exact UI treatment) are my draft of what a reasonable build-out looks like — not something the team has signed off on. Treat anything below as a starting point to edit, not a spec to build against blindly.

## Basic tier (MVP)

Core analytics platform (mostly already built — see [Requirements](../requirements.md) and [ERD](erd.md)):

- Team/player browsing with event-derived season stats (`GameEvent` → `PlayerGameStat`, confirmed in schema)
- Auth via BetterAuth (Google OAuth) — see [ADR-002](../decisions/adr-002-auth.md)

Prediction (new, not yet built):

- **Team-level matchup prediction**: given two teams, output a win-probability estimate, trained on historical game results and season stats already in the database.
- Exposed via one API endpoint and shown in the UI as a single probability with the underlying stats that informed it — not a black box.
- Model choice is intentionally left simple at this tier (e.g. logistic regression over aggregate team stats) — the point of Basic is a working end-to-end pipeline (data → model → API → UI), not model sophistication.

## Intermediate tier (functional and useful)

- **Player-level prediction** added alongside team-level: predicted individual performance (e.g. points/rebounds/assists) for an upcoming or hypothetical matchup, per the team's confirmed scope.
- **Model accuracy shown, not just asserted** — a view comparing predicted vs. actual outcomes for games that have already happened, so the prediction isn't just a number nobody can verify. This also gives you something concrete to show in Sprint 3 reviews.
- Richer feature set feeding the model where the data supports it (recent form, home/away split, head-to-head history) — exact feature list still open.

## Advanced tier (market-ready)

- **Recommendation layer** — the actual "optimisation" in the product name: given a team's current roster/available lineup options, recommend the lineup or rotation that maximises predicted win probability. This is a materially different problem from prediction (search/decision over options vs. estimating one fixed outcome) — budget for it accordingly.
- Possibly a "what-if" simulator: user picks a hypothetical lineup change, sees the predicted effect.
- Model comparison/versioning if time allows.

## Still open

- Exact model/algorithm choices at each tier
- Exact feature set feeding the prediction model
- Whether the recommendation layer (advanced tier) is in-scope for this semester at all, or explicitly a stretch goal — see [Roadmap](roadmap.md) for how this maps to sprint dates, including the client's own guidance on ML timing.

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
