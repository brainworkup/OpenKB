---
sources: [summaries/SESSION_SUMMARY.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/DIAGNOSIS_FIX_SUMMARY.md]
brief: Automated parsing of ICD-10 diagnostic codes and labels from unstructured neuropsychological report text.
---

# ICD-10 Diagnosis Extraction from Clinical Text

ICD-10 diagnosis extraction refers to the automated parsing of structured diagnostic codes and labels from unstructured clinical narrative text — particularly neuropsychological evaluation reports. This is a core component of [[concepts/neuropsych-report-parsing]] and clinical NLP pipelines.

## Why It Is Difficult

Clinical reports do not follow a single universal format. Diagnostic sections may appear under a wide variety of headers, and ICD-10 codes themselves may be written with or without decimal precision. Common obstacles include:

- **Header variability:** A diagnosis section may be labeled `Diagnostic Considerations`, `DSM-5 Diagnoses`, `Diagnostic Summary`, or use multiaxial terminology like `Axis I Diagnostic Considerations`. The header `DIAGNOSTIC CONSIDERATIONS` was entirely unrecognized prior to parser improvements documented in [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]].
- **Code format variability:** The same disorder may be coded as `F90.2` (with decimal) or `F90` (without). An overly strict regex requiring a decimal silently drops valid codes.
- **Subsection noise:** Axis labels and rule-out headers (`Axis I Rule Out`, `Axis II`) can be mistakenly captured as diagnosis lines if no post-processing filter is applied.
- **Non-standard reports:** Textbook examples, recommendations-only reports, and unusual layouts may not yield extractable codes at all.

See [[summaries/DIAGNOSIS_FIX_SUMMARY]] and [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] for concrete case studies of these issues and their resolutions.

## Root Causes of Extraction Failures

Analysis of the neuropsych recommendations dataset (438 recommendations from 104 reports) identified four specific failure modes responsible for 24 reports (23% of the dataset) returning zero diagnoses:

1. **Missing section headers** — `DIAGNOSTIC CONSIDERATIONS` was absent from the recognized header list.
2. **Limited header variants** — `Diagnostic Summary` and `DSM-5 Diagnoses` were not covered.
3. **Strict ICD-10 format** — The regex required a decimal point, rejecting codes like `F90`.
4. **Subsection labels parsed as diagnoses** — Lines such as `Axis I Rule Out:` were incorrectly treated as diagnosis name entries.

These findings emerged from analysis of `data/reports_missing_diagnoses.txt` (generated during data cleaning) and were addressed in `src/report_parser.py` as part of the February 2026 pipeline improvements documented in [[summaries/SESSION_SUMMARY]].

## Extraction Strategy

### Section Header Recognition

The parser must identify the diagnostic section before attempting code extraction. A robust implementation recognizes multiple header variants via regex:

```
Diagnostic\s+Impressions?
Diagnostic\s+Considerations?      # multiaxial format
Diagnostic\s+Summary
(?:Proposed\s+)?(?:DSM[- ]?(?:IV|5|V)\s+)?Diagnostic\s+Formulation
DSM[- ]?(?:IV|5|V)\s+Diagnos[ei]s?
Diagnos[ei]s?
```

The February 2026 improvements (see [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]) added three critical headers that were previously missing:
- `Diagnostic Considerations`
- `Diagnostic Summary`
- `DSM-5 Diagnoses` / `DSM-IV Diagnoses`

Additional headers worth considering for edge cases include `Clinical Impressions`, `Diagnostic Conclusions`, and `Assessment and Diagnosis`.

### Multi-Format Code Pattern Matching

As documented in `src/report_parser.py` (see [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] and [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]), the parser handles **six distinct code format patterns** to accommodate the full range of report styles encountered in practice:

