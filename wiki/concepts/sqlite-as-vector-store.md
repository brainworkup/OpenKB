---
sources: [summaries/DEPENDENCIES.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/vector-store.md, summaries/text-extraction.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/report-generator.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/0002-soul-sqlite-vector-storage.md, summaries/README.md, summaries/TECHNICAL_DOCS.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/index.md]
brief: Using SQLite to persist and query vector embeddings locally, avoiding dedicated vector database infrastructure.
---

# SQLite as a Vector Store

SQLite as a vector store is the practice of persisting dense vector embeddings directly inside a standard SQLite database, avoiding the operational complexity of a dedicated vector database such as Pinecone, Weaviate, or Chroma. Similarity search is then performed in application memory rather than inside the database engine itself.

## How It Works

1. **Schema**: A table (commonly named `chunks`) holds the source text alongside its embedding, serialized as a JSON string.

```sql
CREATE TABLE IF NOT EXISTS chunks (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    source_path TEXT NOT NULL,      -- Original file path
    chunk_id    INTEGER NOT NULL,   -- Position within file
    content     TEXT NOT NULL,      -- Chunk text content
    embedding   TEXT NOT NULL,      -- JSON-serialized vector
    UNIQUE(source_path, chunk_id)
);
```

2. **Write path**: After embedding a text chunk, the float32 array is serialized to a compact JSON string via `_serialize_vector()` (using `separators=(",",":")` for minimal size) and inserted into the `embedding` column with `INSERT OR REPLACE`. This means re-running `build-index` on an updated corpus safely upserts new and changed chunks without duplicating existing ones.
3. **Read path**: At query time, all rows are loaded into RAM, embeddings are deserialized with `_deserialize_vector()`, and cosine similarity is computed in pure Python against the query embedding — no numpy or external library required.
4. **Top-k selection**: The k rows with the highest cosine similarity scores are returned as `ChunkRecord` objects (containing `source_path`, `chunk_id`, and `content`) for the [[concepts/retrieval-augmented-generation]] pipeline.

## Core API

The implementation in `soul/neuro_report_style_agent.py` exposes these functions:

| Function | Purpose |
|---|---|
| `init_db(db_path)` | Creates DB and schema idempotently — safe to call multiple times |
| `_serialize_vector(vec)` | Converts float list to compact JSON string |
| `_deserialize_vector(raw)` | Parses JSON string back to float list |
| `cosine_similarity(a, b)` | Pure-Python dot-product / norm computation; returns 0.0 for zero vectors |
| `retrieve_top_k(db_path, query_embedding, k)` | Returns k most similar `ChunkRecord` objects via full O(n) scan |

The `cosine_similarity` implementation uses only Python builtins:

```python
def cosine_similarity(a, b):
    dot = sum(x * y for x, y in zip(a, b))
    norm_a = math.sqrt(sum(x * x for x in a))
    norm_b = math.sqrt(sum(y * y for y in b))
    if norm_a == 0 or norm_b == 0:
        return 0.0
    return dot / (norm_a * norm_b)
```

## Why SQLite?

| Advantage | Detail |
|-----------|--------|
| **Zero infrastructure** | No separate server process; the DB is a single file. |
| **Portability** | Works on any OS without installation beyond the Python standard library. |
| **Simplicity** | Developers can inspect embeddings with any SQLite browser or the `sqlite3` CLI. |
| **Atomic writes** | SQLite's ACID guarantees prevent partial inserts during indexing. |
| **Minimal dependencies** | No third-party vector DB client or network configuration required. |
| **Auto-initialization** | Database and schema initialize automatically at first run via `init_db()`. |
| **Incremental updates** | `INSERT OR REPLACE` supports safe re-indexing as the report corpus grows. |

This approach fits naturally within the [[concepts/single-file-agent-pattern]] philosophy codified in ADR 0008: the entire knowledge base travels as one portable `.sqlite` file alongside the source code, and all state — including embeddings — lives locally without external services.

## Architectural Decisions

### ADR 0002 — Storage Choice

The use of SQLite for vector storage was formally codified in [[summaries/0002-soul-sqlite-vector-storage]], an [[concepts/architecture-decision-records]] entry for the `soul` RAG system. The ADR evaluated three alternatives:

