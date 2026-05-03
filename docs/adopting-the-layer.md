# Adopting the GovOps GitHub Layer
## Complete guide — Step 7 and day-to-day operations

This document covers Step 7 of the configuration sequence: connecting
a project repository to the governance layer. It also covers day-to-day
operations — what happens when each governance check fires on real work,
and what the team does in response.

Steps 1 through 6 (deploying the layer itself) are in deploying-projops.md.

Plain-language guide to every term: vocabulary.md

---

## Before you start Step 7

Have ready before opening any file:
- The repository name in org/repo format
- The authorising decision reference in ENTITY-YEAR-SEQUENCE-CHECKSUM format
  (example: ACME-2026-0042-A3F9B2C1)
- Names of the four authority roles:
    Project sponsor          @github-username
    Scope change approver    @github-username
    Compliance exception     @github-username (different from sponsor)
    Rollback authority       @github-username
- Uncertainty classification: aleatory, epistemic, or ambiguity
- Compliance obligation level: high, medium, or low
- Rate of change tolerance: high, medium, or low
- Delivery pattern and framework decision

If any of these cannot be answered, the initiative may not be ready
to begin. The governance layer will surface this at Step 7f.

---

## Step 7a — Create GOVERNANCE.md

Create GOVERNANCE.md at the repository root.

Download the template:
  curl -o GOVERNANCE.md \
    https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/templates/GOVERNANCE.md

Or create directly in GitHub:
  Your repository > Add file > Create new file > name it GOVERNANCE.md
  Paste from GovernanceOperations/projops/templates/GOVERNANCE.md

Complete every field. Example for a Tier 2 iterative initiative:

---
# Project Governance Record

<!-- govops-schema: governance-md v0.9 -->

## Governance Scope

Governance substrate: repository
Canonical repository: your-org/your-repo
GitHub Project board (if applicable):
Additional repositories:

## What This Project Is

Initiative title: Customer invoice reconciliation automation
Repository: your-org/invoice-reconciliation
Authorising decision reference: ACME-2026-0042-A3F9B2C1
Decision type: conditional
Governance tier: 2
GovOps layer version: 0.9

## What This Project Is Trying to Achieve

Problem being solved:
Manual invoice reconciliation takes 14 days and generates 40% of all
finance team service complaints.

Outcome targeted:
Reconciliation completes within 2 working days. Service complaints
related to reconciliation fall below 10%.

Constraints accepted:
Must integrate with existing ERP without schema changes.
Budget ceiling 85,000. Must be live before financial year end.

How success will be measured:
Reconciliation cycle time under 2 days by month 3 post go-live.
Service complaint rate measured quarterly.

## Uncertainty Classification

Primary classification: epistemic
Why: ERP integration interface documented but not tested at actual data
volume. Data quality assumed good but not independently verified.
Both are findable through structured discovery before design begins.

## Who Has Authority

Project sponsor: @alice-smith
Approves scope changes: @alice-smith
Authorises compliance exceptions: @bob-jones
Authority to close or pause: @alice-smith
Rollback authority: @carol-lee

## Compliance Obligations

| Obligation | Source | What the project must do | Owner |
|---|---|---|---|
| GDPR data minimisation | Data protection law | Process only invoice references and amounts | @bob-jones |
| Financial audit trail | Internal audit policy | Every automated match produces a traceable record | @carol-lee |

## Delivery System

Delivery pattern: iterative
Execution framework: scrum
Governance cadence: Monthly milestone review at end of every second sprint
Selection rationale:
  1. Uncertainty: epistemic -> iterative indicated
  2. Compliance: medium -> verification built into sprint exit criteria
  3. Change tolerance: medium -> some flexibility while ERP details established
  4. Mandate: none -> no departure required
Approved by: @alice-smith
Departure from standard: no

## Current Status

Status: active
Initiative opened: 2026-04-28
Expected completion: 2026-10-31
Last updated: 2026-04-28 by @alice-smith

