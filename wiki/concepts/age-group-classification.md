---
sources: [summaries/bwu.neuro.reports.recs.adhd.older-adult.md, summaries/SESSION_SUMMARY.md, summaries/QUICK_REFERENCE.md, summaries/PERMANENT_SOLUTION_SUMMARY.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/AGE_OVERRIDE_GUIDE.md]
brief: Mapping patient numeric age to clinical category (pediatric/adolescent/adult/geriatric) in neuropsych pipelines.
---

# Age Group Classification in Clinical Pipelines

Age group classification is the process of mapping a patient's numeric age to a named clinical category used to organize, filter, and contextualize neuropsychological reports within an automated [[concepts/neuropsychological-assessment-pipeline]].

## Standard Age Group Boundaries

The classification scheme used in the pipeline follows conventional clinical developmental divisions:

| Age Range | Category |
|-----------|----------|
| 0–12 | `pediatric` |
| 13–17 | `adolescent` |
| 18–64 | `adult` |
| 65+ | `geriatric` |
| not resolvable | `unknown` |

These boundaries align with broadly accepted clinical and developmental psychology distinctions, and are applied automatically once a numeric age is resolved — whether from PDF extraction or a manual override.

## Role in the Ingestion Pipeline

During ingestion (`python -m src.ingest_recommendations`), the report parser (`src/report_parser.py`) attempts to extract patient age using a cascading series of methods:

1. **Header parsing** — e.g., `PATIENT NAME: John Doe, Age 25`
2. **Body text patterns** — e.g., "is a 25-year-old patient"
3. **Override file fallback** — `data/age_overrides.json` (see [[summaries/AGE_OVERRIDE_GUIDE]] and [[summaries/PERMANENT_SOLUTION_SUMMARY]])

Once a numeric age is obtained by any method, classification into an age group is immediate and deterministic. If no age can be resolved after all three steps, the record is tagged `age_group: "unknown"`.

## Fallback and Override Mechanism

A persistent problem with re-ingestion workflows is that each run of `python -m src.ingest_recommendations` can overwrite previously cleaned metadata, resetting manually corrected age group assignments. The solution is a JSON-based override file (`data/age_overrides.json`) that acts as a permanent, human-curated correction layer.

The override system is implemented in `src/report_parser.py` via a `load_age_overrides()` function and is consulted only when automated extraction has already failed. This is a practical example of a [[concepts/fallback-strategy]] applied to metadata enrichment.

Key design constraint: **if the PDF itself contains an age, that extracted value takes precedence over any override entry.** The override only fires when extraction has already returned `None`. This precedence rule is important to keep in mind when troubleshooting unexpected `unknown` classifications.

### Achieved Coverage (Post-Session February 11, 2026)

Following a dedicated data cleaning session, the override system was applied to 11 reports, dramatically improving coverage across the 438-recommendation dataset sourced from 104 reports:

| Age Group | Count | Percentage |
|-----------|-------|------------|
| Pediatric | 208 | 47.5% |
| Adult | 175 | 40.0% |
| Adolescent | 38 | 8.7% |
| Geriatric | 10 | 2.3% |
| Unknown | 7 | 1.6% |

This represents a reduction in unknown age group records from **16.9% → 1.6%** — a result of identifying 13 reports with unresolvable ages, obtaining ground-truth ages for 11 of them, and encoding those values permanently in `data/age_overrides.json`. Only 7 records remain `unknown`, most of which are duplicate templates or reports with no recoverable patient age information.

### Override Coverage Breakdown

- **Pediatric (age 8)**: 2 reports
- **Adolescent (age 13)**: 1 report
- **Adult (ages 19–42)**: 8 reports

### Adding New Overrides

New entries can be added directly to `data/age_overrides.json` in the form:

```json
{
  "overrides": {
    "new_report.pdf": 25
  }
}
```

After editing, re-running the ingestion script automatically applies the changes. No modification to the original PDF source documents is required.

### Identifying Reports That Need Overrides

