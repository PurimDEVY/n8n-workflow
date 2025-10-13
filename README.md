# n8n Workflow: Self-hosted on Oracle ARM (Docker + GitHub Actions)

This repository bootstraps a self-hosted n8n deployment using Docker Compose on an Oracle Cloud (ARM) instance, with CI/CD via GitHub Actions that deploys over SSH.

## What you get
- Docker Compose stack pinned to `linux/arm64` and a persistent volume
- `.env.example` with safe defaults
- GitHub Actions workflow that:
  - Copies `docker-compose.yml` to your server
  - Creates/updates `.env` on the server from GitHub Secrets
  - Runs `docker compose up -d`

## Prerequisites (on your Oracle VM)
- Docker Engine and Docker Compose plugin installed
- A non-root user with passwordless `sudo` or Docker group membership
- SSH accessible from GitHub Actions runner (public IP or via VPN)
- Firewall/security list allows TCP on `N8N_PORT` (default 5678) or your reverse proxy port (80/443)

## Setup

### 1) Configure GitHub Secrets
Create these repository secrets (Settings → Secrets and variables → Actions):

Required for SSH:
- `SSH_HOST`: server IP or DNS
- `SSH_USER`: SSH username
- `SSH_PRIVATE_KEY`: private key with access to the server
- `SSH_PORT` (optional): port, defaults to 22 when omitted

n8n configuration (used to build `.env` on the server):
- `N8N_HOST`
- `N8N_PORT` (optional, default `5678`)
- `N8N_PROTOCOL` (`http` or `https`)
- `WEBHOOK_URL`
- `GENERIC_TIMEZONE` (optional, default `UTC`)
- `N8N_ENCRYPTION_KEY` (32+ chars)
- `N8N_BASIC_AUTH_USER`
- `N8N_BASIC_AUTH_PASSWORD`
- `N8N_IMAGE` (optional, default `n8nio/n8n:latest`)

### 2) First deployment
Push to `main` (or manually run the workflow). The workflow will:
- Upload `docker-compose.yml`
- Create/overwrite `.env` on the server with your secrets
- Run `docker compose up -d`

Access n8n at `http://<server-ip>:<N8N_PORT>` (or via your reverse proxy).

### Reverse proxy (optional)
If you terminate TLS at a proxy (e.g., Nginx/Traefik/Caddy):
- Set `N8N_PROTOCOL=https` and `WEBHOOK_URL=https://your.domain.com`
- You can remove the port mapping in `docker-compose.yml` and expose only via the proxy.

### Data persistence and backups
All data is stored in the named volume `n8n_data`. Back it up with `docker volume` tooling or snapshot the underlying disk. For larger installations, consider migrating to PostgreSQL and externalizing binary data storage.

### Local development
Run locally:
```bash
docker compose up -d
```
Stop:
```bash
docker compose down
```

## License
MIT

