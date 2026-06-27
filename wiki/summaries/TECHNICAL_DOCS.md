---
doc_type: short
full_text: sources/TECHNICAL_DOCS.md
---

# PAI RAG System — Technical Documentation

## Overview

This document describes the architecture and implementation of a production-ready **Retrieval-Augmented Generation (RAG)** system tailored for clinical neuropsychology, specifically the **Personality Assessment Inventory (PAI)**. The system retrieves relevant clinical knowledge chunks and synthesizes answers using local or cloud LLMs.

See also: [[concepts/retrieval-augmented-generation]], [[concepts/vector-search]], [[concepts/hybrid-search-retrieval]], [[concepts/pai-assessment]], [[concepts/pai-knowledge-base]]

---

## System Architecture

The pipeline operates in three layers:

1. **Layer 1 — Retrieval**: Dual-path retrieval combining semantic (cosine similarity on 768-D vectors) and keyword (SQL `LIKE`) search, merged via **hybrid scoring**. See [[concepts/hybrid-search-retrieval]].
2. **Layer 2 — Context Formatting**: Top-K chunks retrieved, labeled with source and relevance scores, then formatted for prompt injection.
3. **Layer 3 — LLM Integration**: Abstracted provider routing to Ollama (local), OpenAI, or Anthropic Claude. Structured prompts (system + user roles) are used for all calls. See [[concepts/llm-provider-abstraction]].

---

## Knowledge Base

- **Source**: 81 PAI PDFs, producing 2,546 text chunks (~784 chars average)
- **Total text**: ~2.0 million characters
- **Storage**: DuckDB (in-process SQL) + Parquet files
  - `pai_knowledge_base_chunks.parquet` — 0.62 MB
  - `pai_embeddings_complete.parquet` — 6.25 MB
  - DuckDB database — 0.01 MB
  - **Total**: ~6.9 MB

See [[concepts/duckdb-as-vector-store]] and [[concepts/parquet-as-knowledge-store]] for storage pattern details.

### Database Tables

| Table | Key Fields |
|---|---|
| `pai_chunks` | doc_id, chunk_id, origin, text |
| `pai_embeddings` | doc_id, chunk_id, hash, origin, text_preview, embedding_json |

Indexes are maintained on both `doc_id` and `chunk_id` for both tables.

---

## Embedding Model

- **Model**: `nomic-embed-text` via [[concepts/local-llm-inference]]
- **Dimensions**: 768
- **Max context**: 8,192 tokens
- **Key advantage**: Fully local — no API costs, no data leaving the machine

### Similarity Formula (Cosine)
```
similarity = (query_vector · doc_vector) / (||query_vector|| × ||doc_vector||)
```
Range: [-1, 1]

### Hybrid Scoring
```
hybrid_score = (semantic_score_norm × semantic_weight) + (text_score_norm × text_weight)
```
Both scores normalized to [0, 1] before combination.

---

## Performance

| Operation | Latency |
|---|---|
| Semantic search | 500–800 ms |
| Keyword search | 50–100 ms |
| Hybrid search | 600–900 ms |
| LLM call (Ollama) | 2–5 seconds |
| LLM call (OpenAI) | 1–3 seconds |
| End-to-end | 3–8 seconds |

**Retrieval accuracy** (clinical testing):
- Top-1 relevance: 70–75%
- Top-3 relevance: 85–92%
- Top-5 relevance: 92–97%

**Memory usage**: Baseline ~50 MB; peak <100 MB

---

## Security & Privacy

- **Ollama (local)**: HIPAA-compatible — no data leaves the machine, no usage tracking. See [[concepts/privacy-first-software]] and [[concepts/phi-data-handling]].
- **Cloud APIs (OpenAI/Anthropic)**: Data sent to third-party servers; subject to institutional data policies. See [[concepts/clinical-data-privacy]].
- **Recommendation**: Use local inference via [[concepts/local-llm-inference]] for sensitive clinical data.

---

## Key Design Patterns (R Implementation)

### Retrieval
- `pai_semantic_search()` — pure vector search
- `pai_hybrid_search()` — combined semantic + keyword

### LLM Pipeline
- `format_context_for_llm()` → `build_pai_prompt()` → `call_llm()` → `ask_pai_expert()`

The provider abstraction layer (see [[concepts/llm-provider-abstraction]]) routes to Ollama, OpenAI, or Anthropic depending on configuration.

### Error Handling
- `tryCatch` wrapping all API calls
- Returns `NULL` on failure; logs error messages
- Graceful degradation without crashing. See [[concepts/fallback-strategy]].

---

## Optimization Opportunities

1. **Query embedding cache** — avoid re-embedding repeated queries
2. **Approximate Nearest Neighbors (ANN)** — FAISS or Annoy for 10,000+ chunks
3. **Parallel embedding** — via `furrr` / `future_map`
4. **Inverted index** — precomputed keyword matches replacing SQL `LIKE`

---

## Scalability

| Scale | Chunks | Approach |
|---|---|---|
| Current | 2,546 | DuckDB + Parquet |
| 10× | ~25,000 | No changes needed |
| 100× | ~250,000 | Add FAISS ANN |
| 1,000×+ | 2.5M+ | Pinecone / Weaviate, sharding |

---

## Testing & Validation

- **Unit tests** (testthat): semantic search returns valid results, hybrid scoring within [0,1]
- **Integration tests**: end-to-end pipeline returns `answer` and `sources`
- **Clinical validation**: 5 real-world scenarios including differential diagnosis (DEP vs ANX), validity assessment (ICN/INF cutoffs), treatment prognosis, scale interpretation (BOR), and complex comorbidity cases

---

## Extensibility

- Add new assessments (MMPI, WAIS, WISC) by processing PDFs into additional DuckDB tables (see [[concepts/duckdb-as-vector-store]])
- Multi-assessment search via unified `multi_assessment_search()` function
- Custom LLM providers via `call_custom_llm()` with arbitrary REST endpoints, compatible with [[concepts/openai-compatible-api]] patterns

---

## Research Applications

- **Literature synthesis**: retrieve top-50 chunks on a construct, export CSV for qualitative review
- **Meta-analysis support**: extract psychometric properties and statistics from retrieved passages

This system supports the broader [[concepts/neuropsychological-assessment-pipeline]] by providing evidence-based retrieval over [[concepts/clinical-pdf-assessment]] literature.

---

## Planned Enhancements

- Multi-modal support (images, tables)
- Temporal filtering by publication year
- Citation network analysis
- Automated report generation
- Shiny web application / mobile app
- Fine-tuned embedding model on psychology literature
- EMR integration and multi-language support

---

## Related Concepts
- [[concepts/sqlite-as-vector-store]]

- [[concepts/retrieval-augmented-generation]]
- [[concepts/vector-search]]
- [[concepts/hybrid-search-retrieval]]
- [[concepts/local-llm-inference]]
- [[concepts/llm-provider-abstraction]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/parquet-as-knowledge-store]]
- [[concepts/pai-assessment]]
- [[concepts/pai-knowledge-base]]
- [[concepts/phi-data-handling]]
- [[concepts/clinical-data-privacy]]
- [[concepts/fallback-strategy]]
- [[concepts/neuropsychological-assessment-pipeline]]