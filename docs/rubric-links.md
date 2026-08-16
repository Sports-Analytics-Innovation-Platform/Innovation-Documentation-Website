# Rubric Quick Links

One page mapping every rubric criterion (from the COMS3011A project brief) to where the evidence for it actually lives, so a marking tutor doesn't have to hunt across the site or the codebase. Grouped by milestone, in the brief's own order.

!!! warning "Honest about gaps, not just links"
    Where a criterion doesn't have real evidence yet, that's stated plainly below instead of linking to something that doesn't actually cover it. Milestone 1 is the only one currently due (Sprint 1, 2026-08-25) — Milestones 2–4 rows are here for when they're needed, and several are marked as not-yet-existing on purpose.

## Milestone 1: Sprint 1 (due 2026-08-25)

| Criterion | Weight | Evidence |
|---|---|---|
| Version Control | 10% | Repo itself (Gitea, not linkable from this site) + [Git Methodology](git-methodology.md) for how it's used |
| Documentation Site | 10% | This site |
| Getting Started / Dev Guides | 5% | [Getting Started](getting-started.md) |
| Work Tracker | 5% | Gitea Projects/Issues (external — not part of this docs site) |
| Git Methodology | 5% | [Git Methodology](git-methodology.md) |
| Project Methodology | 10% | [Methodology](methodology.md) |
| Tech Stack | 5% | [Tech Stack](tech-stack.md) |
| Stakeholder Interaction | 10% | [Client Meetings](meetings/client/index.md), [Standups](meetings/standups/index.md) |
| Initial Design & Dev Plan | 20% | [Architecture](design/architecture.md), [ERD](design/erd.md), [API Design](design/api-design.md), [UI Overview](design/wireframes.md), [Feature Tiers](design/feature-tiers.md), [Roadmap](design/roadmap.md) |
| Implementation | 20% | Codebase itself (Gitea) — this docs site can't demonstrate working code directly; point the tutor at the repo |

## Milestone 2: Sprint 2 (due 2026-09-15)

| Criterion | Weight | Evidence |
|---|---|---|
| Core Features | 25% | [Feature Tiers](design/feature-tiers.md) (basic tier) + [Requirements Traceability](requirements.md) — traceability table still has `TBD` Owner/Issue columns as of this writing |
| Automated Testing | 10% | ⚠️ No dedicated page — [Tech Stack](tech-stack.md) confirms Vitest is set up, but there's no testing-strategy documentation. Testing Documentation (below) is a separate, also-missing rubric line — worth writing one page that covers both. |
| Stakeholder Reviews | 10% | [Client Meetings](meetings/client/index.md) |
| API | 15% | [API Design](design/api-design.md) |
| User Feedback | 10% | ⚠️ No page exists. No formal user-testing/feedback-collection process is documented anywhere on this site. |
| Project Methodology | 10% | [Methodology](methodology.md) — this rubric line specifically wants evidence of *active* following, not just the document existing |
| Bug Tracker | 5% | Gitea Issues (external) — no docs-site page tracks usage |
| Database Documentation | 5% | [ERD](design/erd.md), [ADR-001: Database](decisions/adr-001-database.md) |
| Third-Party Code Documentation | 5% | ⚠️ Thin — [Tech Stack](tech-stack.md) lists and motivates libraries at a high level, but doesn't document third-party code usage per-component the way this rubric line likely wants |
| Testing Documentation | 5% | ⚠️ No page exists — see Automated Testing above |

## Milestone 3: Sprint 3 (due 2026-09-29)

The brief lists weights only, no descriptions, for this milestone — criteria names are assumed to carry the same meaning as their Sprint 2 counterparts.

| Criterion | Weight | Evidence |
|---|---|---|
| User Feedback | 10% | Same gap as Sprint 2 — not yet addressed |
| Automated Testing | 10% | [Tech Stack](tech-stack.md) (partial only) |
| Feature Implementation | 20% | [Feature Tiers](design/feature-tiers.md) (intermediate tier is the target for this sprint per the [Roadmap](design/roadmap.md)) |
| API Implementation | 20% | [API Design](design/api-design.md) |
| Performance | 5% | ⚠️ No page — this is a property of the running app, not something a docs page demonstrates on its own |
| Improvement | 5% | ⚠️ Would need a changelog or before/after comparison — doesn't exist yet |
| Documentation | 15% | This site generally |
| Project Methodology | 15% | [Methodology](methodology.md) |

## Milestone 4: Project Submission (due 2026-10-11)

| Criterion | Category | Weight | Evidence |
|---|---|---|---|
| Data | Database | 3% | [ERD](design/erd.md) |
| Deployment | Database | 2% | ⚠️ [ADR-003: Hosting Topology](decisions/adr-003-hosting-topology.md) — still a stub, no hosting decision made |
| Structure | Database | 5% | [ERD](design/erd.md), [ADR-001: Database](decisions/adr-001-database.md) |
| Availability | API | 3% | Depends on hosting — see ADR-003 gap above |
| Architecture | API | 5% | [Architecture](design/architecture.md) |
| Deployment | API | 2% | Same ADR-003 gap |
| Performance | API | 5% | ⚠️ Property of the running system, not a doc |
| Design | API | 10% | [API Design](design/api-design.md) |
| Accessibility | App | 5% | ⚠️ No accessibility audit or statement exists on this site yet |
| Aesthetics | App | 3% | [UI Overview](design/wireframes.md) |
| User Experience | App | 5% | ⚠️ No UX documentation beyond the UI Overview page |
| Deployment | App | 2% | Same ADR-003 gap |
| Performance | App | 5% | ⚠️ Property of the running system |
| Features | App | 10% | [Feature Tiers](design/feature-tiers.md) |
| Responsiveness | App | 5% | ⚠️ No responsiveness statement/testing documented |
| Structure | App | 5% | [Architecture](design/architecture.md) |
| Git Methodology | Misc | 5% | [Git Methodology](git-methodology.md) |
| Integration | Misc | 7% | [Tech Stack](tech-stack.md) (`nba_api` — though the ingestion path itself is still an open item) |
| Testing | Misc | 8% | [Tech Stack](tech-stack.md) (partial) |
| Tools | Misc | 5% | [Tech Stack](tech-stack.md), [AI Usage Ledger](ai-usage.md) |

---

*AI Declaration: The preceding document was generated with the assistance of the following: Claude-Web[Claude Sonnet 5]*
