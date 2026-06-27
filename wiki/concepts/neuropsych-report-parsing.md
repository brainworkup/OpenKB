---
sources: [summaries/SESSION_SUMMARY.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/QUICK_REFERENCE.md, summaries/PERMANENT_SOLUTION_SUMMARY.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/DIAGNOSIS_FIX_SUMMARY.md]
brief: Automated extraction of diagnoses, scores, demographics, and recommendations from heterogeneous PDF neuropsychological reports.
---

# Neuropsychological Report Parsing

Neuropsychological report parsing refers to the automated extraction of structured clinical data — diagnoses, test scores, demographic fields, and narrative sections — from PDF-based neuropsychological evaluation reports. Because these reports are produced by many different clinicians using varied templates, parsers must handle significant format heterogeneity.

See also: [[summaries/DIAGNOSIS_FIX_SUMMARY]] and [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] for concrete examples of parser improvement work, [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] for a full codemap of the production pipeline, and [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]] for the complete end-to-end system architecture including PHI de-identification, embedding generation, and LLM-powered recommendation retrieval.

## Core Challenges

### Header Variability
Diagnostic sections appear under many header labels across reports:
- `DIAGNOSTIC CONSIDERATIONS`
- `Diagnostic Summary`
- `DSM-5 Diagnoses` / `DSM-IV Diagnoses`
- `Axis I Diagnostic Considerations` (multiaxial format)
- `Clinical Impressions`, `Diagnostic Conclusions`, `Assessment and Diagnosis` (additional variants)

Parsers must recognize all common variants or risk missing entire diagnostic sections. A failure to handle header variability was responsible for **23% of reports** returning zero diagnoses before the fixes described in [[summaries/DIAGNOSIS_FIX_SUMMARY]] and [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]. The expanded `DIAGNOSIS_SECTION_HEADERS` list in `src/report_parser.py` now includes regex patterns for `Diagnostic\s+Considerations?`, `Diagnostic\s+Summary`, and `DSM[- ]?(?:IV|5|V)\s+Diagnos[ei]s?`, reducing that failure rate from 23% to an expected 9–11%.

The [[summaries/SESSION_SUMMARY]] session reinforced these improvements and confirmed the fix applies to the full 104-report corpus: before the changes, 24 reports had no diagnoses extracted; after, the expected count is ~10–12. New patterns added in that session include `Diagnostic Considerations`, `Diagnostic Summary`, `DSM-5 Diagnoses`, and `DSM-IV Diagnoses`, plus flexible ICD-10 regex to handle codes with and without decimal points.

### Multiaxial vs. Uniaxial Format
Older reports follow the **multiaxial diagnosis format** (DSM-IV era), where diagnoses are organized under Axis I, Axis II, etc. Newer reports use a flat DSM-5 list. Parsers must handle both. The report `np_report_chau_bryan_01_10_20.pdf` is a canonical example of this challenge: before parser improvements it yielded 0 diagnoses; after, it yields 7, all drawn from a `DIAGNOSTIC CONSIDERATIONS` section with Axis I and Axis II subsections. See [[concepts/multiaxial-diagnosis-format]] for detail on this distinction.

### ICD-10 Code Flexibility
[[concepts/icd10-diagnosis-extraction]] is complicated by inconsistent code formatting:
- Some reports use decimals: `F90.2`
- Others omit decimals: `F90`
- Legacy codes appear: `799.9` (ICD-9 era)

Regex patterns must be written to accept optional decimals and mixed code systems. The updated pattern `(?P<icd10>[A-Z]\d{1,2}(?:\.\d+)?[A-Z]?\d*)` makes the decimal optional, handling both `F90.2 ADHD, Combined Type` and `F90 ADHD` correctly. This change was specifically implemented during the [[summaries/SESSION_SUMMARY]] session to address 24 reports returning zero diagnoses.

### Multi-Format Diagnosis Code Patterns
The production parser in `src/report_parser.py` (documented in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] and [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]) handles **six distinct code format patterns**, tried in order of specificity (most specific first to minimize false positives):