- **Dedicated vector databases** (Chroma, Weaviate, Pinecone) — rejected as overkill given expected scale of hundreds to thousands of historical reports.
- **SQLite with JSON serialization** — selected for zero additional dependencies, ACID compliance, and portability.
- **In-memory only storage** — rejected for lack of persistence.

The decision explicitly acknowledges the O(n) scan limitation and sets a scaling ceiling of approximately 10,000 chunks, where CLI response times remain acceptable at roughly 1–2 seconds.

### ADR 0008 — Single-File Agent Architecture

ADR 0008 ([[summaries/0008-soul-single-file-style-agent-architecture]]) reinforces the SQLite choice within the broader decision to keep `soul/neuro_report_style_agent.py` as a single-file script. SQLite and JSON are explicitly called out as sufficient for the expected local scale, aligning with the project's [[concepts/local-first-architecture]] posture.

The ADR notes that the `build-index`, `train-style`, and `write-report` CLI commands all interact with SQLite-backed storage without requiring any additional infrastructure. Network calls use Python stdlib (`urllib`), and state management relies entirely on local SQLite and JSON files. As documented in [[summaries/soul-style-agent]], the full dependency footprint at runtime is Python ≥ 3.13 stdlib only, with PyPDF2 as an optional addition for PDF extraction.

This pairing — single-file agent plus SQLite vector store — defines a coherent operational unit: `uv sync` plus one script is enough to understand and deploy the full runtime surface.

## Build Index Workflow

```python
def build_index(args):
    init_db(db_path)
    files = sorted(iter_report_files(reports_dir))

    for file_path in files:
        text = extract_text(file_path)
        chunks = chunk_text(text, chunk_size=1200, overlap=150)

        for idx, chunk in enumerate(chunks):
            emb = embed_with_fallback(args, chunk)
            conn.execute(
                "INSERT OR REPLACE INTO chunks(source_path, chunk_id, content, embedding)",
                (str(file_path), idx, chunk, _serialize_vector(emb))
            )
```

Chunking uses fixed-size windows (default 1,200 chars, 150 overlap — see [[concepts/text-chunking]]). Embeddings are produced by ModernBERT (~768 dimensions) via the [[concepts/omlx-server]] `/embeddings` endpoint. The full pipeline documented in [[summaries/full-pipeline]] shows a complete run producing roughly 1,247 chunks from 45 files, verifiable via:

```bash
sqlite3 report_style_index.sqlite "SELECT COUNT(*) FROM chunks;"
# Output: 1247
```

As documented in [[summaries/customization]], chunk size is tunable at indexing time: larger windows (e.g., `--chunk-size 2000 --overlap 200`) suit long-form comprehensive reports, while smaller windows (e.g., `--chunk-size 600 --overlap 75`) are preferable for brief screening reports. For structured reports with clear sections, pre-extracting named sections before indexing allows section metadata to be attached for more targeted retrieval.

## Full Pipeline Integration

The SQLite index is the backbone of a three-stage end-to-end workflow documented in [[summaries/full-pipeline]]:

| Stage | Command | SQLite Role |
|---|---|---|
| `build-index` | Populates `chunks` table with embedded text chunks from PDF/TXT/MD reports | Write |
| `train-style` | Reads top-12 style-exemplar chunks via cosine similarity to a seed query | Read |
| `write-report` | Reads top-k chunks (default 6) relevant to the user's task prompt | Read |

Supported input formats are `.pdf`, `.txt`, and `.md`. The pipeline can be run as individual stages or as a single bash script with `set -e` for fail-fast behavior. A one-shot script example:

```bash
#!/bin/bash
set -e
REPORTS_DIR="./neuropsych_report_pdf_bucket"
DB_PATH="./report_style_index.sqlite"
PROFILE_PATH="./report_style_profile.json"
OUTPUT_PATH="./draft_report.txt"

python soul/neuro_report_style_agent.py build-index \
    --reports-dir "$REPORTS_DIR" --db-path "$DB_PATH"
python soul/neuro_report_style_agent.py train-style \
    --db-path "$DB_PATH" --profile-path "$PROFILE_PATH"
python soul/neuro_report_style_agent.py write-report \
    --db-path "$DB_PATH" --profile-path "$PROFILE_PATH" \
    --prompt "Write a comprehensive summary..." --output "$OUTPUT_PATH"
```

