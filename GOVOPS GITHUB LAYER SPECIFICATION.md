# GovOps GitHub Layer — Repository Specification

> **Normative relationship:** This specification defines the GovOps GitHub Layer — the callable governance repository that implements Section 17.11 (Project Operations Discipline) of the *Governance Operations Specification v0.9* for GitHub as an operational substrate. It is the reference platform substrate implementation satisfying the nine Platform Substrate Governance properties defined in GOS §17.11. Where any element of this specification appears to conflict with the Governance Operations Specification, the Governance Operations Specification governs. This document uses the notation **[GOS §X.Y]** to identify corresponding sections of the specification.

> **Status:** v0.1 — First Release Draft. Versioned independently of the Governance Operations Specification.

---

## Table of Contents

1. Purpose and Scope
2. Repository Structure
3. Quick Start — Calling the Layer
4. Governance Initiation Requirements
5. Workflow Specifications
   - 5.1 governance-init-check
   - 5.2 uncertainty-classification-check
   - 5.3 pr-governance-event
   - 5.4 ial-ceiling-check
   - 5.5 operational-readiness-gate
   - 5.6 rollback-readiness-gate
   - 5.7 milestone-governance-checkpoint
   - 5.8 exception-logger
   - 5.9 portfolio-health-aggregator
   - 5.10 pir-trigger
   - 5.11 asset-handover-validator
6. Governance Template Specifications
   - 6.1 GOVERNANCE.md
   - 6.2 EXCEPTION.md
   - 6.3 MILESTONE-REVIEW.md
   - 6.4 PIR.md
   - 6.5 ASSET-HANDOVER.md
7. Immutable Ledger Bridge Specification
8. Configuration Reference
9. Identity Assurance Level Mapping
10. Compliance Verification
11. Versioning and Maintenance
12. Glossary of GitHub-to-GovOps Mappings

---

## 1. Purpose and Scope

The GovOps GitHub Layer is a callable GitHub repository that provides a governance layer for any git repository operating on GitHub as its project delivery substrate. It implements the nine governance gaps identified between GitHub's native capabilities and the requirements of the Project Operations Discipline **[GOS §17.11]**.

**What this layer does:**

It converts GitHub's delivery infrastructure — repositories, pull requests, milestones, deployments, workflows, issues — into governed entities that satisfy the governance requirements of the Governance Operations Specification. It does not replace GitHub's native features. It governs them — adding the accountability structures, audit records, identity verification, and lifecycle controls that GitHub does not natively provide.

**What this layer does not do:**

It does not substitute for the Governance Operations Centre **[GOS §6]**, the Memory Layer **[GOS §6, Layer 6]**, or any other component of the governance architecture. It is the bridge between GitHub and those components — the mechanism by which activity on GitHub produces the Governance Events, the initiative records, and the operational readiness evidence that the specification requires.

**Scope:**

- Any GitHub repository representing a governed initiative — public, internal, or private
- Any GitHub organisation whose repositories contain governed project work
- Any git client whose changes are pushed to a GitHub-hosted repository covered by this layer
- GitHub Actions workflows that represent Act Layer execution under **[GOS §6, Layer 5]**

**What is not in scope:**

- Non-GitHub git hosts (GitLab, Bitbucket, Azure DevOps) — this specification is GitHub-specific; the governance principles apply universally but the implementation is GitHub-native
- Infrastructure state reconciliation (ArgoCD, Flux) — this layer governs the initiative, not the infrastructure deployment
- Technical policy enforcement (OPA, Checkov, Sentinel) — this layer governs governance obligations, not technical compliance

---

## 2. Repository Structure

The GovOps GitHub Layer is hosted as a public GitHub repository at:

```
https://github.com/GovernanceOperations/projops
```

All consuming repositories reference workflows from this repository by version tag. The repository structure is:

```
projops/
├── .github/
│   └── workflows/
│       ├── governance-init-check.yml         # Workflow 5.1
│       ├── uncertainty-classification-check.yml  # Workflow 5.2
│       ├── pr-governance-event.yml           # Workflow 5.3
│       ├── ial-ceiling-check.yml             # Workflow 5.4
│       ├── operational-readiness-gate.yml    # Workflow 5.5
│       ├── rollback-readiness-gate.yml       # Workflow 5.6
│       ├── milestone-governance-checkpoint.yml # Workflow 5.7
│       ├── exception-logger.yml              # Workflow 5.8
│       ├── portfolio-health-aggregator.yml   # Workflow 5.9
│       ├── pir-trigger.yml                   # Workflow 5.10
│       ├── asset-handover-validator.yml      # Workflow 5.11
│       └── govops-suite.yml                  # Composite caller (all 11)
├── templates/
│   ├── GOVERNANCE.md                         # Template 6.1
│   ├── EXCEPTION.md                          # Template 6.2
│   ├── MILESTONE-REVIEW.md                   # Template 6.3
│   ├── PIR.md                                # Template 6.4
│   └── ASSET-HANDOVER.md                     # Template 6.5
├── bridge/
│   ├── README.md                             # Bridge architecture
│   ├── app.yml                               # GitHub App manifest
│   ├── webhook-handler/                      # Event-to-Governance-Event translation
│   └── ledger-client/                        # Immutable Ledger write interface
├── schemas/
│   ├── governance-event.json                 # Governance Event JSON Schema
│   ├── governance-md.json                    # GOVERNANCE.md validation schema
│   └── govops-config.json                    # govops.yml configuration schema
├── govops.yml.example                        # Reference configuration file
├── GOVERNANCE.md                             # This repository's own governance record
└── README.md
```

---

## 3. Quick Start — Calling the Layer

**Step 1: Copy the GOVERNANCE.md template to your repository root**

Every governed repository must have a `GOVERNANCE.md` at root before any initiative work begins. Copy from `templates/GOVERNANCE.md` and complete all required fields. Incomplete `GOVERNANCE.md` files block the first merge to a protected branch.

**Step 2: Add a govops.yml configuration file to your repository root**

```yaml
# govops.yml — GovOps GitHub Layer configuration
govops_version: "0.9"
initiative:
  tier: 2                          # Operational tier 1-4 [GOS §30]
  decision_class: "conditional"   # reversible | conditional | irreversible
  ial_ceiling: 2                   # Maximum achievable IAL on this platform [GOS §15]
  ial_required_for_merge: 2        # Minimum IAL for protected branch merges
  ial_required_for_deploy: 2       # Minimum IAL for production deployments
ledger:
  endpoint: "https://your-ledger-endpoint/events"  # Immutable Ledger write endpoint
  auth_secret: "LEDGER_API_KEY"   # GitHub secret name containing ledger credentials
portfolio:
  org_label: "govops-tier-2"       # Label applied to all repos in this initiative's portfolio
  health_report_schedule: "0 9 * * 1"  # Weekly Monday 09:00 UTC
```

**Step 3: Add the caller workflow to your repository**

Create `.github/workflows/govops.yml` in your repository:

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
    - cron: '0 9 * * 1'   # Portfolio health aggregation

jobs:
  govops:
    uses: GovernanceOperations/projops/.github/workflows/govops-suite.yml@v0.9
    with:
      config-path: govops.yml
    secrets: inherit
