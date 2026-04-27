# Vocabulary — Plain Language Guide to GovOps and Project Operations Terms

This document explains every term, acronym, and concept used across the
GovernanceOperations/projops repository and the Project Operations practitioner
reference. It is written for readers with no prior knowledge of project
management, governance, or technology. If you encounter a word or abbreviation
that is not here, please raise an issue.

---

## Why This Document Exists

Governance and project management have accumulated decades of jargon.
Most of it is shorthand for important ideas — but shorthand that excludes
anyone who was not already in the room when the shorthand was invented.
This document translates the shorthand back into plain language.

The technical terms are kept throughout the other documents because precision
matters — but every technical term now has a plain-language home here.

---

## A

**Accountability**
Being responsible for an outcome and answerable to others when it does or does
not happen. Governance exists to make accountability visible and enforceable —
not just assumed.

**Adapt step**
The ninth step in the governance execution cycle. After reviewing what happened,
the governance system incorporates what it learned — updating its understanding,
its risk picture, and its rules so that it does better next time. Without this
step, an organisation repeats its failures indefinitely.

**AIG — Automated Intelligence Governance**
The discipline responsible for governing what automated systems and artificial
intelligence tools are allowed to do within a governed organisation. AIG defines
the boundaries of machine authority and ensures no automated system makes
decisions that only a human should make. See also: *Bounded Capability*.

**Aleatory uncertainty**
Uncertainty that cannot be reduced no matter how much information you gather.
The weather next month, a regulatory change by a government body, a competitor's
decision — these outcomes are inherently variable. The right governance response
is not to plan them away but to set risk tolerances and prepare contingencies.
Plain language: *things you cannot know in advance, no matter how hard you try.*

**Ambiguity** (as a type of uncertainty)
When the objective of a project is unclear or contested — different people have
genuinely different ideas about what success looks like. This is not a risk to
be managed. It is a condition that must be resolved before any project begins.
Starting work on an ambiguous objective produces confident delivery of the
wrong thing.
Plain language: *everyone agrees there is a problem, but nobody agrees on what
solving it would look like.*

**API — Application Programming Interface**
A defined way for computer systems to communicate with each other. When a
governance workflow sends information to a permanent record system, it does so
through an API. You do not need to understand the technical details — what
matters is that APIs are the plumbing that connects different systems together.
Plain language: *a standard connection point between software systems.*

**Approve step**
The fifth step in the governance execution cycle. A human authority reviews the
proposed action and either approves or rejects it. This step cannot be
automated for significant decisions — a human must decide.

**Asset** (operational asset)
Any outcome of a project that continues to exist and be used after the project
closes. Processes, systems, datasets, policies, and documents are all assets.
An asset without a named owner is a liability — no one is accountable for it.
Plain language: *anything a project creates that an organisation continues to rely on.*

**Asset handover**
The formal transfer of a project's outputs to the people who will own and
maintain them after the project closes. A handover without a named owner,
a service commitment, and a maintenance plan is not a handover — it is an
abandonment with a filing ceremony.

**Audit trail**
A permanent, chronological record of every significant action taken on a
project — who did what, when, and why. The audit trail is not created after
the fact for inspections. It is created as events happen, because an action
not recorded is an action that cannot be verified.

---

## B

**Bounded Capability**
The defined limit of what an automated system or AI tool is permitted to do
without human approval. A system operating within its Bounded Capability is
working as governed. A system that exceeds it — intentionally or through
failure — has produced an authority violation.
Plain language: *the fence around what a machine is allowed to do on its own.*

---

## C

**CHANGELOG**
A running record of what has changed in a piece of software or a document
between versions. It exists so users can understand what is new, what has
been fixed, and what they need to do differently.

**Compliance**
Meeting the obligations imposed by laws, regulations, contracts, policies, or
standards. Compliance is not a box to tick at the end of a project. It is a
requirement that shapes what the project must do from the beginning.

**Compliance debt**
Compliance obligations that accumulate because they are deferred instead of
addressed during delivery. Like financial debt, compliance debt compounds —
the longer it is ignored, the more expensive it becomes to resolve.

**Constitutional Condition**
A fundamental rule in the GovOps governance system that cannot be broken under
any circumstances — not during a crisis, not for efficiency, not by exception.
There are three. Breaking any one of them is a governance failure regardless
of everything else going well.
Plain language: *the rules that hold no matter what.*

**Continuous Delivery**
An approach to software development and deployment where changes are made
available frequently and reliably, in small increments, rather than in large,
infrequent releases.

**COBIT**
Control Objectives for Information and Related Technologies. A governance
framework developed by ISACA for IT management and governance. One of several
established frameworks that organisations may already follow and that GovOps
sits above as a meta-governance layer.

