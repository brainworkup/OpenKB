---
sources: [summaries/SESSION_SUMMARY.md, summaries/CLAUDE.md, summaries/DEPENDENCIES.md, summaries/vector-store.md, summaries/neuropsych-data-extractor.md, summaries/conversation-export.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/README.md, summaries/QUICK_REFERENCE.md]
brief: DuckDB serves as an embedded analytical database and vector store for local, privacy-first RAG and neuropsych data pipelines.
---

# DuckDB as Vector Store

DuckDB is a high-performance, embedded analytical database that can serve double duty as a vector store in [[concepts/retrieval-augmented-generation]] pipelines. Rather than deploying a dedicated vector database service, DuckDB allows developers to store document chunks, metadata, and embedding vectors in a single portable `.duckdb` file — making it an attractive choice for local, privacy-first RAG systems. It also appears as a core staging layer in the `cingulate` neuropsychological reporting package, where it underpins the CSV → Parquet → in-memory query workflow that feeds the PDF report pipeline.

## How It Works

DuckDB's columnar storage engine efficiently handles the dense numerical arrays that represent text embeddings. When combined with an R or Python RAG library, it can persist:

- **Document chunks** — raw text segments derived from source files
- **Embedding vectors** — high-dimensional float arrays produced by an embedding model
- **Metadata** — source file name, page number, document type, etc.

At query time, the system encodes the user's question with the same embedding model, then performs **vector similarity search** (cosine or dot-product) against stored embeddings to retrieve the most relevant chunks.

## DuckDB in the Cingulate Neuropsych Pipeline

In the `cingulate` R package (see [[summaries/CLAUDE]]), DuckDB plays a performance-oriented staging role via the `DuckDBProcessorR6` class:

- Input CSVs are loaded into DuckDB, then materialized as Parquet files for fast columnar access
- In-memory queries are run against these Parquet-backed tables via the `query_neuropsych()` helper
- This staging layer sits between raw user-supplied CSVs and the downstream R6 domain processors that generate per-domain Quarto includes

This use aligns with [[concepts/parquet-as-knowledge-store]] as a complementary pattern: DuckDB manages the query interface while Parquet provides compressed, portable storage. The `cingulate` pipeline is documented further in [[summaries/README_PIPELINE]] and the broader [[concepts/neuropsychological-assessment-pipeline]].

## Hybrid Search

One strength of DuckDB-backed stores is the ability to combine **vector similarity search** with **full-text search** in a single query. This [[concepts/hybrid-search-retrieval]] approach improves recall for both semantic and keyword-specific queries — important in clinical contexts where exact scale names or acronyms matter.

In the PAI RAG system, hybrid scoring is computed as:

```
hybrid_score = (semantic_score_norm × semantic_weight) +
               (text_score_norm × text_weight)
```

Both scores are normalized to [0, 1] before combination to ensure fair weighting. Keyword extraction automatically removes stop words, and per-keyword SQL `LIKE` matches are summed to produce a raw text score before normalization.

## DuckDB in the PAI RAG System

DuckDB functions as the knowledge base backend in two complementary configurations within the [[concepts/pai-knowledge-base]] ecosystem.

### ragnar-based Configuration

The rebuild documented in [[summaries/REBUILD_COMPLETE]] produced:

- **File:** `pai_knowledge_base.duckdb` (18.4 MB)
- **Documents stored:** 98 total — 79 PAI clinical reports + 19 research/source documents
- **Embedding model:** `snowflake-arctic-embed2:568m` (run locally via [[concepts/local-llm-inference]])
- **Search mode:** Vector similarity (VSS) + full-text search (FTS) — hybrid
- **R interface:** the `ragnar` v2 package via `ragnar_store_connect()`, `ragnar_store_ingest()`, `markdown_chunk()`, and `ragnar_retrieve()`

### Parquet + DuckDB Configuration

A complementary approach uses Parquet files alongside DuckDB (documented in [[summaries/TECHNICAL_DOCS]] and demonstrated hands-on in [[summaries/conversation-export]]):

- **`pai_knowledge_base_chunks.parquet`** (0.62 MB) — chunk text and metadata
- **`pai_embeddings_complete.parquet`** (6.25 MB) — 768-D embedding vectors serialized as JSON strings
- **DuckDB** (0.01 MB) — in-process SQL database with indexes
- **Total**: ~6.9 MB for 2,546 chunks from 81 documents

