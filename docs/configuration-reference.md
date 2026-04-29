# Configuration Reference
## Every field in govops.yml explained

This document covers every field in the govops.yml configuration file
in detail — what it does, what values are valid, what happens if it is
missing or wrong, and guidance for choosing the right value.

For the step-by-step process of connecting a repository to the governance
layer, see adopting-the-layer.md in this folder.

Plain-language guide to every term: vocabulary.md

---

## governance_scope block

The governance_scope block declares what GitHub entities this initiative
spans and which repository holds the canonical GOVERNANCE.md.
One canonical GOVERNANCE.md per initiative — regardless of how many
repositories or project boards are involved.

---

## governance_scope.substrate

  governance_scope:
    substrate: "repository"

What it does: Identifies the primary delivery substrate for this initiative.
Determines which GitHub events trigger governance checks.

Required: Yes.

Valid values:

  repository     - A single GitHub repository is the primary substrate.
                   Repository events (push, PR, deployment, milestone,
                   archive) trigger governance checks.

  github-project - A GitHub Projects board is the primary substrate.
                   Projects_v2 events (project closed) trigger PIR
                   and asset handover checks. Use for multi-repository
                   initiatives tracked through a project board.

  both           - Both a repository and a GitHub Projects board are
                   active substrates. Both repository events and
                   projects_v2 events trigger governance checks.
                   The canonical repository holds GOVERNANCE.md.

---

## governance_scope.canonical_repository

  governance_scope:
    canonical_repository: "your-org/your-repo"

What it does: Identifies the repository that holds the canonical GOVERNANCE.md
for this initiative. For single-repository initiatives, this is the repository
itself. For multi-repository initiatives, this is the designated canonical
record holder that all other repositories reference.

Required: Yes.

Valid values: org/repo format. Example: acme-corp/payroll-modernisation

Guidance: Once set, do not change without updating all repositories in
the initiative that reference this canonical record.

---

## governance_scope.github_project_url

  governance_scope:
    github_project_url: "https://github.com/orgs/acme-corp/projects/7"

What it does: The URL of the GitHub Projects board for this initiative.
Used by the Ledger Bridge to match projects_v2 events to the correct
initiative governance record.

Required: Yes when substrate is github-project or both.
Leave blank for repository-only initiatives.