1. **Format 4 (Name-First)**: `ADHD, Combined Type 314.01 (F90.2)` — diagnosis name followed by codes
2. **Format 1 (Codes-First)**: `314.01 (F90.2) ADHD, Combined Presentation` — DSM and ICD-10 codes precede the name
3. **Format 2 (ICD-10 Only)**: `F90.2 ADHD` — only an ICD-10 code; broadest pattern tried last
4. Additional format variants combining DSM-IV numeric codes with or without ICD-10 parenthetical codes

This layered matching strategy reduces false positives while maximizing recall across the heterogeneous report corpus.

### Subsection Header False Positives
Lines like `Axis I Diagnostic Considerations` or `Axis I Rule Out:` are section labels, not diagnoses. Without filtering, they are incorrectly parsed as diagnosis name entries. A post-processing filter using a regex guard removes any parsed entry whose `name` field matches the pattern `^Axis\s+[IV]+\s+(?:Diagnostic\s+)?(?:Considerations?|Rule\s+Out|R/?O)\s*:?$`. This subsection header filtering eliminates a common false-positive class in multiaxial-format reports. The [[summaries/SESSION_SUMMARY]] session explicitly lists this filter as one of three targeted improvements to `src/report_parser.py`.

### Non-Standard and Textbook Reports
Some documents in an ingestion folder may be textbook examples or sample reports (e.g., `donders__*`, `nelson_sample_*`, `neuropsychological_report_writing___*`) rather than real patient evaluations. These often lack standard diagnostic sections and will always return low extraction yields. They may warrant exclusion from the ingestion pipeline or special handling in [[concepts/report-parser-quality]] review. The [[summaries/SESSION_SUMMARY]] session explicitly flags these as candidates for removal from the PDF bucket.

## Age Group Classification and Override System

Age group classification is a metadata field derived during report parsing. The parser attempts to extract patient age from the PDF text and assign one of four groups: pediatric, adolescent, adult, or geriatric. When auto-extraction fails, reports default to "unknown."

Before the session documented in [[summaries/SESSION_SUMMARY]], 16.9% of reports (74 entries) had unknown age groups. An override system was implemented to make manual corrections permanent across re-ingestion:

- **Configuration file**: `data/age_overrides.json` maps report filenames to patient ages
- **Parser integration**: `src/report_parser.py` loads overrides at parse time, after attempting auto-extraction
- **Parse flow**: `Parse PDF → Try extract age → Check overrides → Classify age_group`

The result was a reduction from 16.9% → 1.6% unknown age (7 remaining out of 438 recommendations). See [[summaries/AGE_OVERRIDE_GUIDE]] and [[summaries/PERMANENT_SOLUTION_SUMMARY]] for full documentation. The override system is designed to be maintainable: new overrides are added by editing `age_overrides.json` with `{"overrides": {"report.pdf": 25}}`.

This correction is critical for downstream [[concepts/recommendation-rag-pipeline]] quality: age group is a primary metadata filter in retrieval, so unknown ages degrade the precision of age-targeted recommendation lookup. See also [[concepts/age-group-classification]] for the broader concept.

## Parser Architecture

A robust neuropsychological report parser (as implemented in `src/report_parser.py`) typically includes:

1. **PDF Text Extraction** — converting PDF pages to raw text using PyMuPDF (`fitz`) or similar libraries (see [[concepts/pdf-data-extraction]] and [[concepts/docling-pdf-parsing]])
2. **Text Cleaning** — normalizing whitespace, removing page artifacts, and stripping headers/footers before downstream processing
3. **Section Segmentation** — identifying report sections by header pattern matching using an expanding list of recognized section labels
4. **Field Extractors** — targeted regex or NLP routines for diagnoses, scores, and demographics
5. **Age Override Resolution** — post-extraction check against `data/age_overrides.json` to apply permanent corrections for known edge-case reports
6. **Multi-Format Diagnosis Matching** — layered pattern matching across all known code format variants (name-first, codes-first, ICD-10 only, etc.)
7. **Diagnosis Canonicalization and Deduplication** — normalizing variant names to canonical forms (e.g., all ADHD subtypes) via `canonicalize_diagnosis_name()` and merging equivalent diagnoses via `_merge_equivalent_diagnoses()`; see [[concepts/dsm5-diagnosis-normalization]]
8. **DSM-5 Category Assignment** — classifying each diagnosis into one of 12 DSM-5 categories using `classify_dsm5_category()` based on ICD-10 code and name patterns
9. **Post-Processing Filters** — removing false positives such as subsection headers (e.g., Axis I/II labels) and non-actionable content from recommendation sections
10. **Override Systems** — manual correction mechanisms for fields that cannot be auto-extracted (see [[summaries/AGE_OVERRIDE_GUIDE]])

