# Runtime Requirements

This system is not just a Python web app. It is a deployment control system, so the runtime requirements include API infrastructure, SSH permissions, Docker, nginx, Git, secrets, and Odoo-side modules.

## API Host

Supported baseline:

- Ubuntu 24.04
- Python 3.12+
- PostgreSQL for production state
- systemd service for `odoo-deployment-api`
- outbound SSH access to deployment hosts
- API token or DB-backed scoped tokens

System packages:

```bash
sudo apt update
sudo apt install -y \
  python3 python3-venv python3-pip \
  git curl openssh-client \
  build-essential libffi-dev libssl-dev \
  postgresql postgresql-client
```

Important environment variables:

| Variable | Purpose |
| --- | --- |
| `ODOO_DEPLOY_API_DATABASE_URL` | SQLAlchemy database URL. Use PostgreSQL in production. |
| `ODOO_DEPLOY_API_API_TOKEN` | Bootstrap token. |
| `ODOO_DEPLOY_API_HOST` | Bind host. Often `127.0.0.1` or private IP. |
| `ODOO_DEPLOY_API_PORT` | API port. |
| `ODOO_DEPLOY_API_DEFAULT_SSH_USER` | Operational SSH user for deployment hosts. |
| `ODOO_DEPLOY_API_DEFAULT_SSH_KEY_PATH` | Private key used by API workers. |
| `ODOO_DEPLOY_API_REMOTE_WORKSPACE_ROOT` | Usually `/srv/doodba`. |
| `ODOO_DEPLOY_API_REMOTE_GIT_CACHE_DIR` | Usually `/srv/doodba/git-cache`. |
| `ODOO_DEPLOY_API_SECRET_BASE_DIR` | Usually `/srv/secrets`. |
| `ODOO_DEPLOY_API_GIT_AGGREGATOR_COMMAND` | Usually `gitaggregate`. |

## Deployment Host

Supported baseline:

- Ubuntu 24.04
- Docker Engine 29+
- Docker Compose v2
- SSH server
- `gitaggregate`
- host nginx and Certbot for public domains
- deployment workspace directories
- secret files with restricted permissions

Required packages before Docker-specific install:

```bash
sudo apt update
sudo apt install -y \
  git curl ca-certificates openssh-server \
  python3 python3-venv
```

Required directories:

```bash
sudo mkdir -p /srv/doodba/git-cache /srv/doodba/backups /srv/secrets
sudo chown -R deploy:deploy /srv/doodba /srv/secrets
sudo chmod 755 /srv/doodba
sudo chmod 700 /srv/secrets
```

Install `gitaggregate`:

```bash
sudo python3 -m venv /opt/git-aggregator-venv
sudo /opt/git-aggregator-venv/bin/pip install --upgrade pip git-aggregator
sudo ln -sf /opt/git-aggregator-venv/bin/gitaggregate /usr/local/bin/gitaggregate
gitaggregate --help
```

nginx permissions needed by the operational SSH user:

- write access to `/etc/nginx/sites-available`
- write access to `/etc/nginx/sites-enabled`
- passwordless sudo for `nginx -t`
- passwordless sudo for `systemctl reload nginx`
- passwordless sudo for `certbot certonly` when Let's Encrypt is used

Example sudoers rule:

```text
deploy ALL=(root) NOPASSWD: /usr/sbin/nginx -t, /usr/bin/systemctl reload nginx, /usr/bin/certbot certonly *
```

## Odoo Side

Required Odoo-side pieces:

- Odoo 19
- Odoo control-plane module
- Odoo repository registry module
- Python dependencies declared by the module: `requests`, `PyYAML`
- Deployment API URL configured in Odoo settings
- Deployment API token configured in Odoo settings
- repository records and branches configured in the registry
- node records with host/IP/role/proxy defaults

## Network Requirements

Minimum network paths:

```text
Odoo -> Deployment API over HTTP/private network
Deployment API -> Deployment Host over SSH
Deployment Host -> GitHub/OCA/private Git remotes
Deployment Host -> Let's Encrypt when TLS is issued
Users -> nginx public domain for deployed Odoo instance
```

Recommended exposure:

- keep the API private or behind VPN/firewall
- do not expose the API directly to the public internet
- expose Odoo instances through nginx domains only
- keep `/srv/secrets` readable only by the deployment user
