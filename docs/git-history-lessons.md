# Git History Lessons

The git history shows an architecture that became more explicit and less magical over time.

## Major Rewrite

| Commit | What Changed | Why It Matters |
| --- | --- | --- |
| `cdcdbf2 Bootstrap Odoo deployment API for production use` | Created the manifest-oriented API with schema, tasks, workspace renderer, proxy handling, auth, CLI, docs, and tests. | This is the architectural foundation of the current system. |
| `fa4d609 Rewrite deployment module for manifest-based deployment API` | Reworked the Odoo module around manifest generation and API task records. | The control plane became a manifest client instead of an infrastructure executor. |

## From Generic Proxy Ideas To nginx Reality

| Commit | What Changed | Lesson |
| --- | --- | --- |
| `a3b4b4a Fix Traefik proxy validation for local deployments` | Early system still had Traefik concepts. | Proxy abstraction existed before production reality settled. |
| `3003eaf Align deployment API with nginx-based production hosts` | API moved toward nginx-hosted deployments. | The system adapted to actual infrastructure. |
| `b7f2f8c Standardize proxy rendering on nginx` | Removed broad proxy branches and simplified schema/rendering. | Removing unsupported paths improves reliability. |
| `4559968 Align deployment UI with nginx proxy mode` | Odoo module removed UI fields that no longer matched API behavior. | UI should reflect actual supported operations. |

## Observability Scope Changed

| Commit | What Changed | Lesson |
| --- | --- | --- |
| `5821fc5 Add observability snapshots and local backend support` | API gained observability storage/backend behavior. | Useful during exploration, but it increased API responsibility. |
| `e2e4509 Add Grafana observability stack support` | Added Grafana-oriented runtime pieces. | Observability became a subsystem. |
| `a4c0309 Remove API observability runtime` | Removed API-owned observability runtime and thousands of lines/config. | The API should not own every platform subsystem. |
| `65613e8 Add WireGuard observability node automation` | Added node-level observability agent automation and dashboards. | Observability moved closer to nodes and external monitoring. |

## Operational Hardening

| Commit | What Changed | Lesson |
| --- | --- | --- |
| `880c7b5 feat: add idempotency, resource locking, and resumable tenant creation` | Older API learned idempotency and locks. | Long operations must survive retries. |
| `6359ede Capture remote output on task failures` | Captured stdout/stderr from failed remote commands. | Operators need actionable failure output. |
| `15f523c Refresh rendered workspace on task resume` | Re-rendered workspace during resumed tasks. | Durable tasks must not assume filesystem state survived. |
| `e96d8ce Ensure remote git cache exists before apply` | Preflight creates required git cache. | Remote host drift should be repaired when possible. |
| `6d19855 Fix redeploy cleanup for stale compose projects` | Cleans stale Compose projects. | Docker state can drift and needs reconciliation. |

## Backup And Restore Maturity

| Commit | What Changed | Lesson |
| --- | --- | --- |
| `572ba18 Add manual instance backup API flow` | Added explicit backup tasks. | Backups are an operator workflow, not just an implementation detail. |
| `2a4923d Preserve SQL dump and filestore backup artifacts` | Preserved useful artifacts. | Restore/debug needs inspectable backup contents. |
| `b7283ff Ignore transient sessions in backup archives` | Avoided session noise in archives. | Odoo filestore layout has runtime-only data. |
| `10cbce2 Add backup import restore flow` | Added import/restore flow. | Backup features are incomplete without restore. |
| `e869387 Allow backup import into live instances` | UI/API flow evolved for real operator needs. | Production workflows often require controlled live restore. |

## Repository Composition Became First-Class

| Commit | What Changed | Lesson |
| --- | --- | --- |
| `311c798 add: ability to deploy repos with custom modules` | Older API added custom module repos. | Repo composition became unavoidable. |
| `fa4d609 Rewrite deployment module for manifest-based deployment API` | Odoo module added `odoo.instance.repo`. | Repository choices belong in Odoo UX. |
| `acab5e9 Refactor repository ownership between registry and GitHub integration` | Refined ownership between registry and deployment. | Repo metadata should be centralized, not copied into every deployment component. |

## Summary

The system moved through these lessons:

1. Imperative deployment works, but becomes hard to reason about.
2. Long-running infrastructure operations need idempotency, locks, logs, and resumability.
3. The Odoo UI should produce desired state, not execute infrastructure work.
4. The API should execute desired state, not duplicate Odoo business concepts.
5. Unsupported generality should be removed when production reality is known.
6. Manifest history and task logs are operational features, not nice-to-haves.
