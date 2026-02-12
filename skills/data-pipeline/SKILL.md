---
name: data-pipeline
description: Build an ETL pipeline end-to-end. Ingestion → validation → transform → load → verify. One pipeline per data source.
disable-model-invocation: true
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Pipeline: $ARGUMENTS

## Step 1: Define the Contract

Before writing code, answer these:

1. **Source**: What/where is the data? (API, scrape, file export, database)
2. **Schema**: What fields do we need? What types? What's nullable?
3. **Frequency**: One-time, daily, hourly, event-driven?
4. **Volume**: Rows per run? Total dataset size?
5. **Destination**: Where does it land? (Postgres, DuckDB, Airtable, parquet files)
6. **Idempotency**: Can we re-run safely? How do we handle duplicates?

Present contract. Wait for confirmation.

## Step 2: Build Ingestion Layer

```
Source → Raw Landing (exact copy, no transforms)
```

Rules:
- Always save raw data before transforming (you can't un-transform)
- Timestamp every ingestion run
- Log: source, rows fetched, duration, errors
- Handle pagination, rate limits, retries with exponential backoff
- For file sources: validate file exists, check format, handle encoding

Anti-patterns:
- ❌ Transforming during ingestion — separate concerns
- ❌ Silent failures — every error must be logged and surfaced
- ❌ No deduplication strategy — decide upfront (upsert, skip, overwrite)

## Step 3: Build Validation Layer

```
Raw Landing → Validated (clean, typed, checked)
```

Validate:
- Required fields present and non-null
- Types correct (dates are dates, numbers are numbers)
- Ranges reasonable (no negative prices, no future dates for historical data)
- Referential integrity (foreign keys exist)
- Row count sanity (not 0, not 10x expected)

On failure:
- Log the bad rows with reasons
- Route to dead letter table/file
- Continue processing good rows (don't fail the whole batch)
- Alert if failure rate exceeds threshold (>5% is suspicious)

## Step 4: Build Transform Layer

```
Validated → Transformed (business logic applied)
```

Rules:
- Each transform is a pure function: same input → same output
- Document business logic in comments (why, not what)
- Keep transforms composable and testable
- Derived fields should be traceable to source fields

Common transforms for CRE data:
- Normalize addresses (standardize format, geocode)
- Calculate derived metrics (price/sqft, cap rate, rent/sqft)
- Categorize/tag (property type, submarket, deal box fit)
- Enrich with external data (census, walkability, flood zones)

## Step 5: Build Load Layer

```
Transformed → Destination (final resting place)
```

Rules:
- Upsert by natural key (not auto-increment ID)
- Track load timestamps (created_at, updated_at)
- Maintain history if needed (SCD Type 2 or append-only)
- Verify row counts match: ingested → validated → loaded

## Step 6: Verify Pipeline

Run end-to-end with real data (not mocks):

```
1. Ingest a small batch (10-50 rows)
2. Check raw landing: data arrived, correct format
3. Check validated: bad rows caught, good rows passed
4. Check transformed: business logic correct
5. Check destination: data queryable, counts match
6. Run twice: verify idempotency (no duplicates)
```

## Step 7: Add Monitoring

Minimum viable monitoring:
- Run log: start time, end time, status, row counts per stage
- Error log: failed rows with reasons
- Freshness check: alert if pipeline hasn't run in expected window
- Schema drift: alert if source schema changes

## Step 8: Review & Commit

Spawn pipeline-reviewer agent. Address any issues.
Commit with message: `feat(pipeline): add [source] → [destination] pipeline`

## File Structure Convention

```
pipelines/
  [source_name]/
    ingest.py        # Raw data fetching
    validate.py      # Schema + quality checks
    transform.py     # Business logic
    load.py          # Write to destination
    pipeline.py      # Orchestrator (runs all steps)
    schema.py        # Data models / schemas
    tests/
      test_transform.py
      test_validate.py
      fixtures/       # Sample data for tests
```
