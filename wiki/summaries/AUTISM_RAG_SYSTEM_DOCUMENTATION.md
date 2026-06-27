---
doc_type: short
full_text: sources/AUTISM_RAG_SYSTEM_DOCUMENTATION.md
---

# Autism RAG System Documentation

## Overview

The Autism RAG System is a dual-pipeline [[concepts/retrieval-augmented-generation]] (RAG) application designed for two related but distinct use cases: (1) research document Q&A powered by FLAN-T5, and (2) clinical report parsing with LLM-powered recommendation generation. Both pipelines share a common [[concepts/faiss-vector-index]] retrieval infrastructure built on FAISS.

## Pipeline 1: Research Document Q&A

### Ingestion
- **Entry point:** `ingest_documents()` in `src/ingest.py:22`
- Processes PDF and EPUB files using [[concepts/pdf-data-extraction]] via PyMuPDF (fitz)
- Chunks text into **800-word segments with 120-word overlap** — see [[concepts/text-chunking]]
- Generates **384-dimensional embeddings** using `all-MiniLM-L6-v2` (SentenceTransformers)
- Stores vectors in a **FAISS IndexFlatIP** for inner-product similarity search

### Query & Answer Generation
- **Entry point:** `query_rag_system()` in `src/query.py:9`
- Retrieval phase: encodes query, searches FAISS index, returns `{chunk, score, metadata}` tuples — see [[concepts/vector-search]]
- Generation phase: uses **FLAN-T5** (`google/flan-t5-base`) via a text2text-generation pipeline
- Citation phase: extracts source document names and similarity scores

### Filtered Search
- `search_filtered()` in `src/retrieval.py:89`
- Uses an **over-fetch strategy** (k × 3) to compensate for post-filter losses — related to [[concepts/hybrid-search-retrieval]]
- Supports list membership filters (e.g., diagnoses) and equality filters (e.g., [[concepts/age-group-classification]])

## Pipeline 2: Clinical Recommendation System

### Clinical Report Ingestion & De-identification
- **Entry point:** `ingest_recommendations()` in `src/ingest_recommendations.py:82`
- Parses neuropsychological PDF reports, extracts metadata and [[concepts/dsm5-diagnosis-normalization]] codes
- **PHI de-identification:** redacts names, dates, and identifiers before storage — see [[concepts/phi-deidentification-pipeline]]
- Stores rich metadata (diagnoses, age groups) alongside embeddings in FAISS

### LLM-Powered Recommendation Generation
- **Entry point:** `generate_recommendations()` in `src/llm.py:124`
- Supports multiple providers via [[concepts/llm-provider-abstraction]]: Anthropic Claude, OpenAI GPT, Ollama (local)
- Uses [[concepts/neuropsychological-prompt-engineering]] combining clinical guidelines, patient context, and prior evaluation examples via LangChain
- Constructs `SystemMessage` (guidelines + context) and `HumanMessage` (query + examples)

## User Interfaces

### Shiny Web UI (`app.py`)
- Reactive search interface for research Q&A
- Displays answer card and citation cards with similarity scores
- Supports Markdown export of results

### FastAPI REST API (`api/server.py`)
- `POST /query` — Research Q&A with citations, returns JSON
- `GET /ingest` — Triggers document re-indexing

## Evaluation System

- **Entry point:** `run_evaluation()` in `eval/run_eval.py:34`
- [[concepts/yaml-configuration]]-based test suite with expected answer keywords
- Heuristic keyword-matching scoring, averaged across test questions
- Outputs results to JSON

## Technology Stack

| Component | Technology |
|-----------|------------|
| Vector Database | FAISS (IndexFlatIP) |
| Embeddings | SentenceTransformers (all-MiniLM-L6-v2) |
| Text Generation | FLAN-T5 (google/flan-t5-base) |
| LLM Integration | LangChain + Claude/GPT/Ollama |
| Web Framework | Shiny for Python |
| API Framework | FastAPI |
| Document Parsing | PyMuPDF (fitz) |
| Evaluation | YAML test suite + keyword matching |

## Data Flow Summary

- **Research Pipeline:** Documents → Chunks → Embeddings → FAISS → Query → Retrieved Chunks → FLAN-T5 → Answer + Citations
- **Clinical Pipeline:** Reports → Parsed & De-identified Chunks → Embeddings → FAISS → Patient Query → Retrieved Examples → LLM → Tailored Recommendations

## Key Design Decisions

- Shared retrieval infrastructure across both pipelines promotes code reuse
- [[concepts/phi-data-handling]] is embedded in the ingestion step, not a separate post-process
- Over-fetch strategy for filtered search avoids under-retrieval after metadata filtering
- Multiple LLM provider support via lazy imports allows flexibility between cloud and [[concepts/local-llm-inference]]

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/rag-chunking]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/clinical-report-structure]]
- [[concepts/knowledge-base-architecture]]
- [[concepts/local-first-architecture]]
- [[concepts/narrative-report-generation]]
