---
sources: [summaries/vector-store.md, summaries/responses_to_claude.md, summaries/conversation-export.md, summaries/TECHNICAL_DOCS.md, summaries/REBUILD_FINAL_STATUS.md, summaries/README_AS_PROCESSING.md, summaries/README.md, summaries/OCR_PDF_GUIDE.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md]
brief: Parquet files serve as portable, versionable columnar storage for RAG text chunks and embeddings.
---

# Parquet as Knowledge Store

Parquet is an open-source **columnar storage format** widely used in data engineering. In the context of local AI and RAG (Retrieval-Augmented Generation) systems, Parquet files serve as a lightweight, portable, and efficient alternative to purpose-built vector databases or embedded SQL databases for storing text chunks and embeddings.

See [[summaries/KNOWLEDGE_BASE_EXPLAINED]] for a detailed explanation of this pattern, and [[summaries/REBUILD_FINAL_STATUS]] for a concrete case study of a corpus rebuild using this architecture. The [[summaries/TECHNICAL_DOCS]] document provides a detailed production example with specific storage metrics. The [[summaries/conversation-export]] documents the original construction of the PAI RAG system, including the discovery of the embedding serialization bug and the iterative process of getting full 768-D vectors stored correctly.

## Why Parquet Instead of a Database File?

Some RAG tooling (such as the R `ragnar` package) creates database instances — like DuckDB — **ephemerally in temp memory**, making them unsuitable for persistent storage. Parquet files sidestep this problem entirely: they are plain files on disk, versioned like any other asset, and read efficiently into memory when needed.

A real-world illustration from the original PAI system build: a corpus rebuild discovered that no persistent `.duckdb` file had ever existed — the system had always used Parquet for storage and DuckDB only for in-memory queries. This was not a bug but a deliberate design advantage.

Key advantages:
- **Columnar format** — fast reads of specific columns (e.g., only embeddings, only chunk text)
- **Portable** — no server, no daemon, no connection string
- **Versionable** — easy to back up, diff, and swap (e.g., `_OLD` / `_NEW` naming conventions)
- **Interoperable** — readable by R (`arrow`), Python (`pyarrow`, `pandas`), DuckDB, Spark, and more
- **Corruption-resistant** — plain files avoid database corruption issues common with persistent DB files

## Typical File Layout

In a Parquet-backed RAG system, storage is typically split across two files:

| File | Contents | Role |
|------|----------|------|
| `knowledge_base_chunks.parquet` | Document text chunks + metadata | Source text retrieval |
| `embeddings_complete.parquet` | Vector embeddings per chunk | Semantic similarity search |

This separation allows the text corpus to be updated independently from embeddings, and makes regenerating embeddings (a slow step) a distinct, resumable operation. Backup files follow a simple naming convention (`_OLD`, `_NEW`) for safe rollback.

### PAI System: Two Known Configurations

The PAI RAG system provides concrete size benchmarks across two corpus generations:

**Original build (conversation-export)** — 81 source PDFs, 2,546 chunks:

| File | Size | Purpose |
|------|------|--------|
| `pai_knowledge_base_chunks.parquet` | 0.62 MB | Main text corpus |
| `pai_embeddings_complete.parquet` | 6.25 MB | 768-D vectors per chunk |
| DuckDB (in-process) | 0.01 MB | Query layer only |
| **Total** | **~6.9 MB** | Full knowledge base |

Note: the Parquet embeddings file was initially **larger** than the RDS equivalent (0.62 MB Parquet vs ~0.36 MB RDS) for the text chunks alone, because the small corpus offered little compression benefit over R's native format. The real advantage of Parquet is in enabling SQL queries without loading data into RAM, not necessarily in file size at small scales.

**Rebuild corpus (REBUILD_FINAL_STATUS)** — 98 source PDFs, 4,830 chunks:

| File | Size | Status | Purpose |
|------|------|--------|--------|
| `pai_knowledge_base_chunks.parquet` | 1.1 MB | Updated | Main text corpus |
| `pai_knowledge_base_chunks_NEW.rds` | 759 KB | New | R native backup |
| `pai_knowledge_base_chunks_OLD.parquet` | 638 KB | Archived | Prior version |
| `pai_embeddings_complete.parquet` | 6.2 MB | Stale | Pending regeneration |

The embedding file in the rebuild scenario was stale because regeneration had not yet completed — illustrating why the two-file split is operationally important.

## Embedding Storage Format

In the PAI system documented in [[summaries/TECHNICAL_DOCS]] and originally constructed in [[summaries/conversation-export]], embeddings are stored as JSON-serialized 768-dimensional float vectors in the `embedding_json` column of the embeddings Parquet file.

A critical implementation detail emerged during the original build: `jsonlite::toJSON()` applied to a list element (`embedding_matrixi`) only serialized the first element of each 768-D vector, producing 1-D rather than 768-D embeddings. The fix was to extract by **row index** (`embedding_matrix[i, ]`), which correctly returns all 768 dimensions. This was discovered when all cosine similarity scores came back identical — a diagnostic signal that all stored vectors were effectively scalar.

The per-chunk storage cost is approximately:

```
768 dimensions × 4 bytes = 3.1 KB per chunk
2,546 chunks × 3.1 KB = ~7.9 MB uncompressed
Compressed in Parquet: 6.25 MB
```

This compression ratio (~21%) demonstrates Parquet's efficiency for floating-point columnar data. The embedding model used is `nomic-embed-text` via [[concepts/local-llm-inference]], producing 768-D vectors.

## Integration with DuckDB

DuckDB is not absent from this architecture — it is used at **query time** for complex filtering and aggregation over Parquet files. The production PAI system uses DuckDB as an in-process SQL engine with dedicated indexes for fast lookup:

- `idx_doc_id` on `pai_chunks(doc_id)`
- `idx_chunk_id` on `pai_chunks(chunk_id)`
- `idx_emb_doc_id` on `pai_embeddings(doc_id)`
- `idx_emb_chunk_id` on `pai_embeddings(chunk_id)`

DuckDB can also query Parquet files directly without importing them into a persistent database:

```sql
-- Example: query Parquet directly with DuckDB
SELECT chunk_text, source_file
FROM read_parquet('knowledge_base_chunks.parquet')
WHERE source_file LIKE '%report%'
LIMIT 10;
```

In the original build, DuckDB was used to demonstrate keyword search capabilities across 2,546 chunks, returning results in milliseconds without loading the full corpus into R. The build also illustrated that DuckDB indexes must be dropped and recreated when tables are replaced, as `CREATE INDEX` will fail if a same-named index already exists on the old table.

See [[concepts/duckdb-as-vector-store]] for more on DuckDB's role in vector retrieval workflows.

## Chunking Strategy

The quality of a Parquet knowledge store depends heavily on how source documents are chunked before storage. Two configurations are documented:

**Original PAI system** (from [[summaries/conversation-export]] and [[summaries/TECHNICAL_DOCS]]):
- **Chunks**: 2,546 from 81 PDFs
- **Avg chunk size**: 784 characters
- **Method**: Custom `chunk_long_text()` function, 800-char max with 100-char overlap
- **Total text**: ~2.0 million characters

The original chunking approach worked around `ragnar`'s `markdown_chunk()` function, which does not accept `chunk_size` or `chunk_overlap` parameters directly. Instead, `markdown_segment()` was applied first, then a custom R function split oversized segments.

**Rebuilt corpus** (from [[summaries/REBUILD_FINAL_STATUS]]):
- **Chunk size**: 1,000 characters
- **Overlap**: 100 characters between adjacent chunks
- **Source**: 98 PDF documents (79 clinical reports + 19 research documents)
- **Output**: 4,830 chunks — a **+89.7% increase** from the prior 2,546 chunks

The overlap strategy preserves context at chunk boundaries, improving retrieval quality. A mismatch between chunk store and embedding store causes semantic search to fail, which is the primary operational risk of the two-file architecture.

