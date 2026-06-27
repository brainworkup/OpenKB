---
doc_type: short
full_text: sources/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md
---

# Neuropsychological Report Analysis: DSM-5 Diagnosis Parsing, PHI De-identification, and Recommendation RAG Pipeline

## Overview

This document describes a clinical NLP pipeline that ingests neuropsychological PDF reports and transforms them into a searchable, de-identified vector store for [[concepts/retrieval-augmented-generation]] (RAG)-based recommendation generation. The system spans six major processing traces, from raw PDF ingestion through LLM-powered output.

## Pipeline Architecture

The pipeline consists of six interconnected traces:

### Trace 1: Full Report Ingestion
- Orchestrated by `src/ingest_recommendations.py`
- Flow: PDF → PyMuPDF text extraction → report structure parsing → [[concepts/phi-deidentification-pipeline]] → embedding generation → [[concepts/faiss-vector-index]]
- Key libraries: PyMuPDF (`fitz`), SentenceTransformer, FAISS
- Output: Persisted vector index with de-identified recommendation chunks and metadata

### Trace 2: DSM-5 Diagnosis Extraction
- Implemented in `src/report_parser.py`
- Handles **6 code format patterns** (name-first, codes-first, ICD-10 only, etc.)
- Normalizes diagnosis variants to canonical names (e.g., all ADHD subtypes → canonical form) via [[concepts/dsm5-diagnosis-normalization]]
- Assigns diagnoses to one of **12 DSM-5 categories** using [[concepts/icd10-diagnosis-extraction]]
- Deduplicates and merges equivalent diagnoses via `_merge_equivalent_diagnoses()`

### Trace 3: PHI De-identification
- Implemented in `src/report_deidentify.py`
- Multi-stage process:
  1. Strip page artifacts (CONFIDENTIAL headers, page numbers, standalone name lines)
  2. Infer patient name via fallback chain (metadata → text patterns → source filename)
  3. Build patient-specific regex replacement patterns (name variants, dates, case numbers)
  4. Apply patient-specific replacements using deterministic 6-char hash placeholders via [[concepts/redaction-tokens]]
  5. Apply general PHI safety nets (SSN, phone, email, MRN)
- Deterministic hashing ensures consistent anonymization across multiple reports for the same patient
- Related: [[concepts/phi-data-handling]], [[concepts/pii-redaction-pipelines]]

### Trace 4: Recommendation Section Parsing
- Identifies section boundaries using header/footer heuristics
- Filters non-actionable clinical impressions from recommendation text
- Splits content into `RecommendationChunk` objects by subsection header (Title Case, ALL CAPS, PHASE X patterns)
- Each chunk carries metadata: diagnoses, age group, context, sub-header
- Related: [[concepts/rag-chunking]], [[concepts/text-chunking]]

### Trace 5: Semantic Search with Metadata Filtering
- Implemented in `app_recommendations.py` + `src/retrieval.py`
- Flow: User query → SentenceTransformer embedding → `VectorStore.search_filtered()` → post-filter by diagnosis
- Over-fetches candidates (k×3) from FAISS, then applies metadata filters ([[concepts/age-group-classification]], context, diagnoses list membership)
- Uses cosine similarity via inner product on normalized vectors
- Related: [[concepts/retrieval-augmented-generation]], [[concepts/vector-search]], [[concepts/recommendation-rag-pipeline]]

### Trace 6: LLM-Powered Recommendation Generation
- Implemented in `src/llm.py`
- Supports multiple providers (Anthropic, OpenAI, Ollama) via dynamic `importlib` loading using [[concepts/llm-provider-abstraction]]
- Formats retrieved chunks with metadata as few-shot examples
- Constructs LangChain `SystemMessage` (guidelines + patient context) and `HumanMessage` (query + examples + instructions)
- Invokes LLM to produce tailored clinical recommendations
- Related: [[concepts/neuropsychological-prompt-engineering]], [[concepts/clinical-narrative-generation]]

## Key Components

| File | Role |
|---|---|
| `src/ingest_recommendations.py` | Pipeline orchestrator |
| `src/report_parser.py` | Diagnosis extraction and recommendation chunking |
| `src/report_deidentify.py` | PHI removal and de-identification |
| `src/retrieval.py` | FAISS vector store and filtered semantic search |
| `src/llm.py` | LLM integration and prompt construction |
| `app_recommendations.py` | Streamlit UI for search and generation |

## Key Features

- **Multi-format DSM-5 parsing** with ICD-10 code mapping across 6 format variants — see [[concepts/dsm5-diagnosis-normalization]] and [[concepts/icd10-diagnosis-extraction]]
- **Deterministic PHI de-identification** with patient-specific regex patterns and hash-based placeholders — see [[concepts/phi-deidentification-pipeline]]
- **Metadata-filtered semantic search** supporting [[concepts/age-group-classification]], context, and diagnosis filters
- **Subsection-aware chunking** preserving [[concepts/clinical-report-structure]] of recommendation sections
- **Provider-agnostic LLM generation** supporting multiple backends via [[concepts/llm-provider-abstraction]]

## Entry Points

- **Ingestion**: `python src/ingest_recommendations.py --reports-dir data/reports --output data/recommendations_index`
- **UI**: `streamlit run app_recommendations.py`
- **Evaluation**: `python eval/run_eval.py`

## Key Functions

- `process_single_report()` — main pipeline entry; see [[concepts/neuropsychological-assessment-pipeline]]
- `extract_diagnoses()` — DSM-5 parsing via [[concepts/dsm5-diagnosis-normalization]]
- `deidentify_recommendations()` — PHI removal via [[concepts/pii-redaction-pipelines]]
- `VectorStore.search_filtered()` — filtered semantic search via [[concepts/faiss-vector-index]]
- `generate_recommendations()` — LLM-powered generation via [[concepts/recommendation-rag-pipeline]]

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/neuropsych-report-parsing]]
- [[concepts/clinical-data-privacy]]
- [[concepts/local-first-architecture]]
- [[concepts/narrative-report-generation]]
