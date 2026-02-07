---
name: railway-deploy
description: Deploy to Railway without leaving the terminal. Generates configs, provisions services, and deploys.
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Railway Deploy: $ARGUMENTS

## Prerequisites Check
1. Verify `railway` CLI installed: `railway version` (if missing: `brew install railway`)
2. Verify logged in: `railway whoami` (if not: `railway login`)
3. Verify project linked: `railway status` (if not: guide through `railway init` or `railway link`)

## Step 1: Detect Project Type

Scan for:
- `requirements.txt` / `pyproject.toml` → Python/FastAPI
- `package.json` → Node.js/Next.js
- `Dockerfile` → Already containerized

## Step 2: Generate Dockerfile (if needed)

### FastAPI Pattern (standard across projects)
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Next.js Pattern
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

Present the Dockerfile. Wait for confirmation before writing.

## Step 3: Railway Configuration

Generate `railway.toml`:
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

## Step 4: Environment Variables

Ask user for each required env var. Common patterns:

**FastAPI projects:**
- `DATABASE_URL` — from Railway PostgreSQL addon
- `API_ENV` — production
- `CORS_ORIGINS` — comma-separated allowed origins
- `SECRET_KEY` — generate with `python -c "import secrets; print(secrets.token_urlsafe(32))"`

**Next.js projects:**
- `NEXT_PUBLIC_API_URL` — backend URL from Railway
- `NODE_ENV` — production

Set via: `railway variables set KEY=VALUE`

## Step 5: Database (if needed)

```bash
# Add PostgreSQL
railway add --plugin postgresql

# Get connection string
railway variables get DATABASE_URL
```

For Redis: `railway add --plugin redis`

## Step 6: Deploy

```bash
# Deploy current directory
railway up

# Watch logs
railway logs

# Open deployed URL
railway open
```

## Step 7: Custom Domain

```bash
# Get the Railway-provided URL first
railway domain

# Add custom domain
railway domain add yourdomain.com
```

Then guide DNS setup:
- CNAME record pointing to Railway domain
- Wait for SSL provisioning

## Step 8: Verify

1. Hit health endpoint: `curl https://[deployed-url]/health`
2. Check logs for errors: `railway logs --tail 50`
3. Verify env vars loaded: `railway variables`

## Multi-Service Setup (frontend + backend)

If project has both:
1. Deploy backend first (it provides the API URL)
2. Set `NEXT_PUBLIC_API_URL` on frontend service
3. Deploy frontend
4. Verify end-to-end

## Rollback

```bash
# List recent deployments
railway deployments

# Rollback to previous
railway rollback
```

## Common Issues
- **Port binding:** Railway sets `PORT` env var. Use `--port ${PORT:-8000}` in CMD.
- **Build failures:** Check `railway logs --build` for build-time errors.
- **Health check fails:** Ensure `/health` endpoint returns 200 before deploy timeout.
- **CORS errors:** Update `CORS_ORIGINS` to include Railway domain.