### Incremental Updates

Because `build-index` uses `INSERT OR REPLACE`, adding new reports to the corpus requires only re-running the first stage. The SQLite database does not need to be recreated. Optionally, `train-style` can be re-run afterward to refresh the style profile with the expanded corpus:

```bash
# Re-index after adding new reports
python soul/neuro_report_style_agent.py build-index \
    --reports-dir ./neuropsych_report_pdf_bucket \
    --db-path ./report_style_index.sqlite

# Optionally refresh style profile
python soul/neuro_report_style_agent.py train-style \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json
```

## Customization of Retrieval Parameters

The [[concepts/rag-chunking]] strategy and retrieval depth are both configurable at runtime:

| Mode | `--top-k` | `--temperature` |
|---|---|---|
| High-context (complex cases) | 12 | 0.15 |
| Quick drafting (standard cases) | 4 | 0.25 |
| Balanced (default) | 6 | 0.2 |

Increasing `--top-k` draws more chunks from the SQLite store into the generation context, which can improve draft quality for complex cases at the cost of a slightly larger prompt. The `--top-k` parameter can be overridden freely at the CLI without any schema changes to the database.

Iterative drafting at multiple temperatures is also a documented pattern:

```bash
# Conservative draft
python ... write-report ... --temperature 0.1 --output draft_v1.txt
# Balanced draft
python ... write-report ... --temperature 0.2 --output draft_v2.txt
# Creative draft
python ... write-report ... --temperature 0.3 --output draft_v3.txt
```

### Per-Clinician and Per-Population Indexes

Multiple SQLite databases can coexist for different populations or clinicians. The `--db-path` flag selects the target database at each stage, enabling workflows such as:

- A `smith_index.sqlite` trained exclusively on Dr. Smith's historical reports.
- Separate `pediatric_index.sqlite`, `adult_index.sqlite`, and `forensic_index.sqlite` stores, each paired with a corresponding [[concepts/style-profiles]] JSON file.

This multi-database pattern requires no changes to the SQLite schema or retrieval logic — only the file path differs.

### Large Corpus Handling

For corpora exceeding 1,000 reports, the recommended approach (documented in [[summaries/customization]]) is batch indexing by year:

```bash
for year in 2020 2021 2022 2023 2024; do
    python ... build-index --reports-dir ./reports/$year --db-path ./index_${year}.sqlite
done
python merge_indices.py --inputs ./index_*.sqlite --output ./master_index.sqlite
```

Parallel PDF extraction via GNU `parallel` can further accelerate ingestion for large batches. The O(n) scan limitation means that beyond ~10,000 chunks, query latency will begin to exceed acceptable thresholds.

## Storage Estimates

| Metric | Value |
|---|---|
| Embedding dims | ~768 (ModernBERT) |
| JSON size per embedding | ~5–10 KB |
| 1,000 chunks | ~5–10 MB |
| 10,000 chunks | ~50–100 MB |

## Integration Points

The SQLite store acts as the central hub in the style training pipeline. From [[summaries/style-training-to-report-drafting]]:

| From | To | Mechanism |
|---|---|---|
| `report_style_profile.json` | `write-report` | Style profile injected as JSON system prompt |
| `report_style_index.sqlite` | `write-report` | Top-k cosine-similarity retrieval |
| `draft_report.txt` | `.qmd` sections | Manual copy-paste into Quarto partials |
| `.qmd` sections | PDF | Quarto `{{< include >}}` + render |

The `embed_with_fallback` and `generate_with_fallback` wrappers provide single call-sites for embedding and generation backends, making it straightforward to swap the [[concepts/omlx-server]] backend (or substitute Ollama via `OMLX_URL` environment variable) without touching SQLite interaction code.

