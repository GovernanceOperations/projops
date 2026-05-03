# Deploying the GovOps GitHub Layer
## Complete administrator guide — Steps 1 through 6

This document covers everything needed to stand up the
GovernanceOperations/projops repository as a functioning governance
layer. It reflects every specific configuration step, exact UI path,
command, verification check, and troubleshooting case from initial
deployment.

For connecting a project repository to this layer after it is deployed,
see adopting-the-layer.md.

Plain-language guide to every term: vocabulary.md

---

## Step 1 — Organisation Security Settings

These settings establish the security baseline. Nothing else configures
correctly until these are in place.

### 1a — Enable two-factor authentication enforcement

1. Go to:
   https://github.com/organizations/GovernanceOperations/settings/security
   Or: GovernanceOperations > Settings > Authentication security

2. Under Two-factor authentication check:
   [x] Require two-factor authentication for everyone

3. Click Save

   GitHub will warn that members without 2FA will be removed.
   Warn them before enabling and give 48 hours to enrol.
   This sets the baseline for Identity Assurance Level 2.

### 1b — Enable SAML SSO (optional but recommended)

If your organisation uses Okta, Azure AD, or Google Workspace:
1. Go to Settings > Authentication security > SAML single sign-on
2. Follow your identity provider's connection instructions
3. Once enabled update all IAL register entries (Step 5) from
   "org-2fa-enforced" to "org-2fa-enforced-saml"

### 1c — Set base permissions

1. Go to Settings > Member privileges
2. Set base repository permission to No permission

### 1d — Configure GitHub Actions permissions

1. Go to Settings > Actions > General
2. Under Actions permissions select:
   [x] Allow all actions and reusable workflows
3. Under Workflow permissions select:
   [x] Read and write permissions
   [x] Allow GitHub Actions to create and approve pull requests
4. Click Save

### 1e — Verify

Confirm the security settings page shows 2FA requirement as active.
Check the member list for anyone removed due to missing 2FA and
re-invite once they have enrolled.

---

## Step 2 — Permanent Record System

### Evaluation path

The evaluation endpoint is a lightweight Python server sufficient to
verify the full governance layer works before committing to production
storage infrastructure.

### 2a — Create the evaluation server

Create govops-ledger.py on your local machine:

```python
#!/usr/bin/env python3
import json, os, hashlib
from datetime import datetime, timezone
from http.server import HTTPServer, BaseHTTPRequestHandler

LEDGER_FILE = "govops-ledger.ndjson"
API_KEY     = "eval-key-change-before-use"
PORT        = 8080

class LedgerHandler(BaseHTTPRequestHandler):

    def do_POST(self):
        if self.path != "/v1/events":
            self._respond(404, {"error": "Not found"}); return
        auth = self.headers.get("Authorization", "")
        if auth != f"Bearer {API_KEY}":
            self._respond(401, {"error": "Unauthorised"}); return
        length = int(self.headers.get("Content-Length", 0))
        body = self.rfile.read(length)
        try:
            event = json.loads(body)
        except json.JSONDecodeError:
            self._respond(400, {"error": "Invalid JSON"}); return
        required = {"identity","timestamp","type","payload",
                    "signature","integrity_reference"}
        missing = required - set(event.keys())
        if missing:
            self._respond(400, {"error": "Missing fields",
                                "missing": sorted(missing)}); return
        event["_received_at"] = datetime.now(timezone.utc).isoformat()
        event["_chain_position"] = self._current_position()
        record_hash = hashlib.sha256(
            json.dumps(event, sort_keys=True).encode()).hexdigest()
        event["_record_hash"] = record_hash
        with open(LEDGER_FILE, "a") as f:
            f.write(json.dumps(event) + "\n")
        print(f"[{event['_received_at']}] "
              f"type={event['type']} "
              f"id={event['identity'][:8]}... "
              f"pos={event['_chain_position']}")
        self._respond(201, {"event_id": event["identity"],
                            "chain_position": event["_chain_position"],
                            "record_hash": record_hash,
                            "received_at": event["_received_at"]})

    def do_GET(self):
        if self.path == "/health":
            self._respond(200, {"status": "ok",
                                "events": self._current_position(),
                                "warning": "EVALUATION ONLY"}); return
        if self.path == "/v1/events":
            auth = self.headers.get("Authorization", "")
            if auth != f"Bearer {API_KEY}":
                self._respond(401, {"error": "Unauthorised"}); return
            events = []
            if os.path.exists(LEDGER_FILE):
                with open(LEDGER_FILE) as f:
                    for line in f:
                        if line.strip():
                            events.append(json.loads(line.strip()))
            self._respond(200, {"events": events, "count": len(events)}); return
        self._respond(404, {"error": "Not found"})

    def _current_position(self):
        if not os.path.exists(LEDGER_FILE): return 0
        with open(LEDGER_FILE) as f:
            return sum(1 for line in f if line.strip())

    def _respond(self, status, body):
        payload = json.dumps(body, indent=2).encode()
        self.send_response(status)
        self.send_header("Content-Type", "application/json")
        self.send_header("Content-Length", len(payload))
        self.end_headers()
        self.wfile.write(payload)

    def log_message(self, format, *args): pass

if __name__ == "__main__":
    print(f"GovOps Evaluation Ledger — http://localhost:{PORT}")
    print(f"Ledger file : {LEDGER_FILE}")
    print(f"API key     : {API_KEY}")
    print(f"Health      : http://localhost:{PORT}/health")
    print("WARNING: For evaluation only.")
    HTTPServer(("0.0.0.0", PORT), LedgerHandler).serve_forever()
```

