# Git Platform Coverage
## What this layer covers, what it does not, and how any git host can be governed

---

## The honest answer

The GovernanceOperations/projops repository is built on GitHub Actions.
GitHub Actions only runs inside GitHub. The eleven automated governance
workflow files in this repository therefore run only on GitHub-hosted
repositories. They cannot run natively on GitLab, Bitbucket, Azure DevOps,
Gitea, Forgejo, self-hosted Gitea instances, or any other git host.

That is the platform-specific boundary. It is worth stating plainly.

---

## What is platform-specific and what is not

The governance *workflows* are GitHub-specific. The governance
*principles, requirements, and permanent record structure* are not.

The Governance Operations Specification (GOS §17.11) defines nine
properties that any delivery platform must satisfy to be governed.
These nine properties make no reference to GitHub. They apply to any
platform used as a delivery substrate for governed initiatives:

1. Governance initialisation — every initiative has a governance identity before work begins
2. Execution records — every significant change produces a permanent Governance Event
3. Identity ceiling declaration — the platform's IAL ceiling is declared
4. Act Layer boundary enforcement — automated execution systems have defined limits
5. Deployment gating — production deployments are gated on operational readiness
6. Exception logging — governance bypasses are automatically recorded
7. Milestone accountability — stage boundaries connect to governance checkpoints
8. Permanent record — records are held in an append-only external store
9. Governed closure — projects close with named asset owners and a learning review

Any git host that implements these nine properties through its own
native automation — GitLab CI/CD pipelines, Azure DevOps pipelines,
Bitbucket Pipelines, Forgejo Actions, or direct webhook integrations —
satisfies the governance requirement. The projops repository is the
reference implementation for GitHub. It is not the only possible
implementation.

---

## How non-GitHub git hosts are covered in principle

**Option A — A parallel platform layer**

The same architecture used in GovernanceOperations/projops can be
replicated for any git host that supports:
- Triggerable automation on push, merge, deployment, and lifecycle events
- HTTPS webhook output to the Immutable Ledger
- Issue or ticket creation for governance records
- Branch or merge protection rules

A GitLab equivalent would be a set of GitLab CI/CD pipeline templates
stored in a central GitLab group, callable by any project in the group
using GitLab's `include:` keyword. A Bitbucket equivalent would use
Bitbucket Pipelines shared configurations. Azure DevOps would use
shared pipeline templates in a central repository.

The GovOps GitHub Layer Specification defines the nine governance gaps
and their solutions. Any equivalent implementation on another platform
follows the same gap-solution mapping with platform-native tooling.

**Option B — The Ledger Bridge as the platform-agnostic layer**

The Ledger Bridge in this repository is specified as a GitHub App —
but its core function is platform-agnostic: receive webhook events,
translate them into Governance Events, write them to the Immutable Ledger.

Any git host that emits webhooks — which every major git host does —
can feed a bridge that translates its events into GovOps Governance
Events. The bridge's webhook-handler module (bridge/webhook-handler/)
is the only component that needs to be adapted per platform. The
ledger-client module (bridge/ledger-client/) is identical regardless
of platform.

A multi-platform organisation can therefore:
- Run GitHub repos through the GitHub Actions governance workflows
  plus the GitHub Ledger Bridge
- Run GitLab repos through GitLab CI pipeline templates plus a
  GitLab-adapted Ledger Bridge (same ledger-client, different webhook-handler)
- Write all Governance Events to the same Immutable Ledger

The permanent record is unified. The governance coverage is consistent.
The platform tooling is different. The governance outcome is the same.

**Option C — Manual governance with permanent record integration**

For teams on git hosts that support neither automation pipelines nor
webhooks — or for very small teams where automation overhead is
disproportionate — the governance layer can be operated manually:

1. The GOVERNANCE.md template in templates/GOVERNANCE.md is platform-agnostic.
   It is a markdown file. It can be placed at the root of any repository
   on any git host.
2. Governance Events can be written to the Immutable Ledger directly using
   a simple HTTPS POST call — via curl, a script, or the bridge client.
