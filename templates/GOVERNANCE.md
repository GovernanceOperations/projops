# Project Governance Record

<!-- govops-schema: governance-md v0.9 -->
<!-- This document is the governance identity record for this initiative. -->
<!-- It must be completed before any project work begins. -->
<!-- Every field is required. Incomplete records block the first -->
<!-- merge of changes to the main project branch. -->
<!-- Plain-language guide to every term: docs/vocabulary.md -->

<!-- CANONICAL RECORD POSITION: -->
<!-- This GOVERNANCE.md in the GovernanceOperations/projops repository -->
<!-- is the canonical governance record for the projops initiative itself. -->
<!-- For initiatives adopting this layer: GOVERNANCE.md lives at the root -->
<!-- of the initiative's primary repository (single-repository initiatives) -->
<!-- or at the root of the designated canonical repository (multi-repository -->
<!-- initiatives managed through a GitHub Project board). -->
<!-- One GOVERNANCE.md per initiative. The canonical repository must be -->
<!-- declared in the governance scope field below. -->

---

## Governance Scope

<!-- This section establishes what GitHub entities this initiative -->
<!-- spans and which repository holds the canonical governance record. -->

**Governance substrate:**
<!-- What GitHub entity is the primary delivery substrate for this initiative? -->
<!-- repository — a single GitHub repository -->
<!-- github-project — a GitHub Projects board (may span multiple repositories) -->
<!-- both — a GitHub Projects board with a designated canonical repository -->

**Canonical repository:**
<!-- The repository that holds this GOVERNANCE.md and is the primary -->
<!-- governance substrate. Format: org/repo -->
<!-- For single-repository initiatives: this repository. -->
<!-- For GitHub Project board initiatives: the repository designated as -->
<!-- the canonical record holder. All other repositories in the initiative -->
<!-- reference back to this one. -->

**GitHub Project board (if applicable):**
<!-- If this initiative is tracked through a GitHub Projects board, -->
<!-- provide the board URL. Format: https://github.com/orgs/ORG/projects/N -->
<!-- Leave blank for single-repository initiatives. -->

**Additional repositories in this initiative (if applicable):**
<!-- If this initiative spans multiple repositories beyond the canonical one, -->
<!-- list them here. Format: one org/repo per line. -->
<!-- Each additional repository should have a minimal GOVERNANCE.md that -->
<!-- points back to this canonical record. -->
<!-- Leave blank for single-repository initiatives. -->

---

## What This Project Is

**Project title:**
<!-- One sentence describing what this project will deliver. -->
<!-- Example: "Replace the manual invoice reconciliation process with an -->
<!-- automated matching system." -->

**Repository:**
<!-- The full name of this project in GitHub, in the format org/repo. -->
<!-- Example: acme-corp/invoice-reconciliation -->

**Authorising decision reference:**
<!-- Every project must trace back to a decision that authorised it. -->
<!-- This field holds the reference number of that decision, so anyone -->
<!-- reading this record can find the original approval. -->
<!-- Format: ORGANISATION-YEAR-SEQUENCE-CHECKSUM -->
<!-- Example: ACME-2026-0042-A3F9B2C1 -->
<!-- If you do not have this reference, the project has not been properly -->
<!-- authorised. Obtain it before proceeding. -->

**Decision type:**
<!-- How easily can the main outcome of this project be undone if it -->
<!-- turns out to be wrong? -->
<!-- reversible — can be undone quickly with minimal disruption -->
<!-- conditional — can be undone, but with significant effort -->
<!-- irreversible — cannot practically be undone once done -->
<!-- When in doubt, choose the more cautious category. Irreversible -->
<!-- projects require a higher level of authority to approve and a -->
<!-- tested rollback plan before going live. -->

**Governance tier:**
<!-- How much governance does this project require? -->
<!-- 1 — Low risk, limited scope, reversible. Streamlined governance. -->
<!-- 2 — Moderate scope, some compliance obligations. Standard governance. -->
<!-- 3 — High risk, regulatory exposure, irreversible change. Full governance. -->
<!-- 4 — Organisation-wide or federated scope. Maximum governance. -->
<!-- If unsure, discuss with the governance authority before proceeding. -->