- **Format 4 (Name-First):** `ADHD, Combined Type 314.01 (F90.2)` — diagnosis name appears first, codes at the end
- **Format 1 (Codes-First):** `314.01 (F90.2) ADHD, Combined Presentation` — codes appear before the name
- **Format 2 (ICD-10 Only):** `F90.2 ADHD` — only an ICD-10 code is present; this is the broadest pattern and is tried last to avoid false positives
- Additional variants covering partial codes, parenthetical formats, and legacy DSM-IV codes

The deliberate ordering strategy — trying the most specific patterns first and the broadest pattern last — reduces false positives while maintaining high recall across varied report formats. A flexible regex pattern should make the decimal optional:

```
(?P<icd10>[A-Z]\d{1,2}(?:\.\d+)?[A-Z]?\d*)
```

This handles:
- **With decimal:** `F90.2`, `F43.22`, `F34.1`
- **Without decimal:** `F90`, `F43`, `Z13`
- **Legacy codes:** Non-ICD formats like `799.9` (DSM-IV era) may also appear in older reports

The February 2026 fix made the decimal point optional in the core regex, recovering codes that were previously silently dropped.

### Subsection Header Filtering

After candidate lines are captured, a post-processing filtering pass removes false positives — lines that look structurally valid but are actually section labels:

```python
diagnoses = [
    d for d in diagnoses
    if not re.match(
        r"^Axis\s+[IV]+\s+(?:Diagnostic\s+)?(?:Considerations?|Rule\s+Out|R/?O)\s*:?$",
        d["name"],
        re.IGNORECASE
    )
]
```

Filtered out by this guard:
- `Axis I Diagnostic Considerations`
- `Axis I Rule Out`
- `Axis II`

## Multiaxial Format Handling

Older neuropsychological reports (pre-DSM-5) used a **multiaxial system** where diagnoses were organized across Axis I (clinical disorders), Axis II (personality/developmental), Axis III (medical), and so on. See [[concepts/multiaxial-diagnosis-format]] for details. The extractor must:

1. Recognize axis-labeled headers as section structure, not diagnosis entries.
2. Extract codes listed beneath those headers as actual diagnoses.
3. Filter out the axis header lines themselves from the final diagnosis list.

## Integration with Downstream Normalization

Once ICD-10 codes are extracted, the pipeline performs several downstream normalization steps (see [[concepts/dsm5-diagnosis-normalization]]):

1. **Canonical name normalization** — `canonicalize_diagnosis_name()` maps all variants of a disorder (e.g., all ADHD subtypes) to a single canonical form.
2. **DSM-5 category assignment** — `classify_dsm5_category()` assigns each diagnosis to one of 12 DSM-5 categories using both ICD-10 code and name patterns.
3. **Deduplication and merging** — `_merge_equivalent_diagnoses()` collapses equivalent diagnoses while preserving canonical names and all aliases.

This enables consistent querying and reporting regardless of whether the source report used ICD-10 or older coding conventions, and supports downstream metadata filtering in the [[concepts/recommendation-rag-pipeline]].

## Role in the RAG Pipeline

Extracted diagnoses do not merely populate report metadata — they are attached directly to `RecommendationChunk` objects created during recommendation section parsing. Each chunk carries a `diagnoses` field derived from the extraction results. During semantic search in `src/retrieval.py`, diagnosis list membership is used as a metadata filter (and post-filter in the app layer) to ensure retrieved recommendation examples are clinically relevant to the patient's specific conditions.

Specifically, `VectorStore.search_filtered()` over-fetches candidates (k×3) from the FAISS index and then applies list membership checks against the `diagnoses` metadata field. An additional post-filter step in the app layer further narrows results to the selected disorder. This tight coupling between diagnosis extraction accuracy and search relevance makes ICD-10 parsing a foundational dependency of the entire [[concepts/retrieval-augmented-generation]] workflow.

The full pipeline is:

```
extract_diagnoses() → canonicalize → classify → merge
    └── attach to RecommendationChunk.diagnoses
        └── used as metadata filter in VectorStore.search_filtered()
            └── post-filtered by diagnosis in app layer
```

