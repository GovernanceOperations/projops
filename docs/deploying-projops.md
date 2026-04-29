# Deploying the GovOps GitHub Layer
## How to set up GovernanceOperations/projops so it works as a callable governance layer

This document covers everything needed to stand up the
`GovernanceOperations/projops` repository as a functioning governance
layer that any GitHub repository in your organisation can call.

If you are looking for how to connect an existing project repository
to this layer — not how to deploy the layer itself — see
[adopting-the-layer.md](adopting-the-layer.md).

Plain-language guide to every term used here: [vocabulary.md](vocabulary.md)

---

## Before you begin

**What you need:**

- A GitHub account with permission to create organisations, or an
  existing GitHub organisation where you have Owner access
- A permanent record system endpoint (see Step 4 below — this can be
  set up after the repository is created, but is needed before
  governance records can be written)
- About two hours for the initial setup

**What you are building:**

A GitHub organisation called `GovernanceOperations` containing a
repository called `projops`. This repository holds the eleven governance
workflow files that any project repository in your organisation can
call. When a project team adds three files to their repository and
installs the Ledger Bridge, they are fully connected to this layer.

---

## Step 1 — Create the GitHub organisation

If you already have a GitHub organisation where you want to host this
layer, skip to Step 2.

1. Go to [github.com/organizations/new](https://github.com/organizations/new)
2. Choose a plan. The Free plan is sufficient for public repositories.
   If your governance layer will be private (internal to your organisation),
   you need a paid plan with internal repository support.
3. Set the organisation name to `GovernanceOperations` — or your
   organisation's equivalent name. If `GovernanceOperations` is taken,
   choose a name that clearly identifies this as your governance layer
   organisation. Update all references in the workflow files from
   `GovernanceOperations/projops` to `your-org-name/projops`.
4. Complete the organisation setup.

---

## Step 2 — Configure required organisation security settings

These settings are required for the identity assurance checks to function.
Without them, the layer cannot verify that approvers meet the required
identity assurance level.

Go to your organisation **Settings → Authentication security**:

**Enable two-factor authentication requirement:**
- Check "Require two-factor authentication for everyone in your organisation"
- This sets the baseline for Identity Assurance Level 2 — the maximum
  achievable on GitHub. See [vocabulary.md](vocabulary.md) for what
  Identity Assurance Levels mean.
- Members who have not set up two-factor authentication will be removed
  from the organisation automatically. Warn them before enabling this.

**Enable SAML Single Sign-On (optional but recommended for Level 2):**
- If your organisation uses an identity provider (Okta, Azure AD,
  Google Workspace, or similar), connect it here
- Go to Settings → Authentication security → SAML single sign-on
- Follow your identity provider's instructions for connecting to GitHub
- Once enabled, organisation members authenticate through your identity
  provider, which gives stronger identity assurance than GitHub login alone

**Set the base permissions:**
- Go to Settings → Member privileges
- Set base repository permission to "No permission" — project repositories
  explicitly grant access to the people who need it
- Enable "Allow members to create private repositories" only if needed

---

## Step 3 — Create the projops repository

1. In the `GovernanceOperations` organisation, create a new repository
   named `projops`
2. Set visibility to **Public** — consuming repositories in other
   organisations need to be able to call workflows from this repository.
   If all your consuming repositories are in the same organisation,
   you can set this to **Internal** instead.
3. Initialise the repository with a README (you will replace it)
4. Extract the tarball into the repository root:

```bash
# Extract the tarball
tar -xzf projops.tar.gz

# Move everything from the projops/ directory into your repository
# (after cloning the empty repository)
git clone https://github.com/GovernanceOperations/projops.git
cp -r projops/* GovernanceOperations/projops/
cp -r projops/.github GovernanceOperations/projops/
cd GovernanceOperations/projops
git add .
git commit -m "Initial release: GovOps GitHub Layer v0.1"
git push origin main
```

---

## Step 4 — Set up the permanent record system

The permanent record system (called the Immutable Ledger in the
specification) is the external storage where governance records are
written. It must be set up before governance records can be created.

**What it must do:**
- Accept new records via an HTTPS POST request
- Never allow existing records to be modified or deleted
- Allow authorised readers to query the full history
- Produce a response confirming the record was written

**Compatible options:**

**Option A — AWS S3 with Object Lock (recommended for most organisations)**

1. Create an S3 bucket in your AWS account
2. Enable Object Lock when creating the bucket (cannot be enabled after creation)
3. Set the default retention mode to COMPLIANCE with a minimum retention
   period of 7 years (adjust for your regulatory requirements)
4. Create an IAM user with write-only access to this bucket
5. Generate an access key for this user — this becomes your `LEDGER_API_KEY`
6. Deploy the Ledger Bridge (Step 6) pointing to this bucket

**Option B — Azure Blob Storage with immutability policy**

1. Create a storage account and blob container in Azure
2. Set a time-based immutability policy on the container
3. Lock the policy (once locked, it cannot be reduced or removed)
4. Create a service principal with write access — this becomes your credential
5. Deploy the Ledger Bridge pointing to this container

**Option C — Google Cloud Storage with retention policy**

1. Create a GCS bucket
2. Set a retention policy on the bucket
3. Lock the retention policy
4. Create a service account with objectCreator role — this becomes your credential
5. Deploy the Ledger Bridge pointing to this bucket

**Option D — A specialist compliance audit log service**

Services such as Datadog Audit Trail, Splunk, or dedicated compliance
log management platforms can serve as the permanent record endpoint if
they provide an HTTPS write API and guarantee record immutability.
Verify with the service provider that records meet append-only and
tamper-evident requirements before using.

**Minimum viable option for evaluation only (not for production):**

A simple HTTPS endpoint that writes records to a file or a managed
database is sufficient for evaluating the layer. It is not suitable
for production governance because it does not provide the
append-only and tamper-evident guarantees the specification requires.
Use it only to verify the layer works before committing to a
permanent record infrastructure.

---

## Step 5 — Set organisation-level secrets

Secrets are credentials stored securely in GitHub that workflows can
use without exposing them in code. Set these at the organisation level
so they are available to all repositories using the layer.

Go to your organisation **Settings → Secrets and variables → Actions**:

**Create the following organisation secrets:**

| Secret name | What it contains | How to get it |
|---|---|---|
| `LEDGER_ENDPOINT` | The HTTPS address of your permanent record system | From Step 4 — the write endpoint of your chosen storage |
| `LEDGER_API_KEY` | The access key for writing to your permanent record system | From Step 4 — the credential you created for write access |

To create each secret:
1. Click "New organisation secret"
2. Enter the name exactly as shown above (names are case-sensitive)
3. Paste the value
4. Set repository access to "All repositories" or to a specific set of
   repositories if you want to limit which repositories can write records

**Note on secret access:** When a consuming repository calls a reusable
workflow from `GovernanceOperations/projops`, the workflow uses the
secrets from the `GovernanceOperations` organisation, not from the
consuming repository. This means you only need to set these secrets once,
in the governance layer organisation.

---

## Step 6 — Deploy the Ledger Bridge

The Ledger Bridge is the component that listens to GitHub events and
writes them to the permanent record system. Without it, the governance
checks run and enforce requirements, but permanent records are only
created by the individual workflow steps — not by the continuous event
stream that makes the record complete.

**What you need:**
- A server or cloud function that can receive HTTPS requests from GitHub
  and make HTTPS requests to your permanent record endpoint
- This can be a small cloud function (AWS Lambda, Azure Functions,
  Google Cloud Functions) — it does not require a dedicated server

**Setting up the GitHub App:**

The bridge runs as a GitHub App — a registered integration that GitHub
notifies when events occur.

1. Go to your organisation **Settings → Developer settings → GitHub Apps**
2. Click "New GitHub App"
3. Fill in the basic details:
   - Name: `GovOps Ledger Bridge`
   - Homepage URL: `https://github.com/GovernanceOperations/projops`
   - Webhook URL: the HTTPS address of your deployed bridge server
   - Webhook secret: generate a random string and keep it — you will
     need it as `GITHUB_WEBHOOK_SECRET` in Step 6c
4. Set the permissions from `bridge/app.yml` in this repository —
   these define exactly what the bridge is allowed to see and do
5. Subscribe to the events listed in `bridge/app.yml`
6. Click "Create GitHub App"
7. On the next screen, generate a private key — download it and keep it
   securely as `GITHUB_APP_PRIVATE_KEY`
8. Note the App ID shown on the settings page — this is `GITHUB_APP_ID`

**Deploying the bridge server:**

The bridge server code in `bridge/webhook-handler/` and
`bridge/ledger-client/` contains the architecture and interface
specifications. The implementation must be built or commissioned
separately — it is specified but not yet coded in this v0.1 release.
See CHANGELOG.md for the known gaps.

For organisations that need the full permanent record capability
immediately, two paths are available:

*Path A — Commission the bridge implementation:*
Use the specifications in `bridge/README.md`, `bridge/webhook-handler/README.md`,
and `bridge/ledger-client/README.md` as the technical brief for a
development engagement. The specifications are complete enough to
implement without ambiguity.

*Path B — Use workflow-level record writing only (v0.1 interim):*
Each of the eleven governance workflows writes a record to the ledger
endpoint when it runs. This produces governance records for every
explicit governance act — every merge, every deployment, every milestone
review. It does not produce the continuous event stream that the bridge
provides. This is a partial record, not a complete one. It is acceptable
for evaluation and for organisations beginning their governance journey.
It should be upgraded to full bridge deployment as the organisation matures.

**Setting the bridge environment variables:**

On your bridge server or cloud function, set:

```
LEDGER_ENDPOINT          = (same as organisation secret above)
LEDGER_API_KEY           = (same as organisation secret above)
SIGNING_KEY              = (your post-quantum signing key — see below)
CHAIN_STORE              = (connection string for chain state storage)
GITHUB_APP_ID            = (from GitHub App setup above)
GITHUB_APP_PRIVATE_KEY   = (downloaded in GitHub App setup above)
GITHUB_WEBHOOK_SECRET    = (set during GitHub App setup above)
```

**Generating a signing key:**

The bridge signs every record with a post-quantum cryptographic key
(Dilithium3 by default — see [vocabulary.md](vocabulary.md) for what
post-quantum means and why it matters).

To generate a Dilithium3 key pair:
```bash
# Using the pqcrypto Python library
pip install pqcrypto
python3 -c "
from pqcrypto.sign.dilithium3 import generate_keypair
pk, sk = generate_keypair()
print('Public key (store this):', pk.hex())
print('Private key (keep secret):', sk.hex())
"
```

Store the private key as `SIGNING_KEY` in your bridge environment.
Store the public key in your organisation's governance record so
that signed records can be independently verified.

---

## Step 7 — Set up the IAL register

The IAL register records the verified identity assurance level of each
organisation member. The identity check workflow reads it when validating
approvals.

The register file is at `.github/govops/ial-register.json` in the
`GovernanceOperations/projops` repository.

For each person in your organisation who will be approving governance
decisions, add an entry:

```json
{
  "members": {
    "their-github-username": {
      "ial": 2,
      "verified_by": "your-github-username",
      "last_verified": "2026-04-26",
      "verification_method": "org-2fa-enforced-saml"
    }
  }
}
```

`verification_method` options:
- `"github-login-only"` → IAL 1
- `"org-2fa-enforced"` → IAL 2
- `"org-2fa-enforced-saml"` → IAL 2 (stronger — recommended)
- `"hardware-key-external-idp"` → IAL 3 (requires out-of-band process)
- `"physical-presence-verified"` → IAL 4 (requires out-of-band process)

The IAL register must be reviewed and updated at minimum annually.
When a member's verification expires or their status changes, update
their entry. The `last_verified` field is used to identify stale entries.

---

## Step 8 — Configure required GitHub Actions permissions

Go to your organisation **Settings → Actions → General**:

**Workflow permissions:**
- Set to "Read and write permissions"
- Check "Allow GitHub Actions to create and approve pull requests"

**Reusable workflows:**
- No additional setting required — reusable workflows in public
  repositories can be called by any repository by default
- For internal repositories: go to Settings → Actions → General and
  confirm that "Allow all actions and reusable workflows" is selected,
  or configure the specific permissions your organisation requires

---

## Step 9 — Create the first release tag

Consuming repositories call workflows using a version tag
(`@v0.9`). Without a tag, they cannot pin to a stable version —
and pinning to a stable version is itself a governance requirement.

```bash
cd GovernanceOperations/projops
git tag -a v0.9.0 -m "GovOps GitHub Layer v0.9.0 — initial release"
git push origin v0.9.0

# Also create the minor version tag that consuming repos will use
git tag -a v0.9 -m "GovOps GitHub Layer v0.9 — tracking tag"
git push origin v0.9
```

When you release a patch version (v0.9.1), update the `v0.9` tag to
point to it so consuming repositories automatically receive patch fixes
without changing their workflow files:

```bash
git tag -fa v0.9 -m "Update v0.9 to v0.9.1"
git push origin v0.9 --force
```

---

## Step 10 — Verify the deployment

Before telling project teams to connect to this layer, verify that
everything is working by running through a test connection.

**Create a test repository:**

1. Create a new repository in your organisation called `govops-test`
2. Follow the three steps in the main README to connect it to this layer
3. Complete a minimal `GOVERNANCE.md` and `govops.yml`
4. Create a pull request

**Verify each check activates:**

| What to do | Check that should activate | Expected result |
|---|---|---|
| Open a PR | `governance-init-check` | Passes if GOVERNANCE.md and govops.yml are complete |
| Merge the PR | `pr-governance-event` | Governance record written; PR gets confirmation comment |
| Create a milestone | `uncertainty-classification-check` | Warning if no classification marker in milestone |
| Close a milestone without checkpoint | `milestone-governance-checkpoint` | Milestone re-opened |
| Attempt a production deployment | `operational-readiness-gate` | Blocked until checklist complete |
| Archive the repository | `pir-trigger` | Blocked until PIR complete |
| Close a GitHub Project board | `pir-trigger` + `asset-handover-validator` | Blocked until PIR and asset handover complete |

**For GitHub Project board coverage, also verify:**
- The Ledger Bridge App is installed at the **organisation** level
  (not just repository level) — `projects_v2` events require this
- After installing at organisation level, close a test project board
  and confirm the pir-trigger activates

**For non-GitHub git hosts:**
See [git-platform-coverage.md](git-platform-coverage.md) for guidance
on implementing equivalent governance coverage on GitLab, Azure DevOps,
Bitbucket, Gitea, and Forgejo. The permanent record system (Immutable
Ledger) and the governance templates are platform-agnostic — a
multi-platform organisation can write all Governance Events to the
same ledger regardless of which git host each repository lives on.

**Verify the permanent record:**

After merging the test PR, query your permanent record endpoint to
confirm a governance record was written. The record should have the
six-field structure described in `bridge/README.md`. If no record
appears, check that the `LEDGER_ENDPOINT` and `LEDGER_API_KEY` secrets
are correctly set and that the endpoint is reachable from GitHub Actions.

---

## Step 11 — Document your deployment for your organisation

Before rolling this out to project teams, create a short internal guide
that covers:

- The name of your governance layer organisation and repository
- Which permanent record system you are using and who manages it
- How project teams request access (who to contact, what approvals are needed)
- How the IAL register is managed and updated (who to contact for IAL verification)
- The internal escalation path if a governance check blocks work unexpectedly
- How to raise issues with the governance layer itself

This document does not need to be long. Its purpose is to prevent the
first team that encounters a blocked deployment from having nowhere to
turn.

---

## Ongoing maintenance

**When a new version of this layer is released:**

1. Pull the new version into the `GovernanceOperations/projops` repository
2. Review the CHANGELOG for any changes that affect consuming repositories
3. If the changes are backward-compatible: update the `v0.9` tracking tag
4. If the changes are breaking: create a new major version tag and notify
   consuming repository owners before updating the tracking tag

**IAL register maintenance:**
- Review and update at minimum annually
- Update immediately when a member's role changes or they leave the organisation
- An entry that has not been reviewed for more than 12 months is treated
  as expired by the identity check workflow

**Secrets rotation:**
- Rotate `LEDGER_API_KEY` according to your organisation's key rotation policy
- When rotating, update the organisation secret before the old key expires
- The bridge signing key should be rotated on the same schedule as other
  cryptographic keys in your organisation

**Permanent record system:**
- Verify at minimum quarterly that records are still being written and
  that the chain integrity is intact
- Test the audit query path — confirm that records written six months ago
  are still retrievable and verifiable
- Review storage costs and retention policies annually

---

## Troubleshooting

**Governance check fails with "govops.yml not found":**
The repository does not have a `govops.yml` file at its root, or the
file is in a subdirectory. Copy `govops.yml.example` from this repository
to the project repository root and fill it in.

**Governance check fails with "GOVERNANCE.md is missing required fields":**
Open the GOVERNANCE.md in the project repository and compare it against
the template. The error message lists the specific missing fields.
Every field in the template is required unless marked otherwise.

**Governance check passes but no permanent record appears:**
Check that the `LEDGER_ENDPOINT` and `LEDGER_API_KEY` organisation
secrets are set correctly. Verify the endpoint is reachable by running
a manual curl from a GitHub Actions workflow. Check that the endpoint
returns HTTP 201 on a successful write — some endpoints return 200,
which the bridge treats as a success, but others return unexpected codes.

**Workflow cannot be called from consuming repository:**
Check that the `GovernanceOperations/projops` repository is public or
internal (visible to the consuming repository's organisation). Check that
the workflow file uses `on: workflow_call` at the top. Check that the
consuming repository's Actions settings allow calling reusable workflows
from other repositories.

**Milestone re-opens immediately after closing:**
The milestone checkpoint check did not find a completed checkpoint issue
labelled `milestone-checkpoint`. Create an issue in the project repository
using the `govops-milestone-review` issue template, complete the review,
and close the issue. Then close the milestone again.

**Deployment blocked by operational readiness gate:**
Create an issue in the project repository using the
`govops-operational-readiness` template. Have the named responsible
person for each condition check the box next to their condition.
Once all five boxes are checked, close the issue. Then re-trigger
the deployment.

**Identity check fails with "IAL ceiling exceeded":**
The decision requires a higher identity assurance level than GitHub
can provide. Follow the out-of-band approval process in
[ial-out-of-band.md](ial-out-of-band.md) to obtain and document
an approval at the required level.

---

## Reference

- [adopting-the-layer.md](adopting-the-layer.md) — for project teams
  connecting to this layer
- [configuration-reference.md](configuration-reference.md) — complete
  field reference for govops.yml
- [vocabulary.md](vocabulary.md) — plain-language guide to every term
- [bridge/README.md](../bridge/README.md) — permanent record bridge detail
- [ial-out-of-band.md](ial-out-of-band.md) — high identity assurance approvals