**GovOps layer version:**
<!-- The version of the GovernanceOperations/projops layer this project -->
<!-- is running under. Use 0.9 unless a later version has been released. -->
<!-- 0.9 -->

---

## What This Project Is Trying to Achieve

**The problem being solved:**
<!-- What condition, gap, risk, or opportunity is this project responding to? -->
<!-- Be specific. "Improve efficiency" is not a problem statement. -->
<!-- "Customer onboarding takes 14 days and generates 40% of all service -->
<!-- complaints" is a problem statement. -->

**The outcome being targeted:**
<!-- What will be observably different when this project succeeds? -->
<!-- Not what will be built, but what will be different in the organisation. -->
<!-- Example: "Customer onboarding will take 3 days or fewer. Service -->
<!-- complaints related to onboarding will fall below 10%." -->

**The constraints this project must work within:**
<!-- What limits apply? Budget ceiling? Regulatory deadlines? -->
<!-- Systems that cannot be changed? Dependencies on other projects? -->
<!-- Staff who cannot be redeployed? List them explicitly. -->
<!-- Constraints discovered later create scope changes. Constraints -->
<!-- identified now shape the design. -->

**How success will be measured:**
<!-- How will the project sponsor know the authorising decision has been -->
<!-- satisfied? These must be verifiable observations, not aspirations. -->
<!-- "Customers report higher satisfaction" is an aspiration. -->
<!-- "Customer satisfaction score for onboarding rises above 85% as -->
<!-- measured by the quarterly survey" is verifiable. -->

---

## What Kind of Uncertainty Does This Project Face?

<!-- Before any project can begin execution, its uncertainty must be -->
<!-- classified. Different types require different responses. -->
<!-- Getting this wrong is the most reliable predictor of project failure. -->
<!-- Full explanation: docs/vocabulary.md and Section 2 of the practitioner guide. -->

**Uncertainty classification:**
<!-- Choose one: -->
<!-- aleatory — the outcome is inherently unpredictable. We cannot know -->
<!--   what will happen no matter how much information we gather. -->
<!-- epistemic — we do not know yet, but we could find out. Information -->
<!--   is missing but obtainable through structured discovery. -->
<!-- ambiguity — the objective itself is unclear or contested. Different -->
<!--   stakeholders have different pictures of what success looks like. -->
<!-- If the classification is ambiguity, this project CANNOT begin -->
<!-- execution until the field below is completed. -->

**Why this classification applies to this project:**
<!-- One paragraph explaining why the above classification is the right -->
<!-- one for this project. This rationale will be reviewed at milestones. -->

**Ambiguity resolution record:**
<!-- REQUIRED if classification is 'ambiguity'. -->
<!-- Leave blank for aleatory or epistemic classifications. -->
<!-- This field must contain a link to the completed alignment record -->
<!-- that confirms all project authorities agree on the objective. -->
<!-- The automated governance check blocks execution until this is populated. -->

---

## Who Has Authority Over This Project

<!-- These authorities must be named before the project begins. -->
<!-- "To be confirmed" is not acceptable — it means decisions will be -->
<!-- made without clear authority during delivery, which is where most -->
<!-- governance failures begin. -->

**Project sponsor:**
<!-- The person or role who approved this project and is ultimately -->
<!-- accountable for its outcome. Format: @github-username -->

**Approves scope changes:**
<!-- Who has authority to change what this project will deliver? -->
<!-- Format: @github-username -->
<!-- This should be the same as or senior to the project sponsor. -->

**Authorises exceptions to compliance requirements:**
<!-- Who can formally waive a regulatory or policy obligation during -->
<!-- delivery? This should be a different person from the project sponsor -->
<!-- to ensure independent oversight. Format: @github-username -->

**Authority to pause or close this project:**
<!-- Who can decide that this project should stop? -->
<!-- Format: @github-username -->

**Rollback authority:**
<!-- Who has the authority to invoke the rollback procedure — to undo -->
<!-- the project's changes if they cause problems after go-live? -->
<!-- This person must be named and available. -->
<!-- Format: @github-username -->

---

## What Obligations Does This Project Carry?

