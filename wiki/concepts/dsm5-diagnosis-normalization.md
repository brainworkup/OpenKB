---
sources: [summaries/README.md, summaries/SESSION_SUMMARY.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md]
brief: Mapping raw clinical diagnosis strings to canonical DSM-5 names and categories for reliable NLP retrieval.
---

# DSM-5 Diagnosis Normalization

DSM-5 diagnosis normalization is the process of mapping raw diagnosis strings extracted from clinical documents — which may contain ICD-10 codes, legacy DSM-IV names, garbled parsing artifacts, or inconsistent capitalization — to their canonical DSM-5 names and diagnostic categories. It is a critical preprocessing step in any [[concepts/clinical-nlp-pipelines]] or [[concepts/neuropsychological-assessment-pipeline]] that aggregates data across multiple reports. In retrieval-augmented generation systems built on clinical report corpora (such as the Autism RAG System described in [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] and the Neuropsychological Report RAG Pipeline described in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]), normalization happens at ingestion time and is embedded in the metadata attached to every stored chunk.

## Why Normalization Is Hard

Neuropsychological reports are authored by many different clinicians over many years. The same diagnosis may appear as:
- `Posttraumatic Stress Disorder` (DSM-5 spelling)
- `Post-Traumatic Stress Disorder` (hyphenated legacy form)
- `(309.9) Adjustment disorder, unspecified` (DSM-IV numeric code prepended)
- `Expression, 315.2 (F81.81); and Mathematics` (PDF parsing artifact preserving partial text)
- `Schizophrenia, Undifferentiated Type` (DSM-IV subtype removed in DSM-5)
- `Specific spelling disorder` (ICD-10 name, not DSM-5)

Without normalization, each of these becomes a separate entry in a knowledge base, fragmenting retrieval and inflating apparent diagnostic diversity.

## The Extraction Problem: Missing Diagnoses

Before normalization can even begin, the parser must successfully *detect* diagnosis sections in the source PDF. A significant early gap in the autism-rag system was that 24 reports (23% of the dataset) returned zero diagnoses — not because diagnoses were absent, but because the parser did not recognize common section header variants. The fix, documented in [[summaries/DIAGNOSIS_FIX_SUMMARY]], addressed three root causes in `src/report_parser.py`:

1. **New section header patterns**: Added recognition for `Diagnostic Considerations`, `Diagnostic Summary`, `DSM-5 Diagnoses`, and `DSM-IV Diagnoses` alongside previously supported headers. This is essential for multiaxial format reports that use headers like `DIAGNOSTIC CONSIDERATIONS` rather than a bare `Diagnoses:` label — see [[concepts/multiaxial-diagnosis-format]].

2. **Flexible ICD-10 code matching** ([[concepts/icd10-diagnosis-extraction]]): The regex was relaxed to accept both `F90.2` (with decimal) and `F90` (without decimal), catching codes written in the abbreviated form common in older reports.

3. **Subsection header filtering**: False positive matches on lines like `Axis I Diagnostic Considerations`, `Axis I Rule Out`, and `Axis II` are now filtered out before diagnosis records are constructed.

After this fix, the expected number of reports with zero diagnoses dropped from 24 (23%) to approximately 10–12 (9–11%), with the remainder being textbook examples, non-standard format reports, or recommendations-only documents. A concrete example: `np_report_chau_bryan_01_10_20.pdf` went from 0 to 7 diagnoses extracted, including `F34.1 Dysthymic disorder`, `F43.22 Adjustment disorder with anxiety`, and `F90.2 ADHD, combined type`.

## Multi-Format Pattern Matching

The pipeline documented in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]] makes explicit the breadth of format variation that any robust extractor must handle. The `extract_diagnoses()` function in `src/report_parser.py` applies **six distinct regex patterns** in a defined order of specificity:

- **Format 4 (name-first)**: `ADHD, Combined Type 314.01 (F90.2)` — diagnosis name appears before codes, matched by `_DX_FMT4.finditer()`
- **Format 1 (codes-first)**: `314.01 (F90.2) ADHD, Combined Presentation` — codes precede the name, matched by `_DX_FMT1.finditer()`
- **Format 2 (ICD-10 only)**: `F90.2 ADHD` — only the ICD-10 code is present, broadest pattern, tried last via `_DX_FMT2.finditer()`