After ingestion, unknown-age reports can be surfaced with:

```python
import json
with open('data/recommendations_kb.json') as f:
    data = json.load(f)
    for report in data['metadata']['reports']:
        if report['age_group'] == 'unknown':
            print(report['source_file'])
```

Expect approximately 7 unknown-age reports (1.6%) after a clean ingestion with all current overrides applied.

### Verifying an Override Was Applied

```python
import json
with open('data/recommendations_kb.json') as f:
    data = json.load(f)
    for report in data['metadata']['reports']:
        if report['source_file'] == 'your_report.pdf':
            print(f"Age group: {report['age_group']}")
```

### Troubleshooting Overrides

- Filenames are **case-sensitive** — must match exactly
- Validate JSON syntax: `python -m json.tool data/age_overrides.json`
- Confirm the file is located at `data/age_overrides.json`
- If a report still shows `unknown` after adding an override, check whether the PDF itself contains an age value (PDF-extracted age takes precedence)

## Key Files

| File | Purpose |
|------|----------|
| `data/age_overrides.json` | Edit to add or change age overrides |
| `src/report_parser.py` | Auto-loads overrides during ingestion |
| `AGE_OVERRIDE_GUIDE.md` | Full override documentation |
| `data/recommendations_kb.json` | Output knowledge base with resolved age groups |
| `data/recommendations_kb_cleaned.json` | Enhanced dataset with computed fields and quality flags |
| `data/cleaning_summary.txt` | Detailed cleaning report from data quality session |

## Relationship to Data Quality Flags

The cleaned dataset (`data/recommendations_kb_cleaned.json`) introduces explicit quality flags alongside age group metadata:

- **`age_group_unknown`** — boolean flag set when classification could not be resolved
- **`missing_diagnoses`** — boolean flag for records lacking diagnosis entries
- **Computed fields** — `text_length`, `num_diagnoses`, `num_categories`

These flags support downstream filtering and quality monitoring within the [[concepts/recommendation-rag-pipeline]], making it easy to identify records that may require manual review.

## Impact on Clinical Data Management

Age group is a first-class metadata field in the recommendations knowledge base (`data/recommendations_kb.json`). It enables:

- **Filtering** reports by developmental stage
- **Contextualizing** normative score interpretation (e.g., pediatric vs. adult norms)
- **Routing** recommendations to age-appropriate clinical content

This connects directly to broader goals of [[concepts/clinical-data-management]] and [[concepts/neuropsychological-score-interpretation]], where age-appropriate context is essential for valid interpretation.

## Relationship to PDF Data Extraction

The classification system depends entirely on the upstream success of [[concepts/pdf-data-extraction]]. When PDF parsing is robust, age is resolved automatically. When it fails — due to non-standard formatting, scanned images, or anonymized headers — the override file bridges the gap without modifying the original source documents.

## Related Concepts

- [[concepts/pdf-data-extraction]] — upstream source of age values
- [[concepts/fallback-strategy]] — pattern used when extraction fails
- [[concepts/clinical-data-management]] — broader context for metadata quality
- [[concepts/neuropsychological-assessment-pipeline]] — the system in which classification operates
- [[concepts/neuropsychological-score-interpretation]] — downstream consumer of age group metadata
- [[concepts/phi-data-handling]] — privacy considerations for patient demographic data
- [[concepts/report-ingestion-pipeline]] — the broader ingestion system this classifier is embedded in
- [[concepts/recommendation-rag-pipeline]] — the RAG pipeline consuming age group metadata for retrieval
- [[summaries/AGE_OVERRIDE_GUIDE]] — full documentation for the override system
- [[summaries/PERMANENT_SOLUTION_SUMMARY]] — implementation summary for the persistent override solution
- [[summaries/QUICK_REFERENCE]] — quick-reference guide for common override tasks
- [[summaries/SESSION_SUMMARY]] — session log documenting the February 2026 data cleaning effort
- [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] — companion improvements to diagnosis extraction made in the same session

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.older-adult]]