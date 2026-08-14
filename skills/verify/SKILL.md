---
name: verify
description: Use after writing or modifying code to verify it works. Runs lint, type-check, tests, build, and server smoke tests with a 3-attempt fix loop. Auto-triggered after implementation steps or invoke with /verify.
disable-model-invocation: false
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
If errors → read the errors → fix → re-run. Max 3 attempts per check.

### 2b. Tests
Run the test suite.
If failures → read the failure output → identify root cause → fix → re-run. Max 3 attempts.

### 2c. Build
Run the build command (if applicable). Catches issues tests might miss.
If build fails → read errors → fix → re-run. Max 3 attempts.

### 2d. Server Smoke Test
Spin up the local server and verify it starts and responds:

If the server fails to start or endpoints return unexpected status codes:
Read the error output → fix → restart → re-check. Max 3 attempts.

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
Verified

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
Verification blocked after 3 attempts

**Failing check:** [which one]
**Error:** [concise error]
**Attempted fixes:**
1. [what was tried]
2. [what was tried]
3. [what was tried]

**Root cause:** [best assessment]
**Needs:** [what's required to unblock]
```

## IRON LAW
**NO SHIP CLAIMS WITHOUT FRESH TEST EVIDENCE.** "It worked before" is not evidence. "I just ran it and here's the output" is evidence. Every verify must produce fresh results.

## Completion Status

End every verify run with one of:
```
STATUS: DONE — All checks green. Ready for review.
STATUS: DONE_WITH_CONCERNS — Passing, but [specific concern].
STATUS: BLOCKED — Failed after 3 attempts. [What's needed to unblock.]
STATUS: NEEDS_CONTEXT — Can't determine how to test. [What info is needed.]
```

## Anti-Patterns
- BAD: "Tests passed last time, should still work." (RUN THEM AGAIN. Freshness matters.)
- BAD: Modifying test assertions to match broken behavior. (Fix the code, not the tests.)
- BAD: "Build succeeded so it works." (Build passing ≠ functionality working. Run the server.)
- BAD: Skipping the server smoke test to save time. (The server test catches what unit tests miss.)
- BAD: "I'll fix that lint warning later." (Fix it now. It costs 10 seconds.)

## Rules
- Always clean up background processes (kill servers when done)
- Never leave ports occupied — check with `lsof -i :PORT` before starting
- If a virtual environment exists (`venv/`, `.venv/`), activate it before running Python commands
- Don't modify test files to make tests pass — fix the actual code
- If tests don't exist yet, note it but don't fail verification for it
- Respect existing project config (pyproject.toml, .eslintrc, tsconfig.json)
