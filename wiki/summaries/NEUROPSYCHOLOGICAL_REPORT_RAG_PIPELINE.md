---
doc_type: short
full_text: sources/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md
---

# Neuropsychological Report Analysis: DSM-5 Diagnosis Parsing, PHI De-identification, and Recommendation RAG Pipeline

## Overview

This document describes a clinical NLP system that processes neuropsychological PDF reports through a multi-stage pipeline. The system extracts text, parses DSM-5 diagnoses with ICD-10 code mapping, removes Protected Health Information (PHI) via layered regex replacement, chunks recommendations by subsection, and builds a searchable [[concepts/faiss-vector-index]] for [[concepts/recommendation-rag-pipeline]]-based recommendation generation.

## Pipeline Architecture

The full pipeline is implemented across several Python modules under `src/` and an app entry point (`app_recommendations.py`). Six major traces are documented:

### Trace 1: Full Report Ingestion (PDF → Vector Store)

- Entry point: `src/ingest_recommendations.py`
- For each PDF: extract text with **PyMuPDF** (`fitz.open`), parse report structure, de-identify chunks, generate embeddings, and persist to a **FAISS** vector index.
- Key steps: `parse_pdf()` → `parse_report()` → `deidentify_recommendations()` → `generate_embeddings()` → `VectorStore.add_embeddings()` → `vs.save()`
- See [[concepts/neuropsychological-assessment-pipeline]] for related ingestion workflows.

### Trace 2: DSM-5 Diagnosis Extraction

- Implemented in `src/report_parser.py`
- Handles **6 different code format patterns** including:
  - Format 4 (name-first): `ADHD 314.01 (F90.2)`
  - Format 1 (codes-first): `314.01 (F90.2) ADHD`
  - Format 2 (ICD-10 only): `F90.2 ADHD`
- Post-processing: `canonicalize_diagnosis_name()` normalizes variants; `classify_dsm5_category()` assigns diagnoses to one of 12 DSM-5 categories; `_merge_equivalent_diagnoses()` deduplicates.
- See [[concepts/icd10-diagnosis-extraction]] and [[concepts/dsm5-diagnosis-normalization]] for cross-document context.

### Trace 3: PHI De-identification

- Implemented in `src/report_deidentify.py`
- Stages:
  1. `strip_page_artifacts()` — removes CONFIDENTIAL headers, page numbers, standalone name lines.
  2. Name inference fallback chain: `_infer_patient_name_from_text()` and `_infer_patient_name_from_source_file()`.
  3. `build_report_replacements()` — generates name variants via `_name_forms()`, creates date and case-number patterns.
  4. Patient-specific regex substitution with **deterministic 6-char hash** placeholders (`_short_hash()`).
  5. General PHI safety nets: SSN, phone, email, dates, MRNs via `GENERAL_PHI_PATTERNS`.
- See [[concepts/phi-deidentification-pipeline]], [[concepts/pii-redaction-pipelines]], and [[concepts/redaction-tokens]] for related techniques.

### Trace 4: Recommendation Section Parsing

- Detects `RECOMMENDATIONS` section boundary (including variants like "Summary and Recommendations") and end boundary (signature blocks, appendices, score tables).
- Filters non-actionable content via `_filter_non_recommendation_content()`.
- Detects subsection headers using heuristics: PHASE X pattern, ALL CAPS, Title Case.
- Builds `RecommendationChunk` objects with text, sub-header, diagnoses, age, and context metadata.
- See [[concepts/neuropsych-report-parsing]] and [[concepts/text-chunking]] for related approaches.

### Trace 5: Semantic Search with Metadata Filtering

- Query encoded with **SentenceTransformer** (`embeddings.py`); see [[concepts/sentence-transformer-embeddings]].
- `VectorStore.search_filtered()` over-fetches candidates (k×3) from FAISS, then applies metadata filters (list membership for diagnoses, equality for age group/context).
- Post-filters by diagnosis list in app layer.
- Similarity metric: cosine similarity via inner product on normalized vectors.
- See [[concepts/faiss-vector-index]], [[concepts/vector-search]], and [[concepts/retrieval-augmented-generation]].

### Trace 6: LLM-Powered Recommendation Generation

- Implemented in `src/llm.py`
- Supports multiple providers (Anthropic, OpenAI, Ollama) via **dynamic module import** (`importlib.import_module`); see [[concepts/llm-provider-abstraction]].
- Formats retrieved chunks with metadata into numbered examples.
- Constructs LangChain `SystemMessage` (guidelines + patient context) and `HumanMessage` (query + chunks + instructions).
- Invokes LLM via `llm.invoke(messages)` and returns generated recommendation text.
- See [[concepts/recommendation-rag-pipeline]] and [[concepts/clinical-narrative-generation]].

## Key Components

| Component | File | Purpose |
|---|---|---|
| Ingestion orchestrator | `src/ingest_recommendations.py` | Full pipeline from PDF to vector store |
| Report parser | `src/report_parser.py` | DSM-5 diagnosis extraction, recommendation chunking |
| De-identifier | `src/report_deidentify.py` | PHI removal with regex + hashing |
| Embeddings | `src/embeddings.py` | SentenceTransformer encoding |
| Retrieval | `src/retrieval.py` | FAISS search with metadata filtering |
| LLM generation | `src/llm.py` | LangChain-based recommendation generation |
| App UI | `app_recommendations.py` | Shiny app with search and generation UI |

## Key Entry Points

- **Ingestion**: `src/ingest_recommendations.py:109`
- **Diagnosis parsing**: `src/report_parser.py:945–1011`
- **PHI removal**: `src/report_deidentify.py:254–258`
- **Semantic search**: `app_recommendations.py:214` → `src/retrieval.py:106`

## Design Decisions

- **Broadest regex pattern tried last** for diagnosis format matching to avoid false positives; see [[concepts/fallback-strategy]].
- **Over-fetching** (k×3) in FAISS search to allow post-filter without losing recall.
- **Deterministic hashing** for PHI replacement ensures consistent anonymization across reports for the same patient; see [[concepts/phi-data-handling]].
- **Lazy LLM provider import** avoids dependency loading for unused providers; see [[concepts/llm-provider-abstraction]].
- [[concepts/rag-chunking]] governs how recommendation subsections are split and stored with metadata.

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/local-first-architecture]]
- [[concepts/clinical-data-privacy]]
- [[concepts/hybrid-search-retrieval]]