See [[summaries/AS_PROCESSING_COMPLETE]], [[summaries/EMBEDDINGS_COMPLETE]], [[summaries/KNOWLEDGE_BASE_EXPLAINED]], and [[summaries/TECHNICAL_DOCS]] for implementation status and details.

## Trade-offs and Limitations

- **Memory pressure**: All embeddings must be loaded into RAM for similarity search. Memory grows as `4 bytes × embedding_dim × number_of_chunks`.
- **No approximate nearest-neighbour (ANN) index**: Unlike dedicated vector databases, SQLite has no HNSW or IVF index, so search time is O(n) — suitable for fewer than ~10,000 chunks.
- **No persistence of query results**: Similarity scores are ephemeral; re-querying requires reloading all embeddings.
- **Single-threaded writes**: SQLite writes are serialized, which is acceptable for local batch indexing workflows.
- **No metadata fields**: No date, tags, or other attributes are stored alongside embeddings, precluding metadata-based filtering.
- **Migration cost**: Moving to a dedicated vector DB later requires a conversion script to export embeddings from SQLite.
- **Reuse ceiling**: Because the SQLite interaction code lives inside a single-file agent, subcomponent reuse is limited until the script is modularized (a signal also noted in ADR 0008).

## Serialization Format

Embeddings are stored as JSON arrays of numbers using `separators=(",",":")` for minimal byte size — human-readable, easily inspectable, and directly loadable with `json.loads()`. This choice aligns with [[concepts/style-profile-extraction]] design decisions that standardize JSON as the project-wide serialization format.

## When to Graduate to a Dedicated Vector DB

Consider migrating when:
- The number of chunks exceeds ~10,000 and RAM becomes a bottleneck.
- Query latency exceeds ~5 seconds.
- Sub-millisecond ANN search latency is required.
- Multi-user concurrent access is needed.
- Filtering by metadata (date, author, document type) must happen inside the search layer.
- The single-file agent itself is split into modules (a signal tracked in ADR 0008).

Alternative stores explored in related projects include [[concepts/lancedb-vector-store]], [[concepts/duckdb-as-vector-store]], and [[concepts/parquet-as-knowledge-store]].

## Best Practices

From the full pipeline documentation:

1. **Report quality**: Better input reports produce better style profiles — minimum 20–30 reports recommended.
2. **Chunk review**: Spot-check indexed chunks via the `sqlite3` CLI to verify extraction quality.
3. **Profile review**: Always review `report_style_profile.json` before using it for generation.
4. **Clinician review**: Never use generated drafts without clinician review; use `[NEEDS DATA]` as a signal for hallucinations.
5. **Versioning**: Keep style profiles and SQLite databases versioned together for reproducibility.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — The primary consumer of vector similarity search.
- [[concepts/vector-search]] — The broader pattern of embedding-based similarity search that SQLite implements in pure Python.
- [[concepts/local-llm-inference]] — The embedding model runs locally via the same OMLX server.
- [[concepts/omlx-server]] — Provides both embedding and generation endpoints consumed by the agent.
- [[concepts/single-file-agent-pattern]] — SQLite fits naturally into single-file, zero-config deployment as codified in ADR 0008.
- [[concepts/local-first-architecture]] — SQLite and JSON are the canonical local-first storage choices for this project.
- [[concepts/style-profile-extraction]] — Soul profile generation also queries the SQLite vector store.
- [[concepts/text-chunking]] — Fixed-size windowed chunking feeds the build-index pipeline.
- [[concepts/rag-chunking]] — Configurable chunk size and overlap tuned per report type.
- [[concepts/style-profiles]] — Per-clinician and per-population profiles paired with dedicated SQLite indexes.
- [[concepts/python-project-structure]] — The SQLite file is a first-class project artifact alongside source code.
- [[concepts/architecture-decision-records]] — ADR 0002 and ADR 0008 formally document the storage and agent-architecture decisions.

See also: [[summaries/full-pipeline]], [[summaries/vector-store]], [[summaries/soul-style-agent]], [[summaries/README]], [[summaries/report-generator]], [[summaries/style-trainer]], [[summaries/text-extraction]], [[summaries/style-training-to-report-drafting]], [[summaries/customization]]

See also: [[summaries/DEPENDENCIES]]