Change the API_KEY on line 6 before running:
  API_KEY = "your-chosen-evaluation-key-here"

### 2b — Run the server

  python3 govops-ledger.py

Verify immediately:
  curl http://localhost:8080/health

Expected:
  {"status": "ok", "events": 0, "warning": "EVALUATION ONLY"}

### 2c — Expose over HTTPS using ngrok

GitHub Actions requires HTTPS. ngrok tunnels your local server.

  # macOS:  brew install ngrok
  # Linux:  snap install ngrok
  # Or:     https://ngrok.com/download

  ngrok http 8080

ngrok displays:
  Forwarding   https://a1b2c3d4.ngrok-free.app -> http://localhost:8080

Your LEDGER_ENDPOINT is:
  https://a1b2c3d4.ngrok-free.app/v1/events

Alternative — Cloudflare Tunnel (stable URL while tunnel runs):
  cloudflared tunnel --url http://localhost:8080

### 2d — Verify the HTTPS endpoint

  curl -X POST https://YOUR-NGROK-URL/v1/events \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer your-chosen-evaluation-key-here" \
    -d '{
      "identity": "00000000-0000-0000-0000-000000000001",
      "timestamp": "2026-04-28T10:00:00.000000Z",
      "type": "observation",
      "payload": {"check": "manual-test", "result": "pass"},
      "signature": "eval-no-signature",
      "integrity_reference": "eval-genesis-block"
    }'

Expected response:
  {"event_id": "00000000-...", "chain_position": 1, "record_hash": "...", "received_at": "..."}

Important notes:
- ngrok free tier URLs change every restart — update LEDGER_ENDPOINT secret when this happens
- Keep govops-ledger.ndjson backed up — it is the only copy of evaluation records
- Not tamper-evident or immutable — evaluation only

### 2e — Record your endpoint details for Step 3

  LEDGER_ENDPOINT : https://YOUR-NGROK-URL/v1/events
  LEDGER_API_KEY  : the API key set in govops-ledger.py

### Production path

When moving to production, provision a locked storage backend:

Option A — AWS S3 with Object Lock (recommended):
1. Create S3 bucket with Object Lock enabled at creation time
2. Set default retention mode COMPLIANCE, minimum 7 years
3. Create IAM user with write-only access, generate access key
4. Access key becomes LEDGER_API_KEY
5. Deploy Ledger Bridge pointing to this bucket

Option B — Azure Blob with immutability policy:
1. Create storage account and blob container
2. Set time-based immutability policy and lock it
3. Create service principal with objectCreator role

Option C — Google Cloud Storage with retention policy:
1. Create GCS bucket, set retention policy, lock it
2. Create service account with objectCreator role

Option D — Specialist compliance audit log service:
Datadog Audit Trail, Splunk, or equivalent.
Verify append-only and tamper-evident guarantees before using.

