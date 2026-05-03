# Operational Features

The Deployment API is more than create/redeploy. The later history shows work around real operator needs.

## Task System

The API returns `202 Accepted` for long-running work and stores task state in the database.

Task features:

- DB-backed queue
- worker leases
- status polling
- phase tracking
- progress text
- stdout/stderr capture
- correlation IDs
- idempotency keys
- conflict checks per instance
- stale task revocation

Typical phases for apply:

```text
connect -> render -> git -> database -> bootstrap -> pre_cutover -> deploy -> proxy -> cutover
```

## Manifest History

Every applied manifest revision is stored with:

- instance name
- manifest SHA
- manifest version
- action
- submitted by
- idempotency key
- gzipped manifest YAML

This lets operators see what changed and lets the API resolve previous known-good manifests.

## Backups

Manual backups create `.tgz` archives containing database dump and filestore content.

Backup hardening seen in history:

- staged archive creation
- validation of archive members
- handling tar warnings
- preserving SQL dump and filestore artifacts
- ignoring transient session data
- tracking backup state and remote backup IDs

## Import Restore

The API supports uploading backup archives and restoring them into target instances.

Useful workflows:

- restore customer data into staging
- copy production to upgrade instance
- test migration against real data
- recover from a bad deployment

## Copy And Neutralize

Instance copy can clone data from one deployed instance to another.

Neutralization runs before app startup where configured, so copied databases do not accidentally behave like production.

Examples of why this matters:

- disable outgoing email
- adjust base URLs
- prepare upgrade/testing environments
- avoid cron side effects

## Code-Only Redeploy

`code-redeploy` exists because not every deployment changes the manifest. Sometimes the manifest is still correct, but source branches have moved.

The API can rerun source checkout and restart/update the stack without changing business configuration.

## Power Operations

Operators can power instances off and on without destroying manifest state.

This is useful for:

- staging/demo environments
- temporary resource savings
- maintenance windows
- investigation without deletion

## Node Verification

Node verification checks whether a deployment host has the expected capabilities before it is used.

It verifies things like:

- SSH reachability
- Docker Compose availability
- nginx defaults
- SSL/proxy expectations
- role configuration

## Observability

The API no longer owns a full observability runtime. Instead, the current direction is node-level observability automation and dashboards.

Related components:

- WireGuard examples
- Grafana Alloy agent config
- Loki log forwarding
- VictoriaMetrics remote write
- dashboards for node, instance, and logs
- explicit Docker labels rendered into Compose services

This keeps the deployment executor focused while still making deployed instances observable.