This architecture is part of the broader [[concepts/neuropsychological-assessment-pipeline]]. All changes to the header patterns, ICD-10 regex, and subsection filters are designed to be **backward-compatible**: existing patterns continue to work and new patterns are purely additive.

## Recommendation Section Parsing

Beyond diagnosis extraction, the pipeline (documented in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]) also parses the **Recommendations** section of reports into structured chunks for downstream retrieval. This is implemented in `extract_recommendations_section()` and `split_recommendations_by_subsection()` within `src/report_parser.py`.

Key steps:

- **Boundary detection**: Locate the `RECOMMENDATIONS` header (including variants like `Summary and Recommendations`) and identify the end boundary via signature blocks, appendices, or score tables
- **Content filtering**: Remove non-actionable clinical impressions and administrative content using `_filter_non_recommendation_content()` — see [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] for the full rationale and implementation detail
- **Subsection splitting**: Split by subsection headers detected via heuristics — PHASE X patterns, ALL CAPS lines, and Title Case lines
- **Chunk construction**: Each `RecommendationChunk` carries text, sub-header, and metadata (diagnoses, age group, context)

This recommendation chunking feeds directly into the [[concepts/recommendation-rag-pipeline]] and the [[concepts/retrieval-augmented-generation]] vector store.

### Recommendation Content Filtering

A recurring quality problem is the extraction of non-actionable content alongside genuine recommendations. Three categories of unwanted content commonly appear in raw recommendation text:

1. **Clinical Assessment/Impressions** — interpretive paragraphs describing patient characteristics, motivation, and prognosis (e.g., "Ms. [PATIENT] appears to have substantial interest in making changes...").
2. **Administrative Notes** — billing hours, contact information (e.g., "A total of 8 hours was spent by me personally...").
3. **Signature Blocks** — phone numbers and contact details appearing after the recommendations section ends.

The `_filter_non_recommendation_content()` function in `src/report_parser.py` addresses this by:

- **Paragraph-level filtering** using `assessment_markers` — a list of phrase anchors that identify clinical impression language ("appears to have", "is probably pessimistic", "with respect to past", "may have initial difficulty", etc.)
- **Enhanced section-end detection** using additional regex patterns:
  - Phone numbers: `Please call (###) ###-####`
  - Billing disclosures: `A total of N hours was spent`
  - Personal attestation: `I personally performed`

**What is filtered out:**
- ✗ Clinical impressions about patient motivation and prognosis
- ✗ Assessment of treatment readiness
- ✗ Administrative billing disclosures
- ✗ Contact/phone information

**What is preserved:**
- ✓ Accommodations, referrals, and therapeutic strategies
- ✓ Educational recommendations and specific interventions
- ✓ Treatment suggestions

Filtering is applied at the start of `split_recommendations_by_subsection()`, making it transparent to downstream consumers. The change is backward-compatible — it is purely subtractive of unwanted content, with no structural changes to the output data format.

## Diagnosis Extraction Specifically

Diagnosis extraction is one of the highest-value targets in report parsing because diagnoses drive downstream retrieval and recommendation logic in a [[concepts/retrieval-augmented-generation]] knowledge base.

Key implementation details (all in `src/report_parser.py`):
- Section headers are matched case-insensitively with multiple pattern variants
- ICD-10 codes (with optional decimal) serve as reliable anchors: lines containing a code pattern are candidate diagnoses
- Six distinct format patterns are tried in order of specificity (most specific first)
- Subsection headers matching axis/rule-out patterns are removed before yielding results
- [[concepts/dsm5-diagnosis-normalization]] is applied to standardize extracted codes and collapse variant names
- Diagnoses are classified into 12 DSM-5 categories for metadata-filtered retrieval
- Legacy ICD-9 codes (e.g., `799.9 Diagnosis Deferred`) may appear in older multiaxial reports and should be preserved

## PHI De-identification

