---
doc_type: short
full_text: sources/conversation-export.md
---

# Building a RAG System for PAI Neuropsychological Data

## Overview

This conversation documents the complete construction of a [[concepts/retrieval-augmented-generation]] (RAG) system for the [[concepts/pai-assessment]] (Personality Assessment Inventory) using R, the `ragnar` package, Ollama local embeddings, and a [[concepts/duckdb-as-vector-store]] backend with [[concepts/parquet-as-knowledge-store]] for persistent storage. The system was built iteratively, resolving several technical challenges along the way.

## System Architecture

The final system has three primary layers:

1. **Knowledge Base Layer** — 81 PAI PDF reports ingested, chunked into 2,546 text chunks (~780 chars each), embedded into 768-dimensional vectors using `nomic-embed-text` via [[concepts/local-llm-inference]] through Ollama.
2. **Retrieval Layer** — Both semantic (cosine similarity) and keyword (SQL LIKE) search, combined via a weighted [[concepts/hybrid-search-retrieval]] scoring function.
3. **LLM Integration Layer** — Prompt construction and API routing via a [[concepts/llm-provider-abstraction]] supporting Ollama (local), OpenAI, or Anthropic.

## Key Technical Steps

### Document Ingestion
- Used `ragnar::ragnar_read()` iteratively in a `for` loop (not vectorized — `ragnar_read` only accepts single paths)
- `markdown_segment()` was used for segmentation; direct `bind_rows()` on ragnar S7 objects failed, requiring manual text extraction
- Custom `chunk_long_text()` function split segments at 800 characters with 100-char overlap
- PDF source documents processed via the [[concepts/clinical-pdf-assessment]] workflow

### Embedding
- Embeddings created via `embed_ollama(model = "nomic-embed-text")`
- Processed in batches of 100 chunks; total time ~30 seconds for 2,546 chunks
- Critical bug: initial JSON serialization only stored first element of each 768-D vector; fixed by using `combined_embeddings$embedding_matrix[i, ]` (row indexing) instead of `i` (list indexing)

### Storage
- Text chunks stored in `pai_knowledge_base_chunks.parquet` (0.62 MB) as a [[concepts/parquet-as-knowledge-store]] artifact
- Full embeddings stored in `pai_embeddings_complete.parquet` (6.25 MB) as JSON strings
- [[concepts/duckdb-as-vector-store]] database (`pai_knowledge_base.duckdb`) with indexes on `doc_id`, `chunk_id`, `hash`
- Parquet was slightly larger than RDS for the text corpus, but DuckDB enables SQL queries without loading data into RAM

## Core Functions Built

### `pai_semantic_search(query, top_k, con)`
- Embeds query with `embed_ollama()`
- Normalizes query and document vectors
- Computes cosine similarity via matrix multiplication (`doc_vectors %*% query_vector`)
- Returns top-K chunks by similarity score; part of the [[concepts/vector-search]] pipeline
- Validated similarity scores: 0.68–0.74 for relevant matches

### `pai_hybrid_search(query, top_k, semantic_weight, text_weight, con)`
- Extracts keywords from query (removes stop words)
- Runs SQL `LIKE` search for each keyword, counting matches per chunk
- Normalizes both semantic and text scores to [0,1]
- Computes `hybrid_score = semantic_norm * semantic_weight + text_norm * text_weight`
- Tested configurations: 50/50 (balanced), 80/20 (semantic-heavy), 20/80 (text-heavy)
- Semantic-heavy recommended for conceptual queries; text-heavy for specific scale names/acronyms

### `ask_pai_expert(question, con, provider, model, ...)`
- Full pipeline: hybrid search → context formatting → prompt construction → LLM call
- Supports three providers via `call_llm()` implementing a [[concepts/llm-provider-abstraction]]: Ollama, OpenAI, Anthropic
- System roles: `"expert neuropsychologist"`, `"clinical supervisor"`, `"researcher"`
- Returns structured list with question, answer, sources, context, prompt
- Constitutes a complete [[concepts/neuropsychological-assessment-pipeline]] query interface

## Clinical Testing Results

Five real-world clinical scenarios tested:

| Scenario | Top Hybrid Score | Result Quality |
|---|---|---|
| DEP vs ANX differential diagnosis | 0.600 | Excellent |
| ICN/INF validity cutoffs | 0.400 | Very Good |
| Treatment prognosis indicators | 0.700 | Excellent |
| BOR scale interpretation | 0.500 | Good |
| Complex comorbid case (DEP+ANX, normal BOR) | 0.650 | Excellent |

## PAI Instrument Context

The [[concepts/pai-assessment]] (PAI; Morey, 1991) is a 344-item self-report personality/psychopathology measure with 22 non-overlapping scales:
- 4 validity scales (ICN, INF, NIM, PIM)
- 11 clinical scales (including DEP, ANX, ARD, BOR, SCZ, PAR, MAN, etc.)
- 5 treatment scales
- 2 interpersonal scales

Scales use T-scores (mean=50, SD=10). Key interpretive thresholds documented in retrieved chunks (e.g., ICN ≥ 73T = invalid, INF ≥ 75T = invalid; NIM > 81T indicates negative impression management). These scores map onto the broader domain of [[concepts/neuropsychological-test-scores]].

The corpus also includes a published study (McCredie & Morey, 2019, *Assessment*) characterizing MTurk workers on the PAI: MTurk workers showed higher social detachment (SCZ-S), moderately higher depression (DEP), and slightly higher anxiety, while validity scales did not differ from community norms.

## Bugs and Resolutions

| Problem | Cause | Fix |
|---|---|---|
| `bind_rows()` fails on doc list | S7 object not subsettable | Used `for` loop with manual extraction |
| All similarity scores identical | Only 1st element of 768-D vector stored | Used `embedding_matrix[i, ]` row indexing |
| JSON serialization produced 1-element vectors | `jsonlite::toJSON()` on list element | Extract via row index before serializing |
| Ollama 404 on `/api/generate` | Model name mismatch or not pulled | Verify with `ollama list`; pull required model |

## Files Created

- `pai_knowledge_base_chunks.parquet` — text chunks (0.62 MB)
- `pai_embeddings_complete.parquet` — full 768-D embeddings (6.25 MB)
- `pai_knowledge_base.duckdb` — queryable database with indexes
- `pai_knowledge_base_chunks.rds` — legacy RDS backup

## Related Concepts
- [[concepts/pai-knowledge-base]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/neuropsychological-reporting]]