```

That is the complete integration for a consuming repository. All eleven governance workflows activate from this single caller based on the triggering event type. The consuming repository does not need to configure or maintain individual workflows.

**Step 4: Install the Ledger Bridge GitHub App**

Install the GovOps Ledger Bridge GitHub App (Section 7) on your organisation or repository. The bridge converts GitHub webhook events into Governance Events written to your Immutable Ledger endpoint.

---

## 4. Governance Initiation Requirements

Before the first commit representing initiative work is merged to a protected branch, the following must be true:

**4.1 GOVERNANCE.md is present and complete**

A valid `GOVERNANCE.md` must exist at the repository root. "Valid" means all required fields in Template 6.1 are populated and the file passes the `GovernanceOperations/projops/schemas/governance-md.json` schema validation. A repository with an absent or invalid `GOVERNANCE.md` is not a governed initiative — it is an ungoverned repository that has chosen a filename.

**4.2 govops.yml is present and valid**

A valid `govops.yml` must exist at the repository root, passing the `GovernanceOperations/projops/schemas/govops-config.json` schema validation.

**4.3 Decision Contract reference is resolvable**

The `decision_contract_ref` field in `GOVERNANCE.md` must reference a valid, approved Decision Contract record. The governance-init-check workflow validates that this reference is non-empty and follows the required format. The consuming organisation is responsible for ensuring the referenced Decision Contract exists in its Memory Layer.

**4.4 Uncertainty classification is recorded**

The `uncertainty_classification` field in `GOVERNANCE.md` must contain one of three valid values: `aleatory`, `epistemic`, or `ambiguity`. If the classification is `ambiguity`, the workflow blocks the repository from accepting initiative work until the `ambiguity_resolution_ref` field is also populated with a link to the completed alignment record.

**Why 4.4 is a hard gate:** An initiative classified as `ambiguity` has an objective that is contested or undefined. Constitutional Condition III **[GOS §3]** cannot be satisfied for any decision resting on an ambiguous objective. Proceeding to execution without resolving ambiguity is not a governed initiative. The gate enforces this structurally rather than relying on practitioner discipline.

---

## 5. Workflow Specifications

Each workflow is a reusable GitHub Actions workflow called from the composite `govops-suite.yml` or directly by consuming repositories that require specific governance gates.

All workflows share the following behaviour:
- They run on GitHub-hosted runners unless the consuming organisation overrides with self-hosted runners via `govops.yml`
- On pass: they write a Governance Event to the configured Immutable Ledger endpoint via the bridge, and record a pass status to the consuming repository's governance record
- On fail: they block the triggering action (merge, deployment, milestone close, archive), write a Governance Event of type `alert`, and post a structured failure reason to the PR, deployment, or issue that triggered them
- All Governance Events conform to the six-field structure of Section 6.6 **[GOS §6.6]**: identity, timestamp, type, payload, signature, integrity_reference

---

### 5.1 governance-init-check

**Purpose:** Validates that a repository is a properly initialised governed initiative before any initiative work is accepted.

**Triggers:** `push` to default branch (first push only), `pull_request` opened (first PR to a protected branch), `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` in the calling repository |
| `governance-md-path` | string | no | Path to `GOVERNANCE.md` (default: `GOVERNANCE.md`) |

**Logic:**

```
1. Check GOVERNANCE.md exists at specified path
   → FAIL if absent: "GOVERNANCE.md not found. Repository is not a governed initiative."

2. Validate GOVERNANCE.md against GovernanceOperations/projops/schemas/governance-md.json
   → FAIL if invalid: list all missing or malformed required fields

3. Check decision_contract_ref is non-empty and matches format:
   ^[A-Z]{2,}-[0-9]{4}-[0-9]{4}-[A-Z0-9]{8}$
   (format: ENTITY-YEAR-SEQUENCE-CHECKSUM)
   → FAIL if absent or malformed: "Decision Contract reference is missing or invalid."

4. Check uncertainty_classification is one of: aleatory | epistemic | ambiguity
   → FAIL if absent or invalid value

5. If uncertainty_classification == ambiguity:
   Check ambiguity_resolution_ref is non-empty
   → FAIL if absent: "Uncertainty classified as ambiguity. Initiative cannot proceed
     until ambiguity_resolution_ref references a completed alignment record. [GOS §17.11]"

6. Check delivery_system fields in GOVERNANCE.md:
   a. delivery_pattern is present and one of:
      staged-gate | iterative | parallel-track | hybrid
      → FAIL if absent or invalid: "Delivery pattern is required. [GOS §17.11, §10.5]"

   b. execution_framework is present and one of:
      scrum | kanban | prince2 | safe | internal | hybrid
      → FAIL if absent or invalid

   c. If execution_framework == hybrid OR internal:
      framework_description is non-empty
      → FAIL if absent: "Framework description is required for hybrid or internal frameworks."

   d. governance_cadence is non-empty
      → FAIL if absent: "Governance cadence is required. Specify how delivery cycles
        connect to milestone reviews."

   e. selection_rationale is non-empty
      → FAIL if absent: "Selection rationale is required. Map the selection to the
        four criteria: uncertainty classification, compliance density, rate of change
        tolerance, organisational mandate. [Section 10.5]"

   f. approved_by is a valid organisation member
      → FAIL if absent or not an organisation member

   g. If deviation_from_mandate == true:
      Check deviation_rationale is non-empty
      Check deviation_approved_by is a valid organisation member
      → FAIL if either absent: "Deviation from organisational mandate requires
        rationale and approval. [Section 10.5]"

   h. Consistency check — uncertainty vs. pattern:
      If uncertainty_classification == ambiguity AND delivery_pattern != none:
      → FAIL: "Delivery pattern cannot be set for an ambiguity-classified initiative.
        Resolve the ambiguity classification first."

      If uncertainty_classification == aleatory AND delivery_pattern == staged-gate:
      → WARN: "Staged-gate delivery under aleatory uncertainty concentrates
        irreducible risk at gates. Confirm this selection is intentional and
        document the rationale explicitly."

7. Check govops.yml exists and is valid against govops-config.json schema
   → FAIL if absent or invalid

8. Validate delivery_system block in govops.yml matches GOVERNANCE.md:
   govops.yml pattern must match GOVERNANCE.md delivery_pattern
   govops.yml framework must match GOVERNANCE.md execution_framework
   → FAIL if mismatch: "govops.yml delivery_system does not match GOVERNANCE.md.
     Both records must reflect the same governed delivery system."

9. All checks pass:
   → PASS: write Governance Event (type: observation)
```

**Governance Event produced:**
```json
{
  "type": "observation",
  "payload": {
    "check": "governance-init",
    "result": "pass | fail",
    "repository": "<repo-full-name>",
    "decision_contract_ref": "<value>",
    "uncertainty_classification": "<value>",
    "tier": "<value>",
    "decision_class": "<value>",
    "delivery_system": {
      "pattern": "<value>",
      "framework": "<value>",
      "governance_cadence": "<value>",
      "deviation_from_mandate": true | false,
      "deviation_approved_by": "<username or null>"
    }
  }
}
```

**Failure behaviour:** Blocks merge to protected branch. Posts failure reason as PR check with link to `templates/GOVERNANCE.md` for remediation.

---

### 5.2 uncertainty-classification-check

**Purpose:** Validates that every milestone and every issue labelled `governance-scope` carries an uncertainty classification before work begins.

**Triggers:** `milestones` created or edited, `issues` opened or labelled with `governance-scope`, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `issue-number` | number | no | Issue number (when triggered by issue event) |
| `milestone-number` | number | no | Milestone number (when triggered by milestone event) |

**Logic:**

```
1. On milestone creation or edit:
   Check milestone body contains one of the uncertainty classification markers:
   - <!-- uncertainty: aleatory -->
   - <!-- uncertainty: epistemic -->
   - <!-- uncertainty: ambiguity -->
   → WARN if absent: posts comment requesting classification.
     Milestone is not blocked — classification is required before first associated
     issue moves to "In Progress" status.

2. On issue labelled governance-scope:
   Check issue body contains uncertainty classification marker
   OR issue's parent milestone has a classification
   → WARN if absent: posts comment with classification guidance

3. If classification is ambiguity (at issue or milestone level):
   Check for <!-- ambiguity-resolution: <url> --> marker
   → BLOCK transition to "In Progress" if absent

4. Write classification to repository governance record via bridge
   → Governance Event (type: record)
```

**Governance Event produced:**
```json
{
  "type": "record",
  "payload": {
    "check": "uncertainty-classification",
    "entity_type": "milestone | issue",
    "entity_number": "<number>",
    "classification": "aleatory | epistemic | ambiguity",
    "resolution_ref": "<url or null>"
  }
}
```

**Failure behaviour:** Warns on creation. Blocks status transition to In Progress if classification is absent. Does not block issue creation.

---

### 5.3 pr-governance-event

**Purpose:** Produces a Governance Event for every merged pull request, linking it to its Decision Contract and recording it in the Immutable Ledger as a governed execution act.

**Triggers:** `pull_request` closed where `merged == true`, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `pr-number` | number | yes | Pull request number |

**Logic:**

```
1. Extract from merged PR:
   - PR number, title, merge commit SHA
   - Merger identity (GitHub username)
   - Approver identities (all required reviewers who approved)
   - Associated milestone (if any)
   - Associated issues (linked via "Closes #N" or "Fixes #N")
   - Decision class from govops.yml
   - Labels

