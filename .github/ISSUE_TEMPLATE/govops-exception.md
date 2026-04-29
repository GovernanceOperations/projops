---
name: GovOps Governance Exception
about: Record a governance requirement that was bypassed or deferred
labels: govops-exception
---

<!-- govops-schema: exception v0.9 -->

<!-- WHAT THIS IS: -->
<!-- An exception record documents a formal departure from a governance -->
<!-- requirement — a quality standard waived, a compliance step deferred, -->
<!-- a required review overridden. -->
<!-- -->
<!-- Exceptions are not inherently wrong. Sometimes they are the right -->
<!-- operational call. What makes them acceptable is that they are: -->
<!-- recorded (so they can be reviewed), time-limited (so they cannot -->
<!-- quietly become permanent), and owned (so someone is accountable for -->
<!-- resolving them). -->
<!-- -->
<!-- An unrecorded exception is invisible governance debt. -->
<!-- This record exists to make exceptions visible, accountable, and -->
<!-- trackable. The automated governance check creates this record -->
<!-- automatically when a bypass is detected — but exceptions can also -->
<!-- be created manually when a requirement is being formally deferred. -->

---

## What Requirement Was Bypassed or Deferred?

**Project:**
<!-- Full repository name: org/repo -->

**Date and time:**
<!-- When did this exception occur? Format: YYYY-MM-DDTHH:MM:SSZ -->

**Person who bypassed the requirement:**
<!-- @github-username -->

**Requirement bypassed:**
<!-- Name the specific requirement. Examples: -->
<!-- "Required approval from compliance function before deployment" -->
<!-- "Operational readiness checklist — people readiness condition" -->
<!-- "Mandatory review by security team before merging changes" -->
<!-- "Documentation standard for process changes" -->

**Exactly what was overridden:**
<!-- Be precise. Not "security review" but "the govops-operational -->
<!-- readiness-gate check on production deployment #47, specifically -->
<!-- the compliance readiness condition." -->

---

## Why Was This Necessary?

**Operational reason for the bypass:**
<!-- "Urgency" alone is not an acceptable rationale. Urgency is a -->
<!-- description of pressure, not a reason for the bypass. -->
<!-- What specific operational condition made compliance with the -->
<!-- requirement impossible or harmful in this instance? -->
<!-- Example: "A critical production system failure affecting customers -->
<!-- required an emergency rollback that could not wait for the standard -->
<!-- 24-hour compliance review cycle." -->

**What governance gap has been created:**
<!-- What could now go wrong that the bypassed requirement was designed -->
<!-- to prevent? Be honest. This is not a criticism — it is the -->
<!-- information needed to manage the risk the exception has created. -->

---

## How Will This Be Resolved?

**What will be done to close the governance gap:**
<!-- What specific action will restore the governance requirement that -->
<!-- was bypassed? -->
<!-- Example: "Compliance review will be conducted retroactively within -->
<!-- 5 business days and documented in the project record." -->

**Expiry date:**
<!-- This exception must be resolved by this date. Maximum 30 days. -->
<!-- Format: YYYY-MM-DD -->
<!-- Overdue exceptions are automatically flagged in the portfolio -->
<!-- health report. They cannot be hidden or quietly extended. -->

**Owner:**
<!-- Who is responsible for resolving this exception? -->
<!-- Format: @github-username -->
<!-- Automatically assigned to the person who created the bypass, -->
<!-- unless reassigned here. -->

**Authority who approved this exception:**
<!-- Who authorised this bypass? Must be the person designated in -->
<!-- GOVERNANCE.md as having authority to approve compliance exceptions. -->
<!-- Format: @github-username -->
<!-- If this bypass was not authorised by anyone — that is itself a -->
<!-- governance issue that needs to be noted here. -->

---

## Closing This Exception

<!-- Do not close this issue until all three conditions below are met. -->

- [ ] The governance requirement has been restored or a formal permanent
  exception has been granted by the Oversight Body
- [ ] The remediation action above has been completed
- [ ] If this is the third or more exception of the same type on this
  project or portfolio — the pattern has been reported to the portfolio
  health process (it signals a standards review is needed, not an
  enforcement action)

<!-- govops-end -->
