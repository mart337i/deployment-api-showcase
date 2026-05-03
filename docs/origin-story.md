# Origin Story

The current Deployment API is the result of moving from an imperative tenant provisioner to a manifest-oriented deployment engine.

This page is written as a public summary. It does not require access to the private source history.

## Phase 1: Imperative Deployment

The first version directly performed tenant operations on Ubuntu servers.

Early responsibilities included:

- creating tenants
- creating PostgreSQL databases
- creating Linux users
- creating Python virtualenvs
- cloning Odoo source and addons
- creating nginx proxies
- creating backups
- changing Odoo versions
- copying tenant data

This approach worked for early automation, but it made every new feature another imperative path. Each endpoint had to know about Odoo, source code, databases, nginx, filesystems, permissions, task state, and recovery behavior.

Lessons from this phase:

- long-running infrastructure operations need idempotency
- tenant operations need locks and resumability
- backups are not complete without restore paths
- custom repositories and addons must be first-class deployment inputs
- direct host mutation becomes difficult to audit over time

## Phase 2: Manifest Rewrite

The next version changed the core model: Odoo sends a complete desired-state manifest, and the API applies it.

The rewrite introduced:

- manifest schema and validation
- manifest history
- canonical manifest SHA calculation
- task executor with durable database state
- token authentication and scopes
- SSH execution on remote deployment hosts
- Docker Compose workspace rendering
- nginx proxy rendering
- command execution
- tests and setup documentation

The key idea was simple: Odoo should send complete desired state, not ask the API to perform many unrelated mutations.

## Phase 3: Odoo Control Plane

The Odoo module was rewritten around the same idea.

Instead of directly managing deployment internals, the Odoo side became the control plane:

- operators create and manage instances in Odoo
- repository choices are selected from an Odoo repository registry
- node inventory decides where instances run
- proxy domains and SSL defaults are modeled in Odoo
- jobs mirror remote API tasks
- manifests can be previewed, submitted, and tracked

The module became the Odoo-side manifest builder and job UI.

## Phase 4: Production Simplification

Several parts became simpler after the production environment became clearer.

Examples:

- host nginx became the standard proxy mode
- unsupported proxy branches were removed
- Odoo UI fields were simplified to match real API behavior
- the API stopped owning a local observability runtime
- observability moved toward node agents, dashboards, labels, and external monitoring

This is an important product lesson: the right deployment platform is not the most general one. It is the one that reflects the actual operating environment.

## Phase 5: Operational Hardening

Later work focused on operator needs:

- manual backups
- backup import and restore
- code-only redeploy
- node verification
- captured stdout/stderr on remote failures
- node manifest automation
- stale Docker Compose cleanup
- safer nginx writes
- better dashboard panels and labels

These changes made the system less of a prototype and more of an operator tool: recoverable tasks, readable failures, backfill, verification, backups, imports, and safe cleanup.
