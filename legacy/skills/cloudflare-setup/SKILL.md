---
name: cloudflare-setup
description: Set up Cloudflare Workers, D1, KV, and custom domains without leaving the terminal.
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Cloudflare Setup: $ARGUMENTS

## Prerequisites Check
1. Verify `wrangler` installed: `npx wrangler --version` (if missing: `npm install -g wrangler`)
2. Verify logged in: `npx wrangler whoami` (if not: `npx wrangler login`)

## Step 1: Determine What's Needed

Ask user:
- **Worker?** (API, proxy, edge function)
- **D1 database?** (structured data)
- **KV store?** (sessions, cache, config)
- **Custom domain?**
- **Environment?** (production + staging, or just production)

## Step 2: Generate wrangler.toml

```toml
name = "[project-name]"
main = "src/index.ts"
compatibility_date = "[today's date]"

# Environment variables (non-secret)
[vars]
ENVIRONMENT = "production"

# D1 Database binding (if needed)
[[d1_databases]]
binding = "DB"
database_name = "[project]-db"
database_id = "[will be filled after creation]"

# KV Namespace binding (if needed)
[[kv_namespaces]]
binding = "KV"
id = "[will be filled after creation]"

# Staging environment
[env.staging]
name = "[project-name]-staging"
[env.staging.vars]
ENVIRONMENT = "staging"

# Routes / Custom domain
# routes = [{ pattern = "api.yourdomain.com/*", zone_name = "yourdomain.com" }]
```

## Step 3: Create Resources

### D1 Database
```bash
# Create database
npx wrangler d1 create [project]-db

# Copy the database_id into wrangler.toml

# Create schema
npx wrangler d1 execute [project]-db --file=./schema.sql

# For staging
npx wrangler d1 create [project]-db-staging
```

### KV Namespace
```bash
# Create namespace
npx wrangler kv namespace create KV

# Copy the id into wrangler.toml

# For staging
npx wrangler kv namespace create KV --preview
```

## Step 4: Secrets

```bash
# Set secrets (not in wrangler.toml!)
npx wrangler secret put API_KEY
npx wrangler secret put JWT_SECRET

# List secrets
npx wrangler secret list
```

Never put secrets in `wrangler.toml` — use `wrangler secret put` only.

## Step 5: Worker Code Scaffold

### Basic API Worker
```typescript
export interface Env {
  DB: D1Database;
  KV: KVNamespace;
  ENVIRONMENT: string;
  API_KEY: string;
}

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const url = new URL(request.url);

    // CORS headers
    const corsHeaders = {
      "Access-Control-Allow-Origin": "*", // Restrict in production
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type, Authorization",
    };

    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }

    // Route handling
    if (url.pathname === "/health") {
      return Response.json({ status: "ok" }, { headers: corsHeaders });
    }

    return Response.json({ error: "Not found" }, { status: 404, headers: corsHeaders });
  },
};
```

## Step 6: Local Development

```bash
# Start local dev server
npx wrangler dev

# With local D1 (persisted)
npx wrangler dev --persist-to=.wrangler/state

# Test locally
curl http://localhost:8787/health
```

## Step 7: Deploy

```bash
# Deploy to production
npx wrangler deploy

# Deploy to staging
npx wrangler deploy --env staging

# View live logs
npx wrangler tail
```

## Step 8: Custom Domain

```bash
# Option A: Custom domain via Cloudflare dashboard route
# Add route in wrangler.toml:
# routes = [{ pattern = "api.yourdomain.com/*", zone_name = "yourdomain.com" }]

# Option B: Workers custom domain
npx wrangler deploy  # then set custom domain in dashboard
```

Guide:
1. Domain must be on Cloudflare DNS
2. Add CNAME or route in wrangler.toml
3. Redeploy
4. SSL auto-provisions

## D1 Common Operations

```bash
# Run a query
npx wrangler d1 execute [db-name] --command "SELECT * FROM users LIMIT 10"

# Run migrations file
npx wrangler d1 execute [db-name] --file=./migrations/001_init.sql

# Export data
npx wrangler d1 export [db-name] --output=backup.sql
```

## KV Common Operations

```bash
# Put a value
npx wrangler kv key put --binding=KV "key" "value"

# Get a value
npx wrangler kv key get --binding=KV "key"

# List keys
npx wrangler kv key list --binding=KV
```

## Common Issues
- **D1 binding error:** Ensure `database_id` in wrangler.toml matches `wrangler d1 list` output.
- **KV not found:** Run `wrangler kv namespace list` and verify ids match.
- **CORS in production:** Replace `*` with actual allowed origins.
- **Cold starts:** Workers have ~0ms cold start, but D1 first query may be slower.
- **Size limits:** Worker script max 10MB (compressed 1MB). KV value max 25MB. D1 row max 1MB.
