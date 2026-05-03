# Control Plane Vs Executor

The split between `egeskov-group/me_odoo_deployment` and `odoo_deployment_api` is deliberate.

## Why Not Put Everything In Odoo?

Odoo is excellent for business workflows, forms, permissions, chatter, customers, and operator UX. It is not the right place to run infrastructure mutations directly.

Direct deployment execution from Odoo would mean the Odoo process needs access to:

- SSH keys for deployment hosts
- Docker socket permissions
- nginx write permissions
- Certbot permissions
- backup archive paths
- destructive delete/copy/import operations

That would make the business application and the infrastructure executor the same trust boundary. A bug in an Odoo model method or wizard could become a server-level incident.

## Why Not Put All Business Logic In The API?

The API should not own customer/accounting/business concepts.

Odoo already knows:

- customers and contacts
- responsible users
- languages and locales
- Odoo versions from `ir.odoo.release`
- repository records from `me_repo_registry`
- deployment purpose such as production, staging, upgrade, and demo
- operator groups and permissions
- UI actions and wizards

Duplicating that in FastAPI would create a second business system.

## The Contract

The Odoo module produces a YAML manifest. The API validates and applies it.

This creates a clean contract:

```text
Odoo business state + repo registry + node inventory
  -> YAML manifest
  -> Deployment API validation
  -> SSH/Docker/nginx execution
  -> task state/logs/status
  -> Odoo job mirror
```

## Operational Advantages

- The API can be restarted independently from Odoo.
- Long-running tasks continue through durable DB state and leases.
- Operators can call emergency endpoints without using the Odoo UI.
- The API can be tested with plain manifests and HTTP clients.
- Odoo upgrades do not directly affect deployment host execution code.
- Deployment host permissions stay out of the main Odoo application.

## Example Boundary

Odoo action:

```python
payload = {
    "action": "redeploy",
    "manifest_yaml": instance._prepare_manifest_yaml(raise_on_error=True),
}
```

API responsibility:

```text
validate manifest
store manifest history
queue task
render workspace
run gitaggregate
run docker compose
sync nginx
return task status and logs
```
