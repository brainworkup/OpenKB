---
doc_type: short
full_text: sources/KNOWLEDGE_BASE_EXPLAINED.md
---

# KNOWLEDGE_BASE_EXPLAINED — Summary

This document explains the architecture and current state of the PAI (Personal AI) knowledge base system, clarifying a key discovery: the system uses **Parquet files** for persistent storage, not DuckDB database files.

## Key Discovery

The original system was mistakenly assumed to rely on `.duckdb` files for storage. Investigation revealed that the Ragnar R package creates DuckDB instances in **temp directories or memory**, making them ephemeral. The actual storage layer uses **Parquet files**, which are more efficient for this use case. See [[concepts/parquet-as-knowledge-store]] for broader context on this pattern.

## System Architecture

The PAI [[concepts/retrieval-augmented-generation]] system consists of:

1. **Text Chunk Storage** — `pai_knowledge_base_chunks.parquet` holds chunked text from source PDFs.
2. **Embedding Storage** — `pai_embeddings_complete.parquet` holds vector embeddings per chunk.
3. **Search Process:**
   - Load embeddings into memory
   - Compute similarity between query and chunk vectors
   - Return top matches
4. **DuckDB** — Used only for complex ad-hoc queries, not persistent storage.

## Knowledge Base Update (Old vs. New)

| Metric | Old (Jan 7) | New (Jan 29) |
|--------|-------------|---------------|
| PDF Files | ~50–60 | 98 (79 reports + 19 sources) |
| Text Pages | Unknown | 1,663 |
| Chunks | Fewer | 4,830 |
| File Size | 638 KB | 1.1 MB |
| Chunk Size | Unknown | 1,000 chars + 100 overlap |

## Embedding Generation

New chunks require embeddings generated via **nomic-embed-text** (through [[concepts/local-llm-inference]] via Ollama). The R workflow uses the Ragnar and Arrow libraries for [[concepts/r-python-integration]]:

```r
embedded <- chunks |>
  mutate(embedding = embed_ollama(chunk_text, model = "nomic-embed-text"))
write_parquet(embedded, "pai_embeddings_complete_NEW.parquet")
```

Estimated time: 30–60 minutes.

## Key Files

| File | Purpose | Status |
|------|---------|--------|
| `pai_knowledge_base_chunks.parquet` | Text chunks | ✅ Updated (4,830 chunks) |
| `pai_embeddings_complete.parquet` | Vector embeddings | ⚠️ Outdated (needs regeneration) |
| `pai_rag_system.R` | Search functions | ✅ Ready |
| `pai_score_interpreter.R` | Report generation | ✅ Ready |

## Current Status

- ✅ Text extraction from 98 PDFs complete
- ✅ 4,830 chunks created and stored in Parquet
- ✅ Old version backed up
- ⚠️ Embeddings need regeneration for [[concepts/vector-search]] (semantic search)
- ✅ Text/keyword search available immediately

## Design Rationale

Storing chunks and embeddings in separate [[concepts/parquet-as-knowledge-store]] files (columnar format) is well-suited for this workload: efficient reads, portable, and easy to version or back up. DuckDB's role is limited to query-time operations, keeping the storage layer simple and persistent. This approach directly supports the [[concepts/pai-knowledge-base]] used in the broader [[concepts/neuropsychological-assessment-pipeline]].

## Related Concepts
- [[concepts/sqlite-as-vector-store]]
- [[concepts/pdf-score-extraction]]
- [[concepts/pai-assessment]]