This configuration uses [[concepts/parquet-as-knowledge-store]] for columnar, compressed storage of both chunks and embeddings, with DuckDB providing the query interface. The two tables are:

| Table | Key Fields |
|---|---|
| `pai_chunks` | doc_id, chunk_id, origin, text |
| `pai_embeddings` | doc_id, chunk_id, hash, origin, text_preview, embedding_json |

Indexes are maintained on `doc_id` and `chunk_id` for both tables for fast joins.

The embedding model used in this configuration is `nomic-embed-text` via Ollama (768 dimensions), producing ~3.1 KB per embedding vector. At 2,546 chunks, this totals ~7.9 MB uncompressed, compressed to 6.25 MB in Parquet.

#### Building the Embedding Table: A Critical Detail

The [[summaries/conversation-export]] session revealed an important implementation pitfall: when extracting embedding vectors from a batched `embed_ollama()` result, the embedding column is a matrix of shape `(n_chunks × 768)`. Indexing with `i` retrieves a scalar (the first element only), while `[i, ]` correctly retrieves the full 768-dimensional row. The correct pattern is:

```r
full_vector <- combined_embeddings$embedding_matrix[i, ]  # correct
# NOT: combined_embeddings$embedding_matrixi         # wrong — returns scalar
```

Similarly, `jsonlite::toJSON()` must be called on `as.numeric(full_vector)` to serialize the full array; calling it on a list element produces a one-element JSON array. Verifying embedding dimensionality before writing to storage (e.g., `length(jsonlite::fromJSON(embedding_json_vec[1]))` should equal 768) catches this class of error early.

### Query Performance

| Operation | Latency |
|---|---|
| Semantic search | 500–800 ms |
| Keyword search | 50–100 ms |
| Hybrid search | 600–900 ms |

**Retrieval accuracy** (clinical testing on PAI questions):
- Top-1 relevance: 70–75%
- Top-3 relevance: 85–92%
- Top-5 relevance: 92–97%

Clinical testing confirmed the system returns meaningful results (hybrid scores 0.40–0.70) for differential diagnosis, validity assessment, treatment prognosis, and scale interpretation queries.

**Memory usage**: Baseline ~50 MB; peak <100 MB

### Standard Query Pattern (ragnar)

```r
library(ragnar)

# Connect to the store
store <- ragnar_store_connect("pai_knowledge_base.duckdb")

# Retrieve top-k relevant chunks
results <- ragnar_retrieve(
  store = store,
  text = "What does an elevated Mania scale mean?",
  top_k = 5L
)

# Access the most relevant passage
results$text[1]
```

### Standard Query Pattern (Parquet + DuckDB)

```r
library(duckdb)
library(jsonlite)

con <- dbConnect(duckdb(), "pai_knowledge_base.duckdb")

# Load embeddings and compute cosine similarity
all_emb <- dbGetQuery(con, "SELECT * FROM pai_embeddings")
query_vector <- embed_query("depression cognitive symptoms")  # normalized

doc_matrix <- do.call(rbind, lapply(all_emb$embedding_json, fromJSON))
doc_matrix <- sweep(doc_matrix, 1, sqrt(rowSums(doc_matrix^2)), "/")
similarities <- as.numeric(doc_matrix %*% query_vector)
```

### Rebuild Workflow

The main rebuild script (`rebuild_pai_ragnar_v2.R`) automates the full pipeline:
1. Remove old database
2. Re-scan `reports/` and `source/` folders
3. Ingest all PDFs
4. Generate embeddings with `snowflake-arctic-embed2:568m`
5. Build FTS + VSS indexes

A full rebuild of ~100 documents takes approximately 5–10 minutes when Ollama is running locally. The rebuild also resolved several prior issues including wrong embedding model, incorrect ragnar API usage, and store version mismatch errors — a reminder that DuckDB's SQL interface is always available as a fallback inspection mechanism when higher-level library APIs fail.

## LLM Integration Pattern

Once chunks are retrieved, [[summaries/conversation-export]] demonstrates a clean three-function helper architecture for LLM integration:

1. **`format_context_for_llm()`** — concatenates top-K chunks with relevance scores into a single context string
2. **`build_pai_prompt()`** — wraps context and question into system/user message pairs, with configurable system roles (`"expert neuropsychologist"`, `"clinical supervisor"`, `"researcher"`)
3. **`call_llm()`** — routes to the appropriate provider (Ollama, OpenAI, Anthropic) via [[concepts/llm-provider-abstraction]]

The complete `ask_pai_expert()` function chains all three steps into a single call, returning the AI answer alongside source chunks and scores.

## Scalability Characteristics

The Parquet + DuckDB approach scales predictably:

| Scale | Chunks | Approach |
|---|---|---|
| Current | 2,546 | DuckDB + Parquet |
| 10× | ~25,000 | No code changes needed |
| 100× | ~250,000 | Consider ANN (FAISS/Annoy) |
| 1,000×+ | 2.5M+ | Specialized vector DB (Pinecone, Weaviate) |

For the neuropsychological assessment use case — small corpora (dozens to hundreds of documents), strict data privacy requirements — DuckDB's simplicity and portability make it the right tool.

## Advantages Over Dedicated Vector Databases

| Property | DuckDB | Dedicated Vector DB |
|---|---|---|
| Deployment | Single file, no server | Requires running service |
| Privacy | Fully local | Often cloud-hosted |
| Query flexibility | SQL + vectors + full-text | Vector search primarily |
| Setup complexity | Low | Moderate to high |
| Scalability | Moderate (single node) | High (distributed) |

This setup means the entire knowledge base is a single portable file that requires no running server process, aligning with [[concepts/privacy-first-software]] principles and [[concepts/phi-data-handling]] requirements for clinical data.

## Comparison with SQLite

DuckDB complements [[concepts/sqlite-as-vector-store]] as an alternative embedded store. DuckDB's columnar, analytical orientation gives it an edge for batch embedding ingestion and complex analytical queries, while SQLite is more universally available and simpler for small key-value retrieval patterns.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — The broader pipeline DuckDB supports
- [[concepts/vector-search]] — The similarity search mechanism
- [[concepts/hybrid-search-retrieval]] — Combined semantic and keyword retrieval
- [[concepts/pai-knowledge-base]] — Specific knowledge base using this approach
- [[concepts/pai-assessment]] — Clinical context driving the RAG system
- [[concepts/parquet-as-knowledge-store]] — Columnar storage used alongside DuckDB
- [[concepts/local-llm-inference]] — Embedding model inference powering vector creation
- [[concepts/privacy-first-software]] — Design philosophy motivating local storage
- [[concepts/phi-data-handling]] — Clinical data handling requirements
- [[concepts/sqlite-as-vector-store]] — Comparable embedded database approach
- [[concepts/llm-provider-abstraction]] — Multi-provider LLM routing used with the RAG system
- [[concepts/fallback-strategy]] — Graceful degradation patterns in the retrieval pipeline
- [[concepts/neuropsychological-assessment-pipeline]] — Broader pipeline context for cingulate's DuckDB use
- [[concepts/r6-class-architecture]] — The class design pattern used by `DuckDBProcessorR6`
- [[summaries/CLAUDE]] — Cingulate package guide describing `DuckDBProcessorR6` and the staging pipeline
- [[summaries/TECHNICAL_DOCS]] — Technical implementation details of the PAI RAG system
- [[summaries/REBUILD_COMPLETE]] — Record of the knowledge base rebuild
- [[summaries/REBUILD_FINAL_STATUS]] — Final status of the rebuild
- [[summaries/QUICK_REFERENCE]] — Operational reference for the PAI RAG system
- [[summaries/KNOWLEDGE_BASE_EXPLAINED]] — Deeper explanation of the knowledge base design
- [[summaries/README_PIPELINE]] — Pipeline architecture overview
- [[summaries/README_WORKFLOW]] — Workflow documentation
- [[summaries/README]] — Project readme
- [[summaries/conversation-export]] — End-to-end hands-on build session with debugging details
- [[summaries/WORKFLOW_INSTRUCTIONS]]
- [[summaries/neuropsych-data-extractor]]
- [[summaries/vector-store]]
- [[summaries/DEPENDENCIES]]

See also: [[summaries/SESSION_SUMMARY]]