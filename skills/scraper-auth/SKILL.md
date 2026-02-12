---
name: scraper-auth
description: Build an authenticated web scraper. Handles login flows, session management, anti-detection, and structured data extraction.
disable-model-invocation: true
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Scraper: $ARGUMENTS

## Step 1: Reconnaissance

Before writing code:

1. **Target**: What site/service? What data do we need?
2. **Auth type**: Username/password? OAuth? MFA? API key?
3. **Data location**: HTML pages? API endpoints? Exports/downloads?
4. **Volume**: How many pages/records? How often?
5. **Legal check**: Do we have legitimate access? (paid account, TOS review)
6. **API first**: Does an official API exist? Always prefer API over scraping.

Present findings. Wait for confirmation.

## Step 2: Choose Approach

Decision tree:
```
Has API? → Use API (skip to Step 4)
  ↓ No
Data in HTML? → Playwright (browser automation)
  ↓ No
Data via XHR/fetch? → requests + session (lighter weight)
  ↓ No
Requires JS rendering? → Playwright
```

For CoStar specifically:
- Requires browser automation (Playwright)
- Login flow with credentials
- Session cookies must be maintained
- Export/download features may be more reliable than scraping pages

## Step 3: Build Auth Flow

```python
# Pattern: Session manager
class SessionManager:
    - login(credentials) → authenticated session
    - is_authenticated() → bool
    - refresh_session() → handle expiry
    - save_session(path) → persist cookies for reuse
    - load_session(path) → restore without re-login
```

Rules:
- NEVER hardcode credentials — use environment variables or .env
- Store session cookies to avoid re-login on every run
- Check if session is still valid before making requests
- Handle session expiry gracefully (re-login, retry the failed request)
- Add .env to .gitignore immediately

Anti-detection basics:
- Realistic User-Agent header
- Random delays between requests (2-8 seconds, not uniform)
- Don't parallelize aggressively — 1-2 concurrent requests max
- Respect robots.txt rate limits
- Rotate through a small set of realistic headers

## Step 4: Build Extraction

```
Page/Response → Structured Data (typed, clean)
```

Rules:
- Define output schema FIRST (what fields, what types)
- Use CSS selectors or XPath for HTML, JSON paths for APIs
- Every extracted field should have a fallback (missing data ≠ crash)
- Log what was found vs expected (catch site changes early)
- Screenshot pages on extraction failure for debugging

Pattern for multi-page scraping:
```
1. Get listing/search results page
2. Extract item URLs/IDs
3. Visit each item page
4. Extract structured data
5. Save incrementally (don't lose progress on crash)
```

## Step 5: Build Resilience

Must-haves:
- Retry with exponential backoff (3 attempts, 5s/15s/45s)
- Checkpoint/resume: save progress so crashes don't restart from zero
- Rate limiting: respect the target, don't get banned
- Error classification: network error (retry) vs auth error (re-login) vs data error (log and skip)
- Graceful degradation: partial data is better than no data

For Playwright specifically:
- Handle page load timeouts
- Wait for elements (not arbitrary sleep)
- Handle popup/modal dismissal
- Screenshot on failure for debugging

## Step 6: Store Credentials Safely

```
.env (local, gitignored)
├── COSTAR_USERNAME=...
├── COSTAR_PASSWORD=...
└── PROXY_URL=... (if needed)
```

- Load via python-dotenv or os.environ
- Document required env vars in README
- Never log credentials (even in debug mode)

## Step 7: Test & Verify

1. Run auth flow — confirm login succeeds
2. Extract a single record — verify all fields populate
3. Extract a small batch (10-20) — verify consistency
4. Run twice — verify no duplicates, session reuse works
5. Simulate failure — verify retry and checkpoint work

## Step 8: Review & Commit

Spawn pipeline-reviewer agent. Key review points:
- No hardcoded credentials
- Rate limiting in place
- Error handling is comprehensive
- Anti-detection measures present
- Data schema documented

Commit: `feat(scraper): add [target] authenticated scraper`

## File Structure Convention

```
scrapers/
  [target_name]/
    auth.py          # Login + session management
    extract.py       # Data extraction logic
    scraper.py       # Orchestrator
    schema.py        # Output data models
    config.py        # Settings (URLs, selectors, timing)
    .env.example     # Template for credentials
    tests/
      test_extract.py
      fixtures/       # Sample HTML/responses
```

## CoStar-Specific Notes

- CoStar is a paid service — you must have a valid subscription
- Use their export features when available (cleaner than page scraping)
- Property search filters can be set via URL params
- Watch for session timeouts (typically 30-60 min)
- CoStar may update their UI — use data-testid or stable selectors when possible
- Consider their API if available through your subscription tier
