---
sources: [summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/SKILL.md, summaries/DEPENDENCIES.md, summaries/style-training-to-report-drafting.md, summaries/vector-store.md, summaries/text-extraction.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/embedding-client.md, summaries/0002-soul-sqlite-vector-storage.md, summaries/conversation-export.md, summaries/TECHNICAL_DOCS.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_AS_PROCESSING.md, summaries/README.md, summaries/QUICK_REFERENCE.md, summaries/OCR_PDF_GUIDE.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/COMPLETE_STATUS.md]
brief: Retrieval technique using dense vector embeddings and similarity search to find semantically related text chunks.
---

# Vector Search and Embedding-Based Retrieval

Vector search is a retrieval technique that represents text as dense numerical vectors (embeddings) and finds relevant documents by measuring geometric similarity between a query vector and stored document vectors. Unlike keyword search, which requires exact term matches, vector search captures semantic meaning — synonyms, paraphrases, and related concepts can surface relevant results even when surface words differ.

## How It Works

1. **Embedding generation** — A language model (encoder) converts text chunks into high-dimensional float vectors. Each vector encodes the semantic content of the passage.
2. **Index storage** — Vectors are stored in a vector store or database alongside the original text.
3. **Query encoding** — At retrieval time, the user's query is encoded with the same model into a query vector.
4. **Similarity search** — The system computes distance (cosine similarity, dot product, or L2) between the query vector and all stored vectors, returning the top-k closest matches.
5. **Context assembly** — Retrieved chunks are assembled into a prompt context for a language model to generate a grounded response.

This pipeline is the retrieval backbone of [[concepts/retrieval-augmented-generation]] (RAG) systems.

## Embeddings vs. Keyword Search

| Dimension | Keyword (text) Search | Vector (Semantic) Search |
|-----------|----------------------|-------------------------|
| Mechanism | Exact/regex term match | Similarity in embedding space |
| Handles synonyms | ❌ No | ✅ Yes |
| Setup required | None | Embedding model + index |
| Speed | Very fast | Moderate (ANN index helps) |
| Sophistication | Low | High |
| Works offline | ✅ Yes | ✅ Yes (local models) |

A practical illustration: searching "high energy symptoms" via keyword search would miss a passage containing "Mania scale T=67", but a semantic vector search correctly retrieves it because the vectors encode conceptual relatedness.

## Cosine Similarity

The most common similarity metric for vector search is cosine similarity, which measures the angle between two vectors regardless of their magnitude:

```python
def cosine_similarity(a, b):
    dot = sum(x * y for x, y in zip(a, b))
    norm_a = math.sqrt(sum(x * x for x in a))
    norm_b = math.sqrt(sum(y * y for y in b))
    if norm_a == 0 or norm_b == 0:
        return 0.0
    return dot / (norm_a * norm_b)
```

This pure-Python implementation (from the [[concepts/sqlite-as-vector-store]] component in `soul/neuro_report_style_agent.py`) requires no external libraries and is suitable for small corpora where brute-force O(n) full-table scans are acceptable.

## Storage Backends

Embedding vectors must be persisted for reuse. Multiple backends are used across projects:

- **SQLite** — Lightweight, zero-dependency store serializing vectors as compact JSON strings. Used in the `soul` style agent with a `chunks` table (see [[concepts/sqlite-as-vector-store]] and [[summaries/0002-soul-sqlite-vector-storage]]). Full-scan similarity is O(n); practical for <10,000 chunks.
- **Parquet** (`pai_embeddings_complete_NEW.parquet`, 16.4 MB) — Portable columnar format; primary file for embeddings + text in the PAI system
- **R native RDS** (`pai_embeddings_complete_NEW.rds`, 46.2 MB) — Fast loading within R workflows; see [[concepts/r-python-integration]]
- **DuckDB** (`pai_embeddings_NEW.duckdb`, 38.8 MB) — SQL-queryable database with embedded vectors; see [[concepts/duckdb-as-vector-store]]

### SQLite Storage Efficiency

For the SQLite-based store using ~768-dimensional ModernBERT embeddings:

| Scale | Approximate Size |
|---|---|
| 1,000 chunks | 5–10 MB |
| 10,000 chunks | 50–100 MB |

Each embedding serializes to ~5–10 KB as a compact JSON string. When chunk counts exceed ~10,000 or query latency grows beyond acceptable bounds, migrating to a dedicated vector database with approximate nearest neighbor (ANN) indexing is recommended.

## PAI Knowledge Base Example

The PAI (Personality Assessment Inventory) [[concepts/pai-knowledge-base]] provides a detailed real-world case study of vector search in a clinical interpretation system.

### Evolution of the System

**Old version** (`pai_knowledge_base_copy.duckdb`, 17.5 MB): 81 documents with pre-generated embeddings. Full semantic search available but corpus incomplete.