2. Determine operation_class from PR labels or govops.yml decision_class:
   - Label "irreversible" → irreversible
   - Label "conditional" → conditional
   - Default from govops.yml → as configured

3. Construct Governance Event (type: execute) with full payload

4. Post event to Immutable Ledger endpoint via bridge

5. Comment on merged PR with Governance Event ID for traceability
```

**Governance Event produced:**
```json
{
  "type": "execute",
  "payload": {
    "pr_number": "<number>",
    "pr_title": "<title>",
    "merge_commit_sha": "<sha>",
    "merger": "<github_username>",
    "approvers": ["<username>", "..."],
    "operation_class": "reversible | conditional | irreversible",
    "decision_contract_ref": "<from GOVERNANCE.md>",
    "milestone_ref": "<milestone_number or null>",
    "issues_closed": ["<issue_number>", "..."],
    "repository": "<repo-full-name>",
    "branch": "<target_branch>"
  }
}
```

**Failure behaviour:** The PR is already merged when this workflow runs. On failure to write the Governance Event, the workflow posts an alert to the PR and writes a Governance Event of type `alert` flagging that the execution act was not recorded. The consuming organisation's Oversight Body is notified via the configured alert channel.

---

### 5.4 ial-ceiling-check

**Purpose:** Validates that approver identities meet the Identity Assurance Level required for the decision class of the action being approved.

**Triggers:** `pull_request_review` submitted on a protected branch, `deployment_review` submitted, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `reviewer-username` | string | yes | GitHub username of the approving reviewer |
| `action-type` | string | yes | `merge` or `deploy` |

**Logic:**

```
1. Determine required IAL from govops.yml:
   - action-type == merge → ial_required_for_merge
   - action-type == deploy → ial_required_for_deploy

2. Look up reviewer-username in the organisation's IAL register:
   - The IAL register is a JSON file maintained in the consuming
     organisation's .github repository at:
     .github/govops/ial-register.json
   - Format: {"username": {"ial": 2, "last_verified": "2026-01-15"}}

3. If reviewer not in register → default IAL = 1

4. IAL platform ceiling check:
   This platform (GitHub) has an IAL ceiling of 2 (GitHub 2FA = IAL2).
   If required IAL > 2:
   → FAIL with message: "This action requires IAL <N>, which exceeds the
     platform ceiling of IAL 2 on GitHub. An out-of-band IAL <N> approval
     record is required. See GovernanceOperations/projops/docs/ial-out-of-band.md"

5. If reviewer IAL >= required IAL:
   → PASS: write Governance Event (type: approval)

6. If reviewer IAL < required IAL:
   → FAIL: block approval, post message identifying the IAL gap
```

**Governance Event produced:**
```json
{
  "type": "approval",
  "payload": {
    "reviewer": "<github_username>",
    "reviewer_ial": "<ial_value>",
    "required_ial": "<ial_value>",
    "action_type": "merge | deploy",
    "decision_class": "<value>",
    "result": "pass | fail",
    "out_of_band_required": true | false
  }
}
```

**Failure behaviour:** Blocks the approval from counting as a valid governance approval. Does not prevent GitHub from recording the review — it blocks the status check that makes the review sufficient for merge.

**Important note on IAL ceiling:** GitHub's authentication model supports a maximum of IAL 2 (organisation membership + enforced 2FA). IAL 3 and IAL 4 approvals required for Irreversible decisions **[GOS §14]** cannot be achieved natively on GitHub. This workflow surfaces that gap explicitly and requires an out-of-band approval record (a signed document or external identity provider attestation linked to the PR) rather than silently downgrading the requirement.

---

### 5.5 operational-readiness-gate

**Purpose:** Enforces the five operational readiness conditions as a required status check before any deployment to a production or production-equivalent environment proceeds.

**Triggers:** `deployment` event where environment matches configured production pattern, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `environment` | string | yes | Target environment name |
| `deployment-id` | string | yes | GitHub deployment ID |

**Logic:**

```
1. Check if environment matches production pattern from govops.yml
   (default pattern: production | prod | release)
   → If no match: SKIP (non-production deployments exempt)

2. Look for operational readiness checklist issue:
   Search open issues with label "operational-readiness" linked to
   the milestone or branch being deployed.
   → If absent: FAIL — "No operational readiness checklist found.
     Create an issue using templates/OPERATIONAL-READINESS.md
     before deploying to production."

3. Validate checklist completeness:
   Check that the issue body contains all five signed-off markers:
   - [x] Process readiness confirmed by @<username>
   - [x] Technology readiness confirmed by @<username>
   - [x] People readiness confirmed by @<username>
   - [x] Compliance readiness confirmed by @<username>
   - [x] Rollback readiness confirmed by @<username>
   → FAIL for each unsigned condition, listing which conditions remain

4. Validate that each confirming username appears in the
   organisation's authority register for their respective domain.

5. All five conditions signed by authorised confirmers:
   → PASS: write Governance Event (type: verification)
   → Record checklist issue URL in Governance Event payload
```

**Governance Event produced:**
```json
{
  "type": "verification",
  "payload": {
    "check": "operational-readiness",
    "environment": "<environment>",
    "deployment_id": "<id>",
    "conditions": {
      "process_readiness": {"confirmed": true | false, "confirmed_by": "<username>"},
      "technology_readiness": {"confirmed": true | false, "confirmed_by": "<username>"},
      "people_readiness": {"confirmed": true | false, "confirmed_by": "<username>"},
      "compliance_readiness": {"confirmed": true | false, "confirmed_by": "<username>"},
      "rollback_readiness": {"confirmed": true | false, "confirmed_by": "<username>"}
    },
    "checklist_issue": "<issue_url>",
    "result": "pass | fail"
  }
}
```

**Failure behaviour:** Blocks deployment. Posts structured failure message to deployment status. Lists which conditions remain unsigned and which authorities must confirm them.

---

### 5.6 rollback-readiness-gate

**Purpose:** Validates that a rollback procedure exists, is tested, and its invoking authority is named before any deployment classified as Irreversible proceeds. Surfaces the IAL escalation requirement for Irreversible deployments.

**Triggers:** `deployment` event where `operation_class` is `irreversible`, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `deployment-id` | string | yes | GitHub deployment ID |
| `operation-class` | string | yes | Decision class of this deployment |

**Logic:**

```
1. If operation_class != irreversible:
   → SKIP: rollback-readiness-gate applies only to Irreversible deployments

2. Check for rollback workflow file in repository:
   Look for .github/workflows/rollback-*.yml or rollback.yml
   → FAIL if absent: "No rollback workflow found. Irreversible deployments
     require a defined, tested rollback procedure. [GOS §17.11]"

3. Check for rollback test record:
   Look for issue with label "rollback-tested" closed within the last 30 days
   linked to this deployment's milestone or branch.
   → FAIL if absent or older than 30 days: "Rollback procedure has not been
     tested within 30 days. Re-test and close a rollback-tested issue
     before proceeding."

4. Check for named rollback authority in govops.yml:
   rollback_authority: "@<github_username>"
   → FAIL if absent: "No rollback authority named in govops.yml.
     Specify rollback_authority before an Irreversible deployment."

5. Surface IAL escalation requirement:
   → POST advisory: "This deployment is classified as Irreversible.
     Under [GOS §6.1], Irreversible executions require Human Authority
     approval at the applicable authority level. GitHub's IAL ceiling is 2.
     If this decision class requires IAL > 2, an out-of-band approval record
     must be linked to this deployment before proceeding."

6. All checks pass:
   → PASS: write Governance Event (type: verification)