3. The governance templates (exception records, milestone reviews, PIR,
   asset handover) are markdown files that can be used in any issue
   tracker — GitHub Issues, GitLab Issues, Jira, Linear, or even a
   shared document store.

Manual governance is less reliable than automated governance — it depends
on practitioner discipline rather than structural enforcement. It does
not satisfy the automated enforcement requirements of the highest
governance tiers. But it satisfies the record-keeping and accountability
requirements, and it is available to every team regardless of platform.

---

## Non-GitHub git hosts — specific guidance

**GitLab**

GitLab supports reusable CI/CD pipeline components through its
Component Catalog and `include:` syntax. A GovOps GitLab Layer would:
- Store pipeline templates in a central GitLab group project
- Use GitLab merge request rules and approval policies for IAL enforcement
- Use GitLab webhook integration for Ledger Bridge connectivity
- Use GitLab Environments for deployment gating
- Use GitLab Milestones for milestone checkpoint triggers

GitLab's API is sufficiently rich to implement all nine governance properties.
A GitLab implementation is a direct port of the projops architecture.

**Azure DevOps**

Azure DevOps supports shared YAML pipeline templates through a central
repository. A GovOps Azure DevOps Layer would:
- Store pipeline templates in a central Azure DevOps repository
- Use branch policies and required reviewers for IAL enforcement
- Use service hooks for Ledger Bridge webhook connectivity
- Use Azure DevOps Environments with approval gates for deployment gating
- Use Azure Boards work items for milestone checkpoint tracking

The identity assurance model in Azure DevOps with Entra ID (formerly
Azure Active Directory) integration supports IAL 3 natively — higher
than GitHub's IAL 2 ceiling. For organisations requiring IAL 3 for
significant decisions, Azure DevOps with Entra ID is a stronger
identity substrate than GitHub alone.

**Bitbucket**

Bitbucket Pipelines supports shared pipeline configurations through
the `pipe:` mechanism. A GovOps Bitbucket Layer would follow the
same nine-property structure using Bitbucket's native automation.
Bitbucket's identity model through Atlassian Access supports
stronger identity assurance than basic Bitbucket authentication.

**Gitea and Forgejo (self-hosted)**

Gitea and its fully open-source fork Forgejo support Gitea Actions —
a GitHub Actions-compatible workflow syntax. This means the workflow
files in GovernanceOperations/projops can run on Gitea/Forgejo with
minimal adaptation. The `uses:` syntax for reusable workflows is
supported in recent versions. The primary adaptation needed is
the replacement of GitHub-specific API calls with Gitea/Forgejo
API equivalents.

For organisations running self-hosted Gitea or Forgejo as their git
infrastructure, this is the most direct path to full governance layer
coverage without platform migration.

---

## The canonical GOVERNANCE.md across platforms

The GovernanceOperations/projops repository holds the canonical
GOVERNANCE.md for the projops initiative itself. This principle —
one initiative, one canonical GOVERNANCE.md, one canonical repository
— applies regardless of platform.

For multi-platform initiatives where code lives on both GitHub and
GitLab (a common pattern during platform migrations), the canonical
repository is the one designated as such in the governance record.
The other platform's repositories carry a minimal GOVERNANCE.md
pointing to the canonical record. The Immutable Ledger receives
Governance Events from both platforms, unified by the same chain_id.

---

## The governance principle that transcends platform

The GovOps framework is not a GitHub framework. It is a governance
framework. GitHub is one substrate on which it can run. The projops
repository is the reference implementation for that substrate.

An organisation that uses GitHub today and migrates to a different
platform tomorrow does not lose its governance framework. It migrates
the platform substrate implementation. The Governance Operations
Specification, the permanent record, the governance identity of each
initiative, and the institutional learning accumulated in post-project
reviews — all of that lives outside GitHub, in the Immutable Ledger
and in the governance documentation. The platform is replaceable.
The governance record is not.

That is the design. It is intentional.

---

## Reference

- GOS §17.11 — Platform Substrate Governance (the nine properties)
- GOS §33 — GovOps Protocol Layer (platform-agnostic message format)
- bridge/README.md — Ledger Bridge architecture (platform-agnostic core)
- vocabulary.md — plain-language guide to every term
