---
sources: [summaries/SESSION_SUMMARY.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md]
brief: The DSM-IV multiaxial system organizing diagnoses across labeled Axes, common in older neuropsychological reports.
---

# Multiaxial Diagnosis Format

The **multiaxial diagnosis format** is a structured diagnostic framework introduced in DSM-IV that organizes clinical findings across multiple labeled axes (Axis I through Axis V). Although the DSM-5 (2013) retired this system in favor of a unified dimensional approach, many neuropsychological reports produced before and during the transition period continue to use the multiaxial layout. Parsers and automated extraction tools must explicitly handle this format to avoid missing valid diagnoses.

## Structure of the Multiaxial System

| Axis | Content |
|------|---------|
| Axis I | Clinical psychiatric disorders (e.g., ADHD, depression, anxiety) |
| Axis II | Personality disorders and intellectual disabilities |
| Axis III | General medical conditions |
| Axis IV | Psychosocial and environmental problems |
| Axis V | Global Assessment of Functioning (GAF) score |

In practice, most neuropsychological reports using this format focus on Axes I and II, with Axis I often subdivided into **Diagnostic Considerations** and **Rule Out** (R/O) categories.

## Typical Report Layout

A multiaxial diagnostic section commonly appears like this:

```
DIAGNOSTIC CONSIDERATIONS
Axis I Diagnostic Considerations:
  F34.1  Dysthymic disorder
  F43.22 Adjustment disorder with anxiety
  F90.2  ADHD, combined type
Axis I Rule Out:
  F32.9  Major depressive disorder
  F40.1  Social phobia
Axis II:
  799.9  Diagnosis Deferred on Axis II
```

This structure creates two parsing challenges:
1. **Section header recognition** — The section is headed "DIAGNOSTIC CONSIDERATIONS" rather than the simpler "Diagnosis" or "Diagnostic Impressions".
2. **Subsection label interference** — Lines like "Axis I Rule Out:" appear inside the diagnostic block and can be mistakenly captured as diagnosis names.

## Parsing Challenges and Solutions

As documented in [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]], the `report_parser.py` module previously failed to extract any diagnoses from multiaxial-format reports because:

- `"DIAGNOSTIC CONSIDERATIONS"` was not listed as a recognized section header.
- Subsection labels (`"Axis I Diagnostic Considerations"`, `"Axis I Rule Out"`, `"Axis II"`) were passed through as diagnosis name strings.

Three targeted fixes resolved these issues:

### 1. Expanded Header Matching
Added `Diagnostic\s+Considerations?` and `Diagnostic\s+Summary` to the `DIAGNOSIS_SECTION_HEADERS` list, enabling the parser to enter multiaxial sections.

### 2. Optional ICD-10 Decimal
The [[concepts/icd10-diagnosis-extraction]] regex was relaxed from requiring a decimal point (`F90.2`) to making it optional (`F90`), accommodating shorthand codes sometimes used in multiaxial sections.

### 3. Subsection Header Filtering
A post-processing filter strips any "diagnosis" whose name matches the pattern:
```
^Axis\s+[IV]+\s+(?:Diagnostic\s+)?(?:Considerations?|Rule\s+Out|R/?O)\s*:?$
```
This cleanly removes axis labels while preserving the actual coded diagnoses beneath them.

## Rule-Out Diagnoses

The "Axis I Rule Out" subsection records diagnoses that are being actively considered but not yet confirmed — equivalent to provisional or differential diagnoses. The current parser extracts these diagnoses but does not yet flag them as rule-outs. A planned future enhancement is to tag such entries with a `provisional: true` attribute or similar marker to distinguish confirmed from tentative diagnoses. See [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] for the full roadmap.

## Relationship to DSM-5

DSM-5 removed the multiaxial system, encouraging clinicians to list all conditions in a single unified section, typically headed "Diagnoses," "Diagnostic Impressions," or "DSM-5 Diagnoses." Many contemporary reports use this simpler format, but a substantial number of archival and transitional reports retain the Axis-based layout. Any robust [[concepts/neuropsych-report-parsing]] system must support both conventions.

## Impact on Data Quality

In the autism RAG dataset, 23% of reports (24 out of ~104) initially showed zero extracted diagnoses, with multiaxial format being a primary cause. After the parser fixes, that figure dropped to an estimated 10–12 reports. The remaining zero-diagnosis documents are predominantly textbook samples and recommendations-only reports without formal diagnostic sections. See [[concepts/report-parser-quality]] for broader coverage metrics.

## Related Concepts

- [[concepts/icd10-diagnosis-extraction]] — Regex patterns for extracting ICD-10 and DSM codes from text
- [[concepts/neuropsych-report-parsing]] — Overall pipeline for parsing neuropsychological report documents
- [[concepts/dsm5-diagnosis-normalization]] — Normalizing varied diagnostic label formats to DSM-5 conventions
- [[concepts/clinical-report-structure]] — Structural conventions across different report types
- [[concepts/report-parser-quality]] — Metrics and improvement strategies for diagnosis extraction coverage
- [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] — Source document detailing the multiaxial parsing fixes
- [[summaries/DIAGNOSIS_FIX_SUMMARY]] — Related fix summary for diagnosis extraction issues


See also: [[summaries/SESSION_SUMMARY]]