```

**Governance Event produced:**
```json
{
  "type": "verification",
  "payload": {
    "check": "rollback-readiness",
    "operation_class": "irreversible",
    "rollback_workflow": "<workflow_path>",
    "rollback_tested_issue": "<issue_url>",
    "rollback_authority": "<github_username>",
    "ial_advisory_posted": true,
    "result": "pass | fail"
  }
}
```

**Failure behaviour:** Blocks Irreversible deployment. Does not block Reversible or Conditional deployments.

---

### 5.7 milestone-governance-checkpoint

**Purpose:** Blocks milestone closure until a structured governance checkpoint is completed and produces a Governance Event recording the milestone review.

**Triggers:** `milestone` closed event, `issues` closed event where issue is the last open issue in a milestone, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `milestone-number` | number | yes | Milestone number being closed |

**Logic:**

```
1. On milestone close event:
   Check for an open issue with label "milestone-checkpoint" linked
   to this milestone.
   → If absent: RE-OPEN milestone and post comment:
     "Milestone cannot close without a completed governance checkpoint.
      Create an issue using templates/MILESTONE-REVIEW.md, complete
      the checkpoint, and close this issue to proceed. [GOS §17.11]"

2. If checkpoint issue exists but is still open:
   → BLOCK milestone closure: "Milestone checkpoint issue #<N> is not yet closed.
     Complete the checkpoint review before closing this milestone."

3. If checkpoint issue is closed:
   Validate it was closed with at least one of:
   - A decision recorded (checkbox: [x] Decision recorded)
   - A formal continuation confirmed (checkbox: [x] Continuation confirmed)
   → FAIL if neither: "Milestone checkpoint produced no governance decision.
     A milestone review that produces no decisions is not a governance event.
     Re-open the checkpoint, record a decision or formal continuation,
     and re-close. [GOS §17.11]"

4. Checkpoint valid:
   → PASS: write Governance Event (type: decision)
   → Allow milestone to close
```

**Governance Event produced:**
```json
{
  "type": "decision",
  "payload": {
    "check": "milestone-checkpoint",
    "milestone_number": "<number>",
    "milestone_title": "<title>",
    "checkpoint_issue": "<issue_url>",
    "decision_recorded": true | false,
    "continuation_confirmed": true | false,
    "checkpoint_closed_by": "<github_username>",
    "result": "pass | fail"
  }
}
```

**Failure behaviour:** Re-opens milestone if closed without checkpoint. Posts structured guidance to the milestone's associated issues.

---

### 5.8 exception-logger

**Purpose:** Detects all governance requirement bypasses and admin overrides on the repository, converts them to exception log entries, and writes them to the Immutable Ledger as Governance Events of type `alert`.

**Triggers:** `workflow_run` completed with bypass status, push to protected branch by admin override, `check_run` bypass, `deployment` to production without required checks, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |

**Logic:**

```
1. On any protected branch push that bypassed a required status check:
   Extract: bypassing user, bypassed check name, commit SHA, timestamp
   → Write exception log entry

2. On any deployment that bypassed a required environment reviewer:
   Extract: bypassing user, environment, deployment ID, reason (if provided)
   → Write exception log entry

3. On any workflow run completed with a bypassed governance workflow:
   Extract: bypassed workflow, bypass actor, run ID
   → Write exception log entry

4. For each exception detected:
   a. Create GitHub issue with label "govops-exception" using EXCEPTION.md template
      Pre-populated with: requirement bypassed, bypassing actor, timestamp,
      expiry date (default: 14 days), owner assigned to bypassing actor
   b. Write Governance Event (type: alert) to Immutable Ledger
   c. Post alert to configured notification channel

5. Aggregate exception count for the repository:
   Update repository-level exception rate metric in governance record
```

**Governance Event produced:**
```json
{
  "type": "alert",
  "payload": {
    "check": "exception-logger",
    "exception_type": "branch-protection-bypass | environment-bypass | workflow-bypass",
    "bypassing_actor": "<github_username>",
    "bypassed_requirement": "<requirement_name>",
    "exception_issue": "<issue_url>",
    "expiry_date": "<ISO8601>",
    "assigned_owner": "<github_username>"
  }
}
```

**Failure behaviour:** Does not block the bypass (the bypass has already occurred). Creates the exception record, writes the alert, and notifies. The exception issue blocks the repository from achieving "clean" governance status in the portfolio health report until it is formally closed.

---

### 5.9 portfolio-health-aggregator

**Purpose:** Produces a portfolio health report across all repositories in the initiative's portfolio on a defined schedule, aggregating governance metrics for submission to the Think Layer and Oversight Body.

**Triggers:** `schedule` (as configured in `govops.yml`), `workflow_dispatch`, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `org-name` | string | yes | GitHub organisation name |
| `portfolio-label` | string | yes | Label identifying repos in this portfolio |

**Logic:**

```
1. Query all repositories in org-name with label portfolio-label
   via GitHub API

2. For each repository, collect:
   a. Open exception count (issues labelled govops-exception, open)
   b. Overdue exception count (open exceptions past expiry date)
   c. Milestone checkpoint completion rate (checkpoints completed / milestones closed)
   d. Operational readiness pass rate (OR gates passed / production deployments)
   e. Governance Event chain integrity (bridge confirms no gaps)
   f. GOVERNANCE.md validity status
   g. Days since last PIR (for closed initiatives)

3. Compute portfolio-level aggregates:
   a. Total open exceptions across portfolio
   b. Exception closure rate (exceptions closed / exceptions opened, rolling 30 days)
   c. Portfolio delivery integrity score (OR pass rate × checkpoint completion rate)
   d. Repositories in alert state (any overdue exception or missing GOVERNANCE.md)

4. Generate health report as markdown artifact:
   Portfolio Health Report — <date>
   - Summary table: one row per repository
   - Alert section: repositories requiring governance intervention
   - Exception patterns: recurring exceptions across ≥ 3 repositories
     (signals a standards review requirement under [GOS §17.11])
   - Trend section: 30/60/90 day exception rate trend

5. Post health report as:
   a. GitHub Actions artifact (retained 90 days)
   b. Issue in designated portfolio-management repository (if configured)
   c. Write Governance Event (type: record) to Immutable Ledger

6. If any repository is in alert state:
   Write Governance Event (type: alert) for each alerting repository
```

**Governance Event produced:**
```json
{
  "type": "record",
  "payload": {
    "check": "portfolio-health",
    "report_date": "<ISO8601>",
    "portfolio_label": "<label>",
    "repositories_assessed": "<count>",
    "total_open_exceptions": "<count>",
    "exception_closure_rate": "<0.0-1.0>",
    "delivery_integrity_score": "<0.0-1.0>",
    "repositories_in_alert": ["<repo-name>", "..."],
    "report_artifact_url": "<url>"
  }
}
```

---

### 5.10 pir-trigger

**Purpose:** Blocks repository archiving and GitHub Project closure until a Post-Implementation Review is created, completed, and linked to the initiative's governance record.

**Triggers:** `repository` archived event (via webhook), GitHub Project closed event, `workflow_dispatch` (manual closure check), `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |
| `closure-type` | string | yes | `archive` or `project-close` |

**Logic:**

```
1. On repository archive or project close:
   Check for an issue with label "pir" in the repository.
   → If absent: BLOCK and post:
     "Repository cannot archive without a completed Post-Implementation Review.
      Create an issue using templates/PIR.md, complete the review,
      and close this issue to proceed. [GOS §17.11]"

2. If PIR issue exists but is open:
   → BLOCK: "PIR issue #<N> is not yet closed. Complete the review first."

3. If PIR issue is closed:
   Validate it contains required sections (see Template 6.4):
   - [x] Governance worked (at least one item)
   - [x] Governance failed (at least one item, or explicit "none identified")
   - [x] Improvement backlog submitted (link to Meta-Evolution input)
   → FAIL if any section absent

4. PIR valid:
   → PASS: write Governance Event (type: record) confirming PIR completion
   → Allow archiving to proceed
   → Submit PIR summary to portfolio-health-aggregator for institutional
      learning pattern analysis
```

**Governance Event produced:**
```json
{
  "type": "record",
  "payload": {
    "check": "pir-trigger",
    "closure_type": "archive | project-close",
    "pir_issue": "<issue_url>",
    "pir_closed_by": "<github_username>",
    "improvement_backlog_ref": "<url>",
    "result": "pass | fail"
  }
}
```

**Failure behaviour:** Blocks archiving via GitHub API (sets repository to non-archivable state while check is pending). Posts blocking message as a repository notice.

---

### 5.11 asset-handover-validator

**Purpose:** Validates that all operational outputs of the initiative have named owners, SLAs, versions, and review cadences recorded before repository archiving is declared a valid governance closure.

