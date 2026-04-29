# Changelog

Every significant change to the GovOps GitHub Layer is recorded here.
The most recent version is at the top.

Plain language: this is the record of what changed between versions,
so users can understand what is new, what has been fixed, and whether
they need to do anything differently.

---

## [v0.1] — 2026-04-26 — First release

### What was added

**The eleven governance checks** covering the nine gaps between GitHub's
native project features and what good governance requires:
- Governance identity check — confirms every project has a mandate,
  an authorising decision reference, and a named authority structure
  before work begins
- Uncertainty classification check — confirms the type of uncertainty
  a project faces has been identified and, if the objective is
  contested, resolved before execution begins
- Change record — produces a permanent governance record entry for
  every significant merge of project changes
- Identity assurance check — confirms approvers meet the required
  level of verified identity for the decision they are approving
- Operational readiness gate — enforces five readiness conditions
  before any production deployment proceeds
- Rollback readiness gate — confirms a tested rollback procedure
  exists before irreversible deployments proceed
- Milestone governance checkpoint — blocks milestones from closing
  without a structured accountability review
- Exception logger — automatically records every governance bypass
  and creates a tracked exception with an expiry date
- Portfolio health aggregator — produces a regular health report
  across all projects in the portfolio
- Post-project review trigger — blocks project archiving until a
  structured learning review is completed
- Asset handover validator — blocks project archiving until every
  project output has a named owner, a service commitment, and a
  maintenance plan

**The five governance templates** — plain-language documents that
project teams, sponsors, and governance reviewers complete as part
of managing a governed project:
- GOVERNANCE.md — the project's governance identity record
- EXCEPTION.md — for recording governance bypasses and deferrals
- MILESTONE-REVIEW.md — for milestone accountability checkpoints
- PIR.md — the post-project learning review
- ASSET-HANDOVER.md — the formal handover of project outputs

**The Immutable Ledger Bridge** — the component that converts
GitHub activity into permanent, tamper-proof governance records
stored outside of GitHub.

**JSON validation schemas** — machine-readable rules that the
automated checks use to validate that templates are complete
and correctly formatted.

**The delivery system governance framework** — the selection
criteria, authority structure, and milestone checkpoint for
ensuring the way a project is run is a governed decision, not
a team default or political outcome.

**Plain-language documentation** — the vocabulary guide, the
plain-language README, and the rewritten practitioner reference —
so that executives, business leaders, and readers without
project management background can understand what this is,
what it does, and what it means for them.

**GitHub Projects (ProjectsV2) coverage** — closing a GitHub Project
board triggers the same PIR and asset handover checks as archiving a
repository. The projects_v2 event is handled in the suite workflow
and the Ledger Bridge.

**Governance scope declaration** — GOVERNANCE.md and govops.yml now
include a governance_scope block that explicitly distinguishes between
initiative (the governed entity), repository (code substrate), and
GitHub Project board (planning substrate). One canonical GOVERNANCE.md
per initiative regardless of how many repositories or boards are involved.

**Git platform coverage guide** — docs/git-platform-coverage.md documents
how the nine Platform Substrate Governance properties apply to non-GitHub
git hosts including GitLab, Azure DevOps, Bitbucket, Gitea, and Forgejo.

### Normative foundation
- Governance Operations Specification v0.9 §17.11
  (Project Operations Discipline)
- GOS v0.9 §6.6 (Governance Record structure)
- GOS v0.9 §11.2 (Security standards)

### Known gaps — work for v1.0

The following items are specified in this release but not yet
fully implemented. They are documented openly here because a
governance system that obscures its own gaps is not a governance system.

- The Ledger Bridge (bridge/) is architecturally specified and the
  connection manifest is complete — but the code that translates
  GitHub events into governance records and writes them to the
  external ledger is not yet built. Teams wanting the full permanent
  record capability will need to build or commission this component.
- The portfolio health aggregator works for small portfolios but
  needs tuning for organisations with more than 200 active repositories.
- The out-of-band identity verification process (for decisions
  requiring the highest levels of identity assurance) is documented
  but not yet automatically validated in the identity check workflow.
- The configuration file schema is validated structurally but not
  yet validated using the JSON schema file in the schemas/ directory.