---

## Step 3 — Organisation Secrets

### 3a — Navigate to organisation secrets

  https://github.com/organizations/GovernanceOperations/settings/secrets/actions

Or: GovernanceOperations > Settings > Secrets and variables > Actions > Secrets tab

### 3b — Create LEDGER_ENDPOINT

1. Click New organisation secret
2. Name: LEDGER_ENDPOINT
3. Secret: https://xxxx.ngrok-free.app/v1/events
4. Repository access: All repositories
5. Click Add secret

### 3c — Create LEDGER_API_KEY

1. Click New organisation secret
2. Name: LEDGER_API_KEY
3. Secret: your API key from govops-ledger.py
4. Repository access: All repositories
5. Click Add secret

### 3d — Verify both secrets exist

The secrets page should show:
  LEDGER_API_KEY     Updated just now    All repositories
  LEDGER_ENDPOINT    Updated just now    All repositories

Values are never shown after creation. Correct a value by updating,
not by trying to read it.

### 3e — Secret inheritance

When an adopting repository calls govops workflows with secrets: inherit,
GitHub passes the calling repository's secrets — not the GovernanceOperations
organisation secrets directly.

Same organisation: organisation secrets flow through automatically.
Different organisation: that organisation must set its own LEDGER_ENDPOINT
  and LEDGER_API_KEY secrets with the same values.
Repository-level fallback: set both secrets under the specific repository's
  Settings > Secrets and variables > Actions.

### 3f — Verify secrets reach workflows

1. Go to https://github.com/GovernanceOperations/projops/actions
2. Select GovOps — Portfolio Health Aggregator
3. Click Run workflow > Run workflow
4. Watch the terminal where govops-ledger.py is running

Expected terminal line:
  [2026-04-28T10:15:00Z] type=record id=a1b2c3d4... pos=2

If nothing appears: check the workflow run log, expand
Write Record Governance Event, and read for ledger write warnings.

---

## Step 4 — Ledger Bridge GitHub App

The Ledger Bridge captures the continuous GitHub event stream.
The bridge server implementation is the known v0.1 gap.
Step 4 registers the App so when the bridge is built, it drops in
without further GitHub configuration.

### 4a — Create the GitHub App

  https://github.com/organizations/GovernanceOperations/settings/apps/new

Or: GovernanceOperations > Settings > Developer settings > GitHub Apps > New GitHub App

Basic information:
  GitHub App name : GovOps Ledger Bridge
  Homepage URL    : https://github.com/GovernanceOperations/projops
  Description     : Translates GitHub events into GovOps Governance Events

Webhook:
  Active          : [x] Checked
  Webhook URL     : https://placeholder.example.com/webhook
                    (update when bridge server is deployed)
  Webhook secret  : generate and save immediately:
                    python3 -c "import secrets; print(secrets.token_hex(32))"
                    This cannot be retrieved from GitHub after creation.

### 4b — Set permissions

Repository permissions:
  Contents        : Read-only
  Issues          : Read and write
  Pull requests   : Read and write
  Deployments     : Read-only
  Checks          : Read-only
  Metadata        : Read-only (mandatory, pre-selected)

Organisation permissions:
  Members         : Read-only
  Administration  : Read-only
  Projects        : Read-only

### 4c — Subscribe to events

Repository events — check all:
  [x] Push
  [x] Pull request
  [x] Pull request review
  [x] Issues
  [x] Milestone
  [x] Deployment
  [x] Deployment status
  [x] Workflow run
  [x] Check run
  [x] Repository
  [x] Member

Organisation events — check all:
  [x] Organization
  [x] Projects v2
  [x] Projects v2 item

### 4d — Complete creation

Under Where can this GitHub App be installed: Only on this account
Click Create GitHub App
Note the App ID shown on the settings page -> GITHUB_APP_ID

### 4e — Generate a private key

1. Scroll to Private keys on the App settings page
2. Click Generate a private key
3. A .pem file downloads — save it securely -> GITHUB_APP_PRIVATE_KEY
   This cannot be re-downloaded.

### 4f — Install at organisation level

Projects_v2 events require organisation-level installation.

