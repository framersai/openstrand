# Deployment (Internal)

This guide documents the production deployment of the OpenStrand monorepo to Linode via GitHub Actions and Docker Compose. It is intended for maintainers of the private monorepo, not the public docs.

---

## Overview
- Origin host (Linode): hosts system Nginx and Docker-based services
- Services:
  - Backend (Fastify) on port 8000
  - Frontend (Next.js) on port 3000
  - Admin (Next.js) on port 3002
- Public routing via Cloudflare → system Nginx → container upstreams

---

## Required repository secrets (GitHub → Settings → Secrets and variables → Actions)
- LINODE_HOST: Linode IP or hostname (e.g. 50.116.42.25)
- Authentication for CI (choose ONE):
  - LINODE_SSH_KEY_B64 (preferred): Base64 of the private ED25519 key for user `openstrand`
  - OR LINODE_SSH_KEY: Raw private ED25519 key contents
- GH_PAT: GitHub Personal Access Token with `repo` and `workflow` scopes
- SLACK_WEBHOOK_URL: optional; if set, Slack notifications will be sent

### Secrets quick reference
- Use EXACTLY one of: `LINODE_SSH_KEY_B64` (recommended) OR `LINODE_SSH_KEY` (raw). If both are set, CI uses the base64 one.
- How to add (GitHub UI): Repo → Settings → Secrets and variables → Actions → New repository secret → Name + Value → Add secret.
- GH_PAT scopes: `repo`, `workflow`.

Base64 value for the key (preferred):
```
# Git Bash
base64 -w0 ~/.ssh/openstrand_deploy

# PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\.ssh\openstrand_deploy"))
```

Raw value (alternative):
```
# Git Bash
cat ~/.ssh/openstrand_deploy

# PowerShell
Get-Content "$env:USERPROFILE\.ssh\openstrand_deploy" -Raw
```

### Generate and install SSH key (no LISH required)
On your local machine:

```
ssh-keygen -t ed25519 -f ~/.ssh/openstrand_deploy -N "" -C "gha-deploy@framers"
scp ~/.ssh/openstrand_deploy.pub root@LINODE_HOST:/tmp/openstrand.pub && \
ssh root@LINODE_HOST 'set -e; \
  id -u openstrand >/dev/null 2>&1 || useradd -m -s /bin/bash openstrand; \
  install -d -m 700 -o openstrand -g openstrand /home/openstrand/.ssh; \
  touch /home/openstrand/.ssh/authorized_keys; chmod 600 /home/openstrand/.ssh/authorized_keys; \
  grep -qxF "$(cat /tmp/openstrand.pub)" /home/openstrand/.ssh/authorized_keys || cat /tmp/openstrand.pub >> /home/openstrand/.ssh/authorized_keys; \
  chown -R openstrand:openstrand /home/openstrand/.ssh; rm -f /tmp/openstrand.pub'
```

Test the key:

```
ssh -i ~/.ssh/openstrand_deploy openstrand@LINODE_HOST 'whoami'
```

### Add the key to GitHub Actions secrets
- GitHub → Repo → Settings → Secrets and variables → Actions → New repository secret
- If using base64 (recommended):

```
# Git Bash
base64 -w0 ~/.ssh/openstrand_deploy

# PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\.ssh\openstrand_deploy"))
```

Add as `LINODE_SSH_KEY_B64`.

If using raw (not base64), copy the contents of `~/.ssh/openstrand_deploy` and set as `LINODE_SSH_KEY`.

> Security: Never commit private keys to the repository. Use GitHub Secrets only.

---

## Releasing / re-deploying
- Push to `master` or manually run the workflow: GitHub → Actions → "Deploy to Linode Production" → Run workflow.
- Or from CLI:
```
gh workflow run "Deploy to Linode Production" --ref master
```

---

## GitHub Actions workflow
- File: `.github/workflows/deploy-production.yml`
- Triggers on pushes to `master` and `workflow_dispatch`
- Performs:
  1) Checkout (no submodules fetched in CI — the server pulls submodules)
  2) SSH setup and known-hosts
  3) Create deployment tar (compose files + scripts)
  4) SCP to server
  5) Remote deploy via `appleboy/ssh-action`:
     - `git reset --hard origin/master`
     - `git submodule update --init --recursive --remote`
     - `docker compose down && up -d` with production override
     - DB backup, Prisma migrations, and health checks
  6) Optional Slack notification if `SLACK_WEBHOOK_URL` is set

Notes:
- If the same SHA needs to be redeployed, use “Re-run all jobs” or `workflow_dispatch` instead of dummy commits.

---

## Server layout and compose
- Project dir: `/home/openstrand/openstrand`
- Compose files shipped:
  - `docker-compose.yml` (base)
  - `docker-compose.production.yml` (build and prod overrides)
