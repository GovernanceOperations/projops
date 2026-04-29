# Adopting the GovOps GitHub Layer
## A complete guide for project teams connecting to the governance layer

This document covers every step a project team needs to take to connect
their GitHub repository to the GovOps governance layer and have all
eleven governance checks active and working.

If you are looking for how to deploy the governance layer itself —
not how to connect a project to it — see
[deploying-projops.md](deploying-projops.md).

Plain-language guide to every term: [vocabulary.md](vocabulary.md)

---

## What connecting does

When your repository is connected to this layer, eleven automatic
governance checks activate. They run as part of your existing GitHub
workflow — when you open a pull request, merge changes, close a
milestone, or deploy to production. You do not use a separate system.
The governance happens where the work happens.

What changes for your team:
- Pull requests cannot be merged without completing the checks
- Milestones cannot close without a governance review
- Production deployments cannot proceed without five readiness
  confirmations from named people
- Projects cannot archive without a post-project review and
  a named owner for every output

What does not change:
- How you write code or manage your project day-to-day
- Which tools you use
- Your existing review and approval processes (the governance
  layer sits alongside them, not instead of them)

---

## Before you begin

**What you need:**
- A GitHub repository for your project
- The name of your organisation's governance layer repository
  (ask whoever set up the layer — it will be something like
  `GovernanceOperations/projops`)
- Access to your organisation's permanent record endpoint details
  (ask the governance layer administrator — you will need this
  for your `govops.yml` configuration)
- About 30 minutes

**What you need to know about your project:**
Before filling in the configuration files, have answers ready for:
- What authorised this project? (You will need the decision reference number)
- What type of uncertainty does this project face? (Things you cannot
  know in advance, things you could find out, or a contested objective?)
- Who has authority to approve scope changes?
- Who can grant compliance exceptions?
- How is the project being delivered — and was that choice deliberate?

If any of these cannot be answered, the project may not be ready to
begin. The governance layer will surface this explicitly.

---

## Step 1 — Copy the GOVERNANCE.md template to your repository root

The GOVERNANCE.md file is the governance identity record for your
project. It must exist at the root of your repository before any
project work can be merged to the main branch.

```bash
# From your project repository root
curl -o GOVERNANCE.md \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/templates/GOVERNANCE.md
```

Or copy it manually from `templates/GOVERNANCE.md` in the governance
layer repository.

**Now open GOVERNANCE.md and complete every field.** The comments in
the template explain exactly what each field requires and why. Fields
that the automated check validates:

- Initiative title — must be non-empty
- Authorising decision reference — must match the format
  `ENTITY-YEAR-SEQUENCE-CHECKSUM` (example: `ACME-2026-0042-A3F9B2C1`)
- Decision type — must be one of: `reversible`, `conditional`, `irreversible`
- Governance tier — must be one of: `1`, `2`, `3`, `4`
- Uncertainty classification — must be one of: `aleatory`, `epistemic`, `ambiguity`
- Ambiguity resolution reference — required if classification is `ambiguity`
- All authority fields — must name a valid GitHub username
- Delivery pattern — must be one of: `staged-gate`, `iterative`,
  `parallel-track`, `hybrid`
- Execution framework — must be one of: `scrum`, `kanban`, `prince2`,
  `safe`, `internal`, `hybrid`
- Governance cadence — must be non-empty and specific
- Selection rationale — must be non-empty

**If your project's objective is currently contested (ambiguity):**
The governance check will block all execution until you complete
an alignment process among the project's sponsors and provide a
link to the resolution record in the `Ambiguity resolution record`
field. This is intentional — starting work on a contested objective
is the most reliable way to deliver the wrong thing with confidence.

---

## Step 2 — Copy and configure govops.yml

The `govops.yml` file tells the governance layer how your project is
classified and where its permanent records should be stored.

```bash
# From your project repository root
curl -o govops.yml \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/govops.yml.example
```

Or copy it manually from `govops.yml.example` in the governance layer
repository.

**Open govops.yml and fill in the required fields.** Every field has
a plain-language comment explaining what it does. Required fields are
marked `# REQUIRED`.

**The first block to fill in — governance scope:**