1. Click Install App in the left sidebar
2. Click Install next to GovernanceOperations
3. Select All repositories
4. Click Install
5. Note the installation ID from the URL:
   https://github.com/organizations/GovernanceOperations/settings/installations/XXXXXXXX
   -> GITHUB_APP_INSTALLATION_ID

### 4g — Save all credentials

  GITHUB_APP_ID               : App settings page (retrievable later)
  GITHUB_APP_PRIVATE_KEY      : Downloaded .pem file (NOT retrievable later)
  GITHUB_WEBHOOK_SECRET       : Generated in Step 4a (NOT retrievable later)
  GITHUB_APP_INSTALLATION_ID  : Installation URL (retrievable later)

### 4h — Prevent webhook delivery failures

GitHub retries failed webhooks for 72 hours then may disable delivery.
Replace the placeholder with the health endpoint to return 200:

1. GovernanceOperations > Settings > Developer settings >
   GitHub Apps > GovOps Ledger Bridge > Edit
2. Update Webhook URL to: https://xxxx.ngrok-free.app/health
3. Click Save changes
4. Advanced tab > Redeliver on most recent failed delivery
   Confirm it now returns 200

### 4i — Verify installation

  https://github.com/organizations/GovernanceOperations/settings/installations

GovOps Ledger Bridge should show as Installed with All repositories.

### When the bridge server is ready

1. Edit the GitHub App
2. Update Webhook URL to the bridge server HTTPS address
3. Save changes
4. Use Redeliver on recent failed deliveries to backfill missed events

---

## Step 5 — IAL Register

### 5a — What each level means on GitHub

  Level 1 : Basic GitHub login
  Level 2 : Organisation member with enforced 2FA
             (GitHub maximum achievable level)
  Level 3 : Hardware security key via external IdP — out-of-band process
  Level 4 : Physical presence verified — out-of-band process

Everyone in a properly secured organisation (Step 1 enforcing 2FA)
starts at IAL 2.

### 5b — Open the register

  https://github.com/GovernanceOperations/projops/blob/main/.github/govops/ial-register.json

Click the pencil icon Edit this file.

### 5c — Add entries for every approving member

Replace the example entry with real entries. One per person:

  {
    "members": {
      "alice-smith": {
        "ial": 2,
        "verified_by": "your-username",
        "last_verified": "2026-04-28",
        "verification_method": "org-2fa-enforced"
      },
      "bob-jones": {
        "ial": 2,
        "verified_by": "your-username",
        "last_verified": "2026-04-28",
        "verification_method": "org-2fa-enforced"
      }
    }
  }

verification_method values:
  "github-login-only"          : IAL 1
  "org-2fa-enforced"           : IAL 2 — 2FA enforcement confirmed
  "org-2fa-enforced-saml"      : IAL 2 stronger — 2FA plus SAML SSO
  "hardware-key-external-idp"  : IAL 3 — out-of-band required
  "physical-presence-verified" : IAL 4 — out-of-band required

### 5d — Commit the register

  Commit message: Update IAL register — initial population 2026-04-28
  Commit directly to the main branch

### 5e — Validate the JSON

After committing, the file should render with GitHub syntax highlighting.
Plain text rendering means malformed JSON — edit and fix.

Local validation:
  python3 -c "import json; json.load(open('ial-register.json')); print('Valid')"

Common mistakes: trailing comma after last entry, missing comma between
entries, unmatched braces.

### 5f — Maintenance rules

Annual review: update last_verified for every entry at minimum annually.
Entries not reviewed within 12 months are treated as expired (defaulted to IAL 1).

Immediate updates required:
  Member leaves            : remove entry immediately
  Member loses hardware key: downgrade from IAL 3 immediately
  Organisation enables SAML: update all "org-2fa-enforced" to
                             "org-2fa-enforced-saml"

---

## Step 6 — Release Tags

### 6a — Confirm repository is ready

All workflow files and IAL register committed to main:
  https://github.com/GovernanceOperations/projops/commits/main

### 6b — Create v0.9.0 (exact version tag)

  https://github.com/GovernanceOperations/projops/releases/new

  Choose a tag   : type v0.9.0 > Create new tag: v0.9.0 on publish
  Release title  : GovOps GitHub Layer v0.9.0
  Pre-release    : leave unchecked