<!-- List every law, regulation, contract, or internal policy that -->
<!-- requires something of this project. Do not leave this section empty. -->
<!-- "We are not aware of any obligations" is a valid entry IF it has -->
<!-- been reviewed by the appropriate compliance or legal function. -->
<!-- Obligations discovered after design is complete produce expensive -->
<!-- rework or, worse, get quietly dropped. -->

| Obligation | Where it comes from | What the project must do to satisfy it | Who is responsible |
|---|---|---|---|
| <!-- Example: GDPR data minimisation --> | <!-- Data protection law --> | <!-- Ensure only the minimum necessary customer data is processed --> | <!-- @privacy-lead --> |

---

## How This Project Will Be Delivered

<!-- The way a project is run is a governance decision, not a default. -->
<!-- It must be chosen deliberately, recorded here, and approved. -->
<!-- Full explanation and selection criteria: Section 10.5 of the practitioner guide. -->

**Delivery pattern:**
<!-- How is the work structured? -->
<!-- staged-gate — the project passes through formal review checkpoints -->
<!--   before each phase. Best for: fixed requirements, high compliance -->
<!--   obligations. -->
<!-- iterative — the project works in short cycles, each producing a -->
<!--   working output. Best for: emerging requirements, discovery work. -->
<!-- parallel-track — exploration and delivery run simultaneously. -->
<!--   Best for: complex transformations. -->
<!-- hybrid — a defined combination of the above. Describe in the field below. -->

**Delivery framework:**
<!-- The specific practices being used: -->
<!-- scrum — iterative cycles (sprints) with defined roles and ceremonies -->
<!-- kanban — continuous flow with visual work management -->
<!-- prince2 — structured staged-gate methodology -->
<!-- safe — scaled agile for large organisations -->
<!-- internal — the organisation's own named framework (specify below) -->
<!-- hybrid — a defined combination (describe below) -->

**Framework description:**
<!-- Required if delivery framework is 'hybrid' or 'internal'. -->
<!-- Describe what practices are being used and how they are combined. -->

**Governance cadence:**
<!-- How often do formal governance reviews occur, and how do they -->
<!-- connect to the delivery cycle? Be specific. -->
<!-- Example: "Monthly governance milestone review at the end of every -->
<!-- second two-week sprint. Portfolio health check weekly." -->
<!-- Vague answer to avoid: "Agile with regular reviews." -->

**Why this delivery approach was chosen:**
<!-- Map the selection to the four criteria from Section 10.5: -->
<!-- 1. Uncertainty type: [aleatory/epistemic] → pattern selected: [pattern] -->
<!-- 2. Compliance obligation level: [high/medium/low] → implication: [text] -->
<!-- 3. How much can requirements change: [high/medium/low] → implication: [text] -->
<!-- 4. What the organisation has mandated: [framework name or 'none'] → applied as: [adopted/adapted/formally departed from] -->

**Approved by:**
<!-- Who approved this delivery system selection? -->
<!-- This is the same authority who approves scope changes. -->
<!-- Format: @github-username -->

**Departure from organisational standard:**
<!-- Does this project use a different approach from the organisation's -->
<!-- officially mandated framework? yes or no -->

**Why this project departs from the standard (if applicable):**
<!-- Required if the answer above is 'yes'. -->
<!-- Why is the standard approach not appropriate for this project? -->
<!-- Reference the selection criteria above. -->

**Who approved the departure (if applicable):**
<!-- Required if departure is 'yes'. -->
<!-- Format: @github-username -->

---

## Current Status

**Status:**
<!-- active — project is running -->
<!-- paused — project is temporarily stopped -->
<!-- closing — project is completing final governance steps -->
<!-- closed — project is complete -->

**Project opened:**
<!-- Date this project was formally initiated. ISO format: YYYY-MM-DD -->

**Expected completion:**
<!-- Date or condition under which this project will close. -->
<!-- Example: "2026-09-30" or "On completion of compliance certification." -->

**Last updated:**
<!-- Date this record was last updated, and by whom. -->
<!-- Format: YYYY-MM-DD by @github-username -->

---

<!-- govops-end -->
