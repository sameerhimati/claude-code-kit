---
name: doc-parser
description: Build a document intelligence pipeline. PDF/lease/OM → structured data with citations. Every extracted field links back to its source.
disable-model-invocation: true
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Doc Parser: $ARGUMENTS

## Step 1: Define Extraction Schema

Before writing code, define exactly what you need to extract:

For an **Offering Memorandum (OM)**:
```
- Property address, type, size, year built
- Asking price, price/sqft, cap rate
- Current rent roll (tenant, sqft, rent, lease dates, options)
- Operating expenses (itemized)
- Capital expenditure history
- Debt assumptions
- Tenant credit quality indicators
- Vacancy and absorption data
```

For a **Lease**:
```
- Parties (landlord, tenant, guarantor)
- Premises (address, suite, sqft)
- Term (commencement, expiration, options)
- Rent (base rent, escalations, CPI adjustments)
- Expenses (NNN, modified gross, full service, CAM caps)
- TI allowance, free rent periods
- Assignment/subletting restrictions
- Default and termination clauses
- Key dates and notice requirements
```

For a **PSA (Purchase & Sale Agreement)**:
```
- Parties, property description
- Purchase price, earnest money
- Due diligence period and close date
- Contingencies and conditions
- Representations and warranties
- Prorations and closing costs
```

Present schema. Wait for confirmation.

## Step 2: Choose Extraction Approach

Decision tree:
```
Simple structured PDF (tables, forms)?
  → pdfplumber/tabula for table extraction
  → regex for structured fields

Complex/scanned document?
  → OCR first (Tesseract or cloud OCR)
  → Then LLM extraction

Narrative text with embedded data?
  → Claude API (best for unstructured extraction)
  → Chain: extract → validate → structure
```

For most CRE documents: Claude API extraction is the right call. Documents are semi-structured, clauses vary widely, and LLMs handle ambiguity well.

## Step 3: Build PDF Processing

```python
# Pattern: PDF → text → chunks → extraction
import pdfplumber  # or PyMuPDF

def process_pdf(path):
    pages = []
    with pdfplumber.open(path) as pdf:
        for i, page in enumerate(pdf.pages):
            pages.append({
                'page_num': i + 1,
                'text': page.extract_text(),
                'tables': page.extract_tables(),
            })
    return pages
```

Rules:
- Preserve page numbers (critical for citations)
- Extract tables separately from text
- Handle multi-column layouts
- Log pages that fail to extract (scanned, image-only)
- Keep raw text alongside structured output

## Step 4: Build LLM Extraction

```python
# Pattern: Structured extraction with Claude
def extract_fields(document_text, schema, page_numbers):
    prompt = f"""
    Extract the following fields from this document.
    For each field, provide:
    - value: the extracted value
    - confidence: high/medium/low
    - source: exact quote from the document
    - page: page number where found

    Fields to extract:
    {schema}

    Document:
    {document_text}
    """
    # Call Claude API
    # Parse structured response
```

Rules:
- Always ask for source citations (exact quotes + page numbers)
- Ask for confidence levels
- Process in chunks if document exceeds context window
- Use structured output (JSON mode) for consistent parsing
- Validate extracted values against expected types/ranges

## Step 5: Build Validation Layer

After extraction, validate:

```
1. Required fields present?
2. Values in expected ranges?
   - Dates are valid and in reasonable range
   - Dollar amounts are positive and reasonable
   - Percentages are 0-100 (or 0-1, be consistent)
   - Sqft matches property type expectations
3. Cross-field consistency?
   - Price/sqft = price / sqft (within rounding)
   - NOI / price ≈ cap rate
   - Lease dates: commencement < expiration
   - Total rent roll sqft ≤ building sqft
4. Flag anomalies for human review
```

Output:
```json
{
  "extracted": { ... },
  "validation": {
    "passed": true/false,
    "warnings": ["rent roll sqft exceeds building sqft by 5%"],
    "errors": ["missing required field: purchase_price"],
    "confidence": "high/medium/low"
  },
  "citations": [
    {"field": "purchase_price", "source": "$4,500,000", "page": 3}
  ]
}
```

## Step 6: Build the Pipeline

```
PDF Upload → Text Extraction → Chunking → LLM Extraction → Validation → Structured Output
                                                                              ↓
                                                                    Human Review Queue
                                                                    (for low-confidence items)
```

Key: Low-confidence extractions go to a human review queue, not straight to the database.

## Step 7: Verify

1. Process a known document with manually verified data
2. Compare extraction against ground truth
3. Check all citations are accurate (quotes actually appear in doc)
4. Verify validation catches intentionally bad data
5. Test with different document formats (different brokers format OMs differently)

## Step 8: Review & Commit

Spawn pipeline-reviewer agent. Key points:
- Citations are accurate and traceable
- Confidence scoring is calibrated
- Validation catches real errors
- Schema is complete for the use case

Commit: `feat(docs): add [doc_type] extraction pipeline`

## File Structure Convention

```
doc_intelligence/
  extractors/
    om_extractor.py       # Offering Memorandum
    lease_extractor.py    # Lease agreements
    psa_extractor.py      # Purchase & Sale
  processing/
    pdf_reader.py         # PDF → text + tables
    chunker.py            # Split for LLM context window
    validator.py          # Cross-field validation
  schemas/
    om_schema.py          # OM output schema
    lease_schema.py       # Lease output schema
  pipeline.py             # End-to-end orchestrator
  tests/
    test_extraction.py
    fixtures/              # Sample PDFs for testing
```
