# Manifest Contract

The manifest is the most important design choice in `odoo_deployment_api`.

Instead of calling separate endpoints for every small mutation, Odoo sends a complete desired-state document. The API validates it, stores it, computes a canonical SHA, and applies it through a durable task.

## Core Fields

| Section | Purpose |
| --- | --- |
| `manifest_version` | Version of the manifest schema. |
| `min_api_version` | Minimum API version required to understand this manifest. |
| `instance` | Odoo instance identity, version, edition, ports, workers, locale, and passwords. |
| `runtime` | Docker host, Compose project, Doodba/Odoo base image, env vars, limits. |
| `git` | Git targets rendered into git-aggregator specs. |
| `addons` | Addon groups and names/globs used to render `addons.yaml`. |
| `services` | PostgreSQL, Redis, Mailhog, backup, and replication-related settings. |
| `proxy` | nginx domains, TLS mode, upstream ports, redirects, rate limits, basic auth. |
| `secrets` | Optional secret references using `secret://...` URIs. |

## Example

```yaml
manifest_version: 1.0.0
min_api_version: 0.1.0
instance:
  name: acme-production
  version: "19.0"
  edition: community
  http_port: 18079
  gevent_port: 18080
  workers: 2
  cron_threads: 1
  demo_data: false
  environment: production
  locale: en_US
runtime:
  compose_project: acme-production
  docker_host: odoo-node-01
  doodba_image: odoo:19.0
  limits:
    cpu: 1.0
    memory: 2Gi
git:
  /opt/odoo/custom/addons/queue:
    remotes:
      origin: https://github.com/OCA/queue.git
    target: origin 19.0
    depth: 20
addons:
  queue:
    - name: queue_job
proxy:
  mode: nginx
  letsencrypt_email: ops@example.com
  domains:
    - host: acme.example.com
      ssl: letsencrypt
      upstream_port: 18079
```

See the full example in [`../examples/instance-manifest.yaml`](../examples/instance-manifest.yaml).

## Why The SHA Matters

The API canonicalizes the validated manifest and computes a SHA256 hash.

That SHA becomes:

- the identity of a manifest revision
- the workspace folder name
- the task correlation key
- the comparison target for `If-Match`
- the basis for history and diffs
- the way stale tasks can be detected and revoked

Example path:

```text
/srv/doodba/acme-production/0f4c2e.../docker-compose.yml
/srv/doodba/acme-production/current -> /srv/doodba/acme-production/0f4c2e...
```

## Why Idempotency Matters

Infrastructure operations fail in the middle. SSH sessions drop. Docker pulls time out. nginx reloads can fail. A deployment API must let operators retry safely.

The current API supports:

- `X-Idempotency-Key`
- `If-Match`
- task leases
- per-instance task conflict checks
- manifest history
- stale task revocation
- workspace regeneration when files are missing

## Why Not Many Small Endpoints?

Small imperative endpoints create hidden ordering problems:

```text
create tenant
set version
add repo
add addon
create proxy
enable ssl
run migration
restart service
```

If step 5 fails, what is the desired state? If step 7 runs twice, is it safe? If Odoo and the API disagree, which one is correct?

The manifest avoids that by saying: this whole document is the desired state. Apply it until reality matches it.