```yaml
governance_scope:
  # What GitHub entities does this initiative span?
  # repository = single repository
  # github-project = GitHub Projects board (may span multiple repos)
  # both = Projects board with a designated canonical repository
  substrate: "repository"

  # The repository holding the canonical GOVERNANCE.md
  canonical_repository: "your-org/your-repo"

  # Required if substrate is github-project or both:
  github_project_url: ""

  # Required if the initiative spans multiple repositories:
  additional_repositories: []
```

**If your initiative is tracked through a GitHub Project board:**
- Set `substrate` to `github-project` or `both`
- Provide the board URL in `github_project_url`
  (format: `https://github.com/orgs/YOUR-ORG/projects/N`)
- The GOVERNANCE.md in the canonical repository is the single
  governance record for the entire initiative regardless of how
  many repositories are involved
- The suite workflow caller (Step 3) must be added to the canonical
  repository. Additional repositories in the initiative do not each
  need their own full govops.yml — they need a minimal GOVERNANCE.md
  that points back to the canonical record

**The core initiative fields:**

```yaml
# How much governance does this project need?
initiative:
  tier: 2                        # 1, 2, 3, or 4

  # How easily can the main outcome be undone?
  decision_class: "conditional"  # reversible, conditional, or irreversible

# Where should permanent governance records be written?
ledger:
  endpoint: ""                   # The address of your permanent record system
  auth_secret: "LEDGER_API_KEY"  # Leave as-is — this references the organisation secret
  chain_id: "your-org-your-project-2026"  # A unique ID for this project's record chain

# How is this project being delivered?
delivery_system:
  pattern: "iterative"           # staged-gate, iterative, parallel-track, or hybrid
  framework: "scrum"             # scrum, kanban, prince2, safe, internal, or hybrid
  governance_cadence: "Monthly governance review at end of every second sprint"
```

**Getting the ledger endpoint:**
Ask your governance layer administrator for the permanent record system
endpoint. This is the HTTPS address where governance records will be
written. You do not need to manage the permanent record system yourself —
you only need to know its address.

---

## Step 3 — Add the caller workflow

Create a new file in your repository at
`.github/workflows/govops.yml` with the following content:

```yaml
name: GovOps Governance Layer

on:
  push:
    branches: [main, release/**]
  pull_request:
    types: [opened, synchronize, ready_for_review, closed]
  milestone:
    types: [closed]
  deployment:
  workflow_dispatch:
  schedule:
    - cron: '0 9 * * 1'   # Portfolio health check — every Monday 09:00 UTC

jobs:
  govops:
    uses: GovernanceOperations/projops/.github/workflows/govops-suite.yml@v0.9
    with:
      config-path: govops.yml
    secrets: inherit
```

Replace `GovernanceOperations/projops` with the actual name of your
organisation's governance layer repository if it is different.

---

## Step 4 — Set up branch protection

Branch protection rules enforce that the governance checks must pass
before changes can be merged. Without this, the checks run but do not
block merges — they become advisory rather than mandatory.

Go to your repository **Settings → Branches → Add branch protection rule**:

**Branch name pattern:** `main`