## Keyword vs. Semantic Search

A Parquet-backed system supports two retrieval modes with different dependencies:

**Keyword/regex search** — available immediately using only the chunk file:
```r
library(arrow)
chunks <- read_parquet("knowledge_base_chunks.parquet")
results <- chunks |>
  filter(str_detect(chunk_text, regex("mania", ignore_case = TRUE)))
```
The production system benchmarks keyword search at **50–100 ms**. During the original build, SQL `LIKE` queries via DuckDB returned results for terms like `'%depression%'` across 2,546 chunks with no perceptible delay.

**Semantic search** — requires up-to-date embeddings aligned to the current chunk corpus. The production system benchmarks semantic (vector cosine) search at **500–800 ms**. The original build achieved validated similarity scores of 0.68–0.74 for highly relevant results after the embedding serialization bug was corrected. When the corpus is rebuilt, old embeddings become stale and must be regenerated before [[concepts/vector-search]] can work correctly.

**Hybrid search** — combines both approaches via weighted scoring, benchmarked at **600–900 ms**. The original build tested three configurations: balanced (50/50), semantic-heavy (80/20), and text-heavy (20/80). See [[concepts/hybrid-search-retrieval]] for the full pattern.

## Embedding Regeneration

When the chunk corpus is updated, embeddings must be regenerated. In R-based systems using Ollama, this looks like:

```r
library(arrow)
chunks <- read_parquet("knowledge_base_chunks.parquet")
embedded <- chunks |>
  mutate(
    embedding = embed_ollama(
      chunk_text,
      model = "nomic-embed-text"
    )
  )
write_parquet(embedded, "embeddings_complete_NEW.parquet")
```

In the original PAI build, all 2,546 chunks were embedded in approximately **30 seconds** (~4,800 chunks/minute) using `nomic-embed-text` via Ollama in batches of 100. This was dramatically faster than the initial 15–30 minute estimate, suggesting Ollama's batch throughput is efficient for this model size.

The two-file architecture means this long-running step can be performed without disrupting text-based search.

## Scalability Characteristics

The production PAI system documents how the Parquet-based architecture scales:

| Scale | Chunks | Approach | Notes |
|-------|--------|----------|-------|
| Current | 2,546 | Parquet + DuckDB in-process | ~7 MB total |
| 10× | ~25,000 | No changes needed | ~70 MB |
| 100× | ~250,000 | Add approximate nearest neighbor (ANN) search | e.g., FAISS |
| 1,000×+ | 2.5M+ | Purpose-built vector database | Pinecone, Weaviate |

For most single-user clinical workloads, the Parquet + DuckDB combination remains viable well beyond current corpus sizes.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — the broader system pattern this storage layer supports
- [[concepts/vector-search]] — the runtime search operation over stored embeddings
- [[concepts/hybrid-search-retrieval]] — combining semantic and keyword search over Parquet-stored data
- [[concepts/local-llm-inference]] — the inference environment (e.g., Ollama) that generates embeddings
- [[concepts/pai-knowledge-base]] — the specific PAI system using this storage pattern
- [[concepts/pdf-score-extraction]] — upstream process that produces text fed into the chunk store
- [[concepts/r-python-integration]] — cross-language tooling (`arrow`, `pyarrow`) used to read/write Parquet
- [[concepts/sqlite-as-vector-store]] — an alternative lightweight storage approach for vector data
- [[concepts/phi-data-handling]] — important consideration when storing clinical text in Parquet files
- [[concepts/duckdb-as-vector-store]] — DuckDB's complementary role as an in-memory query layer
- [[concepts/pai-assessment]] — the assessment instrument whose documents populate this knowledge store

See also: [[summaries/OCR_PDF_GUIDE]] · [[summaries/README]] · [[summaries/README_AS_PROCESSING]] · [[summaries/TECHNICAL_DOCS]] · [[summaries/conversation-export]]

See also: [[summaries/responses_to_claude]]

See also: [[summaries/vector-store]]