---

## D

**Decision Architecture**
The explicit structure of who has authority to make which decisions during a
project — who approves scope changes, who can grant exceptions, who can pause
or close the initiative. Defined at initiation, not discovered during a crisis.

**Decision Class**
The category of a decision based on how easily it can be undone:
- *Reversible*: Can be undone quickly if it turns out to be wrong.
- *Conditional*: Can be undone, but with significant effort.
- *Irreversible*: Cannot be undone, or can only be undone at very high cost.
Irreversible decisions require more authority and more careful review than
reversible ones.

**Decision Contract**
The formal record of a governance decision — what was decided, why, who decided
it, what success looks like, and what happens if the decision produces unexpected
results. Think of it as the governance receipt for a significant decision.
Plain language: *a written record of a decision that captures why it was made
and what it is supposed to achieve.*

**Deviation** (from organisational mandate)
When a project team uses a different delivery approach from the one the
organisation has officially adopted. Deviations are not inherently wrong —
sometimes a mandate does not fit the project. But they must be recorded, justified,
and approved. A silent deviation is an ungoverned delivery system.

**DevOps**
The practice of integrating software development and IT operations teams so
that code can be developed, tested, and deployed faster and more reliably.
DevOps is a delivery culture and set of practices, not a single methodology.

---

## E

**Epistemic Field**
A map of what is known and how confidently it is known, across three regions:
- *Confirmed*: We know this and can verify it independently.
- *Disputed*: Different observers are seeing different things — we cannot agree
  on what is true.
- *Unknown*: We do not have the information to know.
Good governance decisions are honest about which region they are drawing from.
A decision presented as confirmed when its basis is actually disputed is a
governance failure.
Plain language: *a map of what we know, what we are unsure about, and what
we do not know at all.*

**Epistemic uncertainty**
Uncertainty that exists because information is missing — but could be obtained.
We have not mapped the process yet. We have not confirmed the regulatory
requirement. We have not verified the dependency. This type of uncertainty
should be reduced through structured discovery work before the project begins.
Plain language: *things we do not know yet, but could find out.*

**Exception**
A formal departure from a governance requirement — a quality standard waived,
a compliance step deferred, a documentation obligation postponed. Exceptions
are legitimate when properly authorised. Unrecorded exceptions are where
governance debt accumulates fastest.

**Execution cycle** (nine-step)
The repeating sequence through which every governance act moves:
Sense → Verify → Evaluate → Propose → Approve → Execute → Record → Review → Adapt.
Each step has a defined purpose. Skipping a step leaves a gap.

---

## F

**Formal record**
Any document or entry in the governance record system that has been officially
created as part of a governance act. Informal notes, emails, and verbal
agreements are not formal records. A formal record is traceable, attributable,
and permanent.

---

## G

**GitHub**
A platform where software development teams store, share, and collaborate
on code and projects. The GovOps GitHub Layer runs within GitHub, using
its built-in features to enforce governance requirements automatically.

**GitHub Actions**
An automation system built into GitHub that can run sequences of steps
automatically when certain events happen — like when a change is submitted
or a project reaches a milestone. The governance workflows in this repository
run as GitHub Actions.

**GitHub App**
A type of software integration that connects to a GitHub organisation or
repository with defined permissions. The GovOps Ledger Bridge is a GitHub App.

**Governance**
The system of rules, practices, and processes by which an organisation makes
decisions, exercises authority, and ensures accountability. Good governance
does not slow organisations down — it prevents the rework, legal exposure,
and trust damage that come from operating without it.
Plain language: *how an organisation makes sure the right people make the
right decisions in ways that can be explained and verified.*

**Governance cadence**
How often formal governance reviews occur during a project, and how those
reviews connect to the project's delivery cycle. A project with weekly delivery
sprints but monthly governance reviews has three ungoverned weeks between
every accountability checkpoint.

**Governance Capacity**
A measure of how capable an organisation's governance system is. It is
calculated from four components: Fidelity (commitment to stakeholders),
Capacity to Act (ability to make decisions stick), Institutional Integrity
(structural soundness), and Adaptive Transparency (honest self-awareness).
If any component is near zero, the whole measure is near zero — governance
cannot be strong in three areas and weak in one.

**Governance Event**
The fundamental unit of the governance record. Every significant governance
action — a decision made, an approval given, an exception granted, a milestone
reviewed — produces a Governance Event. It has six required fields: identity
(a unique ID), timestamp (when it happened), type (what kind of event),
payload (the full content), signature (proof of who produced it), and
integrity reference (a link to the preceding event in the chain).
Plain language: *a permanent, tamper-proof record entry for a governance action.*