Enable the following:
- ✅ Require a pull request before merging
  - ✅ Require approvals: set to 1 (or more based on your project's tier)
  - ✅ Dismiss stale pull request approvals when new commits are pushed
- ✅ Require status checks to pass before merging
  - ✅ Require branches to be up to date before merging
  - In the search box, add these required status checks (they will
    appear after the first PR is opened):
    - `governance-init-check`
    - `ial-ceiling-check`
- ✅ Do not allow bypassing the above settings

**Add a protection rule for release branches:**

Repeat the above for the pattern `release/**` if your project uses
release branches.

**Note on bypasses:** Branch protection bypasses — where an admin
overrides these rules — are automatically logged by the exception
logger workflow as governance exceptions. They are not invisible.

---

## Step 5 — Set up deployment environment protection

If your project deploys to a production environment, configure
environment protection so the operational readiness gate and
rollback readiness gate activate before deployments.

Go to your repository **Settings → Environments → New environment**:

1. Create an environment named `production` (or the name that matches
   your `production_environment_pattern` in govops.yml)
2. Enable **Required reviewers** and add the person named as your
   project sponsor or deployment authority
3. Enable **Wait timer** if your deployment process requires a settling
   period before go-live

The governance checks (`operational-readiness-gate` and
`rollback-readiness-gate`) will activate automatically when a
deployment targets this environment.

---

## Step 6 — Create the issue templates

The governance layer uses GitHub issue templates for milestone reviews,
exception records, operational readiness checklists, post-project
reviews, and asset handovers. Copy them to your repository:

```bash
# Create the issue template directory if it does not exist
mkdir -p .github/ISSUE_TEMPLATE

# Copy the templates
curl -o .github/ISSUE_TEMPLATE/govops-exception.md \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/.github/ISSUE_TEMPLATE/govops-exception.md

curl -o .github/ISSUE_TEMPLATE/govops-milestone-review.md \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/.github/ISSUE_TEMPLATE/govops-milestone-review.md

curl -o .github/ISSUE_TEMPLATE/govops-operational-readiness.md \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/.github/ISSUE_TEMPLATE/govops-operational-readiness.md

curl -o .github/ISSUE_TEMPLATE/govops-pir.md \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/.github/ISSUE_TEMPLATE/govops-pir.md
```

The asset handover template goes at the repository root:

```bash
curl -o ASSET-HANDOVER.md \
  https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/templates/ASSET-HANDOVER.md
```

---

## Step 7 — Set up required labels

The governance checks search for issues by label. Create these labels
in your repository before the checks run:

Go to your repository **Issues → Labels → New label**:

| Label name | Suggested colour | Purpose |
|---|---|---|
| `govops-exception` | Red `#d73a4a` | Marks governance exception records |
| `milestone-checkpoint` | Orange `#e4e669` | Marks milestone governance reviews |
| `operational-readiness` | Blue `#0075ca` | Marks operational readiness checklists |
| `pir` | Green `#0e8a16` | Marks post-project review issues |
| `governance-scope` | Purple `#5319e7` | Marks issues that carry governance obligations |
| `rollback-tested` | Teal `#006b75` | Marks confirmation that rollback has been tested |
| `portfolio-health` | Grey `#e4e669` | Marks portfolio health reports |

To create labels programmatically using the GitHub CLI:

```bash
gh label create "govops-exception" --color "d73a4a" --description "Governance exception record"
gh label create "milestone-checkpoint" --color "e4e669" --description "Milestone governance review"
gh label create "operational-readiness" --color "0075ca" --description "Operational readiness checklist"
gh label create "pir" --color "0e8a16" --description "Post-project review"
gh label create "governance-scope" --color "5319e7" --description "Issue carries governance obligations"
gh label create "rollback-tested" --color "006b75" --description "Rollback procedure has been tested"
gh label create "portfolio-health" --color "e4e669" --description "Portfolio health report"
```

---

## Step 8 — Commit and push

Commit all the files you have created:

```bash
git add GOVERNANCE.md govops.yml .github/workflows/govops.yml \
        .github/ISSUE_TEMPLATE/ ASSET-HANDOVER.md
git commit -m "Connect to GovOps governance layer v0.9

- GOVERNANCE.md: project governance identity record
- govops.yml: layer configuration
- .github/workflows/govops.yml: governance check caller
- Issue templates for milestone review, exceptions, OR, PIR
- ASSET-HANDOVER.md template"
git push origin main
```

---

## Step 9 — Verify the connection

Open a pull request with any small change. The governance checks should
activate as required status checks. Verify:

**governance-init-check** passes:
- If it fails, open the check details for the specific error.
  The most common causes are missing required fields in GOVERNANCE.md,
  an incorrectly formatted decision reference, or a delivery system
  configuration mismatch between govops.yml and GOVERNANCE.md.

**Check that governance records are being written:**
After merging a pull request, ask your governance layer administrator
to confirm that a record appeared in the permanent record system.
If no record appears, the `LEDGER_ENDPOINT` secret may not be
set correctly, or the endpoint may not be reachable from GitHub Actions.

**If your initiative uses a GitHub Project board — additional checks:**

Close a test GitHub Project board and confirm that the pir-trigger
and asset-handover-validator checks activate and block closure until
the PIR and asset handover records are complete.

If the trigger does not activate, confirm that:
- The Ledger Bridge GitHub App is installed at the **organisation** level,
  not just on individual repositories — `projects_v2` events fire at
  organisation scope and require organisation-level App installation
- The `govops.yml` `governance_scope.substrate` field is set to
  `github-project` or `both`
- The `governance_scope.github_project_url` field contains a valid
  project board URL in the format `https://github.com/orgs/ORG/projects/N`

**Check that the portfolio is discoverable:**
If your organisation uses the `portfolio_label` field in govops.yml,
add that label to your repository so the portfolio health aggregator
can find it.

---

## Step 10 — Run your first milestone governance review

When you close your first project milestone, the governance check will
block it and ask for a checkpoint review. This is the check working
correctly — not an error.

To complete the review:
1. Go to your repository Issues
2. Click New issue
3. Select the "GovOps Milestone Governance Checkpoint" template
4. Fill in all sections — the template explains each one
5. At the Decisions section: record a decision OR check the
   "Continuation confirmed" box with an explicit confirmation statement
6. Close the issue
7. Close the milestone again — it will now close successfully

The first time a team completes this process, it often produces a
useful conversation about what has actually been decided, learned, and
changed since the last review. That conversation is governance working.

---

## Ongoing usage

**For every pull request:**
The governance checks run automatically. For most pull requests, they
pass without any action required beyond the existing code review process.

**For production deployments:**
Create an operational readiness issue using the template before
the deployment. Have the responsible person for each condition
confirm it. Close the issue. The deployment gate will then allow
the deployment to proceed.

**For every milestone:**
Create a milestone review issue using the template before or as
you close the milestone. Complete the review, record a decision
or continuation confirmation, and close the issue.

**For governance exceptions:**
When a requirement must be bypassed for operational reasons, the
exception logger creates the exception record automatically. Open
the exception issue, fill in the rationale and remediation sections,
and track it to closure within the expiry date.

**When the project closes:**
1. Create a PIR issue using the template
2. Complete the review with the team, sponsor, and at least one
   person from the receiving operational team
3. Submit the improvement suggestions to the governance improvement process
4. Complete the ASSET-HANDOVER.md — name an owner for every output
5. Get each owner to confirm receipt
6. Close the PIR issue
7. The repository can now be archived

---

## Getting help

**A check is blocking my work unexpectedly:**
Read the check's failure message carefully — it explains exactly
what is needed. Most failures are resolved by completing a template
or supplying a missing field. If the failure reason is not clear,
raise an issue in `GovernanceOperations/projops`.

**I need to override a check urgently:**
Override using your admin account if operationally necessary.
The exception logger will automatically create an exception record.
Fill in the rationale and remediation sections in the exception issue
within 24 hours. An override that is not followed up with a rationale
is an ungoverned bypass — visible in the portfolio health report.

**I think a governance standard is wrong for this project:**
Raise a formal delivery system review at your next milestone
checkpoint. Record it as a governance decision. If the standard
is systematically wrong across multiple projects, raise it through
the governance improvement process — not as an individual project
exception.

**I have a question about a specific field or term:**
See [vocabulary.md](vocabulary.md) for plain-language explanations
of every term. If the answer is not there, raise an issue in
`GovernanceOperations/projops` to suggest an addition.

---

## Quick reference — what triggers what

| What you do | Check that activates | What must be true |
|---|---|---|
| Open the first PR | `governance-init-check` | GOVERNANCE.md and govops.yml present and complete |
| Merge any PR | `pr-governance-event` | Produces a permanent record — no action needed |
| Someone approves a PR | `ial-ceiling-check` | Approver's identity assurance level meets the requirement |
| Create or edit a milestone | `uncertainty-classification-check` | Milestone description has an uncertainty classification marker |
| Close a milestone | `milestone-governance-checkpoint` | A completed checkpoint issue is closed first |
| Deploy to production | `operational-readiness-gate` | All five readiness conditions confirmed in a closed issue |
| Deploy something irreversible | `rollback-readiness-gate` | Rollback workflow exists, tested within 30 days, authority named |
| A check is overridden | `exception-logger` | Exception issue created automatically — fill in rationale |
| Every Monday | `portfolio-health-aggregator` | Produces health report — no action normally needed |
| Archive or close project | `pir-trigger` | PIR issue completed and closed first |
| Archive or close project | `asset-handover-validator` | ASSET-HANDOVER.md complete with all owners confirmed |

---

## Reference

- [deploying-projops.md](deploying-projops.md) — for governance layer administrators
- [configuration-reference.md](configuration-reference.md) — all govops.yml fields
- [vocabulary.md](vocabulary.md) — plain-language guide to every term
- [ial-out-of-band.md](ial-out-of-band.md) — for decisions requiring high identity assurance
