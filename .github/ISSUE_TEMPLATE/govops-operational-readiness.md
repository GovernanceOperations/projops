---
name: GovOps Operational Readiness Checklist
about: Five conditions that must be confirmed before any production deployment
labels: operational-readiness
---

<!-- govops-schema: operational-readiness v0.9 -->

<!-- WHAT THIS IS: -->
<!-- Before any project output goes live in the production environment, -->
<!-- five readiness conditions must be confirmed by named people. -->
<!-- -->
<!-- A project may be technically complete — all the development work done, -->
<!-- all the tests passed — and still fail if the organisation is not -->
<!-- genuinely ready to operate what was built. This checklist ensures -->
<!-- that "done" means operationally ready, not just technically finished. -->
<!-- -->
<!-- The automated governance check will block the deployment until all -->
<!-- five conditions are checked and confirmed. -->
<!-- Do not close this issue until all five boxes are marked. -->

---

## Which Deployment Is This For?

**Project:**
<!-- Repository full name: org/repo -->

**Deployment target:**
<!-- The environment this deployment is going to. -->
<!-- Example: production, prod, release -->

**What is being deployed:**
<!-- What change is going live? Reference the pull request, release tag, -->
<!-- or a plain description. Example: "Invoice matching service v1.0 — -->
<!-- PR #47 and #51, release tag v1.0.0" -->

**Date of this checklist:**
<!-- Format: YYYY-MM-DD -->

---

## The Five Readiness Conditions

<!-- Each condition must be confirmed by a named person before this -->
<!-- issue can be closed and the deployment can proceed. -->
<!-- -->
<!-- "Confirmed" means the named person has personally verified the -->
<!-- condition is true — not that they have been told it is true. -->

---

**Condition 1 — Process readiness**

- [ ] Confirmed by @<!-- username -->

The new or changed processes are documented, approved, and accessible
to the people who will use them. Process owners have walked through
the changes — not just received documentation.

*Not confirmed means:* People will discover how things work by trial and
error after go-live, which is both a quality risk and a compliance risk.

---

**Condition 2 — Technology readiness**

- [ ] Confirmed by @<!-- username -->

The system has been tested in an environment that matches the live
production environment as closely as possible. Documentation for
supporting the system (runbooks, troubleshooting guides) is written
and accessible. The team responsible for ongoing support has been
confirmed capable — not just informed of their new responsibility.

*Not confirmed means:* Support teams discover how to support the system
while customers are affected by its failures.

---

**Condition 3 — People readiness**

- [ ] Confirmed by @<!-- username -->

The people whose work is affected by this change understand what is
changing, why, and what is now expected of them. Training has been
delivered and comprehension has been confirmed — not just attendance
at a training session recorded.

*Not confirmed means:* People revert to the old way of working because
they were not genuinely prepared for the new one.

---

**Condition 4 — Compliance readiness**

- [ ] Confirmed by @<!-- username -->

Every regulatory notification, approval, or certification required
before this change goes live has been obtained. The evidence is
documented and accessible in the project record.

*Not confirmed means:* The deployment may constitute a compliance breach
from the moment it goes live.

---

**Condition 5 — Rollback readiness**

- [ ] Confirmed by @<!-- username -->

A procedure exists for undoing this change if it causes problems.
The procedure has been tested. The person authorised to invoke it
is named and available.

Rollback procedure location: <!-- link to rollback workflow or document -->
Person authorised to invoke rollback: @<!-- username -->
Estimated time to complete a rollback: <!-- e.g. less than 15 minutes -->

**Important:** If no rollback procedure exists, this deployment cannot
be undone once it goes live. That makes it an irreversible decision under
the governance specification — and irreversible decisions require a higher
level of authority review before they can proceed. If this applies,
confirm that the appropriate authority has approved the deployment
*knowing* it cannot be undone.

*Not confirmed means:* If this deployment causes problems, the only
options are forward or chaos.

---

## After Go-Live — Hypercare Plan

**Hypercare period:**
<!-- The period during which the project team remains available to -->
<!-- support the receiving teams after go-live. -->
<!-- Minimum 30 days for governance-critical capabilities. -->
<!-- State the end date: YYYY-MM-DD -->

**Hypercare owner:**
<!-- Who is responsible for the hypercare period and for confirming -->
<!-- stability at its end? Format: @github-username -->

**What must be true before hypercare closes:**
<!-- What specific conditions confirm that the deployed capability is -->
<!-- stable and the receiving teams are genuinely independent? -->

---

## Closing This Checklist

<!-- Close this issue only when all five conditions above are confirmed. -->
<!-- Closing this issue signals to the deployment governance check that -->
<!-- the deployment may proceed. -->

<!-- govops-end -->
