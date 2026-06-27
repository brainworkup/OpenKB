---
doc_type: short
full_text: sources/0002-soul-sqlite-vector-storage.md
---

# 0002 — SQLite for Vector Storage

**Type:** Architecture Decision Record (ADR)  
**Date:** 2025-01-20  
**Status:** Accepted

## Summary

This ADR documents the decision to use **SQLite with JSON-serialized embeddings** as the vector store for a [[concepts/retrieval-augmented-generation]] pipeline. The choice prioritizes simplicity and portability over advanced vector-search capabilities.

## Problem Context

The system needs to store text chunks alongside their [[concepts/embeddings]] for semantic similarity search. Three options were evaluated:

1. **Dedicated vector databases** (Chroma, Weaviate, Pinecone) — rejected as overkill for the expected scale
2. **SQLite with JSON serialization** — selected
3. **In-memory only storage** — rejected for lack of persistence

## Decision

Use SQLite to persist chunks and their embeddings, serialized as JSON strings via `json.dumps()` / `json.loads()`.

## Rationale

The [[concepts/single-file-architecture]] constraint and modest expected scale (hundreds to thousands of historical reports) make a dedicated vector DB unnecessary. SQLite offers:

- Zero additional dependencies
- ACID compliance
- Portable `.sqlite` file
- Sufficient performance for the target data volume

## Database Schema

```sql
CREATE TABLE IF NOT EXISTS chunks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source_path TEXT NOT NULL,
    chunk_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    embedding TEXT NOT NULL,  -- JSON serialized vector
    UNIQUE(source_path, chunk_id)
)
```

## Trade-offs

| Aspect | Detail |
|---|---|
| **Search** | O(n) [[concepts/cosine-similarity]] scan — no approximate nearest neighbor optimizations |
| **Memory** | All embeddings loaded into memory at query time |
| **Portability** | Single `.sqlite` file, easily moved between machines |
| **Config** | Auto-initializes, zero external setup |

## Scaling Limits

- Suitable for up to **~10,000 chunks**
- Acceptable CLI response times of ~1–2 seconds at that scale
- Migration to a proper vector database recommended if scale is exceeded

## Implementation Notes

- [[concepts/cosine-similarity]] computed in pure Python after loading all embeddings
- This approach is documented as an [[concepts/architecture-decision-records]] entry
- References: `neuro_report_style_agent.py:155-169`, `neuro_report_style_agent.py:218-235`

## Related Concepts
- [[concepts/sqlite-as-vector-store]]
- [[concepts/vector-search]]
- [[concepts/clinical-data-privacy]]
