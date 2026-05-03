# Deployment Flow

This page describes the lifecycle of a normal instance create or redeploy.

## 1. Odoo Builds The Manifest

The Odoo module builds a dict from instance fields:

- `name`
- `version_id`
- `edition`
- `language_id`
- `server_node_id`
- `http_port` and `gevent_port`
- `compose_project`
- `doodba_image`
- runtime limits
- proxy domains
- repository lines from `odoo.instance.repo`
- optional overrides

Then it dumps YAML and stores it as the request payload for an `odoo.paas.job`.

## 2. Odoo Submits The Job

For create/redeploy/destroy/plan, Odoo calls:

```http
PUT /api/instances/{name}/manifest
Authorization: Token <token>
X-Idempotency-Key: <job-key>
If-Match: <last-known-sha>
```

Payload:

```json
{
  "action": "redeploy",
  "manifest_yaml": "manifest_version: 1.0.0\n..."
}
```

The API returns:

```json
{
  "task_id": "0a753d88-fd7a-43fe-9246-1d86053be8b2"
}
```

## 3. API Validates And Stores

The API:

- parses YAML
- applies environment overrides
- validates the Pydantic schema
- validates `manifest_version` and `min_api_version`
- computes canonical JSON
- computes `manifest_sha`
- stores or updates the instance definition
- stores manifest history
- queues a durable task

## 4. Worker Renders Workspace

The worker renders files under:

```text
/srv/doodba/{instance}/{manifest_sha}/
```

Rendered files include:

- `manifest.yaml`
- `manifest.json`
- `repos.yaml`
- `addons.yaml`
- `.env`
- `Dockerfile.odoo-runtime`
- `docker-compose.yml`
- `scripts/run-git-aggregator.sh`
- `scripts/bootstrap.sh`
- `scripts/upgrade.sh`
- `scripts/start-odoo.sh`
- `scripts/neutralize.sh`
- `scripts/odoo-shell.sh`
- nginx config files under `proxy/nginx/`

## 5. Worker Runs Git-Aggregator

The worker executes generated `gitaggregate` commands on the deployment host.

The manifest:

```yaml
git:
  /opt/odoo/custom/addons/queue:
    remotes:
      origin: https://github.com/OCA/queue.git
    target: origin 19.0
    depth: 20
```

Becomes a generated `repos.yaml` consumed by `gitaggregate`.

## 6. Worker Runs Docker Compose

The worker starts services with a Compose project derived from the manifest:

```text
{compose_project}_{manifest_sha_short}
```

Typical services:

- `db`
- `odoo`
- `readonly_shell`
- optional `redis`
- optional `mailhog`
- optional `backup_runner`

The runtime image is built from `Dockerfile.odoo-runtime`, using the manifest's `runtime.doodba_image` as base.

## 7. Worker Bootstraps Or Upgrades Odoo

On first deploy, the worker runs Odoo with:

```text
-i base --stop-after-init
```

On redeploy, it runs:

```text
-u all --stop-after-init
```

The generated config includes database credentials, addons path, workers, cron threads, demo-data settings, and proxy mode.

## 8. Worker Syncs nginx

For `proxy.mode: nginx`, the worker renders vhost files and handles:

- domain server blocks
- HTTP to HTTPS redirects
- Let's Encrypt certificate paths
- Odoo websocket/longpolling routing
- rate limits
- basic auth htpasswd file
- `nginx -t`
- `systemctl reload nginx`

## 9. Odoo Polls Task State

Odoo calls:

```http
GET /api/tasks/bulk/?task_ids=<task_id>
GET /api/instances/{name}/logs?task_id=<task_id>
GET /api/instances/{name}
GET /api/instances/{name}/status
```

It mirrors remote task status and writes the final instance state locally.
