---
name: deploy
description: Deploy a project to Railway or Cloudflare Workers. Detects project type, generates configs, provisions services, handles secrets, and deploys. Invoke manually with /deploy.
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Deploy: $ARGUMENTS

## Step 1: Choose Platform

Decision tree:
```
Full-stack app (FastAPI/Next.js + database)?
  → Railway (managed infra, easy DB addons)

Edge function / API proxy / lightweight worker?
  → Cloudflare Workers (fast, cheap, global)

Static site or JAMstack?
  → Cloudflare Pages or Vercel

Already containerized?
  → Railway (just push the Dockerfile)
```

If unclear, ask the user.

---

## Railway Deploy

### Prerequisites
1. `railway version` (if missing: `brew install railway`)
2. `railway whoami` (if not: `railway login`)
3. `railway status` (if not linked: guide through `railway init` or `railway link`)

### Detect Project Type
- `requirements.txt` / `pyproject.toml` → Python/FastAPI
- `package.json` → Node.js/Next.js
- `Dockerfile` → Already containerized

### Generate Dockerfile (if needed)

**FastAPI:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Next.js:**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
EXPOSE 3000
CMD ["npm", "start"]
```

### Railway Configuration

```toml
[build]
builder = "DOCKERFILE"
dockerfilePath = "Dockerfile"

[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 5
```

### Environment Variables
- `DATABASE_URL` — from Railway PostgreSQL addon
- `SECRET_KEY` — generate with `python -c "import secrets; print(secrets.token_urlsafe(32))"`
- Set via: `railway variables set KEY=VALUE`

### Database
```bash
railway add --plugin postgresql
railway variables get DATABASE_URL
```

### Deploy & Verify
```bash
railway up
railway logs
curl https://[deployed-url]/health
```

### Common Issues
- **Port binding:** Use `--port ${PORT:-8000}` in CMD
- **Health check fails:** Ensure `/health` returns 200
- **CORS errors:** Update `CORS_ORIGINS` to include Railway domain

---

## Cloudflare Workers Deploy

### Prerequisites
1. `npx wrangler --version` (if missing: `npm install -g wrangler`)
2. `npx wrangler whoami` (if not: `npx wrangler login`)

### Generate wrangler.toml

```toml
name = "[project-name]"
main = "src/index.ts"
compatibility_date = "[today's date]"

[vars]
ENVIRONMENT = "production"

# D1 Database (if needed)
[[d1_databases]]
binding = "DB"
database_name = "[project]-db"
database_id = "[from wrangler d1 create]"

# KV (if needed)
[[kv_namespaces]]
binding = "KV"
id = "[from wrangler kv namespace create]"
```

### Create Resources
```bash
# D1
npx wrangler d1 create [project]-db
npx wrangler d1 execute [project]-db --file=./schema.sql

# KV
npx wrangler kv namespace create KV

# Secrets (never in wrangler.toml)
npx wrangler secret put API_KEY
```

### Deploy & Verify
```bash
npx wrangler dev          # local
npx wrangler deploy       # production
npx wrangler tail         # live logs
curl https://[worker-url]/health
```

### Custom Domain
1. Domain must be on Cloudflare DNS
2. Add route in wrangler.toml or set custom domain in dashboard
3. Redeploy — SSL auto-provisions

### Common Issues
- **D1 binding error:** Verify `database_id` matches `wrangler d1 list`
- **Size limits:** Worker 10MB, KV value 25MB, D1 row 1MB
- **CORS in production:** Replace `*` with actual allowed origins

## IRON LAW
**NO DEPLOY WITHOUT PASSING VERIFY.** Run /verify before deploying. If it doesn't pass locally, it won't work in production. No exceptions.

## Completion Status

```
STATUS: DONE — Deployed to [URL]. Health check passing. [platform] logs clean.
STATUS: DONE_WITH_CONCERNS — Deployed, but [specific concern].
STATUS: BLOCKED — Deploy failed. [Error and what was tried.]
STATUS: NEEDS_CONTEXT — Missing [secrets/config/platform access].
```

## Anti-Patterns
- BAD: "Let me deploy and see if it works." (Verify locally first. Deploy is not a test environment.)
- BAD: Committing .env or secrets to git. (Use platform secret management. Always.)
- BAD: "I'll set up the health check later." (No health check = no way to know if it's alive.)
- BAD: Deploying without checking logs after. (Deploy succeeding ≠ app working.)

## Rules
- Always verify deployment with a health check
- Never commit secrets — use platform secret management
- Set up staging environment before production
- Document deploy process in project README
