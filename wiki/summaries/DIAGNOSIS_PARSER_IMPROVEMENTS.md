---
doc_type: short
full_text: sources/DIAGNOSIS_PARSER_IMPROVEMENTS.md
---

# Summary: Diagnosis Parser Improvements

## Overview

This document describes targeted improvements to `src/report_parser.py` that address a critical data gap: 24 reports (23% of the dataset) had zero diagnoses extracted, despite many containing real diagnostic content. The fixes are backward-compatible and expected to reduce the zero-diagnosis count from 24 to ~10–12 reports.

## Root Causes Identified

Four distinct failure modes were diagnosed:

1. **Unrecognized section headers** — "DIAGNOSTIC CONSIDERATIONS" (multiaxial format) was not in the header list.
2. **Missing header variants** — "Diagnostic Summary" and "DSM-5 Diagnoses" were absent.
3. **Strict ICD-10 regex** — Required a decimal point (e.g., `F90.2`), rejecting shorthand codes like `F90`.
4. **Subsection labels parsed as diagnoses** — Lines such as "Axis I Rule Out:" were incorrectly treated as diagnosis names.

## Key Code Changes

### Expanded Section Header Patterns
Added three new regex entries to `DIAGNOSIS_SECTION_HEADERS`:
- `Diagnostic\s+Considerations?` — covers [[concepts/multiaxial-diagnosis-format|multiaxial report format]]
- `Diagnostic\s+Summary`
- `DSM[- ]?(?:IV|5|V)\s+Diagnos[ei]s?`

### Optional ICD-10 Decimal
Changed the ICD-10 capture group from requiring a decimal to making it optional:
```
(?P<icd10>[A-Z]\d{1,2}(?:\.\d+)?[A-Z]?\d*)
```
This now accepts both `F90.2` and `F90`. See [[concepts/icd10-diagnosis-extraction]] for broader context on ICD-10 code parsing strategies.

### Subsection Header Filtering
Added a post-processing filter that removes parsed "diagnoses" matching [[concepts/multiaxial-diagnosis-format|multiaxial subsection labels]] (e.g., "Axis I Rule Out", "Axis II", "Axis I Diagnostic Considerations") using a regex guard.

## Concrete Example

The report `np_report_chau_bryan_01_10_20.pdf` previously yielded **0 diagnoses**. After the fix it yields **7 diagnoses** (F34.1, F43.22, F90.2, F32.9, F40.1, F41.1, 799.9), extracted from a "DIAGNOSTIC CONSIDERATIONS" section with multiaxial subsections.

## Expected Impact

| State | Reports with No Diagnoses |
|---|---|
| Before | 24 (23% of dataset) |
| After (expected) | ~10–12 |

Remaining zero-diagnosis reports are expected to be textbook examples (e.g., `donders__*`, `nelson_sample_*`) or genuinely recommendations-only reports — not real patient records.

## Testing Procedure

Re-ingest via `python -m src.ingest_recommendations`, then inspect `data/recommendations_kb.json` for the updated count of reports with empty `diagnoses` lists.

## Future Enhancements

- **Diagnosis override system** — Manual overrides for edge cases (analogous to age overrides, see [[summaries/AGE_OVERRIDE_GUIDE]]).
- **Rule-out flagging** — Mark provisional/rule-out diagnoses distinctly, relevant to [[concepts/dsm5-diagnosis-normalization]].
- **Multi-column layout detection** — Handle non-linear diagnosis table formats, a challenge for [[concepts/neuropsych-report-parsing]].
- **Confidence scores** — Per-diagnosis parsing confidence.
- **Additional header patterns** — "Clinical Impressions", "Diagnostic Conclusions", "Assessment and Diagnosis"

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-report-structure]]
- [[concepts/luria-neuropsych-pipeline]]

- [[concepts/icd10-diagnosis-extraction]] — Regex strategies for ICD-10 code extraction
- [[concepts/neuropsych-report-parsing]] — Overall structure of the neuropsychological report parser
- [[concepts/multiaxial-diagnosis-format]] — DSM-IV multiaxial (Axis I/II) report conventions
- [[concepts/report-parser-quality]] — Dataset completeness and extraction gap analysis
- [[concepts/clinical-nlp-pipelines]] — Broader pipeline context for clinical text extraction
- [[concepts/dsm5-diagnosis-normalization]] — Normalizing diagnostic labels across report formats

## Files Modified

- `src/report_parser.py` — Lines ~794–805 (headers), ~817–823 (ICD-10 regex), ~1043–1052 (subsection filter)