More specific patterns are attempted first; the broadest ICD-10-only pattern is the final fallback. This ordering prevents over-eager matches that would capture partial lines or subsection labels. The pipeline also implements `_merge_equivalent_diagnoses()`, which deduplicates entries sharing the same canonical name regardless of how they originally appeared in the source text. Normalization calls within this flow include `canonicalize_diagnosis_name()` (normalize ADHD variants and other aliases to canonical form) and `classify_dsm5_category()` (assign to one of 12 DSM-5 categories).

## Role in Clinical Ingestion Pipelines

In the Autism RAG System and the broader neuropsychological report pipeline, DSM-5 mapping is applied during report ingestion orchestrated by `src/ingest_recommendations.py`. After PDF text is extracted via PyMuPDF and structured through `parse_report()`, the `extract_diagnoses()` function performs code-to-DSM-5 standardization before chunks are embedded and stored. The full sequence, as documented in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]], is:

1. `parse_pdf()` — raw text extraction via PyMuPDF (`fitz.open`)
2. `extract_report_metadata()` → `extract_diagnoses()` — diagnosis normalization
3. `deidentify_recommendations()` — PHI removal (see [[concepts/phi-deidentification-pipeline]])
4. `generate_embeddings()` via SentenceTransformer → [[concepts/faiss-vector-index]] population via `VectorStore.add_embeddings()`
5. `vs.save()` — persist index to disk

Every FAISS vector carries normalized diagnosis metadata, enabling reliable filtered retrieval via metadata filters — including list-membership checks for multi-valued diagnosis fields (e.g., a patient with comorbid ASD, ADHD, and SLD). The `VectorStore.search_filtered()` function in `src/retrieval.py` over-fetches candidates (k×3) from the FAISS index and applies metadata filters in a post-processing pass, making clean normalized diagnosis tags essential to recall. An additional post-filter by diagnosis list membership is applied at the app layer in `app_recommendations.py`.

## Two-Pass Architecture: Code First, Name Second

The most robust approach runs ICD-10/DSM-IV numeric code matching *before* name-based string matching:

1. **Code-based lookup**: If a diagnosis record contains a valid ICD-10 code (e.g., `F81.81`), map it directly to the canonical DSM-5 name regardless of what the name string says. This catches parsing artifacts where the name is garbled but the code is intact.
2. **Name-based lookup**: Normalize the name string (strip prefixes, lowercase, collapse whitespace) and match against a table of canonical names.
3. **Fallback**: If neither pass produces a match, return the cleaned raw string.

### ICD-10 SLD Mapping Example
```
F81.0  → SLD, With Impairment in Reading
F81.1  → SLD, With Impairment in Written Expression
F81.81 → SLD, With Impairment in Written Expression
F81.2  → SLD, With Impairment in Mathematics
```
This caught the artifact `Expression, 315.2 (F81.81); and Mathematics` and correctly mapped it to SLD Written Expression.

## Category Assignment from ICD-10 Ranges

DSM-5 diagnostic categories can be inferred from ICD-10 code ranges, but the mapping must be precise. The `classify_dsm5_category()` function in `src/report_parser.py` assigns each diagnosis to one of **12 DSM-5 categories** using both ICD-10 code ranges and name patterns. A common error is using an overly broad regex like `F9[0-9]` which conflates:

| ICD-10 Range | DSM-5 Category |
|---|---|
| F70–F79 | Neurodevelopmental (Intellectual Disability) |
| F80–F89 | Neurodevelopmental (SLD, ASD, Language) |
| F90 | Neurodevelopmental (ADHD) |
| F91 | **Disruptive, Impulse-Control, and Conduct** |
| F95 | Neurodevelopmental (Tic Disorders) |
| F98 | **Elimination Disorders** (not Neurodevelopmental) |
| F20, F22, F25 | Schizophrenia Spectrum |
| F21 | **Personality Disorders** (Schizotypal PD — cross-listed in DSM-5 but conventionally filed with PDs) |
| F34.81 | **Depressive Disorders** (DMDD) |