<!-- govops-end -->
---

Fields validated by governance-init-check:
  - Governance substrate (must be: repository, github-project, or both)
  - Canonical repository (must be: org/repo format)
  - Authorising decision reference (must match: ENTITY-YEAR-SEQUENCE-CHECKSUM)
  - Decision type (must be: reversible, conditional, or irreversible)
  - Governance tier (must be: 1, 2, 3, or 4)
  - Primary classification (must be: aleatory, epistemic, or ambiguity)
  - Delivery pattern (must be: staged-gate, iterative, parallel-track, or hybrid)
  - Execution framework (must be: scrum, kanban, prince2, safe, internal, or hybrid)
  - Governance cadence (must be non-empty)
  - All authority fields (must name valid GitHub usernames)

If classification is ambiguity: the Ambiguity resolution reference field
must be populated with a link to a completed alignment record before
the governance check will pass. The layer blocks execution on ambiguity
until the objective is agreed. This is intentional.

---

## Step 7b — Create govops.yml

Create govops.yml at the repository root.

Download and edit:
  curl -o govops.yml \
    https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/govops.yml.example

Minimum required fields:

  govops_version: "0.9"

  governance_scope:
    substrate: "repository"
    canonical_repository: "your-org/your-repo"
    github_project_url: ""
    additional_repositories: []

  initiative:
    tier: 2
    decision_class: "conditional"
    ial_ceiling: 2
    ial_required_for_merge: 2
    ial_required_for_deploy: 2
    rollback_authority: "@carol-lee"
    production_environment_pattern: "production|prod|release"

  ledger:
    endpoint: "https://YOUR-NGROK-URL/v1/events"
    auth_secret: "LEDGER_API_KEY"
    chain_id: "your-org-initiative-name-2026"
    signature_algorithm: "dilithium3"
    alert_channel: ""

  portfolio:
    org_label: "govops-tier-2"
    health_report_schedule: "0 9 * * 1"
    portfolio_repo: ""

  governance:
    require_uncertainty_classification: true
    require_milestone_checkpoints: true
    require_pir: true
    require_asset_handover: true
    exception_expiry_days: 14
    exception_pattern_threshold: 3

  delivery_system:
    pattern: "iterative"
    framework: "scrum"
    framework_description: ""
    governance_cadence: "Monthly milestone review at end of every second sprint"
    org_mandate: ""
    deviation_from_mandate: false
    deviation_approved_by: ""
    deviation_rationale: ""

  runners:
    default: "ubuntu-latest"

Two fields require your specific values:
  ledger.endpoint    : your HTTPS evaluation endpoint URL from Step 2
  ledger.chain_id    : a unique identifier for this initiative's records
                       Recommended format: org-initiative-name-year
                       Example: acme-invoice-reconciliation-2026
                       Do not change this once set.

If using a GitHub Project board (substrate: github-project or both):
  Set governance_scope.github_project_url to the board URL:
  Format: https://github.com/orgs/YOUR-ORG/projects/N

---

## Step 7c — Add the caller workflow

