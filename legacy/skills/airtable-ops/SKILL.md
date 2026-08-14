---
name: airtable-ops
description: Design and build Airtable integrations. Schema design, API patterns, and when to graduate to Postgres.
disable-model-invocation: false
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Airtable: $ARGUMENTS

## Step 1: Decide If Airtable Is Right

Use Airtable when:
- Team needs to view/edit data without code
- < 100k records per table
- Schema changes frequently (early stage)
- You need quick views, filters, forms
- Operational data (deal pipeline, contacts, tasks)

Graduate to Postgres when:
- > 100k records or growing fast
- Complex queries (joins, aggregations, subqueries)
- Need transactions or referential integrity
- API rate limits are a bottleneck (5 req/sec)
- Data is append-only / analytical

For CRE deal pipeline: Airtable is great for deals, contacts, outreach tracking. Use Postgres/DuckDB for property data, market comps, scraped data.

## Step 2: Schema Design

Tables to consider for a CRE deal pipeline:

```
Properties
├── Address, City, State, Zip
├── Property Type, Subtype
├── Size (sqft), Units, Year Built
├── List Price, Price/sqft
├── Owner Name, Owner Type
├── Signal Score, Signal Details
├── Status (New, Contacted, Under Review, Passed, Active Deal)
├── Source (CoStar, Scrape, Broker, Direct)
├── Link → Contacts
├── Link → Deals
└── Link → Outreach Log

Contacts
├── Name, Company, Role
├── Phone, Email, Address
├── Contact Type (Owner, Broker, Lender, Vendor)
├── Link → Properties
├── Link → Outreach Log
└── Notes

Deals
├── Property (linked)
├── Stage (Lead, LOI, PSA, Due Diligence, Closed, Dead)
├── Offer Price, Cap Rate
├── Key Dates (LOI sent, PSA signed, Close target)
├── Link → Documents
└── Notes

Outreach Log
├── Contact (linked)
├── Property (linked)
├── Channel (Call, Email, Direct Mail, AI Voice)
├── Date, Outcome
└── Next Action, Next Date
```

Rules:
- Use linked records instead of duplicating data
- Single select for status fields (enforce consistency)
- Formula fields for calculated values
- One primary field per table that's human-readable
- Created/Modified timestamps on every table

## Step 3: API Integration

```python
# Pattern: Airtable client wrapper
from pyairtable import Api

api = Api(os.environ['AIRTABLE_API_KEY'])
table = api.table('base_id', 'table_name')

# Key operations:
table.all()                    # Get all records (handles pagination)
table.first(formula="...")     # Find one record
table.create(fields)           # Create record
table.update(record_id, fields) # Update record
table.batch_create(records)    # Bulk create (10 per request)
table.batch_update(records)    # Bulk update (10 per request)
```

Rate limit handling:
- Airtable allows 5 requests/second per base
- Use pyairtable (handles pagination and rate limiting)
- For bulk operations, batch in groups of 10
- Add 0.2s delay between batch calls

## Step 4: Sync Patterns

**One-way push** (pipeline → Airtable):
```
1. Fetch existing records from Airtable (keyed by unique ID)
2. Compare with new data
3. Create new records
4. Update changed records
5. Optionally archive removed records (don't delete)
```

**Two-way sync** (team edits in Airtable + pipeline updates):
```
1. Airtable is source of truth for: status, notes, manual fields
2. Pipeline is source of truth for: scraped data, scores, calculated fields
3. Never overwrite manual fields from pipeline
4. Use a "Last Pipeline Update" timestamp to track sync
```

## Step 5: Views and Automations

Set up Airtable views for the team:
- **New Leads**: Filter by Status = "New", Sort by Signal Score desc
- **Active Outreach**: Filter by Next Action date ≤ today
- **Deal Pipeline**: Kanban by Stage
- **Dead Deals**: Archive view with reason

Automations (built in Airtable):
- When status changes → notify team
- When deal moves to "LOI" → create checklist
- Weekly digest of new high-score properties

## Step 6: Verify

1. Create a test record via API — verify it appears in Airtable
2. Update the record — verify changes reflect
3. Run a batch sync — verify no duplicates
4. Check linked records resolve correctly
5. Verify rate limiting doesn't cause failures

## File Structure Convention

```
integrations/
  airtable/
    client.py        # Airtable API wrapper
    sync.py          # Sync logic (push/pull/two-way)
    schema.py        # Table/field definitions
    config.py        # Base IDs, table names
    .env.example     # AIRTABLE_API_KEY template
```