**Governance Operations Specification (GOS)**
The master rulebook that defines what governance means in practice for any
organisation adopting the GovOps framework. It specifies the rules, the
structures, the measurement methods, and the operational disciplines.
When this document refers to `[GOS §X.Y]`, it means "see section X.Y of
the master rulebook for the precise requirement."

**Governance theatre**
The condition where governance processes are being followed in form but not
in substance — meetings are held, documents are produced, checkboxes are
ticked — but no real accountability is being exercised. Governance theatre
is more dangerous than no governance at all, because it creates the
appearance of accountability without its reality.
Plain language: *going through the motions of governance without actually
governing anything.*

**GovOps**
Short for Governance + Operations. The framework that integrates how an
organisation governs itself with how it operates — so that governance is
not a separate department or periodic audit, but a practice woven into
every operational decision.

**GOVERNANCE.md**
A specific document maintained at the root of every governed project
repository. It is the governance identity document for the project —
recording what the project is, who has authority over it, what its
obligations are, and what delivery approach it is using.

---

## H

**Handover**
See *Asset handover*.

**Hypercare**
The period immediately after a project's outputs go live, during which the
project team remains available to support the receiving teams and confirm
that the new capability is working as intended. The minimum period for
governance-critical capabilities is thirty days.

---

## I

**IAL — Identity Assurance Level**
A measure of how certain we are that a person approving a decision is who
they claim to be:
- Level 1: Basic login — we know someone is using this account.
- Level 2: Verified membership and two-factor authentication — we are
  reasonably certain this is the right person.
- Level 3: Hardware security key linked to a verified identity — high
  confidence.
- Level 4: Physical presence verification — the highest level of certainty.
On GitHub, the maximum achievable level is 2. Higher levels require
out-of-band verification processes.
Plain language: *how confidently we can say "yes, this is definitely the
right person making this decision."*

**IAL Register**
The organisation's record of each person's verified Identity Assurance Level.
Maintained by the governance authority and read by the identity check workflow
to confirm that approvers meet the required level for each decision type.

**Immutable Ledger**
A permanent record system where entries can be added but never modified or
deleted. Once a governance action is recorded, the record cannot be changed,
not even by an administrator. Any attempt to alter a record is itself detectable.
Plain language: *a record book where nothing can ever be erased or changed —
only added to.*

**ITIL**
Information Technology Infrastructure Library. A framework for IT service
management. One of several established frameworks that organisations may
follow and that GovOps sits above.

**ISO 27001**
An international standard for information security management. Organisations
certified to ISO 27001 have demonstrated that their information security
practices meet defined international standards.

---

## J

**JSON**
JavaScript Object Notation. A structured text format that computers use to
exchange information. The governance schemas in this repository are written
in JSON. You do not need to understand JSON to use the governance templates —
it is used by the automated systems working behind the scenes.
Plain language: *a standard format computers use to share structured information.*

---

## K

**Kanban**
A visual workflow management method. Work is represented as cards on a board
that move through stages (for example: To Do → In Progress → Done). Kanban
emphasises continuous flow and limiting the amount of work in progress at
any one time.

---

## L

**Ledger Bridge**
The component that connects GitHub's activity stream to the organisation's
permanent governance record. It converts GitHub events (a pull request merged,
a milestone closed, a deployment completed) into Governance Events and writes
them to the Immutable Ledger.
Plain language: *the translator that turns GitHub activity into permanent
governance records.*

---

## M

**Mandate statement**
A clear, agreed description of what a project is trying to achieve — the
problem being solved, the outcome being targeted, the constraints being
accepted, and how success will be measured. Without a mandate statement,
a project has no authoritative definition of done.

**Memory Layer**
The permanent record layer in the GovOps governance architecture. Every
significant governance action is recorded here. It is the organisational
memory — not stored in individual people's email inboxes or desk drawers,
but in a structured, permanent, auditable system.
Plain language: *the organisation's permanent, searchable record of everything
that happened in governance.*

**Meta-Evolution**
The process by which the governance system improves itself over time.
Governance frameworks that cannot be updated become disconnected from
operational reality. Meta-Evolution is the structured, governed process
for changing governance rules, standards, and processes — so that the
changes are themselves accountable.
Plain language: *how the governance rulebook gets updated, in a way that
is itself governed.*

**Methodology mismatch**
When the delivery approach a project is using is not appropriate for the
type of work the project is actually doing. A compliance programme being
run like an exploratory innovation project, or a complex strategic
transformation being run on two-week sprints — both are methodology
mismatches with predictable failure patterns.

