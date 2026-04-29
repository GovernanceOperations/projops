# GovernanceOperations/projops

**The GovOps Project Operations Layer for GitHub**

This repository makes governance real inside any GitHub project.
It is not a policy document. It is not a checklist. It is a working
governance system — a set of automated checks, templates, and permanent
records that ensure every project running on GitHub is accountable,
auditable, and honestly managed from start to finish.

---

## Three things this layer uses the word "project" to mean

Before anything else, a clarification — because GitHub uses the word
"project" to mean three different things, and this layer needs to be
precise about all three.

**An initiative** is the governed delivery entity. The body of work
that has been formally authorised, has a mandate and a sponsor, consumes
resources, and will eventually produce outputs that the organisation
continues to use. This is what governance is applied to. The initiative
is the thing that has a GOVERNANCE.md, a decision class, an uncertainty
classification, and five operational readiness conditions.

**A repository** is the GitHub storage location where code and files
live. For many initiatives, the repository is the primary delivery
substrate — where changes are made, reviewed, and merged. The governance
layer activates on repository events: pushes, pull requests, deployments,
milestones, and archiving.

**A GitHub Project** (also called a project board) is GitHub's built-in
planning and tracking tool — a board, table, or roadmap view of work
items. A GitHub Project board is separate from any repository. It can
span multiple repositories. It has its own lifecycle, including closure.

**This layer covers all three.** The governance checks activate on
repository events and on GitHub Project events. Closing a GitHub
Project board triggers the same post-project review and asset handover
checks as archiving a repository. For initiatives spanning multiple
repositories through a GitHub Project board, the GOVERNANCE.md governance
scope field declares which repository holds the canonical governance record.

**What about git hosts other than GitHub?**
The governance workflows in this repository run on GitHub Actions and
are GitHub-specific. The governance *principles* — the nine Platform
Substrate Governance properties from GOS §17.11 — apply to any git
host. GitLab, Azure DevOps, Bitbucket, Gitea, and Forgejo can all
implement equivalent governance layers using their native automation.
The permanent record (Immutable Ledger) and the governance templates are
platform-agnostic. See [docs/git-platform-coverage.md](docs/git-platform-coverage.md)
for specific guidance on each platform.

---

Most organisations have governance policies. Few have governance practice.
The gap between the two is where projects fail, compliance obligations
get quietly dropped, decisions go unrecorded, and no one can explain
afterward what happened or why.

This layer closes that gap — not by adding bureaucracy, but by embedding
governance into the workflow that project teams are already using.
When a team works in GitHub, this layer works alongside them.
They do not need to fill in a separate governance system, file a report,
or remember to notify a compliance team. The governance happens
as the work happens.

---

## What does it do?

When a team connects this layer to their GitHub repository, eleven
automatic checks activate. Each one closes a specific gap between
how GitHub works and what good governance requires.

**Before any project begins:**
- Checks that the project has a clear identity — a mandate, a named
  authority, a decision class, a delivery approach, and a link to the
  decision that authorised it. A project that cannot answer these
  questions is not ready to begin.
- Checks that the type of uncertainty the project faces has been
  classified. Is the outcome genuinely unpredictable? Is information
  missing but findable? Or is the objective itself contested?
  Each type requires a different governance response.

**As work happens:**
- Records every significant merge of changes as a permanent governance
  entry — who approved it, under what authority, linked to the project's
  authorising decision.
- Confirms that approvers have the right level of verified identity for
  the decision they are approving. A rubber-stamp approval from an
  unverified account does not satisfy governance.
- Logs every governance bypass automatically — when a requirement is
  overridden, it creates a tracked exception with a named owner and
  an expiry date.

**Before anything goes live:**
- Requires confirmation that the organisation is actually ready to
  operate what the project delivered — not just that the technical
  work is complete. Five readiness conditions must be confirmed by
  named people before a production deployment proceeds.
- For changes that cannot be undone, requires that a tested rollback
  procedure exists and that someone with the authority to invoke it
  is named.

**At milestones:**
- Blocks milestone closure until a structured accountability review
  has been completed and a decision or formal continuation has been
  recorded. A milestone that closes without a governance decision is
  not a governance checkpoint — it is a date on a calendar.

**When a project closes:**
- Blocks archiving until a post-project learning review has been
  completed, its lessons submitted to the governance improvement
  process, and every output of the project assigned to a named owner
  with a service commitment and a maintenance plan.

---

## Who is this for?

**Executives and board members:**
This layer gives you assurance that projects running on GitHub are
being governed — not just tracked. Every significant decision is
recorded. Every compliance bypass is logged. Every deployment is
gated on readiness. You can ask "what happened on this project?"
and get a verifiable answer.

**Business leaders and programme sponsors:**
This layer gives you visibility into your projects without requiring
you to learn GitHub. The governance templates — particularly the
GOVERNANCE.md identity document, the milestone checkpoint, and the
portfolio health report — are written in plain language and designed
for business review.