Correct range-based classification feeds directly into [[concepts/clinical-data-management]].

## String Normalization Steps

Before any lookup, raw diagnosis strings should be processed through a `match_key` normalization pipeline:

1. Unicode normalization (NFC → ASCII where safe)
2. Strip parenthesized DSM-IV codes: `(309.9)`, `(315.2)`
3. Strip leading ICD-10 codes: `F81.81 ` at start of string
4. Lowercase
5. Collapse whitespace
6. Strip trailing punctuation: `:;,-`

This creates a stable key for dictionary lookup without altering the display name. The `canonicalize_diagnosis_name()` function in `src/report_parser.py` implements this normalization, called during the `_merge_equivalent_diagnoses()` deduplication pass.

## Common Normalization Rules

### Trauma- and Stressor-Related
- `Post-Traumatic Stress Disorder` → `Posttraumatic Stress Disorder` (DSM-5 uses one word)
- `(309.9) Adjustment disorder, unspecified` → `Adjustment Disorder, Unspecified`

### Neurodevelopmental
- `Intellectual Developmental Disorder, Mild` → `Intellectual Disability, Mild` (DSM-5 preferred term)
- `Specific spelling disorder` → `Specific Learning Disorder, With Impairment in Written Expression`
- All ADHD subtypes → normalized to the canonical combined/inattentive/hyperactive-impulsive presentation names

### Schizophrenia Spectrum
- `Schizophrenia, Undifferentiated Type` → `Schizophrenia` (DSM-5 removed all subtypes)
- `Schizoaffective disorder, unspecified` → `Schizoaffective Disorder`

### Anxiety
- `Fear of flying` / `Other situational type phobia` → `Specific Phobia`

### Depressive
- `Disruptive mood dysregulation disorder` → `Disruptive Mood Dysregulation Disorder`

### Bipolar
- `Bipolar disorder, unspecified, with psychotic features` → `Unspecified Bipolar Disorder`

## Merging Equivalent Diagnoses

After canonicalization, a secondary merge pass collapses entries with the same canonical name regardless of how they originally appeared. The knowledge base in the autism-rag project went from **72 → 66 unique disorders** after applying these rules, with 6 duplicate clusters merged. The `_merge_equivalent_diagnoses()` function in `src/report_parser.py` preserves canonical names while collecting all aliases encountered across reports.

## Relation to Retrieval Quality

In a retrieval-augmented generation system built on clinical reports, fragmented diagnosis names cause retrieval failures: a query for `PTSD` may miss chunks tagged `Post-Traumatic Stress Disorder`. Normalization ensures that all chunks for a given diagnosis share the same metadata field value, enabling reliable filtered retrieval. This is particularly important for multi-valued metadata fields (e.g., a patient with comorbid ASD, ADHD, and SLD) where the filtered search — implemented via `VectorStore.search_filtered()` in `src/retrieval.py` — uses list-membership checks rather than simple equality. The system over-fetches candidates (k×3) from the FAISS index and then applies metadata filters in a post-processing pass, making clean normalized diagnosis tags essential to recall. Diagnosis filtering also applies at the app layer as a post-filter list comprehension in `app_recommendations.py`.

The [[concepts/age-group-classification]] metadata field, stored alongside diagnosis metadata in the same vector records, is similarly normalized to enable age-stratified filtered retrieval in clinical RAG pipelines.

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/clinical-data-management]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/knowledge-base-architecture]]
- [[concepts/pdf-data-extraction]]
- [[concepts/clinical-report-structure]]
- [[concepts/phi-deidentification-pipeline]]
- [[concepts/faiss-vector-index]]
- [[concepts/age-group-classification]]
- [[concepts/icd10-diagnosis-extraction]]
- [[concepts/multiaxial-diagnosis-format]]
- [[concepts/neuropsych-report-parsing]]
- [[concepts/recommendation-rag-pipeline]]
- [[concepts/sentence-transformer-embeddings]]
- [[concepts/text-chunking]]

See also: [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]], [[summaries/DIAGNOSIS_FIX_SUMMARY]], [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]], [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/SESSION_SUMMARY]]

See also: [[summaries/README]]