**Milestone**
A defined checkpoint in a project at which formal progress is reviewed,
decisions are made, and the project's authority to continue is confirmed.
A milestone review that produces no decisions is not a governance event —
it is a status update dressed as governance.

---

## N

**Non-compensatory**
A property of the Governance Capacity formula: strength in one area cannot
make up for weakness in another. An organisation with excellent processes,
a capable team, and strong oversight, but whose leadership is not genuinely
committed to stakeholder welfare — has near-zero governance capacity overall.
This is also true of project disciplines: excellence in ten disciplines does
not compensate for a complete gap in an eleventh.

---

## O

**Operational asset**
See *Asset*.

**Operational readiness**
The condition in which an organisation is genuinely ready to receive, operate,
and support what a project has delivered. There are five conditions: process
readiness, technology readiness, people readiness, compliance readiness, and
rollback readiness. All five must be confirmed before a project can be declared
complete.

**Oversight Body**
An independent group with authority to investigate, declare, and require the
correction of governance failures. It is structurally independent of the
organisation it oversees — meaning it cannot be controlled or silenced by
those whose governance it is reviewing.

---

## P

**PIR — Post-Implementation Review**
A structured review carried out after every project closes, examining what
worked, what failed, what produced unnecessary friction, and what should be
different next time. The PIR is not a blame exercise — it is the mechanism
by which the organisation gets better at governance with each project it runs.
Plain language: *a structured "what did we learn" exercise that feeds into
making the next project better.*

**Plain language**
Language that is clear, direct, and understandable to its intended audience
without requiring specialist knowledge. This document attempts to practise
what it preaches.

**Portfolio**
The complete set of active projects and initiatives being run by an organisation
at any given time, managed as a collective whole rather than as independent
individual efforts. Portfolio governance asks whether the right projects exist,
are resourced correctly, and remain aligned with organisational strategy.

**Portfolio health report**
A regular report on the state of the entire portfolio of projects — aggregate
risk levels, compliance status, resource utilisation, delivery progress, and
any projects requiring governance intervention. A portfolio health report
showing only good news is not a governance instrument. It is a communications
exercise.

**Post-quantum cryptography**
Encryption methods designed to remain secure even against quantum computers,
which are powerful enough to break many current encryption standards. The
GovOps framework specifies post-quantum cryptography for its permanent record
system because governance records must remain secure for decades.
Plain language: *security locks designed to hold even against the computers
of the future.*

**PRINCE2**
Projects IN Controlled Environments. A structured project management framework
used widely in government and large enterprises, particularly in Europe and
Australia. One of several established frameworks that organisations may follow
and that GovOps sits above.

**ProjOps — Project Operations**
Short for Project Operations. The discipline of running initiatives as governed,
accountable operational entities — where governance is embedded in how work
is done, not applied to it afterward.

**Pull request**
In GitHub, a request to merge a set of changes into the main version of a
project. Pull requests are reviewed and approved before changes are accepted.
In a governed repository, every merged pull request produces a Governance Event.

---

## R

**RACI**
A tool for defining who is Responsible (does the work), Accountable (owns
the outcome), Consulted (provides input), and Informed (receives updates)
for each decision or activity. RACI is an accountability structure, not
project management boilerplate.

**Repository**
In GitHub, a storage location for a project's files, history, and
configuration. When a repository is governed under the GovOps layer, it
becomes a governed initiative vehicle — with a governance identity, a
permanent record, and automated governance checks.

**Reversibility**
The degree to which a decision or action can be undone. Reversible decisions
can be corrected quickly if they turn out to be wrong. Irreversible decisions
cannot be undone, or can only be undone at very high cost. Irreversible
decisions require more authority and more careful review.

**Rollback**
The act of undoing a change and restoring the system to its previous state.
Before any significant deployment, the team must confirm that a rollback
procedure exists, is tested, and has a named person authorised to invoke it.
Plain language: *the ability to undo a change if it causes problems.*

---

## S

**SAFe — Scaled Agile Framework**
A framework for applying agile delivery practices across large organisations
with many teams working on related initiatives. SAFe adds structure and
coordination on top of team-level agile practices.

**Scrum**
An iterative delivery framework that organises work into short, fixed cycles
called sprints (usually two weeks). At the end of each sprint, a working
output is produced and reviewed. Scrum includes defined roles (Product Owner,
Scrum Master, Development Team) and ceremonies (sprint planning, daily
standups, reviews, and retrospectives).

**SHA-256**
A cryptographic hashing algorithm that produces a unique fixed-length
"fingerprint" of any piece of information. If even one character in a
document changes, the fingerprint changes completely. Used in the governance
record system to detect tampering.
Plain language: *a mathematical fingerprint that reveals if anything has
been changed.*

