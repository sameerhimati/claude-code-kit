---
name: verify
description: Run, test, and verify code actually works. Spins up local servers, runs tests, checks types/lint, and loops until green or reports what's stuck.
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Verify: $ARGUMENTS

## Step 1: Detect Project & Commands

Scan the project root to determine stack and available commands:

### Python/FastAPI
- Look for: `requirements.txt`, `pyproject.toml`, `app/main.py`
- Test: `pytest` or `python -m pytest`
- Lint: `ruff check .` or `flake8`
- Type check: `mypy .` if configured
- Server: `uvicorn app.main:app --reload --port 8000`

### Next.js/Node
- Look for: `package.json`, `next.config.*`
- Test: `npm test` or `npm run test`
- Lint: `npm run lint` or `npx next lint`
- Type check: `npx tsc --noEmit`
- Build: `npm run build`
- Server: `npm run dev`

### iOS/Swift
- Look for: `*.xcodeproj`, `Package.swift`
- Build: `xcodebuild build`
- Test: `xcodebuild test`

### Generic fallbacks
- Look for `Makefile` → `make test`, `make lint`
- Look for `docker-compose.yml` → `docker compose up`

Report what was detected. If unsure, ask the user.

## Step 2: Run Checks (in order)

### 2a. Type Check / Lint
Run type checker and linter first — fastest feedback.

```
If errors → read the errors → fix → re-run
Max 3 attempts per check
```

### 2b. Tests
Run the test suite.

```
If failures → read the failure output → identify root cause → fix → re-run
Max 3 attempts
```

### 2c. Build
Run the build command (if applicable). Catches issues tests might miss.

```
If build fails → read errors → fix → re-run
Max 3 attempts
```

### 2d. Server Smoke Test
Spin up the local server and verify it starts and responds:

**FastAPI:**
```bash
# Start server in background
uvicorn app.main:app --host 0.0.0.0 --port 8000 &
SERVER_PID=$!
sleep 3

# Hit health endpoint
curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/health

# Hit any new/changed endpoints relevant to current work
curl -s http://localhost:8000/[endpoint]

# Kill server
kill $SERVER_PID
```

**Next.js:**
```bash
# Start dev server in background
npm run dev &
SERVER_PID=$!
sleep 5

# Check it responds
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000

# Kill server
kill $SERVER_PID
```

If the server fails to start:
```
Read the error output → fix → restart → re-check
Max 3 attempts
```

If endpoints return unexpected status codes or errors:
```
Read the response → trace the issue → fix → restart → re-check
Max 3 attempts
```

## Step 3: Feedback Loop Rules

For EVERY failure:
1. **Read** the full error output
2. **Trace** to the root cause (don't just fix symptoms)
3. **Fix** the code
4. **Re-run** the same check
5. If still failing after 3 attempts → **STOP and report**

### What "green" means:
- Lint: 0 errors (warnings OK)
- Types: 0 errors
- Tests: all passing
- Build: exits 0
- Server: starts, health returns 200, relevant endpoints respond correctly

### When to stop looping:
- All checks green → report success
- 3 failed attempts on same issue → report the issue with full context
- Issue requires external action (missing env var, database not running, etc.) → report blocker

## Step 4: Report

### All Green
```
✅ Verified

- Lint: passed
- Types: passed
- Tests: X/X passing
- Build: passed
- Server: starts, /health returns 200
- Endpoints tested: [list]

Ready for review.
```

### Blocked
```
⚠️ Verification blocked after 3 attempts

**Failing check:** [which one]
**Error:** [concise error]
**Attempted fixes:**
1. [what was tried]
2. [what was tried]
3. [what was tried]

**Root cause:** [best assessment]
**Needs:** [what's required to unblock — env var, dependency, user decision, etc.]
```

## Rules
- Always clean up background processes (kill servers when done)
- Never leave ports occupied — check with `lsof -i :PORT` before starting
- If a virtual environment exists (`venv/`, `.venv/`), activate it before running Python commands
- Don't modify test files to make tests pass — fix the actual code
- If tests don't exist yet, note it but don't fail verification for it
- Respect existing project config (pyproject.toml, .eslintrc, tsconfig.json)
