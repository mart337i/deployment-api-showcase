# Development Lessons

The project history shows an architecture that became more explicit and less magical over time. This public summary describes the lessons without depending on access to private commits or repositories.

## Major Rewrite

| Change | What Changed | Why It Matters |
| --- | --- | --- |
| Initial manifest API | Created the manifest-oriented API with schema, tasks, workspace renderer, proxy handling, auth, CLI, docs, and tests. | This became the architectural foundation of the current system. |
| Manifest-based Odoo control plane | Reworked the Odoo module around manifest generation and API task records. | The control plane became a manifest client instead of an infrastructure executor. |

## From Generic Proxy Ideas To nginx Reality

| Change | What Changed | Lesson |
| --- | --- | --- |
| Early proxy abstraction | Early versions still had more than one proxy concept. | Proxy abstraction existed before production reality settled. |
| nginx production alignment | API moved toward nginx-hosted deployments. | The system adapted to actual infrastructure. |
| nginx-only rendering | Removed unsupported proxy branches and simplified schema/rendering. | Removing unsupported paths improves reliability. |
| UI alignment | Odoo UI removed fields that no longer matched API behavior. | UI should reflect actual supported operations. |

## Observability Scope Changed

| Change | What Changed | Lesson |
| --- | --- | --- |
| Local observability backend | API gained observability storage/backend behavior during exploration. | Useful during exploration, but it increased API responsibility. |
| Grafana-oriented runtime | Added dashboards and monitoring concepts. | Observability became a subsystem. |
| Removed API-owned runtime | Removed API-owned observability runtime. | The API should not own every platform subsystem. |
| Node-level observability | Added node-level agent automation and dashboards. | Observability moved closer to nodes and external monitoring. |

## Operational Hardening

| Change | What Changed | Lesson |
| --- | --- | --- |
| Idempotency and locks | Added idempotency keys, resource locking, and resumable behavior. | Long operations must survive retries. |
| Captured remote output | Captured stdout/stderr from failed remote commands. | Operators need actionable failure output. |
| Workspace refresh | Re-rendered workspace during resumed tasks. | Durable tasks must not assume filesystem state survived. |
| Remote preflight | Ensured required remote directories and tools exist before apply. | Remote host drift should be repaired when possible. |
| Stale Compose cleanup | Cleans stale Compose projects. | Docker state can drift and needs reconciliation. |

## Backup And Restore Maturity

| Change | What Changed | Lesson |
| --- | --- | --- |
| Manual backups | Added explicit backup tasks. | Backups are an operator workflow, not just an implementation detail. |
| Inspectable artifacts | Preserved SQL dump and filestore artifacts. | Restore/debug needs inspectable backup contents. |
| Filestore cleanup | Avoided transient session noise in archives. | Odoo filestore layout has runtime-only data. |
| Import restore | Added upload and restore flow. | Backup features are incomplete without restore. |
| Live restore workflows | Refined restore behavior around real operator needs. | Production workflows often require controlled live restore. |

## Repository Composition Became First-Class

| Change | What Changed | Lesson |
| --- | --- | --- |
| Custom module repos | Added support for deploying repositories with custom modules. | Repo composition became unavoidable. |
| Instance repo records | Odoo gained per-instance repository targets. | Repository choices belong in Odoo UX. |
| Registry ownership | Repository metadata became centralized in a registry concept. | Repo metadata should not be copied into every deployment component. |

## Summary

The system moved through these lessons:

1. Imperative deployment works, but becomes hard to reason about.
2. Long-running infrastructure operations need idempotency, locks, logs, and resumability.
3. The Odoo UI should produce desired state, not execute infrastructure work.
4. The API should execute desired state, not duplicate Odoo business concepts.
5. Unsupported generality should be removed when production reality is known.
6. Manifest history and task logs are operational features, not nice-to-haves.