Valid values: https://github.com/orgs/ORG/projects/N format only.
Repository-level projects (https://github.com/ORG/REPO/projects/N)
are an older format — use the organisation-level URL format above.

---

## governance_scope.additional_repositories

  governance_scope:
    additional_repositories:
      - acme-corp/payroll-api
      - acme-corp/payroll-frontend
      - acme-corp/payroll-data

What it does: Lists additional repositories that are part of this
initiative beyond the canonical repository.

Required: No. Leave as empty list for single-repository initiatives.

Guidance: Each additional repository should carry a minimal GOVERNANCE.md
at its root pointing back to the canonical record:

  # Project Governance Record — Secondary Repository

  <!-- govops-schema: governance-md v0.9 -->
  <!-- This is a secondary repository in a multi-repository initiative. -->
  <!-- The canonical governance record is held at: -->
  <!-- [canonical-org/canonical-repo] -->

  ## Governance Scope

  Governance substrate: both
  Canonical repository: acme-corp/payroll-modernisation
  [All other fields: see canonical GOVERNANCE.md]

  <!-- govops-end -->

Secondary repositories do not each need their own full govops.yml
or suite workflow. The governance checks run from the canonical
repository. Secondary repositories need only branch protection
rules that require the canonical repository's checks to pass.

---

## govops_version

  govops_version: "0.9"

What it does: Identifies which version of the governance layer
specification this configuration follows.

Required: Yes.

Valid values: A version number matching a released version of this
layer. Current version: "0.9".

What happens if wrong: The governance check flags a version mismatch.
If the version has been deprecated, the check fails.

Guidance: Always specify the exact version. Do not leave blank.

---

## initiative block

The initiative block classifies the project for governance purposes.

---

## initiative.tier

  initiative:
    tier: 2

What it does: Sets the governance intensity level for this project.

Required: Yes.

Valid values and when to use each:

  1 - Streamlined. Low risk, limited scope, reversible change,
      budget within delegated authority, single function affected.

  2 - Standard. Moderate scope, some compliance obligations,
      cross-functional impact. Default for most projects.

  3 - Full. High regulatory exposure, significant operational impact,
      irreversible or high-risk change, external certification required.

  4 - Maximum. Organisation-wide or federated scope, highest
      stakeholder exposure.

What happens if wrong: Checks calibrate to the declared tier.
Declaring a lower tier than warranted results in under-governed delivery
that oversight functions may identify during portfolio review.

Guidance: When in doubt, choose the higher tier. Tier 2 overhead over
Tier 1 is modest. The cost of under-governance is not.
Tier assignment is a governance decision — agree it with the project
sponsor and record it in GOVERNANCE.md.

---

## initiative.decision_class

  initiative:
    decision_class: "conditional"

What it does: Classifies how easily the main outcome can be undone.
Affects which checks activate and at what intensity.

Required: Yes.

Valid values:

  reversible   - Can be undone quickly with minimal disruption.
                 Small changes, feature additions with easy rollback.

  conditional  - Can be undone but with significant effort.
                 Changes affecting multiple systems or teams.

  irreversible - Cannot practically be undone once done.
                 Regulatory changes, data migrations,
                 system decommissioning, structural reorganisations.

What irreversible activates:
  - The rollback readiness gate
  - Requires rollback_authority to be named
  - Triggers the IAL advisory for deployments

Guidance: Ask: "If we go live and immediately discover it was wrong,
how quickly and completely could we return to the previous state?"
If the answer involves significant time, cost, or regulatory process
— or is simply "we cannot" — choose irreversible.

---

## initiative.ial_ceiling

  initiative:
    ial_ceiling: 2

What it does: Declares the maximum identity assurance level achievable
on this platform. Used to determine whether out-of-band approval is
needed for high-consequence decisions.

Required: Yes.

GitHub's maximum is 2. Do not set above 2 unless you have set up an
out-of-band identity verification process. See ial-out-of-band.md.

  1 - Basic GitHub login
  2 - Organisation membership with enforced two-factor authentication
      (stronger with SAML SSO connected)
  3 - Hardware security key via external identity provider — out-of-band
  4 - Physical presence verified by governance authority — out-of-band

---

## initiative.ial_required_for_merge

  initiative:
    ial_required_for_merge: 2

What it does: Minimum identity assurance required for an approver to
count as a valid governance approver for merging changes.

Required: No. Defaults to 1.

Guidance: Set to 2 for Tier 2 and above. IAL 1 means any authenticated
GitHub user can approve changes — including compromised accounts.

---

## initiative.ial_required_for_deploy

  initiative:
    ial_required_for_deploy: 2

What it does: Minimum identity assurance required to approve a
production deployment.

Required: No. Defaults to 1.

Guidance: Set to 2 for conditional or irreversible projects. For
irreversible deployments, consider whether IAL 3 or 4 is warranted
given the consequence of the decision.

---

## initiative.rollback_authority

  initiative:
    rollback_authority: "@username"

What it does: Names the person authorised to invoke the rollback
procedure if the deployment causes problems.

Required: Yes, if decision_class is irreversible. Strongly recommended
for all other classes.

Valid values: A GitHub username in @username format.

What happens if missing on an irreversible project: The rollback
readiness gate fails until this field is populated.

Guidance: This person must be available during and after the
deployment, genuinely empowered to act without seeking further
approval, and familiar with the procedure before deployment begins.
Do not name someone who has not explicitly agreed to this role.

---

## initiative.production_environment_pattern

  initiative:
    production_environment_pattern: "production|prod|release"

What it does: Tells the readiness gates which GitHub deployment
environments should trigger the full readiness checks.

Required: No. Defaults to production|prod|release.

Valid values: A regular expression matching your production
environment names. Examples:

  Match production, prod, or release (default):
    "production|prod|release"

  Also match live and customer environments:
    "production|prod|release|live|customer-.*"

  Match only exact name "production":
    "^production$"

Environments not matching this pattern bypass the readiness gates
and are treated as non-production.

---

## ledger block

The ledger block configures where permanent governance records are written.

---

## ledger.endpoint

  ledger:
    endpoint: "https://your-ledger-endpoint/v1/events"

What it does: The HTTPS address where governance record entries are sent.

Required: Yes.

What happens if empty or unreachable: Checks post a warning that the
record write failed. The act is still visible in GitHub activity, but
the permanent tamper-proof record outside GitHub will not be created.
This is a governance gap.

Guidance: Get this address from your governance layer administrator.
The endpoint must accept HTTP POST and return HTTP 201 on success.
See deploying-projops.md for compatible permanent record systems.

---

## ledger.auth_secret

  ledger:
    auth_secret: "LEDGER_API_KEY"

What it does: Names the GitHub secret containing the access credential
for your permanent record endpoint.

Required: Yes.

Guidance: Leave as LEDGER_API_KEY unless your administrator has told
you to use a different name. The actual credential is set at the
organisation level by the administrator.

---

## ledger.chain_id

  ledger:
    chain_id: "acme-invoice-reconciliation-2026"

What it does: A unique identifier for this project's chain of governance
records. Every record written for this project carries this identifier.

Required: Yes.

Valid values: Any string without spaces. Recommended format:
  organisation-projectname-year
  Example: acme-payroll-modernisation-2026

Critical: Once set, do not change. Changing breaks the record chain —
records before the change will no longer connect to records after it.

---

## ledger.signature_algorithm

  ledger:
    signature_algorithm: "dilithium3"

What it does: The cryptographic algorithm used to sign governance records.

Required: No. Defaults to dilithium3.

Guidance: Do not change without consulting your governance layer
administrator. Dilithium3 is a post-quantum security standard — designed
to remain secure even against future computing systems far more powerful
than those available today. See vocabulary.md for a plain-language
explanation of what post-quantum means.

---

## ledger.alert_channel

  ledger:
    alert_channel: "https://hooks.slack.com/services/your-webhook-url"

What it does: A notification address where governance alerts are sent —
exception records, bypass detections, overdue exceptions, and
repositories in alert state on the portfolio health report.

Required: No.

Valid values: Any HTTPS webhook URL. Supports Slack, Microsoft Teams,
and any service accepting a JSON POST.

Guidance: Strongly recommended for Tier 2 and above. Without an alert
channel, governance alerts only appear in GitHub Issues and the portfolio
health report. An alert channel ensures time-sensitive events reach the
right people promptly.

---

## portfolio block

Configures portfolio-level governance — how this project appears in
management visibility reports across all active projects.

---

## portfolio.org_label

  portfolio:
    org_label: "programme-customer-transformation"

What it does: A label identifying this repository as part of a specific
portfolio. The portfolio health aggregator uses this to find related
projects when generating reports.

Required: No, but strongly recommended for organisations running more
than one governed project.

Guidance: Use a consistent label format. Apply it to your repository
via Settings -> General -> Topics, or add it as a repository label.
All repositories in the same portfolio must use the same label.

Examples:
  programme-customer-transformation
  bu-finance-2026
  govops-tier-2
  strategic-initiative-alpha

---

## portfolio.health_report_schedule

  portfolio:
    health_report_schedule: "0 9 * * 1"

What it does: Sets when the portfolio health report runs.

Required: No. Default: every Monday at 09:00 UTC (0 9 * * 1).

Common schedules:
  Every Monday 09:00 UTC:     "0 9 * * 1"
  Every day 08:00 UTC:        "0 8 * * *"
  Every Friday 17:00 UTC:     "0 17 * * 5"
  First of month 09:00 UTC:   "0 9 1 * *"

---

## portfolio.portfolio_repo

  portfolio:
    portfolio_repo: "acme-corp/portfolio-governance"

What it does: The repository where portfolio health reports are posted
as issues for governance review.

Required: No.

Valid values: A GitHub repository in org/repo format.

Guidance: Set this to a repository that governance decision-makers
have access to. Reports are most useful when they go somewhere
decisions can be made based on them.

---

## governance block

Enables or adjusts individual governance check behaviour.

---

## governance.require_uncertainty_classification

  governance:
    require_uncertainty_classification: true

What it does: When true, blocks any project classified as ambiguity
from execution until the ambiguity resolution reference is populated.

Required: No. Default: true.

Guidance: Do not set to false. The ambiguity hard stop prevents starting
work on a contested objective — the most reliable path to delivering
the wrong thing. Disabling this removes a structural protection against
one of the most common and expensive project failure modes.

---

## governance.require_milestone_checkpoints

  governance:
    require_milestone_checkpoints: true

What it does: When true, blocks milestones from closing without a
completed governance checkpoint review.

Required: No. Default: true.

Guidance: Do not set to false for Tier 2 and above. Milestone reviews
are the primary mechanism for project sponsors to maintain genuine
accountability. Disabling this converts milestones from governance
checkpoints into calendar dates.

---

## governance.require_pir

  governance:
    require_pir: true

What it does: When true, blocks repository archiving without a completed
post-project review.

Required: No. Default: true.

Guidance: Do not set to false. The post-project review is how the
organisation learns from every project. A governance system that does
not improve from its own experience becomes disconnected from
operational reality over time.

---

## governance.require_asset_handover

  governance:
    require_asset_handover: true

What it does: When true, blocks repository archiving until all project
outputs have named owners who have confirmed receipt.

Required: No. Default: true.

Guidance: Do not set to false. Assets without owners drift and degrade.
This check enforces the minimum condition of valid project closure.

---

## governance.exception_expiry_days

  governance:
    exception_expiry_days: 14

What it does: Days before an open exception issue is flagged as overdue.

Required: No. Default: 14.

Valid values: Any integer from 1 to 90.

Guidance: 14 days suits most projects. Reduce to 7 for Tier 3 and above.
Do not exceed 30 — exceptions open longer than 30 days are no longer
exceptions; they are normalised governance gaps.

---

## governance.exception_pattern_threshold

  governance:
    exception_pattern_threshold: 3

What it does: The number of exceptions of the same type before the
portfolio health report flags it as a pattern requiring a standards
review rather than individual remediation.

Required: No. Default: 3.

Guidance: One exception is an event. Two could be coincidence. Three
is a pattern warranting examination of whether the standard itself is
the problem — not the team failing to comply.

---

## delivery_system block

Records the governed decision about how this project is being run.
For the full decision framework, see Section 10.5 of the practitioner guide.

---

## delivery_system.pattern

  delivery_system:
    pattern: "iterative"

What it does: The structural pattern of the delivery approach.

Required: Yes.

Valid values:

  staged-gate    - Work proceeds through phases with formal review
                   before each phase begins. Best for: fixed requirements,
                   high compliance obligation, low change tolerance.

  iterative      - Short cycles each producing a working output.
                   Best for: emerging requirements, discovery work,
                   epistemic uncertainty.

  parallel-track - Exploration and delivery run simultaneously.
                   Best for: complex transformations.

  hybrid         - A defined combination. Requires framework_description.

---

## delivery_system.framework

  delivery_system:
    framework: "scrum"

What it does: The specific delivery framework being used.

Required: Yes.

Valid values:

  scrum    - Iterative sprints with defined roles and ceremonies
  kanban   - Continuous flow with visual work management
  prince2  - Structured staged-gate with defined management products
  safe     - Scaled Agile Framework for large organisations
  internal - Organisation's own named framework (describe below)
  hybrid   - A combination of frameworks (describe below)

---

## delivery_system.framework_description

  delivery_system:
    framework_description: "Scrum at team level with staged-gate phase reviews at programme level"

What it does: Describes the specific framework when framework is
hybrid or internal.

Required: Yes when framework is hybrid or internal. Optional otherwise.

Guidance: Be precise. "Agile" is not a description. "Two-week Scrum
sprints with monthly programme-level staged-gate reviews at which the
sponsor confirms continuation authority" is a description.

---

## delivery_system.governance_cadence

  delivery_system:
    governance_cadence: "Monthly milestone review at end of every second sprint"

What it does: How delivery cycles connect to governance review checkpoints.

Required: Yes.

Guidance: Be specific. Vague cadences produce vague accountability.

Good: "Monthly milestone review at end of every second two-week sprint"
Good: "Staged-gate review at end of each phase — typically every 8 weeks"
Avoid: "Regular reviews" — how regular?
Avoid: "As needed" — governance that happens as needed happens when
        convenient, not when required.

---

## delivery_system.org_mandate

  delivery_system:
    org_mandate: "prince2"

What it does: The delivery framework the organisation has officially
mandated for projects of this type or tier.

Required: No. Leave blank if no mandate exists.

Guidance: Record the mandate even when this project follows it.
Having the mandate explicit makes the governance decision visible in
both the compliant and departing cases.

---

## delivery_system.deviation_from_mandate

  delivery_system:
    deviation_from_mandate: false

What it does: Whether this project departs from the organisational mandate.

Required: No. Default: false.

Valid values: true or false.

What happens if true: The check validates that deviation_approved_by
and deviation_rationale are both populated. Missing either fails the check.

Guidance: A departure is legitimate when the mandate does not fit the
project's governance needs. What makes it legitimate is that it is
recorded, justified, and approved — not that the team preferred a
different approach.

---

## delivery_system.deviation_approved_by

  delivery_system:
    deviation_approved_by: "@sponsor-username"

What it does: Who approved the departure from the mandate.

Required: Yes when deviation_from_mandate is true.

Valid values: A GitHub username in @username format.

Guidance: Must be the same authority who approves scope changes for
this project. A departure from the mandate carries the same governance
weight as a scope change.

---

## delivery_system.deviation_rationale

  delivery_system:
    deviation_rationale: "PRINCE2 staged-gate structure is incompatible
      with high epistemic uncertainty at initiation. Discovery-led
      iterative delivery is required before fixed phases can be defined."

What it does: Why the mandate is not appropriate for this project.

Required: Yes when deviation_from_mandate is true.

Guidance: Reference the four selection criteria from Section 10.5:
uncertainty type, compliance obligations, rate-of-change tolerance,
and organisational mandate. "The team preferred Scrum" is not a
governance rationale.

---

## runners block

  runners:
    default: "ubuntu-latest"

What it does: Which server type the governance workflow checks run on.

Required: No. Default: ubuntu-latest (GitHub's shared infrastructure).

When to change:
  - Your security policy requires governance workflows on
    self-hosted infrastructure (servers your organisation controls)
  - Your project needs to reach a permanent record endpoint on
    a private network

How to set up self-hosted runners: Go to your repository or organisation
Settings -> Actions -> Runners and follow the instructions. Then set
runners.default to the label you assigned that runner.

---

## Complete example

A filled-in govops.yml for a Tier 2 project:

  govops_version: "0.9"

  initiative:
    tier: 2
    decision_class: "conditional"
    ial_ceiling: 2
    ial_required_for_merge: 2
    ial_required_for_deploy: 2
    rollback_authority: "@alice-smith"
    production_environment_pattern: "production|prod"

  ledger:
    endpoint: "https://governance-records.acme.internal/v1/events"
    auth_secret: "LEDGER_API_KEY"
    chain_id: "acme-invoice-reconciliation-2026"
    signature_algorithm: "dilithium3"
    alert_channel: "https://hooks.slack.com/services/T00/B00/xxxx"

  portfolio:
    org_label: "programme-finance-transformation"
    health_report_schedule: "0 9 * * 1"
    portfolio_repo: "acme-corp/governance-oversight"

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
    governance_cadence: "Monthly milestone review at end of every second sprint"
    org_mandate: "safe"
    deviation_from_mandate: true
    deviation_approved_by: "@alice-smith"
    deviation_rationale: "SAFe programme increment planning assumes stable
      quarterly roadmaps. This project has high epistemic uncertainty
      requiring front-loaded discovery. Scrum at team level with monthly
      programme governance reviews provides flexibility while maintaining
      accountability visibility."

  runners:
    default: "ubuntu-latest"

---

Reference:
  adopting-the-layer.md    - Step-by-step connection guide
  deploying-projops.md     - Layer deployment guide for administrators
  vocabulary.md            - Plain-language guide to every term
  ial-out-of-band.md       - High identity assurance approval process