Release notes:
  Initial release of the GovOps GitHub Layer.
  Implements GOS Section 17.11 (Project Operations Discipline).
  Covers all nine Platform Substrate Governance properties including
  GitHub Projects (projects_v2) event handling.
  See CHANGELOG.md for full details and known gaps.

Click Publish release.

### 6c — Create v0.9 (tracking tag)

  https://github.com/GovernanceOperations/projops/releases/new

  Choose a tag  : type v0.9 > Create new tag: v0.9 on publish
  Release title : GovOps GitHub Layer v0.9 (tracking)

Release notes:
  Tracking tag for the v0.9 minor version. Currently points to v0.9.0.
  Consuming repositories reference @v0.9 in their workflow files.
  This tag moves forward with each patch release within v0.9.

Click Publish release.

Command line alternative:
  git clone https://github.com/GovernanceOperations/projops.git
  cd projops
  git tag -a v0.9.0 -m "GovOps GitHub Layer v0.9.0 — initial release"
  git tag -a v0.9 -m "GovOps GitHub Layer v0.9 — tracking tag"
  git push origin v0.9.0
  git push origin v0.9

### 6d — Verify tags exist

  https://github.com/GovernanceOperations/projops/tags

Both v0.9 and v0.9.0 should appear pointing to the same commit.

### 6e — Test tag resolution

Create a temporary file in any test repository as
.github/workflows/tag-test.yml:

  name: Tag Resolution Test
  on:
    workflow_dispatch:
  jobs:
    test:
      uses: GovernanceOperations/projops/.github/workflows/govops-suite.yml@v0.9
      with:
        config-path: govops.yml
      secrets: inherit

Run it. Expect failure on governance-init-check (no GOVERNANCE.md) —
not on tag resolution. Tag resolution failure shows an error before
any job starts. Delete the file after confirming.

### 6f — Future tag management

Patch release (v0.9.1):
  git tag -a v0.9.1 -m "GovOps GitHub Layer v0.9.1"
  git push origin v0.9.1
  git tag -fa v0.9 -m "GovOps GitHub Layer v0.9 — now at v0.9.1"
  git push origin v0.9 --force

New minor or major version: create new tracking tags.
Never move v0.9 to a new minor version — consuming repositories
pinned to @v0.9 must receive v0.9.x only.

---

## Full deployment status after Steps 1-6

  Step 1 : Organisation security settings       COMPLETE
  Step 2 : Permanent record endpoint            EVALUATION RUNNING
  Step 3 : Organisation secrets                 SET
  Step 4 : Ledger Bridge GitHub App             REGISTERED, PENDING BRIDGE SERVER
  Step 5 : IAL register                         POPULATED
  Step 6 : Release tags                         v0.9 AND v0.9.0 PUBLISHED
  Step 7 : First adopting repository            See adopting-the-layer.md

---

## Ongoing maintenance

When ngrok URL changes: update LEDGER_ENDPOINT secret immediately.
Update Ledger Bridge webhook URL to new health endpoint.

When rotating LEDGER_API_KEY: update govops-ledger.py, restart server,
update the organisation secret before the old key expires.

IAL register: review annually. Update immediately on membership changes.

Release tags: never delete published tags.
Consuming repositories depend on them.

---

## Troubleshooting

Workflow cannot be called from consuming repository:
  Confirm projops is public or internal.
  Confirm Actions settings allow calling reusable workflows from
  other repositories.

Ledger writes failing silently:
  Check workflow run log > expand Write Governance Event step.
  Verify LEDGER_ENDPOINT is the full /v1/events URL.
  Verify ngrok is still running.

Webhook deliveries failing after 72 hours:
  GitHub may have disabled delivery. Go to App settings > Advanced >
  re-enable delivery, then set webhook URL to the health endpoint.

IAL check defaulting everyone to IAL 1:
  IAL register JSON is malformed or file path has changed.
  Validate JSON. Confirm file is at .github/govops/ial-register.json
  on the main branch.

---

Reference:
  adopting-the-layer.md       Step 7 — connecting project repositories
  configuration-reference.md  All govops.yml fields
  git-platform-coverage.md    Non-GitHub git hosts
  vocabulary.md               Plain-language term guide
  ial-out-of-band.md          IAL 3 and 4 approvals
