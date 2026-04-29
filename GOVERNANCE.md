# Project Governance Record

<!-- govops-schema: governance-md v0.9 -->

<!-- CANONICAL RECORD POSITION: -->
<!-- This file is the canonical governance record for the -->
<!-- GovernanceOperations/projops initiative. It governs the projops -->
<!-- repository itself — the governance layer for all adopting initiatives. -->

---

## Governance Scope

**Governance substrate:**
repository

**Canonical repository:**
GovernanceOperations/projops

**GitHub Project board (if applicable):**
N/A

**Additional repositories in this initiative (if applicable):**
N/A

---

## What This Project Is

**Project title:**
GovOps GitHub Layer — the governance system for GitHub-hosted projects

**Repository:**
GovernanceOperations/projops

**Authorising decision reference:**
GOVOPS-2026-0001-P9R0J0P5

**Decision type:**
reversible

**Governance tier:**
1

**GovOps layer version:**
0.9

---

## What This Project Is Trying to Achieve

**The problem being solved:**
Projects running on GitHub have no governance layer. There is no
standardised way to ensure that projects have a clear mandate, that
significant decisions are recorded, that deployments are gated on
genuine readiness, that exceptions are tracked, or that projects close
with their outputs properly owned. Every team builds its own governance
infrastructure — or builds none at all.

**The outcome being targeted:**
Any GitHub repository can call this layer and immediately gain governance
coverage across all nine gaps between GitHub's native features and what
good governance requires — without building custom tooling and without
disrupting the team's existing workflow.

**The constraints this project must work within:**
GitHub's identity assurance ceiling is Level 2. No infrastructure beyond
GitHub Actions and a configured permanent record system is required for
basic operation. No mandatory dependency on any paid GitHub tier above Team.

**How success will be measured:**
A repository calling govops-suite@v0.9 and passing all eleven governance
checks satisfies the GitHub-substrate requirements of GOS §17.11.
The Ledger Bridge produces a complete, unbroken permanent record from
project creation to project closure.

---

## What Kind of Uncertainty Does This Project Face?

**Uncertainty classification:**
epistemic

**Why this classification applies:**
The overall requirements are clear — nine specific governance gaps must be
closed. The precise implementation details of some workflow edge cases,
permanent record endpoint compatibility patterns, and identity assurance
register integration will emerge through structured iteration. Information
that is currently missing is obtainable through implementation experience.

---

## Who Has Authority Over This Project

**Project sponsor:**
@GovernanceOperations

**Approves scope changes:**
@GovernanceOperations

**Authorises exceptions to compliance requirements:**
@GovernanceOperations

**Authority to pause or close this project:**
@GovernanceOperations

**Rollback authority:**
@GovernanceOperations

---

## What Obligations Does This Project Carry?

| Obligation | Where it comes from | What the project must do | Who is responsible |
|---|---|---|---|
| Implement Project Operations Discipline | GOS §17.11 | All eleven governance gaps addressed in workflow suite | @GovernanceOperations |
| Conform to Governance Record structure | GOS §6.6 | All workflow outputs use the six-field record structure | @GovernanceOperations |
| Apply security standards | GOS §11.2 | Ledger Bridge uses dilithium3 post-quantum signing by default | @GovernanceOperations |

---

## How This Project Will Be Delivered

**Delivery pattern:**
iterative

**Delivery framework:**
kanban

**Framework description:**
Continuous flow with explicit work-in-progress limits per stage.
Work moves through: Backlog → Specified → In Review → Done.
No fixed sprint cadence. Governance review at each version release.

**Governance cadence:**
Governance milestone review at each version release. Portfolio health
check monthly. Post-project review before each major version begins.

**Why this delivery approach was chosen:**
1. Uncertainty type: epistemic → iterative pattern indicated
2. Compliance obligation level: low (self-governed) → no staged-gate pressure
3. How much can requirements change: high (early-stage tool) → iterative preferred
4. Organisational mandate: none → no departure required

**Approved by:**
@GovernanceOperations

**Departure from organisational standard:**
no

---

## Current Status

**Status:**
active

**Project opened:**
2026-04-26

**Expected completion:**
On declaration that all nine governance gaps are fully addressed and
the v1.0 milestone is ratified.

**Last updated:**
2026-04-26 by @GovernanceOperations

---

<!-- govops-end -->
