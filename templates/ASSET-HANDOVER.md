# Project Asset Handover Record

<!-- govops-schema: asset-handover v0.9 -->

<!-- WHAT THIS IS: -->
<!-- This document is the formal record of everything this project -->
<!-- produced that the organisation will continue to use after the -->
<!-- project closes. -->
<!-- -->
<!-- Every process, system, dataset, policy, and document that came out -->
<!-- of this project is an "asset" — something the organisation now owns -->
<!-- and depends on. Each one needs a named owner who is accountable for -->
<!-- it, a description of how well it should perform, a version number, -->
<!-- a regular review schedule, and a location where it can be found. -->
<!-- -->
<!-- A handover without named owners is not a handover. It is an -->
<!-- abandonment with a completion ceremony. Assets without owners drift, -->
<!-- degrade, and eventually create the same problem the project was -->
<!-- trying to solve. -->
<!-- -->
<!-- The automated governance check blocks project archiving until this -->
<!-- document is complete and every named owner has confirmed receipt. -->

---

## Project Details

**Project:**
<!-- Repository full name: org/repo -->

**Handover date:**
<!-- Format: YYYY-MM-DD -->

**Handover approved by:**
<!-- The project sponsor confirming all outputs have been handed over. -->
<!-- Format: @github-username -->

---

## Asset Register

<!-- Complete one row for every output this project produced. -->
<!-- Do not leave any cell empty — if a field is genuinely not -->
<!-- applicable, write "N/A" and explain why in the notes column. -->
<!-- -->
<!-- Asset type options: -->
<!-- process — a business process or procedure -->
<!-- system — a software system, application, or technology service -->
<!-- dataset — a database, data file, or structured data collection -->
<!-- policy — a formal policy, standard, or regulatory compliance document -->
<!-- document — operational documentation, user guides, runbooks -->
<!-- model — an AI model, analytical model, or decision-support tool -->

| What it is | Type | Named owner | How well it must perform | Version | How often it is reviewed | Where to find it |
|---|---|---|---|---|---|---|
| <!-- Example: Automated invoice matching service --> | system | <!-- @username --> | <!-- 99.5% availability during business hours; critical faults resolved within 4 hours --> | 1.0 | quarterly | <!-- link to runbook --> |
| <!-- Example: Customer onboarding process (revised) --> | process | <!-- @username --> | <!-- Reviewed within 5 working days of any regulatory change affecting onboarding --> | 2.0 | semi-annual | <!-- link to process documentation --> |
| <!-- Example: Transaction risk scoring model --> | model | <!-- @username --> | <!-- Accuracy reviewed monthly; model retrained if accuracy falls below 94% --> | 3.1 | monthly | <!-- link to model documentation --> |

---

## Ownership Confirmation

<!-- Every named owner in the table above must confirm they have -->
<!-- received the asset(s) assigned to them and accept responsibility -->
<!-- for their ongoing governance. -->
<!-- -->
<!-- Do not close this document without all confirmations checked. -->

- [ ] <!-- @asset_owner_1 --> confirms ownership of <!-- asset name(s) -->
- [ ] <!-- @asset_owner_2 --> confirms ownership of <!-- asset name(s) -->

---

## Support After Handover

**Who do asset owners contact if they need help:**
<!-- For each type of support issue, who should be contacted and how -->
<!-- quickly should they expect a response? -->
<!-- Example: "Technical faults — @platform-team — 4-hour response during -->
<!-- business hours. Process questions — @operations-manager — 1 working day." -->

**Known limitations of what was handed over:**
<!-- Are there any known gaps, workarounds, or limitations in the assets -->
<!-- being handed over that the receiving owners must be aware of? -->
<!-- "None identified" is acceptable but must be written explicitly. -->

**What happens when these assets reach end of life:**
<!-- Who has authority to decommission each asset? -->
<!-- What process must be followed? -->
<!-- Who must be notified before decommissioning begins? -->
<!-- Assets decommissioned without governance are assets that may take -->
<!-- things with them that other parts of the organisation depend on. -->

---

<!-- govops-end -->