The production pipeline integrates PHI removal as a mandatory step between parsing and vector store population, implemented in `src/report_deidentify.py`. See [[concepts/phi-deidentification-pipeline]] for full detail. The pipeline consists of:

1. **Page artifact stripping**: Remove CONFIDENTIAL headers, page numbers, and standalone name lines via `strip_page_artifacts()`
2. **Name inference fallback chain**: `_infer_patient_name_from_text()` searches structured patterns; `_infer_patient_name_from_source_file()` falls back to filename parsing
3. **Replacement pattern construction**: `build_report_replacements()` calls `_name_forms()` to generate name variants, uses `_short_hash()` to create **deterministic 6-character hash placeholders** (ensuring the same patient maps to the same token across multiple reports)
4. **Patient-specific regex substitution**: Ordered patterns applied via `pattern.sub()`
5. **General PHI safety nets**: Patterns for SSN, phone, email, dates, and MRNs

The deterministic hash design means that de-identified chunks from the same patient are consistently anonymized, supporting longitudinal analysis without re-identification.

## Data Cleaning and Quality Metrics

The [[summaries/SESSION_SUMMARY]] session introduced structured data cleaning as a formal step applied to `data/recommendations_kb.json` (438 recommendations from 104 reports). Key activities:

- Normalized whitespace and text formatting across all recommendation text fields
- Added computed metadata fields: `text_length`, `num_diagnoses`, `num_categories`
- Added quality flags: `missing_diagnoses`, `age_group_unknown`
- Created searchable text fields for diagnoses and categories
- Produced `data/recommendations_kb_cleaned.json` and `data/cleaning_summary.txt`

### Pre-Cleaning Statistics
| Metric | Value |
|---|---|
| Total recommendations | 438 |
| Reports | 104 |
| With diagnoses | 77.9% |
| General (no diagnoses) | 22.1% |
| Pediatric age group | 44.3% |
| Adult age group | 28.3% |
| Unknown age group | 16.9% |
| Mean text length | 1,652 chars |
| Text length range | 103–14,305 chars |

### Post-Improvement Expected Statistics
| Metric | Before | After |
|---|---|---|
| Unknown age | 16.9% | 1.6% |
| Missing diagnoses | 22.1% | ~10% |
| Pediatric | 44.3% | 47.5% |
| Adult | 28.3% | 40.0% |
| Adolescent | — | 8.7% |
| Geriatric | — | 2.3% |

### Zero-Diagnosis Report Rate
The primary quality metric for diagnosis extraction is the **zero-diagnosis report rate** — the proportion of real patient reports for which the parser returns an empty diagnosis list. Before improvements: 23% (~24/104 reports). After: expected ~9–11% (~10–12 reports), with remaining cases being textbook examples or genuinely recommendations-only documents.

## Semantic Search and Metadata Filtering

Parsed and de-identified recommendation chunks are embedded via SentenceTransformer (see [[concepts/sentence-transformer-embeddings]]) and stored in a FAISS index (see [[concepts/faiss-vector-index]]). The search workflow implements:

- **Query encoding**: Same SentenceTransformer model used at ingest time ensures embedding space alignment
- **Over-fetching**: Retrieve k×3 candidates from FAISS to allow post-filtering without recall loss
- **Metadata filtering**: List-membership check for diagnoses; equality check for age group and context
- **Post-filtering**: Additional diagnosis list filtering at the app layer
- **Similarity metric**: Cosine similarity via inner product on L2-normalized vectors

The metadata attached to each chunk (diagnoses, age group, context, sub-header) comes directly from the parsing step, making parse quality — including recommendation content filtering — a critical upstream dependency for retrieval precision.

## Relationship to the Full Ingestion Pipeline

Parsing is embedded within a multi-stage ingestion pipeline orchestrated by `src/ingest_recommendations.py`. The complete flow is:

1. **PDF ingestion** → raw text via PyMuPDF (`fitz.open`)
2. **Report parsing** → metadata, diagnoses, recommendation chunks (`src/report_parser.py`)
3. **Age override resolution** → `data/age_overrides.json` applied for known edge cases
4. **Recommendation filtering** → `_filter_non_recommendation_content()` removes assessment and administrative content before chunking
5. **PHI de-identification** → patient-specific regex replacement with deterministic hash placeholders (`src/report_deidentify.py`); see [[concepts/phi-deidentification-pipeline]]
6. **Embedding generation** → SentenceTransformer encoding of de-identified chunks
7. **Vector store population** → FAISS index for semantic search; see [[concepts/faiss-vector-index]]
8. **LLM-powered generation** → retrieved chunks formatted and sent to LangChain chat model via [[concepts/llm-provider-abstraction]]

