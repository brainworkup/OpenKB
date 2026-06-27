---
sources: [summaries/SESSION_SUMMARY.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/QUICK_REFERENCE.md, summaries/PERMANENT_SOLUTION_SUMMARY.md]
brief: Automated pipeline ingesting neuropsych PDFs into a structured knowledge base with age and diagnosis extraction.
---

# Report Ingestion Pipeline

The **report ingestion pipeline** is the automated process responsible for reading raw neuropsychological PDF reports, extracting structured metadata (including patient age, diagnoses, and test scores), and persisting the results to a downstream knowledge base. In the autism-RAG project, the pipeline is invoked via:

```bash
python -m src.ingest_recommendations
```

Each run parses all source PDFs and writes structured records to `data/recommendations_kb.json`.

## Pipeline Stages

The ingestion flow proceeds through several sequential extraction and enrichment steps:

1. **PDF Parsing** — Raw report files are read and text is extracted from each page.
2. **Header Age Extraction** — The parser searches for age patterns in report headers (e.g., `Name, Age 25`).
3. **Body Text Age Extraction** — If no header match is found, body text is scanned for age patterns (e.g., `25-year-old`).
4. **Override Lookup** — If age remains `None` after the two extraction passes, the pipeline consults `data/age_overrides.json` for a manually configured value. See [[summaries/PERMANENT_SOLUTION_SUMMARY]]. Overrides are only applied when the parser cannot find an age in the PDF — PDF-embedded values always take precedence.
5. **Age Group Classification** — The resolved numeric age is mapped to a categorical group. See [[concepts/age-group-classification]] for the full taxonomy.
6. **Diagnosis Extraction** — Section headers and ICD-10 code patterns are matched to pull structured diagnoses from each report. See [[concepts/icd10-diagnosis-extraction]] for details on the extraction logic.
7. **Knowledge Base Write** — All structured fields are serialized and written to the output store.

## The Age Override Problem

A persistent challenge in this pipeline is that many neuropsychological PDFs do not contain machine-readable age information — particularly scanned or hand-formatted documents. Without an override mechanism, each re-ingestion would reset these fields to `None`, undoing any manual corrections.

The solution introduced in [[summaries/PERMANENT_SOLUTION_SUMMARY]] is a **declarative override file** (`data/age_overrides.json`) that maps PDF filenames to integer ages. The file is consulted only as a fallback — it does not interfere with automatic extraction when age data is present. This implements a clean [[concepts/fallback-strategy]] pattern.

### Current Age Group Status

As of the latest ingestion run (post-session improvements):

| Age Group | Count | Percentage |
|---|---|---|
| Pediatric | 208 | 47.5% |
| Adult | 175 | 40.0% |
| Adolescent | 38 | 8.7% |
| Geriatric | 10 | 2.3% |
| Unknown | 7 | 1.6% |

- ✅ **11 reports** have age overrides configured
- ✅ Unknown age reduced from **16.9% → 1.6%**
- ✅ **98.4%** of the dataset has known age groups

## Diagnosis Extraction Improvements

Prior to the February 2026 session, 24 reports (23% of the dataset) had no diagnoses detected despite containing diagnostic sections in non-standard formats. The core issue was that `src/report_parser.py` did not recognize several common section header patterns used in real-world neuropsychological reports.

### Example Format Previously Missed

```
DIAGNOSTIC CONSIDERATIONS
Axis I Diagnostic Considerations:
F34.1 Dysthymic disorder
F43.22 Adjustment disorder with anxiety
F90.2 Attention-deficit hyperactivity disorder, combined type
```

### Changes Made to `src/report_parser.py`

1. **Added Section Headers**: "Diagnostic Considerations", "Diagnostic Summary", "DSM-5 Diagnoses", "DSM-IV Diagnoses" added to `DIAGNOSIS_SECTION_HEADERS`
2. **Flexible ICD-10 Regex**: Pattern now handles codes with or without a decimal point (e.g., both `F90.2` and `F90`)
3. **Subsection Filter**: Lines matching "Axis I Rule Out", "Axis II", and similar subsection labels are excluded from extracted diagnoses

### Impact

| Metric | Before | After |
|---|---|---|
| Reports with no diagnoses | 24 (23%) | ~10–12 (9–11%) |
| Recommendations with diagnoses | 77.9% | ~90% |

See [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] and [[summaries/DIAGNOSIS_FIX_SUMMARY]] for full documentation.

## Data Cleaning & Enrichment

The February 2026 session also introduced a cleaned dataset (`data/recommendations_kb_cleaned.json`) with the following enhancements:

- Normalized whitespace and text formatting across all 438 recommendations
- Added computed fields: `text_length`, `num_diagnoses`, `num_categories`
- Added quality flags: `missing_diagnoses`, `age_group_unknown`
- Created searchable text fields for diagnoses and categories

Text length statistics: mean 1,652 chars, range 103–14,305.

## Key Files

