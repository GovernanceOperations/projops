---
name: GovOps Milestone Governance Checkpoint
about: Required governance review before a project milestone can close
labels: milestone-checkpoint
---

<!-- govops-schema: milestone-review v0.9 -->

<!-- WHAT THIS IS: -->
<!-- A milestone checkpoint is not a progress report. It is a formal -->
<!-- accountability review at which the project sponsor confirms that -->
<!-- the project continues to meet its original mandate, that risks are -->
<!-- acceptable, and that the governance record is current. -->
<!-- -->
<!-- A milestone that closes without this review has no governance standing. -->
<!-- The automated check will block the milestone from closing until this -->
<!-- issue is completed and closed. -->
<!-- -->
<!-- A milestone review that produces no decisions is not a governance event. -->
<!-- At minimum, a formal continuation confirmation must be recorded. -->

---

## Which Milestone Is This Review For?

**Milestone:**
<!-- The milestone title and number. Example: "Phase 1 Complete (#3)" -->

**Review date:**
<!-- When was this review conducted? Format: YYYY-MM-DD -->

**Conducted by:**
<!-- Who conducted this review? Must be the project sponsor or a person -->
<!-- they have formally delegated this to. Format: @github-username -->

---

## Is This Project Still on Track for Its Original Purpose?

**Is the project's work still aligned with its original mandate?**
<!-- yes — the project is delivering what was originally authorised -->
<!-- partially — there have been changes, explained below -->
<!-- no — the project has diverged significantly, explained below -->

**If partially or no — what has changed and why:**
<!-- Describe what is different from the original mandate. -->
<!-- If this requires a formal scope change, record that change as a -->
<!-- decision below and update the GOVERNANCE.md record accordingly. -->
<!-- Undocumented scope change is one of the most common sources of -->
<!-- governance failure. -->

---

## Are Risks Still Acceptable?

**Current risk level:**
<!-- Unchanged from when the project began -->
<!-- Increased — new risks have emerged or existing risks have grown -->
<!-- Decreased — risks have been successfully reduced -->

**Risks currently above the accepted threshold:**
<!-- List any risks that have become real or have grown beyond what -->
<!-- was accepted at the project's start. -->
<!-- If none: write "No risks above threshold as of this review." -->

---

## Is the Delivery Approach Still Right for This Project?

<!-- The way a project is run should be reviewed at every milestone. -->
<!-- A delivery approach that made sense at the start may no longer fit. -->
<!-- If it has stopped working, the right response is to change it -->
<!-- through a governance decision — not to quietly abandon it. -->

**Is the current delivery approach still appropriate?**
<!-- yes — the delivery approach is working -->
<!-- no — it is no longer appropriate, see below -->
<!-- under review — concerns have been identified, see below -->

**If no or under review — which of these warning signs is present:**
<!-- governance theatre — reviews produce no decisions; the same problems -->
<!--   appear repeatedly without resolution; forms are filled in but -->
<!--   nothing is actually being governed -->
<!-- compliance building up — compliance steps that should happen each -->
<!--   cycle are being deferred to the end of the project -->
<!-- scope instability — requirements are changing significantly in a -->
<!--   project that was supposed to have fixed scope -->
<!-- review misalignment — delivery cycles are happening much more often -->
<!--   than governance reviews, leaving long gaps between accountability -->
<!--   checkpoints -->
<!-- other — describe: -->

**If the delivery approach needs to change:**
<!-- Describe the proposed change. It becomes a formal decision below. -->
<!-- Changing the delivery approach has the same governance standing as -->
<!-- changing the project's scope — it requires the same authority. -->
<!-- From: [current pattern/framework] -->
<!-- To: [proposed pattern/framework] -->
<!-- Reason: -->

---

## Is the Governance Record Current?

**Is the project's GOVERNANCE.md record up to date?**
<!-- yes -->
<!-- no — update it before closing this checkpoint -->

**How many open exceptions are there?**
<!-- Count of open exception issues (labelled govops-exception): -->
<!-- Are any past their expiry date? yes / no -->
<!-- Overdue exceptions are flagged in the portfolio health report as -->
<!-- a governance gap. -->

---

## Decisions Made at This Review

<!-- A milestone review that produces no decisions is not a governance event. -->
<!-- Record a substantive decision below, OR record a formal continuation. -->
<!-- Both options are equally valid — what matters is that something is recorded. -->

- [ ] **Decision recorded:**
  <!-- Describe the decision made at this review. -->
  <!-- If no material decision was required, state the continuation explicitly: -->
  <!-- "Confirmed: this project continues within its original mandate, within -->
  <!-- its accepted risk levels, and within the authority granted at initiation. -->
  <!-- No scope change. No exceptions granted. Review date: YYYY-MM-DD." -->

- [ ] **Continuation confirmed** (if no decision was required):
  Confirmed by: <!-- @github-username (project sponsor or delegate) -->

**Actions arising from this review:**
<!-- List any specific actions, with owners and due dates. -->
<!-- "None" is a valid entry if no actions arose. -->

---

## Closing This Review

<!-- Do not close this issue until at least one of the two checkboxes above -->
<!-- is marked as complete. Closing this issue allows the milestone to close. -->

<!-- govops-end -->