See [[concepts/faiss-vector-index]] for details on the vector search layer and [[concepts/sentence-transformer-embeddings]] for the embedding model used.

## Observed Impact

The parser improvements described in [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] and confirmed in [[summaries/SESSION_SUMMARY]] fixed all four root-cause failure modes in `src/report_parser.py`. The result, measured across 104 reports and 438 recommendations:

| Metric | Before | After (Expected) |
|--------|--------|------------------|
| Reports with zero diagnoses | 24 (23%) | ~10–12 (9–11%) |
| Recommendations with diagnoses | 77.9% | ~90% |
| Recommendations without diagnoses | 22.1% | ~10% |

These improvements reduce the ~50% of previously undetected diagnoses to a residual set of genuinely non-standard reports. The remaining zero-diagnosis reports are predominantly:
- Textbook example PDFs (`donders__*`, `nelson_sample_*`)
- Non-standard format reports requiring manual review
- Recommendations-only reports (no formal diagnosis section)

All changes were backward-compatible — existing patterns still function, new patterns are purely additive, and the data structure is unchanged.

## Example Extraction

From `np_report_chau_bryan_01_10_20.pdf`, which previously yielded **0 diagnoses**, the improved parser now extracts **7 diagnoses**:

| Code | Diagnosis |
|------|-----------|
| F34.1 | Dysthymic disorder |
| F43.22 | Adjustment disorder with anxiety |
| F90.2 | ADHD, combined type |
| F32.9 | Major depressive disorder |
| F40.1 | Social phobia |
| F41.1 | Generalized anxiety disorder |
| 799.9 | Diagnosis Deferred on Axis II |

## Relationship to Other Data Quality Improvements

Diagnosis extraction improvements were developed alongside age group classification fixes in the same February 2026 session. The age override system (see [[summaries/AGE_OVERRIDE_GUIDE]] and [[summaries/PERMANENT_SOLUTION_SUMMARY]]) uses a complementary approach: a configuration file (`data/age_overrides.json`) allows manual correction of extracted metadata without modifying source PDFs. A similar override mechanism has been proposed for edge-case diagnosis assignments. Both systems share the design principle of persisting corrections across re-ingestion. See [[concepts/age-group-classification]] for details on the age classification pipeline.

## Future Enhancements

1. **Diagnosis override system** — Manual overrides for edge cases, analogous to the existing age override mechanism in `data/age_overrides.json`.
2. **Rule-out flagging** — Mark provisional/rule-out diagnoses distinctly from confirmed diagnoses.
3. **Multi-column layout detection** — Better handling of non-linear diagnosis table formats.
4. **Confidence scores** — Per-diagnosis parsing confidence indicators.
5. **Additional header patterns** — Cover `Clinical Impressions`, `Diagnostic Conclusions`, `Assessment and Diagnosis`, `Results and Diagnosis`.
6. **Removal of textbook PDFs** — Non-patient source documents in the knowledge base bucket may be excluded to improve overall extraction rates.

## Related Concepts

- [[concepts/neuropsych-report-parsing]]
- [[concepts/multiaxial-diagnosis-format]]
- [[concepts/dsm5-diagnosis-normalization]]
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/pdf-data-extraction]]
- [[concepts/report-parser-quality]]
- [[concepts/recommendation-rag-pipeline]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/phi-deidentification-pipeline]]
- [[concepts/faiss-vector-index]]
- [[concepts/sentence-transformer-embeddings]]
- [[concepts/age-group-classification]]

## Source Documents

- [[summaries/DIAGNOSIS_FIX_SUMMARY]]
- [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]
- [[summaries/SESSION_SUMMARY]]
- [[summaries/AGE_OVERRIDE_GUIDE]]
- [[summaries/PERMANENT_SOLUTION_SUMMARY]]
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]