**Triggers:** Runs after pir-trigger passes, before repository archives, `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|---|---|---|---|
| `config-path` | string | yes | Path to `govops.yml` |

**Logic:**

```
1. Look for ASSET-HANDOVER.md in repository root (Template 6.5)
   → FAIL if absent: "No asset handover record found. Create ASSET-HANDOVER.md
     before archiving. [GOS §17.11]"

2. Validate ASSET-HANDOVER.md:
   Check that at least one asset is listed AND every listed asset has:
   - asset_name: populated
   - asset_type: one of (process | system | dataset | policy | document | model)
   - named_owner: a valid GitHub username in the organisation
   - sla: populated (not empty)
   - version: populated
   - review_cadence: one of (monthly | quarterly | semi-annual | annual)
   → FAIL for each asset missing required fields

3. Validate named_owner is still an active organisation member
   → WARN if named_owner has left the organisation:
     "Asset owner @<username> is no longer an active organisation member.
      Update named_owner before archiving."

4. All assets valid:
   → PASS: write Governance Event (type: record)
   → Write each asset record to Immutable Ledger
   → Allow archiving to proceed (both pir-trigger and
     asset-handover-validator must pass for archiving to be valid)
```

**Governance Event produced:**
```json
{
  "type": "record",
  "payload": {
    "check": "asset-handover",
    "assets": [
      {
        "name": "<asset_name>",
        "type": "<asset_type>",
        "named_owner": "<github_username>",
        "sla": "<sla_description>",
        "version": "<version>",
        "review_cadence": "<cadence>"
      }
    ],
    "assets_count": "<count>",
    "result": "pass | fail"
  }
}
```

**Failure behaviour:** Blocks archiving. Lists all assets with missing fields. Links to Template 6.5 for remediation.

---

## 6. Governance Template Specifications

Templates are canonical documents for governance acts that GitHub does not natively structure. All templates are maintained at `GovernanceOperations/projops/templates/`. Consuming repositories copy them and fill in the required fields. The workflow checks validate completed templates against expected structure.

---

### 6.1 GOVERNANCE.md — Initiative Governance Record

This is the governance identity document for a repository. It must exist before the repository can receive initiative work. It is the machine-readable counterpart to the mandate statement required by **[GOS §17.11]**.

```markdown
# Initiative Governance Record

<!-- govops-schema: governance-md v0.9 -->
<!-- Do not remove schema comment. Required for automated validation. -->

## Initiative Identity

**Initiative title:**
<!-- One line describing what this initiative delivers -->

**Repository:**
<!-- Full repository name: org/repo -->

**Decision Contract reference:**
<!-- Format: ENTITY-YEAR-SEQUENCE-CHECKSUM e.g. ACME-2026-0042-A3F9B2C1 -->
<!-- This must reference an approved Decision Contract in your Memory Layer -->

**Decision class:**
<!-- reversible | conditional | irreversible -->

**Operational tier:**
<!-- 1 | 2 | 3 | 4 -->

**GovOps version:**
<!-- 0.9 -->

---

## Mandate Statement

**Problem being solved:**
<!-- What condition, gap, or decision is this initiative responding to? -->

**Outcome targeted:**
<!-- What will be observably different when this initiative succeeds? -->

**Constraints accepted:**
<!-- What are the known limits within which this initiative must operate? -->

**Success criteria:**
<!-- How will the sponsoring authority know the Decision Contract has been satisfied? -->
<!-- These must be verifiable, not aspirational -->

---

## Uncertainty Classification

**Primary classification:**
<!-- aleatory | epistemic | ambiguity -->

**Classification rationale:**
<!-- One paragraph explaining why this classification applies to this initiative -->

**Ambiguity resolution reference:**
<!-- REQUIRED if classification is 'ambiguity' -->
<!-- Link to the completed alignment record confirming all sponsoring authorities -->
<!-- agree on the objective. Leave blank if classification is not 'ambiguity'. -->

---

## Decision Architecture

**Sponsoring authority:**
<!-- GitHub username of the person who holds Sovereign Decision authority for this initiative -->
<!-- Format: @username -->

**Approves scope changes:**
<!-- GitHub username (may be same as sponsoring authority) -->

**Authorises compliance exceptions:**
<!-- GitHub username (should be different from sponsoring authority) -->

**Authority to close or pause the initiative:**
<!-- GitHub username -->

**Rollback authority:**
<!-- GitHub username of the person who may invoke the rollback procedure -->

---

## Compliance Obligation Map

<!-- List every regulatory, contractual, or policy obligation this initiative must satisfy -->
<!-- One obligation per line. Do not leave this section empty. -->
<!-- If no obligations apply, state: None identified as of <date>. Reviewed by @<username>. -->

| Obligation | Source | Delivery requirement | Owner |
|---|---|---|---|
| <!-- e.g. GDPR Article 25 data minimisation --> | <!-- regulation/contract/policy name --> | <!-- what must be done in this initiative to satisfy it --> | <!-- @username --> |

---

## Delivery System

<!-- The delivery system is a governed decision. It must be recorded here before -->
<!-- initiative work begins and updated through a governed change process if it changes. -->
<!-- See Section 10.5 of the ProjOps Practitioner Reference for selection guidance. -->

**Delivery pattern:**
<!-- staged-gate | iterative | parallel-track | hybrid -->

**Execution framework:**
<!-- scrum | kanban | prince2 | safe | internal | hybrid -->
<!-- 'internal' = a named organisational variant — specify the name in framework description -->
<!-- 'hybrid' = describe precisely in framework description -->

**Framework description:**
<!-- Required if execution framework is 'hybrid' or 'internal'. -->
<!-- Describe what practices are used, how they are combined, and any adaptations. -->

**Governance cadence:**
<!-- How do delivery cycles connect to milestone reviews and governance checkpoints? -->
<!-- Be specific: "Monthly milestone review at end of every second two-week sprint." -->
<!-- Not: "Agile with monthly reviews." -->

**Selection rationale:**
<!-- Map the selection to the four criteria from Section 10.5: -->
<!-- 1. Uncertainty classification: <aleatory|epistemic> → pattern indicated: <pattern> -->
<!-- 2. Compliance obligation density: <high|medium|low> → implications for methodology: <text> -->
<!-- 3. Rate of change tolerance: <high|medium|low> → implications for pattern: <text> -->
<!-- 4. Organisational mandate: <framework name or 'none'> → applied as: <adopted|deviated> -->

**Approved by:**
<!-- @github_username — authority who approved the delivery system selection -->
<!-- This is the same authority who approves scope changes. -->

**Deviation from organisational mandate:**
<!-- yes | no -->
<!-- If yes: the following two fields are required. -->

**Deviation rationale:**
<!-- Required if deviation == yes. -->
<!-- Why is the organisational mandate incompatible with this initiative's characteristics? -->
<!-- Reference specific criteria from the selection rationale above. -->

**Deviation approved by:**
<!-- Required if deviation == yes. -->
<!-- @github_username — must be the authority designated for scope change approval. -->

---

## Initiative Status

**Current status:**
<!-- active | paused | closing | closed -->

**Initiative opened:**
<!-- ISO 8601 date -->

**Expected closure:**
<!-- ISO 8601 date or condition -->

**Last governance record update:**
<!-- ISO 8601 date -->

---

<!-- govops-end -->
```

---

### 6.2 EXCEPTION.md — Governance Exception Issue Template

Create as `.github/ISSUE_TEMPLATE/govops-exception.md` in the consuming repository:

```markdown
---
name: GovOps Governance Exception
about: Record a governance requirement that was bypassed or deferred
labels: govops-exception
---

<!-- govops-schema: exception v0.9 -->

## Exception Record

**Repository:**
<!-- Full repository name -->

**Date of exception:**
<!-- ISO 8601 date and time -->

**Bypassing actor:**
<!-- @github_username -->

**Governance requirement bypassed:**
<!-- Name the specific requirement: branch protection | required review | 
     operational readiness gate | compliance obligation | documentation standard -->

**Specific control bypassed:**
<!-- e.g. "Required status check: govops-operational-readiness-gate on production deployment #47" -->

---

## Rationale

