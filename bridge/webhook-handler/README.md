# Webhook Handler

Receives GitHub webhook events, validates signatures, and translates
each event into a Governance Event for the ledger client.

## Interface

Input:  Raw GitHub webhook payload + X-Hub-Signature-256 header
Output: Governance Event JSON conforming to GOS §6.6 six-field structure

## Translation Logic

See the Event Translation Map in bridge/README.md.

Each handler module is named for the GitHub event type it processes:
- push.js
- pull_request.js
- pull_request_review.js
- issues.js
- milestone.js
- deployment.js
- workflow_run.js
- check_run.js
- repository.js
- member.js

## Chain State

The handler maintains a chain state store recording the certification_hash
of the last event written per repository, used to populate the
integrity_reference field of each new event.
