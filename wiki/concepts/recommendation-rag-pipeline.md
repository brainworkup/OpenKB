---
sources: [summaries/bwu.neuro.reports.recs.autism.adults.md, summaries/bwu.neuro.reports.recs.adhd.merge.md, summaries/autism_recommendations_for_adults_summary.md, summaries/README.md, summaries/SESSION_SUMMARY.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md]
brief: End-to-end pipeline ingesting neuropsychological reports to generate evidence-based clinical recommendations via RAG.
---

# Recommendation RAG Pipeline for Clinical Reports

A **Recommendation RAG Pipeline** for clinical reports is an end-to-end system that ingests neuropsychological assessment documents, de-identifies protected health information, filters non-actionable content, chunks recommendation sections by clinical subsection, builds a searchable vector store, and uses a large language model to generate tailored clinical recommendations from semantically retrieved examples.

The primary reference implementations are documented in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] and [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]. Content filtering improvements are documented in [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]].

## Shared Recommendations Repository

A central `./recommendations` directory serves as a shared knowledge base of **evidence-based intervention materials** available across all patient RAG systems. Files placed in this directory (PDF, DOCX, or plain text) are automatically ingested whenever any patient's RAG system is rebuilt, ensuring a consistent, up-to-date set of reference materials informs recommendation generation for every patient. This design supports the [[concepts/knowledge-base-architecture]] principle of a single source of truth for intervention resources, reducing duplication and promoting consistency across patient workflows. See also [[summaries/README]] for the directory specification.

## Pipeline Architecture Overview

The system is orchestrated by `src/ingest_recommendations.py` and spans six major stages, each implemented in dedicated modules:

| Module | Role |
|---|---|
| `src/ingest_recommendations.py` | Top-level pipeline orchestrator |
| `src/pdf_parse.py` | PyMuPDF-based text extraction |
| `src/report_parser.py` | DSM-5 diagnosis and recommendation parsing |
| `src/report_deidentify.py` | PHI removal with regex and hashing |
| `src/embeddings.py` | SentenceTransformer encoding |
| `src/retrieval.py` | FAISS search with metadata filtering |
| `src/llm.py` | LangChain-based recommendation generation |
| `app_recommendations.py` | Shiny search and generation UI |

## Pipeline Stages

### 1. PDF Ingestion and Text Extraction

Raw PDF neuropsychological reports are loaded using PyMuPDF (`fitz.open()`) and converted to plain text. The orchestrator loops over a directory of reports and passes each through `process_single_report()`, which coordinates parsing, de-identification, embedding, and indexing.

**Key entry point**: `src/ingest_recommendations.py:109`

This connects directly to broader [[concepts/clinical-nlp-pipelines]] and [[concepts/pdf-data-extraction]] practices.

### 2. Diagnosis Parsing and Metadata Extraction

The system extracts DSM-5 diagnoses with ICD-10 code mapping from the report text using multiple distinct regex format patterns:

- **Format 4** (name-first): `ADHD, Combined Type 314.01 (F90.2)` — tried first, most specific
- **Format 1** (codes-first): `314.01 (F90.2) ADHD, Combined Presentation`
- **Format 2** (ICD-10 only): `F90.2 ADHD` — broadest pattern, tried last to avoid false positives

The diagnosis section is located via multiple header pattern variants before matching begins. Post-processing steps:
1. `canonicalize_diagnosis_name()` normalizes variants (e.g., all ADHD subtypes to canonical form)
2. `classify_dsm5_category()` assigns diagnoses to one of 12 DSM-5 categories using ICD-10 code and name patterns
3. `_merge_equivalent_diagnoses()` deduplicates and preserves canonical names with all aliases

**Key entry points**: `src/report_parser.py:945–1011` for format matching; `src/report_parser.py:149` for extraction entry

See [[concepts/dsm5-diagnosis-normalization]] and [[concepts/icd10-diagnosis-extraction]] for related concepts.

### 3. PHI De-identification

Before any text is stored or embedded, all protected health information is removed through a layered multi-stage process implemented in `src/report_deidentify.py`:

1. **Strip page artifacts** (`strip_page_artifacts()`): Remove CONFIDENTIAL headers, page numbers, and standalone name lines
2. **Name inference fallback chain**:
   - `_infer_patient_name_from_text()`: Searches `Name: First Last` patterns, CONFIDENTIAL footers, and standalone name lines
   - `_infer_patient_name_from_source_file()`: Falls back to filename-based inference
3. **Build replacement patterns** (`build_report_replacements()`): Generates name variants via `_name_forms()`, constructs date and case-number regex patterns, ordered longest-first to prevent partial matches
4. **Patient-specific replacement**: Regex substitution replaces names and variants with deterministic 6-character hash placeholders (`_short_hash()`), ensuring consistent anonymization of the same patient across multiple reports
5. **General PHI safety nets**: `GENERAL_PHI_PATTERNS` catches SSNs, phone numbers, and emails; `REPORT_GENERAL_PHI_PATTERNS` catches remaining dates and MRNs

**Key entry points**: `src/report_deidentify.py:254–258`

This approach aligns with [[concepts/phi-deidentification-pipeline]], [[concepts/phi-data-handling]], [[concepts/pii-redaction-pipelines]], and [[concepts/redaction-tokens]].

### 4. Recommendation Section Chunking and Content Filtering

The recommendations section is isolated by detecting its header boundary (including variants like "Summary and Recommendations") and end boundary (signature blocks, appendices, score tables). A key improvement added enhanced stop-pattern detection and a dedicated content filtering stage.

#### 4a. Section End Detection

Additional regex patterns now terminate extraction when administrative or signature content is encountered.

See also: [[summaries/SESSION_SUMMARY]]

#### 4b. Content Quality Filtering

After section extraction, individual recommendation chunks are evaluated for actionability. Non-clinical content such as boilerplate legal disclaimers, page headers, and score tables is discarded before embedding. Only chunks that represent genuine, evidence-based clinical guidance are retained for indexing. This filtering stage ensures that retrieved recommendations are relevant and useful for downstream generation.

### 5. Embedding and Vector Indexing

Filtered recommendation chunks are encoded using SentenceTransformer models into dense vector representations. These embeddings are indexed with [[concepts/faiss-vector-index]] for efficient approximate nearest-neighbor retrieval. Metadata (diagnosis category, ICD-10 codes, source report) is stored alongside each vector to enable filtered retrieval at query time.

### 6. Retrieval and LLM-Based Generation

At query time, a clinician's request or patient profile is encoded and used to retrieve semantically similar recommendation chunks from the FAISS index. Retrieved chunks, filtered by diagnosis metadata, are assembled into a context window and passed to a large language model (via LangChain) to synthesize a tailored, evidence-based recommendation narrative.

This final stage connects to [[concepts/retrieval-augmented-generation]], [[concepts/hybrid-search-retrieval]], and [[concepts/narrative-report-generation]].

## Related Pages

- [[concepts/clinical-guidelines]] — The evidence base informing intervention content
- [[concepts/per-patient-workspace]] — Per-patient RAG system architecture
- [[concepts/knowledge-base-architecture]] — Shared knowledge store design
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] — Module-level code map
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]] — Pipeline narrative documentation
- [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] — Content filtering changelog
- [[summaries/README]] — Shared recommendations directory specification

See also: [[summaries/autism_recommendations_for_adults_summary]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.merge]]

See also: [[summaries/bwu.neuro.reports.recs.autism.adults]]