**Why the bypass was necessary:**
<!-- Operational reason. "Urgency" is not a rationale. What specific condition made -->
<!-- compliance with the requirement impossible or harmful in this instance? -->

**Risk accepted by bypassing:**
<!-- What governance gap has been created? What could now go wrong that the bypassed -->
<!-- requirement was designed to prevent? -->

---

## Remediation

**Remediation action:**
<!-- What will be done to close the governance gap created by this exception? -->

**Expiry date:**
<!-- ISO 8601 date. Maximum 30 days from exception date. -->
<!-- This issue must be closed (remediation complete) before this date. -->
<!-- Overdue exceptions are flagged in portfolio health reports. -->

**Assigned owner:**
<!-- @github_username — responsible for remediation and closing this issue -->

**Authority who approved this exception:**
<!-- @github_username — must be the authority designated in GOVERNANCE.md -->
<!-- for compliance exceptions -->

---

## Closure Criteria

<!-- Do not close this issue until all of the following are confirmed: -->
- [ ] Remediation action completed
- [ ] Governance requirement now satisfied (or formal exception granted by Oversight Body)
- [ ] Exception pattern reported to portfolio health (if this is the 3rd+ exception of this type)

<!-- govops-end -->
```

---

### 6.3 MILESTONE-REVIEW.md — Milestone Governance Checkpoint Template

Create as `.github/ISSUE_TEMPLATE/govops-milestone-review.md`:

```markdown
---
name: GovOps Milestone Governance Checkpoint
about: Structured accountability checkpoint for milestone closure
labels: milestone-checkpoint
---

<!-- govops-schema: milestone-review v0.9 -->

## Milestone Governance Checkpoint