**Intermediate state** (documented in [[summaries/COMPLETE_STATUS]]): 98 documents, 4,830 chunks rebuilt in Parquet format — but embeddings not yet generated. Only keyword search available, illustrating the classic **fallback pattern**: text search as an interim [[concepts/fallback-strategy]] while embeddings are being produced.

**Current version** (documented in [[summaries/EMBEDDINGS_COMPLETE]]): All 4,830 chunks from 98 documents now fully embedded with `snowflake-arctic-embed2:568m`, achieving 100% coverage. Key statistics:

| Feature | Old | New |
|---------|-----|-----|
| Documents | 81 | 98 (+17) |
| Chunks | 2,546 | 4,830 (+89.7%) |
| Embedding model | Unknown | snowflake-arctic-embed2:568m |
| Vector dimensions | Unknown | 1,024 |
| Completeness | Partial | 100% ✅ |

Generation took approximately 10 minutes (97 batches of 50 chunks each), much faster than anticipated due to the efficiency of the snowflake-arctic model.

## Embedding Models

For local, privacy-preserving use cases (see [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]]), embedding models run entirely on-device:

- **snowflake-arctic-embed2:568m** — Used in the PAI system; 568M parameter encoder producing 1,024-dimensional vectors, optimized for retrieval and notably fast in practice
- **ModernBERT** — Used in the `soul` style agent SQLite store; ~768-dimensional vectors
- Models served via [[concepts/local-llm-inference]] infrastructure
- Compatible with [[concepts/openai-compatible-api]] endpoints for drop-in integration

Multimodal variants that encode images alongside text are discussed in [[concepts/multimodal-embeddings]].

## Build Index Workflow

A typical indexing pipeline (as implemented in the `soul` style agent):

1. Initialize storage backend
2. Iterate source files, extract text
3. Split into chunks via [[concepts/text-chunking]] (e.g., `chunk_size=1200`, `overlap=150`)
4. Embed each chunk using the chosen model
5. Serialize and upsert (`INSERT OR REPLACE`) into the store

Batch processing (e.g., 50 chunks/batch) balances throughput and memory during embedding generation.

## Retrieval Quality Techniques

- **Late interaction** — Approaches like ColBERT that score query-document token pairs at retrieval time rather than compressing to a single vector; see [[concepts/late-interaction-retrieval]]
- **Chunk size tuning** — Smaller chunks increase precision; larger chunks preserve context. The PAI system grew from 2,546 to 4,830 chunks with the new corpus rebuild, reflecting finer granularity and more documents.
- **Hybrid search** — Combining keyword and vector scores (BM25 + cosine) for robustness against edge cases; see [[concepts/hybrid-search-retrieval]]
- **Batch processing** — Generating embeddings in batches balances throughput and memory
- **Scaling thresholds** — When chunk count exceeds ~10,000, query latency grows unacceptably with brute-force O(n) search; ANN indexing or a dedicated vector database becomes necessary

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — RAG pipelines that consume vector search output
- [[concepts/late-interaction-retrieval]] — Advanced retrieval with per-token scoring
- [[concepts/multimodal-embeddings]] — Embeddings spanning text and image modalities
- [[concepts/sqlite-as-vector-store]] — Lightweight local vector storage with pure-Python similarity
- [[concepts/duckdb-as-vector-store]] — SQL-queryable alternative backend
- [[concepts/local-llm-inference]] — Running embedding models on-device
- [[concepts/fallback-strategy]] — Text search as interim when embeddings are unavailable
- [[concepts/hybrid-search-retrieval]] — Combining keyword and semantic search
- [[concepts/pai-knowledge-base]] — Applied example knowledge base using this approach
- [[concepts/neuropsychological-assessment-pipeline]] — Clinical pipeline that benefits from semantic retrieval
- [[concepts/phi-data-handling]] — Privacy constraints that motivate local embedding generation
- [[concepts/privacy-first-software]] — Design philosophy requiring on-device processing
- [[concepts/text-chunking]] — Splitting source text into indexable units

See also: [[summaries/KNOWLEDGE_BASE_EXPLAINED]]

See also: [[summaries/OCR_PDF_GUIDE]]

See also: [[summaries/QUICK_REFERENCE]]

See also: [[summaries/README]]

See also: [[summaries/README_AS_PROCESSING]]

See also: [[summaries/REBUILD_COMPLETE]]

See also: [[summaries/REBUILD_FINAL_STATUS]]

See also: [[summaries/TECHNICAL_DOCS]]

See also: [[summaries/conversation-export]]

See also: [[summaries/0002-soul-sqlite-vector-storage]]

See also: [[summaries/embedding-client]]

See also: [[summaries/soul-style-agent]]

See also: [[summaries/style-trainer]]

See also: [[summaries/text-extraction]]

See also: [[summaries/vector-store]]

See also: [[summaries/style-training-to-report-drafting]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/SKILL]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]