Create .github/workflows/govops.yml:

  name: GovOps Governance Layer

  on:
    push:
      branches: [main, release/**]
    pull_request:
      types: [opened, synchronize, ready_for_review, closed]
    pull_request_review:
      types: [submitted]
    milestone:
      types: [closed]
    issues:
      types: [labeled]
    deployment:
    deployment_review:
    workflow_run:
      workflows: ["*"]
      types: [completed]
    check_run:
      types: [completed]
    projects_v2:
      types: [closed]
    workflow_dispatch:
    schedule:
      - cron: '0 9 * * 1'

  jobs:
    govops:
      uses: GovernanceOperations/projops/.github/workflows/govops-suite.yml@v0.9
      with:
        config-path: govops.yml
      secrets: inherit

---

## Step 7d — Copy the issue templates

  mkdir -p .github/ISSUE_TEMPLATE

  for template in govops-exception govops-milestone-review \
                  govops-operational-readiness govops-pir; do
    curl -o .github/ISSUE_TEMPLATE/${template}.md \
      https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/.github/ISSUE_TEMPLATE/${template}.md
  done

  curl -o ASSET-HANDOVER.md \
    https://raw.githubusercontent.com/GovernanceOperations/projops/v0.9/templates/ASSET-HANDOVER.md

---

## Step 7e — Create the required labels

  gh auth login

  gh label create "govops-exception"       --color "d73a4a" --description "Governance exception record"       --repo your-org/your-repo
  gh label create "milestone-checkpoint"   --color "e4e669" --description "Milestone governance review"       --repo your-org/your-repo
  gh label create "operational-readiness"  --color "0075ca" --description "Operational readiness checklist"   --repo your-org/your-repo
  gh label create "pir"                    --color "0e8a16" --description "Post-project review"                --repo your-org/your-repo
  gh label create "governance-scope"       --color "5319e7" --description "Issue carries governance obligations" --repo your-org/your-repo
  gh label create "rollback-tested"        --color "006b75" --description "Rollback procedure has been tested" --repo your-org/your-repo
  gh label create "portfolio-health"       --color "e4e669" --description "Portfolio health report"            --repo your-org/your-repo

---

## Step 7f — Commit everything

  git add GOVERNANCE.md govops.yml ASSET-HANDOVER.md \
          .github/workflows/govops.yml \
          .github/ISSUE_TEMPLATE/
  git commit -m "Connect to GovOps governance layer v0.9

  - GOVERNANCE.md: initiative governance identity record
  - govops.yml: layer configuration
  - .github/workflows/govops.yml: governance check caller
  - Issue templates: exception, milestone review, OR checklist, PIR
  - ASSET-HANDOVER.md: asset handover template"
  git push origin main

After pushing, go to your repository Actions tab. The GovOps Governance
Layer workflow should appear and begin running governance-init-check.
Watch it complete.

If it passes: proceed to Step 7g.

If it fails, read the message carefully:

  GOVERNANCE.md not found
    File is not at repository root or filename has wrong case.

  Missing field: Authorising decision reference
    Field is blank or comment text was not replaced.

  Decision Contract reference format invalid
    Reference must match ENTITY-YEAR-SEQUENCE-CHECKSUM.
    Example: ACME-2026-0042-A3F9B2C1

  Delivery pattern must be one of...
    Value has a typo or extra whitespace.

  govops.yml not found
    File is not at repository root.

  Governance substrate must be...
    substrate field blank or contains an invalid value.

  Uncertainty classified as ambiguity — ambiguity_resolution_ref missing
    The alignment process has not been completed or the URL was left blank.
    If alignment is not complete, the block is correct.

---

## Step 7g — Set up branch protection

Your repository > Settings > Branches > Add branch protection rule

Branch name pattern: main

  [x] Require a pull request before merging
      [x] Require approvals: 1
      [x] Dismiss stale pull request approvals when new commits are pushed

  [x] Require status checks to pass before merging
      [x] Require branches to be up to date before merging
      Search and add required status checks:
        governance-init-check / governance-init-check
        ial-ceiling-check / ial-ceiling-check

  [x] Require conversation resolution before merging
  [x] Do not allow bypassing the above settings

Status check names only appear in the search box after they have run
at least once. If they do not appear yet: open a draft pull request,
wait for the checks to run, then come back to add them.

Repeat for release/** if your project uses release branches.

---

## Step 7h — Set up deployment environment

Your repository > Settings > Environments > New environment

1. Name: production
2. Enable Required reviewers — add @alice-smith (or your deployment authority)
3. Click Save protection rules

This activates operational-readiness-gate and rollback-readiness-gate
for deployments targeting this environment.

---

## Step 7i — Add portfolio label

Your repository > Settings > General > Topics
Add: govops-tier-2 (matching portfolio.org_label in govops.yml)
Click Save changes

---

## Step 7j — End-to-end verification

Run all four tests before declaring the connection complete.

Test 1 — Pull request governance:
1. Create a branch, make any small change, open a pull request
2. Confirm governance-init-check and ial-ceiling-check appear as
   required status checks on the PR
3. Check your terminal — a governance record should appear
4. Have an IAL register member approve the PR
5. Merge the PR
6. Check terminal — a second record of type execute should appear

Test 2 — Milestone checkpoint:
1. Create a milestone: Issues > Milestones > New milestone > Sprint 1
2. Close the milestone immediately
3. The workflow should re-open it with a message
4. Create a milestone review issue using the GovOps Milestone
   Governance Checkpoint template
5. Fill it in — check the Continuation confirmed box
6. Close the issue
7. Close the milestone again — it should now close successfully
8. Check terminal — a record of type decision should appear

Test 3 — Exception logging:
1. Repository Settings > Branches > Edit your branch protection rule
2. Enable Allow specified actors to bypass required pull requests
   and add yourself temporarily
3. Save the rule
4. Push a commit directly to main — bypassing protection
5. Check Issues — a govops-exception labelled issue should appear
6. Remove the bypass rule
7. Fill in the exception issue rationale and close it

Test 4 — Ledger audit:
  curl https://xxxx.ngrok-free.app/v1/events \
    -H "Authorization: Bearer your-eval-key" | python3 -m json.tool

Should return an array of events covering all actions in Tests 1-3.
Count them — at minimum six records should be present.

If Test 2 projects_v2 trigger is also needed (GitHub Project board):
Create a test project board, link issues from your repository,
close the board. The pir-trigger should activate and block closure.
Confirm the Ledger Bridge App is installed at organisation level —
projects_v2 events require this.

---

## Full connection status after Step 7

  GOVERNANCE.md complete and valid           CONFIRMED
  govops.yml configured                      CONFIRMED
  Caller workflow active                     CONFIRMED
  Issue templates installed                  CONFIRMED
  Labels created                             CONFIRMED
  Branch protection enforcing checks         CONFIRMED
  Production environment gated               CONFIRMED
  Portfolio label set                        CONFIRMED
  Pull request governance verified           CONFIRMED
  Milestone checkpoint verified              CONFIRMED
  Exception logging verified                 CONFIRMED
  Permanent record receiving events          CONFIRMED

---

## Day-to-Day Operations

After the connection is verified, governance runs automatically.
Most days nothing unusual happens and the layer is invisible.
It becomes visible in four situations.

---

### Situation 1 — A check fails on a pull request

What the team sees: a red status check, merge button greyed out.

What to do: click Details next to the failing check and read the
message. It is specific. Every failure has a specific resolution.

Common failures after initial connection:

  governance-init-check / GOVERNANCE.md missing required fields
    Someone edited GOVERNANCE.md and blanked a required field.
    Find the field, restore a valid value, commit. Check re-runs.

  ial-ceiling-check / Reviewer has IAL 1 but IAL 2 required
    Approver is not in the IAL register or their entry expired.
    Add or update their entry in the IAL register and re-request review,
    or have a different IAL 2 approver review instead.

  governance-init-check / Decision Contract reference format invalid
    Correct the reference in GOVERNANCE.md to ENTITY-YEAR-SEQUENCE-CHECKSUM.

  governance-init-check / Uncertainty classified as ambiguity
    The initiative objective is contested. The block is correct.
    Either complete the alignment process and add the resolution URL,
    or reclassify the uncertainty if ambiguity was an error.

What not to do: do not use an admin bypass unless there is a genuine
operational emergency. An admin bypass creates an automatic exception
record. Use it when necessary, not when inconvenient.

---

### Situation 2 — An exception is logged

What the team sees: a new issue appears in the repository Issues list,
labelled govops-exception, titled GovOps Exception: [requirement bypassed].
It is assigned to the person who performed the bypass.

This happens automatically. The issue is pre-populated with bypass details.

What the assigned person does:
1. Open the exception issue
2. Fill in the Rationale section — the operational reason the bypass
   was necessary. Urgency alone is not a rationale.
3. Fill in the Remediation section — what will restore the requirement
4. Confirm the expiry date is accurate (default 14 days)
5. Leave it open until remediation is complete
6. When remediation is done, check all three closure criteria and close

What the governance authority does:
Review within 24 hours. If the rationale is not adequate, engage directly.
An exception with no rationale is a governance gap.

Pattern monitoring:
If three or more exceptions of the same type accumulate, the portfolio
health report flags it as a pattern. The response is a standards review —
ask whether the standard is correctly calibrated. Not enforcement escalation.

---

### Situation 3 — A milestone is closing

What the team sees: the milestone re-opens with the message:
"Milestone cannot close without a completed governance checkpoint."

What to do:
1. Issues > New issue > GovOps Milestone Governance Checkpoint template
2. Fill in all sections:
   - Which milestone: title and number
   - Is it still on track: honest answer
   - Risk status: any risks that have materialised or grown
   - Delivery system review: is the approach still appropriate?
   - Governance record currency: is GOVERNANCE.md up to date?
   - Decision or continuation: one of the two checkboxes must be checked
3. Close the issue
4. Close the milestone again — it will now close

What makes a good milestone review:

The decision or continuation must be specific enough that someone
reading it in six months understands what was confirmed and on what basis.

Acceptable:
  "Confirmed: this initiative continues within its original mandate,
  within its accepted risk levels, and within the authority granted
  at initiation. No scope change. No exceptions granted.
  Reviewed by @alice-smith on 2026-05-15."

Not acceptable:
  "Looks good, proceeding."

A milestone review consistently producing empty continuations signals
the review is not functioning as a genuine accountability checkpoint.
Raise it at the next governance retrospective.

Delivery system review signals to watch for:
  governance-theatre     : reviews produce no decisions; same problems
                           appear repeatedly without resolution
  compliance-building-up : compliance steps deferred from cycle exit criteria
  scope-instability      : requirements changing under a locked-scope approach
  cadence-misalignment   : delivery cycles much more frequent than governance reviews

When any signal appears, raise a formal delivery system review at the
checkpoint — record it as a decision regardless of outcome.

---

### Situation 4 — A deployment is targeting production

What the team sees: deployment blocked.
operational-readiness-gate: "No operational readiness checklist found."

What to do before every production deployment:
1. Issues > New issue > GovOps Operational Readiness Checklist template
2. Fill in deployment details
3. Work through each condition with the responsible person:

   Process readiness:
   The process owner confirms processes are documented and the people
   who will use them have walked through them — not just read them.

   Technology readiness:
   The system responsible person confirms stability in a
   production-equivalent environment and that support documentation
   is written and accessible.

   People readiness:
   The workforce responsible person confirms training delivered and
   comprehension confirmed — not just attendance recorded.

   Compliance readiness:
   The compliance responsible person confirms all regulatory approvals
   or notifications obtained and documented.

   Rollback readiness:
   The rollback authority confirms the procedure exists, has been tested,
   and they are available and prepared to invoke it.

4. Each person checks their condition box in the issue
5. All five checked: close the issue
6. Re-trigger the deployment

For irreversible deployments — rollback readiness gate also activates:
  A .github/workflows/rollback*.yml file must exist in the repository
  A rollback-tested labelled issue must be closed within the last 30 days
  rollback_authority must be named in govops.yml

The gate posts an IAL advisory for irreversible deployments noting
GitHub's IAL ceiling is 2. If the decision warrants IAL 3 or 4,
follow docs/ial-out-of-band.md before proceeding.

---

## Regular rhythms

Weekly — automatic (every Monday 09:00 UTC):
The portfolio health aggregator runs. It scans all repositories
labelled govops-tier-2 and produces a health report.

Review the report for:
  Open exceptions past expiry   : need immediate attention
  Repositories missing GOVERNANCE.md : ungoverned
  Milestone checkpoint completion below 100% : milestones closing without review
  Exception patterns : three or more of the same type signals a standards review

Monthly:
  Initiate milestone reviews proactively before milestones close.
  Review the last four portfolio health reports as a set — trends are
  invisible in a single report.
  Check the IAL register for membership changes.

Quarterly — governance review:
  Are governance tiers correctly calibrated?
  Are milestone reviews producing genuine decisions or empty continuations?
  Are exception patterns telling us something about the standards?
  Is the delivery system governance working?
  Is the learning loop closing?

Output: a governance improvement backlog submitted through GOS Section 24.
Even two or three items per quarter compound over time into a governance
framework that fits operational reality.

---

## Project closure

When an initiative completes, closes, or is formally ended:

1. Create a PIR issue using the GovOps Post-Project Review template
2. Complete the review with the team, sponsor, and at least one person
   from the receiving operational team
3. Submit improvement suggestions to the governance improvement process
   — the PIR issue cannot close without this link populated
4. Complete ASSET-HANDOVER.md:
   - Name an owner for every output of the initiative
   - Get each owner to confirm receipt by checking their box
5. Close the PIR issue
6. Archive the repository (or close the GitHub Project board)
   Both PIR and asset handover must pass before closure is allowed

---

## When something unexpected happens

False positive — a check fails on something that should have passed:
  Do not bypass. Open an issue in GovernanceOperations/projops describing
  what triggered it, what was expected, what actually happened.
  This is a bug report against the governance layer. Treat it as such.

Check consistently bypassed by the team:
  Governance signal, not a compliance failure.
  Engage to understand why. Raise a standards review if warranted.
  Address through governance, not enforcement pressure.

Evaluation ledger down because ngrok restarted:
  Update LEDGER_ENDPOINT secret with new ngrok URL.
  Note which events were missed during the outage.
  This is the operational argument for moving to a stable
  permanent record endpoint before production.

Named authority leaves the organisation:
  Update GOVERNANCE.md to replace their role.
  Update govops.yml if they are rollback_authority.
  Remove from IAL register.
  Check ASSET-HANDOVER.md for assets they own — reassign immediately.
  The Ledger Bridge alert will have fired — confirm it was received.

---

## Known gaps to address before production

  1. Ledger Bridge not implemented
     Impact: continuous event stream missing, workflow-level records only
     Address: before production governance begins

  2. Evaluation ledger not tamper-evident or immutable
     Impact: evaluation records cannot be used as production governance records
     Address: provision production Immutable Ledger before first production decision

  3. Out-of-band IAL hash validation not automated
     Impact: IAL 3/4 approvals require manual verification
     Address: before any initiative requires IAL 3+ decisions

  4. govops-config.json schema not fully wired
     Impact: govops.yml validated structurally, not against full JSON schema
     Address: v0.9.1 patch, low urgency

---

## Quick reference — what triggers what

  Open first PR                  : governance-init-check
  Merge any PR                   : pr-governance-event (permanent record)
  PR review approval             : ial-ceiling-check
  Create or edit milestone       : uncertainty-classification-check
  Close a milestone              : milestone-governance-checkpoint (blocks)
  Deploy to production           : operational-readiness-gate (blocks)
  Deploy irreversible change     : rollback-readiness-gate (blocks)
  Check overridden by admin      : exception-logger (creates exception issue)
  Every Monday 09:00 UTC         : portfolio-health-aggregator
  Archive repository             : pir-trigger + asset-handover-validator (blocks)
  Close GitHub Project board     : pir-trigger + asset-handover-validator (blocks)

---

Reference:
  deploying-projops.md        Steps 1-6 — layer deployment
  configuration-reference.md  All govops.yml fields
  vocabulary.md               Plain-language term guide
  ial-out-of-band.md          IAL 3 and 4 approval process
  git-platform-coverage.md    Non-GitHub git hosts