Extracted diagnoses and scores populate the knowledge base architecture used for retrieval. Poor extraction quality — whether from missed diagnoses, incorrect age groups, or contaminated recommendation text — directly degrades recommendation relevance.

## LLM-Powered Recommendation Generation

Once relevant chunks are retrieved, `src/llm.py` generates tailored recommendations using a LangChain-based pipeline. Providers (Anthropic, OpenAI, Ollama) are loaded via dynamic import (`importlib.import_module`) to avoid unnecessary dependencies — an instance of [[concepts/llm-provider-abstraction]]. Retrieved chunks are formatted with metadata (diagnoses, age, context, headers) into numbered examples, then combined with clinical guidelines in a `SystemMessage` and the user query in a `HumanMessage`. The LLM is invoked via `llm.invoke(messages)` and the response content is returned directly.

## Testing and Verification

Testing is performed by re-running `python -m src.ingest_recommendations` and inspecting `data/recommendations_kb.json`. Recommended verification checks (from [[summaries/SESSION_SUMMARY]]):

```python
# Verify age group fix
import json
with open('data/recommendations_kb.json') as f:
    data = json.load(f)
    unknown = [r for r in data['metadata']['reports']
               if r['age_group'] == 'unknown']
    print(f"Unknown age: {len(unknown)} (expect ~2)")

# Verify diagnosis fix
with open('data/recommendations_kb.json') as f:
    data = json.load(f)
    no_dx = [r for r in data['metadata']['reports']
             if len(r['diagnoses']) == 0]
    print(f"No diagnoses: {len(no_dx)} (expect ~10-12, was 24)")
```

For recommendation content quality, the key test is inspecting `data/recommendations_kb.json` post-ingestion to confirm that known problematic files (e.g., `amoozegar_report_081018.pdf`) no longer contain assessment language ("appears to have") or administrative disclosures ("8 hours was spent").

## Planned Enhancements

- **Diagnosis override system** — analogous to age overrides, allowing manual correction of edge cases
- **Rule-out flagging** — marking provisional/rule-out diagnoses distinctly from confirmed diagnoses
- **Multi-column layout detection** — better handling of non-linear diagnosis table formats
- **Confidence scores** — per-diagnosis and per-paragraph parsing confidence to surface uncertain extractions for review
- **Additional header patterns** — `Clinical\s+Impressions?`, `Diagnostic\s+Conclusions?`, `Assessment\s+and\s+Diagnos[ei]s?`
- **ML classifier for recommendation filtering** — train a model to distinguish assessment vs. recommendation language, replacing the current regex-based `assessment_markers` approach
- **Configurable filter patterns** — allow per-deployment customization of content filter lists
- **Manual review flagging** — surface borderline paragraphs for human review before ingestion
- **Remove textbook PDFs** — exclude non-patient documents (e.g., `donders__*`, `nelson_sample_*`) from the ingestion bucket to eliminate persistent zero-diagnosis false negatives

## Related Concepts

- [[concepts/icd10-diagnosis-extraction]]
- [[concepts/multiaxial-diagnosis-format]]
- [[concepts/pdf-data-extraction]]
- [[concepts/docling-pdf-parsing]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/dsm5-diagnosis-normalization]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/clinical-report-structure]]
- [[concepts/pdf-score-extraction]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/report-parser-quality]]
- [[concepts/recommendation-rag-pipeline]]
- [[concepts/phi-deidentification-pipeline]]
- [[concepts/faiss-vector-index]]
- [[concepts/rag-chunking]]
- [[concepts/llm-provider-abstraction]]
- [[concepts/sentence-transformer-embeddings]]
- [[concepts/text-chunking]]
- [[concepts/age-group-classification]]

See also: [[summaries/PERMANENT_SOLUTION_SUMMARY]], [[summaries/QUICK_REFERENCE]], [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]], [[summaries/SESSION_SUMMARY]], [[summaries/AGE_OVERRIDE_GUIDE]]