**SLA — Service Level Agreement**
A formal commitment about how well a service will perform — its availability,
response time, and what happens when it falls short. Every operational asset
handed over at the end of a project needs an SLA so the receiving team
knows what standard they are accountable for maintaining.
Plain language: *a written promise about how good a service will be and what
happens if it falls short.*

**Sovereign Decision**
A decision that only a human being may make. It cannot be delegated to an
automated system, regardless of urgency, convenience, or the system's
capability. Any decision that materially affects people's rights, resources,
or welfare is a Sovereign Decision.
Plain language: *a decision too important to leave to a machine.*

**Sponsor** (initiative sponsor)
The person or body who authorises and is accountable for an initiative — who
commits the resources, sets the mandate, and is ultimately responsible for
the outcome.

**Sprint**
A short, fixed period of work — typically two weeks — used in iterative
delivery frameworks. At the end of a sprint, a working output is produced
and reviewed.

**Staged-gate**
A delivery approach where a project passes through formal review checkpoints
(gates) before proceeding to the next phase. At each gate, the project must
demonstrate it has met defined criteria. Staged-gate governance is well-suited
to projects with high compliance obligations or fixed-scope requirements.

---

## T

**Tier** (governance tier)
The level of governance intensity applied to a project, based on its risk,
complexity, and compliance obligations:
- Tier 1: Low risk, limited scope, simple governance.
- Tier 2: Moderate risk, cross-functional scope, standard governance.
- Tier 3: High risk, significant compliance, full governance.
- Tier 4: Federated or organisation-wide scope with the highest governance requirements.
Tier assignment is a governance decision — not a discretionary label.

---

## U

**UUID**
Universally Unique Identifier. A long string of characters (like
`550e8400-e29b-41d4-a716-446655440000`) that is statistically guaranteed
to be unique. Every Governance Event gets a UUID as its identity — ensuring
no two events can be confused with each other.
Plain language: *a unique ID number that will never be used twice.*

---

## V

**Vocabulary** (this document)
A plain-language reference for every term, acronym, and concept used in the
GovOps Project Operations framework. It exists so that executives, business
leaders, and curious readers without technical or project management background
can understand what they are reading and what it means for their organisation.

---

## W

**Webhook**
An automatic notification that GitHub sends to another system when something
happens in a repository — a change is made, a milestone is reached, a
deployment occurs. The Ledger Bridge listens for these notifications and
converts them into governance records.
Plain language: *an automatic notification that fires when something
important happens.*

**WIP — Work in Progress**
Work that has been started but not yet completed. Limiting work in progress
prevents teams from starting too many things simultaneously and finishing
nothing.

---

## Y

**YAML**
A human-readable configuration file format. The governance workflow files
in this repository are written in YAML. You do not need to write YAML to
use this repository — but if you see a `.yml` file, it is a configuration
or workflow definition in this format.
Plain language: *a structured text file that tells automated systems what
to do.*

---

## Abbreviation Quick Reference

| Abbreviation | Full term | Plain language |
|---|---|---|
| AIG | Automated Intelligence Governance | Governing what machines are allowed to do |
| API | Application Programming Interface | A standard connection between software systems |
| CHANGELOG | — | Record of what changed between versions |
| GOS | Governance Operations Specification | The master governance rulebook |
| GovOps | Governance + Operations | Governance woven into operations |
| IAL | Identity Assurance Level | How certain we are someone is who they say |
| ISO 27001 | International security management standard | — |
| ITIL | IT Infrastructure Library | IT service management framework |
| JSON | JavaScript Object Notation | A data format computers use to share information |
| PIR | Post-Implementation Review | Structured learning review after a project closes |
| PRINCE2 | Projects IN Controlled Environments | Structured project management framework |
| ProjOps | Project Operations | Running initiatives as governed entities |
| RACI | Responsible, Accountable, Consulted, Informed | Accountability mapping tool |
| SAFe | Scaled Agile Framework | Large-scale agile delivery framework |
| SHA-256 | Secure Hash Algorithm 256-bit | Mathematical tamper-detection fingerprint |
| SLA | Service Level Agreement | A formal service performance commitment |
| SOX | Sarbanes-Oxley Act | US financial reporting compliance law |
| UUID | Universally Unique Identifier | A unique ID number used once only |
| WIP | Work in Progress | Work started but not yet finished |
| YAML | YAML Ain't Markup Language | A configuration file format |

---

*This vocabulary is maintained in the GovernanceOperations/projops repository.*
*Raise an issue to suggest additions, corrections, or plain-language improvements.*