**Project managers and delivery teams:**
This layer gives you governance infrastructure that works with how
you already work, rather than alongside it as a separate overhead.
The checks run automatically. The templates tell you exactly what
is needed and why. The exception log catches overrides before they
become invisible debt.

**Compliance and audit functions:**
This layer gives you a permanent, tamper-proof record of every
governance act on every governed project — who decided what, when,
under what authority, with what record of the decision. The record
cannot be altered retroactively. It cannot be selectively deleted.
It is there when you need it.

**Curious readers with no project management background:**
If you are reading this and wondering what all of it means — the
[vocabulary guide](docs/vocabulary.md) explains every term in
plain language. Start there. Then come back to this document.

---

## What does "governed" mean in practice?

A governed project, under this layer, means:

1. **It has a clear mandate.** Everyone knows what it is trying to
   achieve, who authorised it, and how success will be measured.

2. **Its uncertainty is classified.** The team knows whether they are
   dealing with genuinely unpredictable outcomes, missing information
   that can be found, or a contested objective that must be resolved
   before work begins.

3. **Its delivery approach is a decision, not a default.** The way the
   project is being run was chosen deliberately, recorded, and approved —
   not inherited from habit or team preference.

4. **Every significant action is recorded.** Changes, approvals, exceptions,
   milestone reviews, and closures all produce permanent records that
   cannot be altered.

5. **Nothing goes live until the organisation is ready.** Five readiness
   conditions must be confirmed by named people. Projects do not
   "go live" on hope.

6. **Nothing closes until the organisation has learned from it.** Every
   project produces a structured learning review. Every output has a named
   owner. No project disappears into a folder and is forgotten.

---

## Documentation

| Document | Who it is for | What it covers |
|---|---|---|
| [docs/adopting-the-layer.md](docs/adopting-the-layer.md) | Project teams | Complete step-by-step guide to connecting a project repository to this layer |
| [docs/deploying-projops.md](docs/deploying-projops.md) | Governance layer administrators | How to set up the GovernanceOperations/projops repository itself |
| [docs/configuration-reference.md](docs/configuration-reference.md) | Anyone configuring govops.yml | Every field explained — what it does, valid values, what goes wrong |
| [docs/git-platform-coverage.md](docs/git-platform-coverage.md) | Teams on non-GitHub git hosts | How governance coverage works on GitLab, Azure DevOps, Bitbucket, Gitea, Forgejo |
| [docs/vocabulary.md](docs/vocabulary.md) | Everyone | Plain-language explanation of every term, acronym, and concept |
| [docs/ial-out-of-band.md](docs/ial-out-of-band.md) | Governance authorities | High identity assurance approvals beyond GitHub's IAL 2 ceiling |
| [bridge/README.md](bridge/README.md) | Administrators, developers | The permanent record bridge — what it does, how it works, setup |
| [CHANGELOG.md](CHANGELOG.md) | Everyone | What has changed between versions |

---

## How does a team connect to this layer?

Three steps:

**Step 1.** Copy the [GOVERNANCE.md template](templates/GOVERNANCE.md)
to the root of the project repository and complete all required fields.
This is the project's governance identity document.

**Step 2.** Copy [govops.yml.example](govops.yml.example) to `govops.yml`
in the repository root and fill in the configuration fields. This tells
the layer how the project is classified and where its permanent records
should be stored.

**Step 3.** Add a single workflow file to the repository that calls this
layer. The full instruction is in the [quick start](docs/configuration-reference.md).

Once connected, the eleven governance checks activate automatically.
No additional tooling, no additional meetings, no separate governance portal.

---

## What is the Ledger Bridge?

The Ledger Bridge is an optional but strongly recommended component.
It connects GitHub's activity stream to a permanent record system
(the Immutable Ledger) outside of GitHub.

Why outside of GitHub? Because GitHub's own records can be modified by
administrators. A permanent governance record must be held somewhere
that cannot be altered — not even by the organisation's own administrators.
The Ledger Bridge writes every governance event to that external system
in a tamper-proof, cryptographically chained format.

Plain language: *it is the mechanism that ensures "what happened on this
project" can always be answered honestly, even years later, even under
adversarial scrutiny.*

Instructions for setting up the bridge are in [bridge/README.md](bridge/README.md).

---

## Vocabulary

Every technical term, acronym, and concept used in this repository
is explained in plain language in the [vocabulary guide](docs/vocabulary.md).

If you encounter a word that is not there, raise an issue.

---

## Versions

This layer is versioned in alignment with the Governance Operations
Specification. Layer `v0.9.x` implements the requirements of GOS `v0.9`.

Always call a specific version — never `@main`. A team that depends
on the head of a development branch is accepting unreviewed governance
changes into their project automatically. That is itself a governance gap.

See [CHANGELOG.md](CHANGELOG.md) for what has changed between versions.

---

## Normative foundation

This layer implements Section 17.11 of the
*Governance Operations Specification v0.9* — the Project Operations Discipline.
The specification is the authority. Where this layer and the specification
appear to conflict, the specification governs.

---

*GovernanceOperations/projops | v0.1 | This repository governs itself
under this layer. See GOVERNANCE.md.*