- Optional health override used locally: `~/docker-compose.health.yml` (not required in CI)

Set for convenience on host:
```
export COMPOSE_FILE=docker-compose.yml:docker-compose.production.yml
```

---

## Dockerfiles (frontend/admin) – key requirements
- Use Debian slim base images to avoid `apk` permission errors:
  - `FROM node:20-slim`
- Install certificates only (no full build tools required for Next build):
  - `apt-get update && apt-get install -y --no-install-recommends ca-certificates && rm -rf /var/lib/apt/lists/*`
- Use `npm install` (not `npm ci`) to avoid lockfile failures inside CI containers
- Copy `node_modules` from deps and copy the entire `.next` directory (not `standalone`) in runner stage
- Start with `npm run start` (Next.js runner)

Already implemented in:
- `openstrand-admin/Dockerfile`
- `openstrand-app/Dockerfile`

---

## Healthchecks (no curl required in container)
Use Node-based health commands to avoid missing `curl` in minimal images.
- Backend compose healthcheck:
```
healthcheck:
  test: ["CMD","node","-e","const http=require('http');http.get('http://localhost:8000/health',r=>process.exit(r.statusCode===200?0:1)).on('error',()=>process.exit(1));"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

---

## Nginx (system, not in Docker)
- Use system Nginx to reverse proxy:
  - openstrand.ai → http://127.0.0.1:3000
  - openstrand.ai/admin → http://127.0.0.1:3002
  - api.openstrand.ai → http://127.0.0.1:8000
- Prefer `127.0.0.1` over `localhost` to force IPv4 and avoid IPv6 socket issues

---

## Submodules and deployments
- The server pulls submodules: `git submodule update --init --recursive --remote`
- Pushing changes in submodules requires bumping the submodule pointers in the monorepo root before CI sees the updates
  - `git add openstrand-app openstrand-admin && git commit -m "chore: bump submodules" && git push`

---

## Secrets and environment on the host
- `.env.production` must be present on the server at `/home/openstrand/openstrand/.env.production`
- Minimum variables:
```
DATABASE_URL=postgresql://openstrand:changeme@postgres:5432/openstrand
JWT_SECRET=dev-secret-change-me
ENCRYPTION_KEY=<generate>
STRIPE_SECRET_KEY=sk_test_dummy
STRIPE_WEBHOOK_SECRET=whsec_dummy
REDIS_URL=redis://redis:6379
STORAGE_BACKEND=filesystem
STORAGE_PATH=/app/storage
FRONTEND_URL=https://openstrand.ai
BACKEND_URL=https://api.openstrand.ai
ADMIN_URL=https://openstrand.ai/admin
```

---

## Common issues and fixes
- 502 Bad Gateway via Cloudflare
  - Frontend/admin containers are not running or not listening on 3000/3002
  - Fix: `docker compose up -d frontend admin && curl -I 127.0.0.1:3000 127.0.0.1:3002`

- Backend `/health` returns `database: error`
  - Migrations failed (e.g., enums already exist)
  - Fix strategy:
    - Mark failed migration resolved or applied, then deploy or push schema
    - Examples:
      - `npx prisma migrate resolve --rolled-back 20250208_submissions_pipeline`
      - `npx prisma migrate resolve --applied 20250208_submissions_pipeline`
      - `npx prisma migrate deploy || npx prisma db push`

- `curl` not found in containers
  - Healthchecks use Node instead of curl; leave images minimal

- Alpine-specific errors (`apk` permission denied, musl/libssl mismatches)
  - Use `node:20-slim` instead of `node:alpine`

- Next.js `.next/standalone` missing
  - Copy entire `.next` directory instead of `standalone`

- CI ssh/scp failures
  - Ensure `LINODE_HOST` and `LINODE_SSH_KEY` secrets exist
  - Guard `ssh-keyscan` and Slack when secrets are unset

---

## Manual deploy (fallback)
```
cd /home/openstrand/openstrand
export COMPOSE_FILE=docker-compose.yml:docker-compose.production.yml

git fetch origin && git reset --hard origin/master
git submodule update --init --recursive --remote

docker compose down
docker compose up -d --remove-orphans --force-recreate
sleep 15

# Health
docker compose ps
curl -I 127.0.0.1:3000 || true
curl -I 127.0.0.1:3002 || true
curl -s 127.0.0.1:8000/health || true
```

---

## Links
- Workflow: `.github/workflows/deploy-production.yml`
- Compose: `docker-compose.yml`, `docker-compose.production.yml`
- Dockerfiles: `openstrand-app/Dockerfile`, `openstrand-admin/Dockerfile`


