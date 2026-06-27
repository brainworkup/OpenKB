---
doc_type: short
full_text: sources/DIAGNOSIS_FIX_SUMMARY.md
---

# Diagnosis Parser Improvements — Fix Summary

## Overview

This document summarizes improvements made to `src/report_parser.py` to fix a problem where 24 reports (23% of the dataset) had no diagnoses extracted, despite containing valid diagnostic sections. The fix targets [[concepts/neuropsych-report-parsing]] and improves coverage of [[concepts/icd10-diagnosis-extraction]].

## Problem Identified

- **24 reports** (23% of dataset) returned zero diagnoses.
- Root cause: The parser did not recognize common diagnostic section header variants found in neuropsychological reports, particularly **multiaxial format** headers.
- Example missed header: `DIAGNOSTIC CONSIDERATIONS`, `Axis I Diagnostic Considerations:`

## Changes Made to `src/report_parser.py`

### 1. New Section Header Patterns (Lines ~794–805)
Added recognition for:
- `Diagnostic Considerations`
- `Diagnostic Summary`
- `DSM-5 Diagnoses` / `DSM-IV Diagnoses`

### 2. Flexible ICD-10 Code Matching (Lines ~817–823)
- **Before:** Required decimal format, e.g., `F90.2`
- **After:** Accepts both `F90.2` and `F90` (decimal optional)

### 3. Subsection Header Filtering (Lines ~1043–1052)
Filters out false positives such as:
- `Axis I Diagnostic Considerations`
- `Axis I Rule Out`
- `Axis II`

## Expected Improvement

| Metric | Before | After |
|---|---|---|
| Reports with no diagnoses | 24 (23%) | ~10–12 (9–11%) |

Remaining zero-diagnosis reports are expected to be:
- Textbook examples (`donders__*`, `nelson_sample_*`)
- Non-standard format reports
- Recommendations-only reports

## Example: `np_report_chau_bryan_01_10_20.pdf`

**Before:** 0 diagnoses extracted  
**After:** 7 diagnoses extracted, including:
- `F34.1` Dysthymic disorder
- `F43.22` Adjustment disorder with anxiety
- `F90.2` ADHD, combined type
- `F32.9` Major depressive disorder
- `F40.1` Social phobia
- `F41.1` Generalized anxiety disorder
- `799.9` Diagnosis Deferred on Axis II

## Files Modified / Created

- **Modified:** `src/report_parser.py`
- **Created:** `DIAGNOSIS_PARSER_IMPROVEMENTS.md` (full documentation)
- **Created:** `data/reports_missing_diagnoses.txt` (reference list)

## Testing

Run the ingestion pipeline:
```bash
python -m src.ingest_recommendations
```
Then verify with a JSON inspection script against `data/recommendations_kb.json`.

## Next Steps

1. Run ingestion pipeline and verify reduced missing-diagnosis count.
2. Review remaining reports without diagnoses.
3. Add more header patterns or use a diagnosis override system if needed.
4. Consider removing textbook example PDFs from the ingestion folder.

## Related Concepts
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/phi-data-handling]]

- [[concepts/neuropsych-report-parsing]]
- [[concepts/icd10-diagnosis-extraction]]
- [[concepts/multiaxial-diagnosis-format]]
- [[concepts/knowledge-base-architecture]]
- [[concepts/dsm5-diagnosis-normalization]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/pdf-data-extraction]]