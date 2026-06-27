---
doc_type: short
full_text: sources/vector-store.md
---

# Vector Store Component

## Overview

A lightweight, zero-external-dependency vector storage system built on [[concepts/sqlite-as-vector-store]], implementing pure-Python cosine similarity search for retrieving semantically similar text chunks. Located in `soul/neuro_report_style_agent.py:147-235`.

## Schema

Stores chunks in a single `chunks` table:

- `id` — auto-incrementing primary key
- `source_path` — original file path
- `chunk_id` — position within file
- `content` — chunk text
- `embedding` — JSON-serialized float vector

A unique constraint on `(source_path, chunk_id)` prevents duplicate entries. There is **no vector index**; similarity is computed via full table scan.

## Core Functions

| Function | Purpose |
|---|---|
| `init_db(db_path)` | Creates DB and schema (idempotent) |
| `_serialize_vector(vec)` | Converts float list to compact JSON string |
| `_deserialize_vector(raw)` | Parses JSON string back to float list |
| `cosine_similarity(a, b)` | Pure-Python dot-product / norm computation |
| `retrieve_top_k(db_path, query_embedding, k)` | Returns k most similar `ChunkRecord` objects |

## Retrieval Algorithm

`retrieve_top_k` performs a brute-force [[concepts/vector-search]]:
1. Load **all** embeddings from database into memory
2. Compute cosine similarity against the query vector for each chunk
3. Sort descending by score
4. Return top-k as `ChunkRecord(source_path, chunk_id, content)`

Time complexity: **O(n)** — suitable for fewer than ~10,000 chunks.

## Build Index Workflow

Indexing pipeline:
1. `init_db` — initialize storage
2. Iterate report files, extract text
3. [[concepts/text-chunking]] with `chunk_size=1200`, `overlap=150`
4. Embed each chunk via `embed_with_fallback` (ModernBERT, ~768 dims)
5. Serialize and `INSERT OR REPLACE` into SQLite

## Storage Estimates

| Scale | Approximate Size |
|---|---|
| 1,000 chunks | 5–10 MB |
| 10,000 chunks | 50–100 MB |

Embedding JSON size: ~5–10 KB per vector at ~768 dimensions.

## Limitations

- No approximate nearest neighbor (ANN) indexing
- All embeddings must fit in memory at query time
- Single-threaded SQLite writes
- No metadata fields (date, tags, etc.)

## Scaling Thresholds

Migrate to a dedicated vector database when:
- Chunk count exceeds 10,000
- Query latency exceeds 5 seconds
- Metadata filtering is required
- Multi-user concurrent access is needed

## Related Concepts
- [[concepts/retrieval-augmented-generation]]
- [[concepts/local-first-architecture]]
- [[concepts/fallback-strategy]]
- [[concepts/single-file-agent-pattern]]
- [[concepts/lancedb-vector-store]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/parquet-as-knowledge-store]]
