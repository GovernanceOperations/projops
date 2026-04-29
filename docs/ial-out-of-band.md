# IAL 3 and IAL 4 Out-of-Band Approval Process

GitHub's authentication model supports a maximum of IAL 2 (organisation
membership + enforced 2FA + SAML SSO). IAL 3 and IAL 4 are not achievable
natively on GitHub.

When an Irreversible decision requires IAL 3 or IAL 4 approval:

## Step 1 — Obtain out-of-band approval

Obtain the approval through the required identity verification process:
- **IAL 3:** External IdP with hardware security key, identity bound to the
  approver's GitHub SAML identity via the organisation's SSO provider.
- **IAL 4:** Physical presence verification by the designated governance
  authority, producing a signed approval document.

## Step 2 — Create the approval document

Produce a signed document (PDF or equivalent) containing:
- Approver's full legal name and verified identity
- The IAL achieved and the verification method
- The specific decision being approved (PR number, deployment ID, or
  Decision Contract reference)
- A SHA-256 hash of the relevant PR merge commit or deployment ID
- The approver's cryptographic signature or wet signature with witness

## Step 3 — Link to the GitHub event

Add a comment to the relevant PR or deployment containing:

```
govops-ial-approval: <document_hash>
govops-ial-level: <3 or 4>
govops-ial-approver: @<github_username>
govops-approval-document: <link to signed document in Memory Layer>
```

## Step 4 — The ial-ceiling-check workflow

The ial-ceiling-check workflow recognises the `govops-ial-approval` marker,
validates the document hash against the PR/deployment content, and records
the out-of-band approval as a Governance Event of type `approval` with
`out_of_band_required: true` in the payload.

## Normative Reference

GOS §15 — Identity and Access Governance
GOS §14 — Authority and Decision Structures
GOS §17.11 — Project Operations Discipline