| File | Purpose |
|------|----------|
| `data/age_overrides.json` | Maps PDF filenames to integer ages for manual correction |
| `data/recommendations_kb.json` | Primary output knowledge base consumed by downstream RAG |
| `data/recommendations_kb_cleaned.json` | Enhanced dataset with computed fields and quality flags |
| `src/report_parser.py` | Core parser; auto-loads overrides and runs diagnosis extraction |
| `AGE_OVERRIDE_GUIDE.md` | Full documentation for managing age overrides |
| `DIAGNOSIS_PARSER_IMPROVEMENTS.md` | Documentation for diagnosis extraction enhancements |
| `data/reports_missing_diagnoses.txt` | Reference list of reports still lacking diagnoses |

Filenames in `age_overrides.json` are **case-sensitive** and must match the source PDF filename exactly.

## Managing Age Overrides

Adding a new override requires editing `data/age_overrides.json`:

```json
{
  "overrides": {
    "new_report.pdf": 28
  }
}
```

Then re-run ingestion. To validate the JSON before running:

```bash
python -m json.tool data/age_overrides.json
```

## Adding Diagnosis Patterns

To support additional non-standard section headers, edit the `DIAGNOSIS_SECTION_HEADERS` list in `src/report_parser.py` around line 794:

```python
DIAGNOSIS_SECTION_HEADERS = [
    # Add new pattern here
    r"Your\s+New\s+Pattern",
]
```

## Verification Queries

After re-ingestion, verify improvements with:

```python
import json
with open('data/recommendations_kb.json') as f:
    data = json.load(f)
    unknown = [r for r in data['metadata']['reports']
               if r['age_group'] == 'unknown']
    print(f"Unknown age: {len(unknown)} (expect ~2)")
    no_dx = [r for r in data['metadata']['reports']
             if len(r['diagnoses']) == 0]
    print(f"No diagnoses: {len(no_dx)} (expect ~10-12, was 24)")
```

## Integration with report_parser.py

The core logic lives in `src/report_parser.py`, which was modified across this session to:

- Import the `json` standard library
- Add a `load_age_overrides()` function that reads `data/age_overrides.json`
- Extend `extract_report_metadata()` to invoke the override lookup at the appropriate fallback position
- Expand `DIAGNOSIS_SECTION_HEADERS` with additional real-world section patterns
- Make the ICD-10 code regex more permissive
- Filter out subsection header lines from extracted diagnosis lists

This design keeps each concern isolated within the parser module and does not affect upstream PDF reading or downstream storage logic.

## Reliability Properties

| Property | Description |
|---|---|
| **Idempotent** | Re-running produces the same output given the same inputs and overrides |
| **Non-destructive** | Original PDFs are never modified |
| **Transparent** | All manual overrides are visible in a single JSON file |
| **Extensible** | New overrides and diagnosis patterns can be added without major code changes |
| **Precedence-safe** | PDF-embedded age values always override the fallback file |

## Maintenance: Adding New Reports

1. Place PDF in `/Users/joey/knowledge/neuropsych_report_pdf_bucket/`
2. Run `python -m src.ingest_recommendations`
3. Check for unknown ages or missing diagnoses
4. Add to `age_overrides.json` if needed
5. Add to `DIAGNOSIS_SECTION_HEADERS` if the report uses a non-standard section format

## Related Concepts

- [[concepts/age-group-classification]] — The taxonomy applied after age resolution
- [[concepts/recommendation-rag-pipeline]] — Downstream consumer of ingested report records
- [[concepts/neuropsychological-assessment-pipeline]] — Broader context for automated neuropsychological workflows
- [[concepts/neuropsych-report-parsing]] — Techniques for extracting structured data from clinical PDFs
- [[concepts/pdf-data-extraction]] — General approaches to pulling structured content from PDF sources
- [[concepts/report-parser-quality]] — Considerations for validating and improving parser output
- [[concepts/clinical-data-management]] — Governance and persistence of clinical data across pipeline runs
- [[concepts/fallback-strategy]] — The design pattern used by the override lookup step
- [[concepts/knowledge-base-architecture]] — How the output knowledge base is structured and queried
- [[concepts/icd10-diagnosis-extraction]] — ICD-10 code parsing and normalization techniques
- [[summaries/PERMANENT_SOLUTION_SUMMARY]] — The document describing the age override implementation
- [[summaries/AGE_OVERRIDE_GUIDE]] — The companion guide for managing age overrides
- [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] — Full documentation of diagnosis extraction enhancements
- [[summaries/DIAGNOSIS_FIX_SUMMARY]] — Summary of diagnosis parser fix and expected impact
- [[summaries/QUICK_REFERENCE]] — Quick-reference card for common override tasks and troubleshooting
- [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] — Related improvements to recommendation filtering
- [[summaries/SESSION_SUMMARY]] — Session log for the February 2026 data cleaning and pipeline improvements
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]] — Related documentation on the full RAG pipeline