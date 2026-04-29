# The GovOps Ledger Bridge

## What this does

The Ledger Bridge is the component that makes governance records permanent.

GitHub keeps records of what happens in a repository — changes merged,
milestones reached, deployments completed. But those records can be modified
by administrators. They can be deleted if the organisation deletes its
GitHub account. They are not held to the standard a governance record requires.

The Ledger Bridge solves this by listening to everything that happens in
GitHub, converting each significant event into a governance record entry,
and writing it to a permanent record system outside of GitHub — one that
is designed so that entries can never be changed or deleted, even by
system administrators.

Plain language: *it is the mechanism that ensures "what happened on this
project" can always be answered honestly, even years later, even under
adversarial scrutiny.*

## How it works

```
Something happens in GitHub
(a change is merged, a milestone closes, a deployment goes live)
        │
        ▼
The Ledger Bridge receives a notification from GitHub
        │
        ▼
It translates the GitHub event into a governance record entry
(six required fields: identity, timestamp, type, content,
 security signature, and a link to the previous entry in the chain)
        │
        ▼
It signs the entry with a cryptographic signature
(a mathematical seal that proves the entry has not been altered)
        │
        ▼
It writes the entry to the permanent record system
(append-only: entries can be added but never changed or removed)
```

## What the permanent record system must be able to do

The organisation provides the permanent record system. It must satisfy
four requirements:

| Requirement | What it means |
|---|---|
| **Append-only** | Entries can be added. Nothing can be changed or deleted. Not even by administrators. |
| **Tamper-evident** | Any attempt to alter an existing entry is detectable, because each entry is mathematically linked to the one before it. |
| **Auditable** | The full history of any project is queryable by anyone with authorised access — without needing the cooperation of the people who created the records. |
| **Reproducible** | Given the same inputs and the same security keys, the same records are produced. This allows independent verification that records are genuine. |

Compatible storage systems include: cloud object storage with
immutability locks (AWS S3 Object Lock, Azure Blob immutability policy,
Google Cloud Storage retention policy), specialist compliance audit
log systems, and blockchain-anchored audit records.

## What events it records

**Repository-level events:**

| What happens in GitHub | Type of governance record created |
|---|---|
| A change is merged to the main branch | "execute" — a governed execution act |
| A pull request review is approved | "approval" — an authority approval |
| A governance exception is created | "alert" — a governance gap has been created |
| A milestone closes (with checkpoint) | "decision" — an accountability decision was made |
| A deployment is created | "execute" — an act of execution |
| A deployment completes | "record" — the result is recorded |
| A governance workflow completes | "record" — the governance act is recorded |
| An admin overrides a required check | "alert" — a governance bypass has occurred |
| A repository is archived | "record" — the repository-level project is formally closed |
| A team member leaves who owns an asset | "alert" — an asset may now be unowned |

**GitHub Projects events (organisation-level project boards):**

| What happens in GitHub Projects | Type of governance record created |
|---|---|
| A GitHub Project is created | "observation" — a new initiative planning board is established |
| A GitHub Project is closed | "record" — the project board lifecycle is complete; triggers PIR and asset handover checks |
| A GitHub Project is reopened | "alert" — a closed initiative has been reopened; requires governance review |
| An item is added to a GitHub Project | "record" — scope item added to the initiative |
| An item is converted to an issue | "record" — planned work item formalised |
| An item is archived in a GitHub Project | "record" — work item removed from active scope |
| Status field changed on a project item | "observation" — delivery status change recorded |

The GitHub Projects event coverage ensures that initiatives managed through
project boards — including multi-repository initiatives — produce the same
complete permanent record as repository-level initiatives.

## Setting it up

**Step 1:** Create a GitHub App in your organisation using the
configuration in `app.yml`. This registers the bridge as an
authorised component of your governance system.

**Step 2:** Set the following environment variables on the server
where the bridge will run:

```
LEDGER_ENDPOINT     — the address of your permanent record system
LEDGER_API_KEY      — the security key for writing to that system
SIGNING_KEY         — the cryptographic key for signing records
CHAIN_STORE         — where the bridge stores its chain state
GITHUB_APP_ID       — the ID of the GitHub App you created
GITHUB_APP_PRIVATE_KEY — the private key for the GitHub App
GITHUB_WEBHOOK_SECRET  — the shared secret for validating notifications
```

**Step 3:** Install the GitHub App on your organisation or on
the specific repositories you want to govern.

## Security

Records are signed using post-quantum cryptography — security methods
designed to remain secure even against computing systems far more
powerful than those available today. This ensures governance records
created now will still be verifiable decades in the future.

Plain language: *the security locks are designed to hold even against
the computers of the future.*

The algorithm used by default is Dilithium3 — a standard approved by
the US National Institute of Standards and Technology for post-quantum
security.

## The governance record structure

Every entry written by the bridge has six required fields, matching
the Governance Event structure defined in the master specification
(GOS §6.6):

```
identity          — a unique ID that will never be reused
timestamp         — when this happened, to microsecond precision
type              — what kind of governance act this was
content           — the full details of what happened
signature         — the cryptographic seal proving this entry is genuine
preceding_entry   — a mathematical link to the entry before this one,
                    so any gap or alteration in the chain is detectable
```

## Reference

GOS §6.6 — Governance Record structure
GOS §11.2 — Security requirements
GovernanceOperations/projops vocabulary: docs/vocabulary.md