**Milestone:**
<!-- Milestone title and number: Title (#N) -->

**Checkpoint date:**
<!-- ISO 8601 date -->

**Conducted by:**
<!-- @github_username — must be sponsoring authority or delegated reviewer -->

---

## Mandate Alignment

**Does this milestone's output remain aligned with the initiative mandate?**
<!-- yes | no | partially -->

**If partially or no — what has changed and why:**
<!-- Describe the divergence. If this requires a scope change, record it as -->
<!-- a decision below and update GOVERNANCE.md. -->

---

## Risk Status

**Current risk profile:**
<!-- Unchanged from initiation | Increased | Decreased -->

**Active risks above tolerance:**
<!-- List any risks that have materialised or are now above the accepted threshold -->
<!-- If none: "None above tolerance as of this checkpoint" -->

---

## Governance Record Currency

**Is the GOVERNANCE.md current?**
<!-- yes | no — if no, update before closing this checkpoint -->

**Open exceptions:**
<!-- Number of open govops-exception issues: N -->
<!-- Are any overdue? yes | no -->

---

## Delivery System Review

<!-- The delivery system is reviewed at every milestone checkpoint. -->
<!-- A delivery system that is no longer appropriate to the initiative's actual conditions -->
<!-- must be changed through a governed process, not abandoned informally. -->

**Is the current delivery system still appropriate to this initiative's conditions?**
<!-- yes | no | under review -->

**If no or under review — which signal is present:**
<!-- governance-theatre: milestone reviews producing no decisions, ceremonies without governance -->
<!-- compliance-accumulation: compliance obligations deferred from cycle exit criteria -->
<!-- scope-instability: requirements changing significantly under a locked-scope pattern -->
<!-- cadence-misalignment: delivery cycle frequency mismatched to governance checkpoint frequency -->
<!-- other: describe -->

**If delivery system change is required:**
<!-- Record the proposed change here. It becomes a decision item below. -->
<!-- Pattern change: <from> → <to> -->
<!-- Framework change: <from> → <to> -->
<!-- Governance cadence change: <from> → <to> -->
<!-- Deviation from mandate involved: yes | no -->
<!-- A delivery system change requires the same authority as a scope change. [Section 10.5] -->

---

## Decisions and Actions

<!-- A milestone checkpoint that produces no decisions is not a governance event. -->
<!-- At minimum, record a formal continuation or a decision. -->

- [ ] **Decision recorded:**
  <!-- Describe the decision made at this checkpoint. If no material decision, -->
  <!-- state the continuation confirmation explicitly: -->
  <!-- "Confirmed: initiative continues within original mandate, risk tolerance, -->
  <!-- and resource authority as of <date>. No scope change. No exceptions granted." -->

- [ ] **Continuation confirmed** (if no decision required):
  <!-- @sponsoring_authority confirms continuation explicitly -->
  Confirmed by: <!-- @github_username -->

**Actions arising:**
<!-- List any actions with owners and due dates. "None" is valid. -->

---

## Closure

<!-- Do not close this issue until a decision or continuation confirmation is recorded above. -->
<!-- Closing this issue allows the milestone to close. -->

<!-- govops-end -->
```

---

### 6.4 PIR.md — Post-Implementation Review Template

Create as `.github/ISSUE_TEMPLATE/govops-pir.md`:

```markdown
---
name: GovOps Post-Implementation Review
about: Structured review of a completed initiative for institutional learning
labels: pir
---

<!-- govops-schema: pir v0.9 -->

## Post-Implementation Review

**Initiative:**
<!-- Repository full name and initiative title from GOVERNANCE.md -->

**PIR date:**
<!-- ISO 8601 date -->

**Conducted by:**
<!-- @github_username(s) — should include delivery team, sponsoring authority, -->
<!-- and at least one operational team member who received the initiative's output -->

**Decision Contract reference:**
<!-- From GOVERNANCE.md — confirms which approved decision this initiative served -->

---

## Outcome Assessment

**Did the initiative satisfy its success criteria?**
<!-- yes | partially | no -->

**What was delivered:**
<!-- List the operational outputs (processes, systems, capabilities, policies) -->
<!-- that this initiative produced -->

**Divergence from Decision Contract:**
<!-- What, if anything, was different from what the Decision Contract anticipated? -->
<!-- If none: "None identified." -->

---

## Governance Assessment

### What governance worked and should be standardised:
<!-- Minimum one item. What governance practice in this initiative was effective? -->
<!-- Format: Practice | Why it worked | Recommendation for standardisation -->

- <!-- e.g. Uncertainty classified as epistemic at initiation → front-loaded discovery work
       eliminated two mid-delivery scope changes → recommend as standard for all
       data-dependent initiatives -->

### What governance failed and should be redesigned:
<!-- Minimum one item OR explicit "None identified — all governance requirements
     were met throughout this initiative." -->

- <!-- e.g. Operational readiness gate triggered 3 days before planned deployment
       because compliance readiness was left until last → recommend integrating
       compliance readiness tracking into milestone checkpoints from initiation -->

### What produced unintended friction and should be simplified:
<!-- Process that added overhead without adding governance value -->
<!-- "None identified" is acceptable. -->

---

## Exception Analysis

**Total exceptions logged during this initiative:**
<!-- Count of govops-exception issues created -->

**Exception patterns identified:**
<!-- Any exception that recurred ≥ 2 times during this initiative is a pattern -->
<!-- Format: Exception type | Count | Root cause | Governance implication -->

---

## Improvement Backlog

**Governance improvements identified:**
<!-- List changes to governance policy, standards, or this layer's workflows -->
<!-- that would improve future initiatives -->

**Improvement backlog submitted to Meta-Evolution process:**
<!-- Link to the issue, PR, or record in the governance policy register where -->
<!-- these improvements have been submitted [GOS §24] -->
<!-- REQUIRED: this field must be populated before this issue can be closed -->
<!-- If no improvements: create a record stating "No improvements identified" -->
<!-- and link it here. A closed initiative with no PIR input to Meta-Evolution -->
<!-- is a learning gap. -->

- [ ] Improvement backlog submitted: <!-- link here -->

---

## Closure

<!-- Do not close this issue until: -->
- [ ] All sections completed
- [ ] Improvement backlog submitted (link above populated)
- [ ] All named owners for asset handover confirmed (see ASSET-HANDOVER.md)

<!-- govops-end -->
```

---

### 6.5 ASSET-HANDOVER.md — Operational Asset Handover Record

```markdown
# Operational Asset Handover Record

<!-- govops-schema: asset-handover v0.9 -->
<!-- Maintained in repository root. Must be complete before archiving. -->

**Initiative:**
<!-- Repository full name -->

**Handover date:**
<!-- ISO 8601 date -->

**Handover approved by:**
<!-- @github_username — sponsoring authority confirming all assets are owned -->

---

## Asset Register

<!-- One entry per operational output of this initiative -->
<!-- Delete example entries and replace with actual assets -->
<!-- Every row is mandatory — do not leave cells empty -->

| Asset name | Asset type | Named owner | SLA | Version | Review cadence | Location |
|---|---|---|---|---|---|---|
| <!-- e.g. Customer onboarding process v2 --> | process | <!-- @username --> | <!-- e.g. Reviewed within 5 days of any regulatory change --> | 2.0 | quarterly | <!-- link to documentation --> |
| <!-- e.g. Identity verification service --> | system | <!-- @username --> | <!-- e.g. 99.5% availability; P1 incidents resolved within 4 hours --> | 1.3.2 | semi-annual | <!-- link to runbook --> |
| <!-- e.g. Transaction risk model --> | model | <!-- @username --> | <!-- e.g. Performance reviewed monthly; retrained if accuracy < 94% --> | 3.1 | monthly | <!-- link to model card --> |

**Asset type values:** process | system | dataset | policy | document | model

---

## Handover Confirmation

<!-- Each named owner must confirm receipt of their asset(s) -->
<!-- Do not close this document without all confirmations -->

- [ ] <!-- @asset_owner_1 --> confirms ownership of <!-- asset name(s) -->
- [ ] <!-- @asset_owner_2 --> confirms ownership of <!-- asset name(s) -->

---

## Support Transition

**Support escalation path:**
<!-- Who do asset owners contact if they need help with an asset they have received? -->
<!-- Format: Issue type → contact → SLA for response -->

**Known limitations:**
<!-- Limitations of any asset that the receiving owner must be aware of -->
<!-- "None identified" is acceptable but must be explicit -->

**End-of-life plan:**
<!-- What happens to each asset when it is superseded or retired? -->
<!-- Who has authority to decommission? What is the decommission process? -->

---

<!-- govops-end -->
```

---

## 7. Immutable Ledger Bridge Specification

### 7.1 Purpose

The bridge is the component that converts GitHub's mutable event stream into an append-only, cryptographically chained Governance Event record. Without the bridge, the eleven workflows produce enforcement gates but no auditable memory. With the bridge, every governance act produces a Governance Event conforming to **[GOS §6.6]** written to an Immutable Ledger external to GitHub.

### 7.2 Architecture

The bridge is a GitHub App — a small server-side application that:

1. Subscribes to GitHub webhook events from all repositories where it is installed
2. Translates each relevant GitHub event into a Governance Event in the six-field structure
3. Signs each Governance Event with the bridge's cryptographic key
4. Writes each Governance Event to the consuming organisation's configured Immutable Ledger endpoint
5. Maintains a local chain integrity reference (the `integrity_reference` field) linking each event to its predecessor in the chain

```
GitHub webhook event
        │
        ▼
   Bridge App
        │
   ┌────┴────┐
   │ Translate│  GitHub event → Governance Event (six-field structure)
   └────┬────┘
        │
   ┌────┴────┐
   │  Sign   │  Post-quantum signature per [GOS §11.2]
   └────┬────┘
        │
   ┌────┴──────────┐
   │ Chain & Write │  Append to Immutable Ledger at configured endpoint
   └───────────────┘
```

### 7.3 GitHub App Manifest

```yaml
# app.yml — GitHub App manifest for GovOps Ledger Bridge
name: GovOps Ledger Bridge
description: >
  Translates GitHub events into GovOps Governance Events and writes them
  to a configured Immutable Ledger endpoint. Required for Memory Layer
  compliance under [GOS §6.6].
url: https://github.com/GovernanceOperations/projops
webhook_events:
  - push
  - pull_request
  - pull_request_review
  - issues
  - milestone
  - deployment
  - deployment_status
  - workflow_run
  - check_run
  - repository
  - member
  - organization
  - audit_log_streaming
permissions:
  contents: read
  issues: write
  pull_requests: write
  deployments: read
  checks: read
  metadata: read
  members: read
```

### 7.4 Event Translation Map

| GitHub Event | Governance Event Type | Trigger Condition |
|---|---|---|
| `push` to protected branch | `execute` | All pushes to main, release/*, deploy/* |
| `pull_request` merged | `execute` | `merged == true` |
| `pull_request_review` approved | `approval` | Review state == approved |
| `issues` opened with `govops-exception` label | `alert` | Label applied |
| `milestone` closed | `decision` | Checkpoint issue confirmed closed |
| `deployment` created | `execution` | All deployments |
| `deployment_status` success/failure | `record` | Final deployment status |
| `workflow_run` completed | `record` | All govops workflow completions |
| `check_run` bypassed | `alert` | Admin bypass detected |
| `repository` archived | `record` | Archiving confirmed post-PIR |
| `member` removed | `alert` | If removed member is a named asset owner |

### 7.5 Governance Event Structure Produced by Bridge

Every event written by the bridge conforms to **[GOS §6.6]**:

```json
{
  "identity": "<UUID v4, unique and non-reusable>",
  "timestamp": "<ISO 8601 with microsecond precision>",
  "type": "<observation|verification|evaluation|propose|approval|execute|record|review|adapt|alert|escalation|system>",
  "payload": {
    "source": "govops-ledger-bridge",
    "github_event": "<original_event_type>",
    "repository": "<org/repo>",
    "actor": "<github_username>",
    "govops_workflow": "<workflow_name or null>",
    "decision_contract_ref": "<from GOVERNANCE.md or null>",
    "data": "<structured event-specific payload>"
  },
  "signature": "<post-quantum signature of all preceding fields>",
  "integrity_reference": "<certification_hash of immediately preceding event in chain>"
}
```

### 7.6 Immutable Ledger Endpoint Requirements

The consuming organisation must provide an Immutable Ledger endpoint that satisfies the four properties of **[GOS §6.6]**:

| Property | Requirement |
|---|---|
| **Append-only** | The endpoint accepts POST to write new events. It has no PATCH, PUT, or DELETE operations. Any attempt to modify a record returns 405 Method Not Allowed. |
| **Tamper-evident** | Each written event is linked to its predecessor via the `integrity_reference` field. Any gap or modification in the chain is detectable. |
| **Auditable** | The complete event history for any repository or initiative is queryable by any party with authorised read access, without the cooperation of the producing system. |
| **Reproducible** | Given the same input sequence and keys, the same records are produced. |

Reference implementations compatible with these requirements include: append-only object storage with versioning disabled and deletion locked (AWS S3 with Object Lock, Azure Blob with immutability policy, GCS with retention policy), blockchain-anchored audit logs, and WORM-compliant log management systems.

### 7.7 Bridge Configuration in govops.yml

```yaml
ledger:
  endpoint: "https://your-ledger-endpoint/v1/events"
  auth_secret: "LEDGER_API_KEY"          # GitHub secret containing API key
  chain_id: "your-org-your-initiative"   # Unique chain identifier for this initiative
  signature_algorithm: "dilithium3"       # Post-quantum algorithm per [GOS §11.2]
  alert_channel: "https://hooks.slack.com/..."  # Optional: alert webhook
```

---

## 8. Configuration Reference

Complete `govops.yml` schema with all supported fields:

```yaml
# govops.yml — complete configuration reference
govops_version: "0.9"                    # Required. Must match installed layer version.

initiative:
  tier: 2                                # Required. 1 | 2 | 3 | 4 [GOS §30]
  decision_class: "conditional"          # Required. reversible | conditional | irreversible
  ial_ceiling: 2                         # Required. Maximum IAL achievable on this platform.
                                         # GitHub maximum is 2. Override only with documented
                                         # out-of-band IAL mechanism in place.
  ial_required_for_merge: 2              # Minimum IAL for protected branch merges
  ial_required_for_deploy: 2             # Minimum IAL for production deployments
  rollback_authority: "@username"        # Required for irreversible decision_class
  production_environment_pattern: "production|prod|release"  # Regex for production envs

ledger:
  endpoint: ""                           # Required. Immutable Ledger write endpoint.
  auth_secret: "LEDGER_API_KEY"          # Required. GitHub secret name.
  chain_id: ""                           # Required. Unique identifier for this event chain.
  signature_algorithm: "dilithium3"      # Default. Post-quantum algorithm [GOS §11.2]
  alert_channel: ""                      # Optional. Webhook for governance alerts.

portfolio:
  org_label: ""                          # Label identifying repos in this portfolio
  health_report_schedule: "0 9 * * 1"   # Cron schedule for portfolio health aggregation
  portfolio_repo: ""                     # Optional. Repo where health reports are posted as issues.

governance:
  require_uncertainty_classification: true   # Block execution on ambiguity classification
  require_milestone_checkpoints: true        # Block milestone closure without checkpoint
  require_pir: true                          # Block archiving without PIR
  require_asset_handover: true               # Block archiving without asset handover record
  exception_expiry_days: 14                  # Default exception expiry window
  exception_pattern_threshold: 3            # Exceptions of same type → standards review alert

delivery_system:
  pattern: ""                              # Required. staged-gate | iterative | parallel-track | hybrid
  framework: ""                            # Required. scrum | kanban | prince2 | safe | internal | hybrid
                                           # Use 'internal' for a named organisational variant
                                           # Use 'hybrid' only with framework_description populated
  framework_description: ""               # Required if framework == hybrid. Describe the specific hybrid.
  governance_cadence: ""                   # Required. How delivery cycles connect to milestone reviews.
                                           # e.g. "Monthly milestone review at end of every second sprint"
  org_mandate: ""                          # The organisation's mandated framework for this tier.
                                           # Leave blank if no mandate exists.
  deviation_from_mandate: false            # true if selected framework differs from org_mandate
  deviation_approved_by: ""               # Required if deviation_from_mandate == true.
                                           # GitHub username of approving authority.
  deviation_rationale: ""                 # Required if deviation_from_mandate == true.
                                           # Why the mandate is incompatible with this initiative.

runners:
  default: "ubuntu-latest"              # Override for self-hosted runners
```

---

## 9. Identity Assurance Level Mapping

GitHub's authentication model supports IAL 1 and IAL 2. IAL 3 and IAL 4 require out-of-band mechanisms.

| IAL | GitHub Equivalent | Enforcement Mechanism |
|---|---|---|
| IAL 1 | Authenticated GitHub user | GitHub login |
| IAL 2 | Organisation member with enforced 2FA | Organisation 2FA policy + SAML SSO |
| IAL 3 | **Not natively achievable on GitHub** | External IdP with hardware token, linked to GitHub SAML identity. Out-of-band approval document required. |
| IAL 4 | **Not natively achievable on GitHub** | Physical presence verification by designated governance authority. Out-of-band approval document required. |

**IAL Register:** The consuming organisation maintains an IAL register at `.github/govops/ial-register.json`. This file records the verified IAL of each organisation member, maintained by the designated governance authority. The ial-ceiling-check workflow reads this register.

```json
{
  "members": {
    "username": {
      "ial": 2,
      "verified_by": "governance-authority-username",
      "last_verified": "2026-01-15",
      "verification_method": "org-2fa-enforced"
    }
  }
}
```

**Out-of-band approval for IAL 3 and IAL 4:** When an Irreversible decision requires IAL 3 or IAL 4, the consuming organisation must:

1. Obtain the approval through the required out-of-band identity verification process
2. Create a signed approval document (PDF or equivalent) with: the approver's identity, the verified IAL, the specific decision being approved, and a cryptographic hash of the relevant PR or deployment
3. Link this document to the PR or deployment via a comment containing `govops-ial-approval: <document_hash>`
4. The ial-ceiling-check workflow recognises this marker and validates the hash

---

## 10. Compliance Verification

A repository is considered compliant with the GovOps GitHub Layer when all of the following are true:

| Condition | Verification method |
|---|---|
| `GOVERNANCE.md` present, valid, and current | governance-init-check passes on last push |
| `govops.yml` present and valid | governance-init-check passes |
| No overdue exceptions | exception-logger: zero open govops-exception issues past expiry date |
| All milestone checkpoints complete | milestone-governance-checkpoint: no milestones closed without checkpoint |
| All production deployments passed operational readiness gate | operational-readiness-gate: 100% pass rate |
| Governance Event chain intact | Bridge: no gaps in event chain since repository creation |
| PIR complete (for closing repositories) | pir-trigger passes |
| Asset handover complete (for closing repositories) | asset-handover-validator passes |

**Compliance report:** The portfolio-health-aggregator workflow produces a machine-readable compliance summary for each repository in its weekly run. The compliance summary is a JSON artifact consumable by the consuming organisation's governance dashboard or Oversight Body.

---

## 11. Versioning and Maintenance

**Version scheme:** The GovOps GitHub Layer is versioned in alignment with the Governance Operations Specification. Layer v0.9.x implements the requirements of GOS v0.9. Breaking changes require a major version increment and a migration guide.

**Calling a specific version:**
```yaml
uses: GovernanceOperations/projops/.github/workflows/govops-suite.yml@v0.9
```

Using a specific version tag is required. Calling `@main` is not permitted in production governed repositories — consuming the head of a branch is an uncontrolled dependency, which is itself a governance gap.

**Deprecation policy:** Workflow inputs and schema fields are deprecated with a minimum 90-day notice period before removal. Deprecated fields produce a warning Governance Event of type `alert` but do not block governance gates during the deprecation window.

**Maintenance governance:** This repository is itself a governed initiative under GOS v0.9. Its `GOVERNANCE.md` is at the repository root. Changes to any workflow, template, or bridge specification are made through pull requests, subject to the same governance gates the layer enforces on consuming repositories.

---

## 12. Glossary of GitHub-to-GovOps Mappings

| GitHub Concept | GovOps Equivalent | Notes |
|---|---|---|
| Repository | Governed initiative vehicle | Only when `GOVERNANCE.md` is present and valid |
| Pull request | Governed execution act | Produces Governance Event of type `execute` on merge |
| PR approval | Approval step **[GOS §6.1]** | Subject to IAL ceiling and decision class requirements |
| Merge to protected branch | Execute step **[GOS §6.1]** | Logged as Governance Event before merge is complete |
| GitHub Actions workflow | Act Layer automation **[GOS §6, Layer 5]** | `permissions` block = Bounded Capability expression |
| Deployment environment | Controlled execution context | Production environments gated by OR and rollback checks |
| Milestone | Initiative stage / governance boundary | Requires checkpoint before closure |
| Issue with `govops-exception` label | Exception log entry | Written to Immutable Ledger as type `alert` |
| Repository archive | Initiative closure | Requires PIR and asset handover to be complete |
| GitHub organisation audit log | Partial Memory Layer input | Not sufficient alone — bridge required for complete record |
| CODEOWNERS | Partial authority delegation | Does not satisfy **[GOS §14]** delegation requirements alone |
| Branch protection bypass | Governance exception | Auto-logged by exception-logger as type `alert` |
| Required status check | Governance gate | govops workflows produce required status checks |
| GitHub Actions `permissions` block | Bounded Capability constraint **[GOS §25]** | Must be defined per workflow, not inherited |
| Dependabot PR | Automated Act Layer operation | Subject to govops gates at merge |
| `.github` org repository | Organisation-wide governance substrate | Houses IAL register, org-level govops config |
| `delivery_system` block in `govops.yml` | Governed delivery system record | Pattern, framework, and governance cadence — all three required |
| `GOVERNANCE.md` Delivery System section | Delivery system Decision Contract | Selection rationale, approval authority, deviation record |
| Milestone review Delivery System section | Delivery system checkpoint | Required at every milestone — signals methodology mismatch before it produces ungoverned delivery |
| Deviation from org mandate | Governed scope-equivalent decision | Requires same authority as scope change; ungoverned deviation is an ungoverned delivery system |

---

*GovOps GitHub Layer Specification | v0.1 | April 2026*

*This specification implements Section 17.11 of the Governance Operations Specification v0.9 for GitHub as an operational substrate. It is maintained as a governed repository at `GovernanceOperations/projops`. All changes are subject to the governance requirements it defines for consuming repositories.*

*This repository does not exist yet. This document is its founding specification.*

---

**END OF DOCUMENT**
