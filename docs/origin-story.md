# Origin Story

The current `odoo_deployment_api` is the result of moving from an imperative tenant provisioner to a manifest-oriented deployment engine.

## Phase 1: Imperative Deployment API

The older `deployment_api` repository started as a FastAPI service that directly performed tenant operations on Ubuntu servers.

Early responsibilities included:

- creating tenants
- creating PostgreSQL databases
- creating Linux users
- creating Python virtualenvs
- cloning Odoo source and addons
- creating nginx proxies
- creating backups
- changing versions
- copying tenants

Important commits from `deployment_api`:

| Commit | Lesson |
| --- | --- |
| `4364927 Initial implementation of deployment API` | The first design exposed direct tenant/proxy/backup endpoints and remote host operations. |
| `880c7b5 feat: add idempotency, resource locking, and resumable tenant creation` | Long-running infrastructure operations need locks, idempotency, and resumability. |
| `60ba645 feat: add tenant copy flow and CI deploy` | Tenant lifecycle expanded beyond create/delete into copy and automation. |
| `ff8f24f feat: install addon python deps during tenant create/copy` | Custom addons pulled deployment logic deeper into source/runtime management. |
| `311c798 add: ability to deploy repos with custom modules` | Repository composition became a first-class problem. |

The imperative approach worked, but it pushed too much behavior into many separate API calls. Each new feature added another path that had to understand Odoo, source code, databases, nginx, filesystems, and task state.

## Phase 2: Manifest Rewrite

The newer `odoo_deployment_api` started with `cdcdbf2 Bootstrap Odoo deployment API for production use`. That first commit already included the core shape of the rewrite:

- manifest schema
- manifest validation
- manifest history
- task executor
- token auth
- remote SSH execution
- Docker Compose workspace rendering
- proxy rendering
- command execution
- tests and setup docs

The key idea was simple: Odoo should send complete desired state, not ask the API to perform many unrelated mutations.

## Phase 3: Odoo Module Rewritten Around Manifests

The Odoo module followed the same architectural shift.

Important commit:

| Commit | Lesson |
| --- | --- |
| `fa4d609 Rewrite deployment module for manifest-based deployment API` | The module became the Odoo-side manifest builder and job UI. |

That rewrite added models for:

- `odoo.instance`
- `odoo.paas.deployment`
- `odoo.paas.node`
- `odoo.paas.proxy`
- `odoo.paas.job`
- `odoo.instance.repo`

The module now prepares the manifest in `_prepare_manifest_dict()` and submits it from `odoo.paas.job` to `PUT /api/instances/{name}/manifest`.

## Phase 4: Simplification Around Production Reality

The history shows several cases where complexity was removed after production constraints became clearer.

| Commit | Meaning |
| --- | --- |
| `3003eaf Align deployment API with nginx-based production hosts` | Production hosts already used nginx, so proxy behavior moved toward that reality. |
| `b7f2f8c Standardize proxy rendering on nginx` | Traefik/custom proxy paths were removed; nginx became the standard. |
| `4559968 Align deployment UI with nginx proxy mode` | Odoo UI was simplified to match the nginx-only API contract. |
| `a4c0309 Remove API observability runtime` | The API stopped owning a local observability stack. |
| `65613e8 Add WireGuard observability node automation` | Observability moved closer to deployment nodes and external monitoring. |

This is an important product lesson: the right deployment platform is not the most general one. It is the one that reflects the actual operating environment.

## Phase 5: Operational Hardening

Later commits are mostly hardening and operational workflows.

Examples:

- `572ba18 Add manual instance backup API flow`
- `10cbce2 Add backup import restore flow`
- `008ea3e Add code-only redeploy task`
- `7dd89d9 Add node verification endpoint`
- `6359ede Capture remote output on task failures`
- `162dce8 Add node manifest CLI and current manifest export`
- `6d19855 Fix redeploy cleanup for stale compose projects`

These commits show the system becoming less of a prototype and more of an operator tool: recoverable tasks, readable failures, backfill, verification, backups, imports, and safe cleanup.
