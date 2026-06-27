---
sources: [summaries/pai_96.md, summaries/pai_316.md, summaries/pai_23.md, summaries/pai_199.md, summaries/pai_102.md, summaries/pai_09.md, summaries/pai_01.md, summaries/processed_files.md, summaries/conversation-export.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/README_AS_PROCESSING.md, summaries/README.md, summaries/QUICK_REFERENCE.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/FIX_EXPLANATION.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/COMPLETE_STATUS.md, summaries/AS_PROCESSING_COMPLETE.md]
brief: Parquet+DuckDB RAG knowledge base of 81–98 PAI documents enabling semantic clinical query and LLM-grounded interpretation.
---

# PAI Knowledge Base for RAG

The PAI Knowledge Base is a structured document corpus purpose-built to support [[concepts/retrieval-augmented-generation]] for interpreting Personality Assessment Inventory (PAI) results. It provides the evidence layer underlying an automated [[concepts/pai-assessment]] interpretation system, enabling semantically relevant clinical content to be retrieved on demand and supplied to a language model for evidence-based report generation.

The system is documented across several reference files: [[summaries/README]] (project overview and quick-start), [[summaries/README_WORKFLOW]] (end-to-end operator workflow including rebuild, score entry, and report generation), [[summaries/WORKFLOW_INSTRUCTIONS]] (complete step-by-step operator guide for the January 2026 rebuild and new patient AS processing), [[summaries/AS_PROCESSING_COMPLETE]] (concrete demonstration of the knowledge base in operation), [[summaries/COMPLETE_STATUS]] (detailed status report on both corpus versions), [[summaries/EMBEDDINGS_COMPLETE]] (record of full embedding generation completed January 29, 2026), [[summaries/KNOWLEDGE_BASE_EXPLAINED]] (architectural investigation clarifying how storage works), [[summaries/REBUILD_COMPLETE]] (record of the January 29, 2026 full rebuild, including issues resolved and test results), and [[summaries/REBUILD_FINAL_STATUS]] (final confirmed status of the rebuilt corpus, embedding state, and next steps for new patient processing). The [[summaries/QUICK_REFERENCE]] document provides a concise operator's cheat sheet for daily use. The [[summaries/TECHNICAL_DOCS]] document provides a detailed architectural and implementation reference for the full RAG system.

The original construction of this system is documented in [[summaries/conversation-export]], which records the step-by-step build session covering document ingestion, chunking, embedding, storage, semantic search, hybrid retrieval, and LLM integration.

---

## Corpus Composition

The knowledge base has expanded significantly through continued ingestion. A rebuild was triggered in January 2026 by a file naming restructure — reports moved to a numbered scheme (`pai_00` through `pai_318`) and source files gained a `pai_source_` prefix — requiring full reprocessing to ensure all current files are properly indexed.

| Property | Initial Version | Current Version |
|----------|----------------|------------------|
| Total documents | 81 | 98 (79 reports + 19 sources) |
| Total text chunks | 2,546 | 4,830 (+89.7%) |
| Total text pages | — | 1,663 |
| Chunk size | ~800 chars + 100 char overlap | 1,000 chars + 100 char overlap |
| Embedding model | nomic-embed-text (768-D) | snowflake-arctic-embed2:568m (1,024-D) |
| Vector dimensions | 768 | 1,024 per vector |
| Primary storage format | Parquet | Parquet |
| Search method | Hybrid (vector + keyword) | ✅ Vector similarity + full-text (hybrid) |
| Database size | ~7 MB total | 18.4 MB (DuckDB) + Parquet files |

Documents are segmented into fine-grained chunks (approximately 49 chunks per document on average) to support granular retrieval. The chunking strategy prioritizes retrieval precision over broad coverage.

The initial 81-document version used `nomic-embed-text` (768-D vectors) and achieved retrieval accuracy of 70–75% top-1, 85–92% top-3, and 92–97% top-5 in clinical testing. The current version upgrades to `snowflake-arctic-embed2:568m` (1,024-D vectors) for improved speed and quality.

---

## Technical Requirements

The system runs in **R 4.0+** and depends on the following packages:

- **tidyverse** — data wrangling
- **arrow** — Parquet read/write
- **duckdb** — ad-hoc query interface
- **httr2** — HTTP calls to LLM APIs
- **ragnar** v2 — RAG store management (using `ragnar_store_ingest()` and `markdown_chunk()`)
- **digest** — hash generation for chunk deduplication
- **jsonlite** — embedding vector serialization/deserialization

Local LLM inference via [[concepts/local-llm-inference]] (Ollama) is recommended for privacy-preserving operation. Cloud providers (OpenAI, Claude) are also supported through an OpenAI-compatible API interface via [[concepts/openai-compatible-api]]. See [[summaries/README]] for the full requirements list.

If `ragnar` is not installed, it can be added with `install.packages("ragnar")`. If Ollama is unavailable, the system falls back to BM25 keyword search, which remains effective for most clinical queries.

---

## Build Process: Original Construction

The system was initially built from scratch in a single session (documented in [[summaries/conversation-export]]) following this sequence:

### 1. Document Ingestion
- PDFs read with `ragnar::ragnar_read()` in a `for` loop (one path at a time; the function does not accept vectors)
- `markdown_segment()` applied for initial segmentation
- Combining ragnar S7 objects via `bind_rows()` fails; text must be extracted manually with `as.character()`

### 2. Chunking
A custom `chunk_long_text()` function split each segment at 800 characters with 100-character overlap, producing 2,546 chunks from 81 documents (average ~780 chars per chunk):

```r
chunk_long_text <- function(text, max_chars = 800, overlap = 100) {
  if (nchar(text) <= max_chars) return(text)
  chunks <- character()
  start <- 1
  while (start <= nchar(text)) {
    end <- min(start + max_chars - 1, nchar(text))
    chunks <- c(chunks, substr(text, start, end))
    start <- start + max_chars - overlap
  }
  chunks
}
```

### 3. Embedding
- `embed_ollama(model = "nomic-embed-text")` called on batches of 100 chunks
- Total embedding time: ~30 seconds for 2,546 chunks
- **Critical bug resolved:** initial JSON serialization via `jsonlite::toJSON()` only stored the first element of each 768-D vector. Fix: extract full row with `embedding_matrix[i, ]` (row index) rather than `embedding_matrixi` (list index).

### 4. Storage
Embedding vectors are serialized as JSON strings and stored in Parquet via the arrow package. The resulting files:

- `pai_knowledge_base_chunks.parquet` — text chunks (~0.62 MB)
- `pai_embeddings_complete.parquet` — full 768-D embedding vectors (~6.25 MB)
- `pai_knowledge_base.duckdb` — runtime query interface

---

## Storage Architecture

A key architectural finding — documented in [[summaries/KNOWLEDGE_BASE_EXPLAINED]] and confirmed in [[summaries/REBUILD_FINAL_STATUS]] — is that the system does **not** rely on persistent `.duckdb` files for storage. The ragnar R package creates DuckDB instances in temporary directories or in memory, making them ephemeral. **The `.duckdb` file that appeared "missing" never existed as a required persistent artifact.** The true persistence layer is [[concepts/parquet-as-knowledge-store]].

This is by design and is actually advantageous: Parquet files are more portable, easier to back up and version, and resistant to database corruption issues.

### How It Works

1. **Text Storage** — Chunked text is stored in `pai_knowledge_base_chunks.parquet` (columnar Parquet format, efficient and portable).
2. **Embedding Storage** — Vector embeddings are stored separately in `pai_embeddings_complete.parquet` as JSON-serialized arrays.
3. **Search Process** — At query time, embeddings are loaded into memory, cosine similarity is computed against the query vector, and top-matching chunks are returned.
4. **DuckDB** — Used for ad-hoc complex queries at runtime via `ragnar_store_connect()` and `ragnar_retrieve()`; not the primary persistent store. Direct SQL queries replace the deprecated `ragnar_store_inspect()` function.

This separation of text chunks and embeddings into distinct [[concepts/parquet-as-knowledge-store]] files is well-suited for this workload: efficient columnar reads, easy versioning, and straightforward backup. The [[concepts/duckdb-as-vector-store]] pattern provides a lightweight in-process SQL layer for complex query operations at runtime without requiring a standalone database server.

### Key Files

| File | Purpose | Status |
|------|---------|--------|
| `pai_knowledge_base.duckdb` | RAG store interface (98 documents, 18.4 MB) | ✅ Connected via ragnar |
| `pai_knowledge_base_chunks.parquet` | Text chunks (1.1 MB) | ✅ Updated (4,830 chunks) |
| `pai_knowledge_base_chunks_NEW.rds` | R native backup (759 KB) | ✅ New |
| `pai_knowledge_base_chunks_OLD.parquet` | Previous version backup (638 KB, Jan 7) | 📦 Archived |
| `pai_embeddings_complete.parquet` | Vector embeddings (6.2 MB) | ⚠️ Stale — needs regeneration |
| `pai_embeddings.parquet` | Older embeddings (272 KB) | ⚠️ Stale |
| `rebuild_pai_ragnar_v2.R` | **Main rebuild script** for all current files | ✅ Ready |
| `rebuild_pai_ragnar.R` | Previous rebuild script | ✅ Ready |
| `test_knowledge_base.R` | Test KB with sample queries | ✅ Ready |
| `input/AS_scores_template.json` | T-score entry template for patient AS | ✅ Ready |
| `generate_as_interpretation.R` | Generate interpretation report for patient AS | ✅ Ready |
| `process_new_patient_complete.R` | Complete automated workflow | ✅ Ready |
| `pai_rag_system.R` | Search functions (`ask_pai_expert()`) | ✅ Ready |
| `pai_score_interpreter.R` | Report generation | ✅ Ready |
| `WORKFLOW_INSTRUCTIONS.md` | Complete step-by-step operator guide | ✅ Ready |

> **Note on embeddings:** As of January 29, 2026, the old embeddings (2,546 chunks, generated January 7) are mismatched with the new corpus (4,830 chunks). Text-based keyword search works immediately; semantic vector search requires regenerating embeddings before use.

---

## System Architecture (RAG Pipeline)

The PAI RAG system operates through three processing layers:

**Layer 1 — Retrieval**: Dual-path retrieval combining semantic cosine similarity search on dense vectors and SQL keyword search, merged via hybrid scoring:
```
hybrid_score = (semantic_score_norm × semantic_weight) +
               (text_score_norm × text_weight)
```
Both scores are normalized to [0, 1] before combination. See [[concepts/hybrid-search-retrieval]] for the general pattern.

**Layer 2 — Context Formatting**: Top-K chunks are retrieved, labeled with source identifiers and relevance scores, then formatted for prompt injection into the LLM via `format_context_for_llm()` and `build_pai_prompt()`.

**Layer 3 — LLM Integration**: Provider routing is abstracted via [[concepts/llm-provider-abstraction]] to support Ollama (local), OpenAI, or Anthropic Claude. All calls use structured prompts with system and user roles. System roles include `"expert neuropsychologist"`, `"clinical supervisor"`, and `"researcher"`.

### Query Performance (Initial 81-Document Version)

| Operation | Latency |
|---|---|
| Semantic search | 500–800 ms |
| Keyword search | 50–100 ms |
| Hybrid search | 600–900 ms |
| LLM call (Ollama) | 2–5 seconds |
| LLM call (OpenAI) | 1–3 seconds |
| End-to-end | 3–8 seconds |

Memory usage: baseline ~50 MB; peak <100 MB during query processing.

### Key R Functions

```r
# Pure vector search
pai_semantic_search(query, top_k = 5, con = con)

# Combined semantic + keyword search
pai_hybrid_search(query, top_k = 5,
                  semantic_weight = 0.6,
                  text_weight = 0.4,
                  con = con,
                  return_full_text = FALSE)

# Format retrieved chunks for LLM
format_context_for_llm(retrieved_results, max_chunks = 5)

# Build structured prompt
build_pai_prompt(question, context, system_role = "expert neuropsychologist")

# Route to LLM provider
call_llm(prompt, provider = "ollama", model = NULL, api_key = NULL)

# Complete end-to-end pipeline
ask_pai_expert(question, con, provider = "ollama", model = NULL,
               top_k = 5, semantic_weight = 0.6, text_weight = 0.4)
```

Error handling uses `tryCatch` wrapping all API calls, returning `NULL` on failure with informative messages — a [[concepts/fallback-strategy]] design for graceful degradation.

---

## Retrieval: Semantic and Hybrid Search

### Semantic Search

The `pai_semantic_search()` function embeds the query with the same model used during indexing, normalizes both the query and document vectors, then computes cosine similarity via matrix multiplication:

```r
# Normalize vectors
query_vector <- query_vector / sqrt(sum(query_vector^2))
doc_vectors[i,] <- doc_vec / sqrt(sum(doc_vec^2))

# Compute all similarities at once
similarities <- as.numeric(doc_vectors %*% query_vector)
```

Validated similarity scores for the initial corpus: 0.68–0.74 for clinically relevant matches. Scores below ~0.40 typically indicate poor relevance.

### Hybrid Search

The `pai_hybrid_search()` function layers keyword scoring on top of semantic scoring:

1. Extract keywords from query (strip stop words)
2. Run SQL `LIKE` search counting matches per chunk
3. Normalize both score types to [0, 1]
4. Compute `hybrid_score = semantic_norm × semantic_weight + text_norm × text_weight`

**Recommended weight configurations:**

| Use Case | Semantic Weight | Text Weight |
|---|---|---|
| Conceptual questions ("What causes...") | 0.7–0.9 | 0.1–0.3 |
| General clinical queries | 0.5 | 0.5 |
| Specific scale names / acronyms | 0.2–0.4 | 0.6–0.8 |

### Clinical Testing Results (81-Document Version)

| Scenario | Top Hybrid Score | Result Quality |
|---|---|---|
| DEP vs ANX differential diagnosis | 0.600 | Excellent |
| ICN/INF validity cutoffs | 0.400 | Very Good |
| Treatment prognosis indicators | 0.700 | Excellent |
| BOR scale interpretation | 0.500 | Good |
| Complex comorbid case (DEP+ANX, normal BOR) | 0.650 | Excellent |

---

## LLM Integration

The system supports three LLM providers through a common `call_llm()` interface:

| Provider | Function | Default Model | Notes |
|---|---|---|---|
| Ollama | `call_ollama()` | `llama3.2` | Local, free, private |
| OpenAI | `call_openai()` | `gpt-4o-mini` | Requires `OPENAI_API_KEY` |
| Anthropic | `call_anthropic()` | `claude-3-5-sonnet-20241022` | Requires `ANTHROPIC_API_KEY` |

API keys are read from environment variables if not passed directly:
```r
Sys.setenv(OPENAI_API_KEY = "sk-...")
Sys.setenv(ANTHROPIC_API_KEY = "sk-ant-...")
```

All providers receive the same structured prompt containing a system message (clinical role) and a user message (question + retrieved context). Temperature defaults to 0.3 for consistent, factual responses.

---

## Common Operations

### Rebuild the Knowledge Base

After adding new files or following a restructure, run either of the following (v2 is preferred):
```r
# Preferred (v2 API)
source("/Users/joey/rag/pai/rebuild_pai_ragnar_v2.R")

# Also supported
source("/Users/joey/rag/pai/rebuild_pai_ragnar.R")
```
This removes the old database, re-scans the `reports/` and `source/` folders, ingests all PDFs, generates embeddings with snowflake-arctic, and builds FTS + VSS indexes. Estimated time: 5–10 minutes for ~100 documents.

### Query the Knowledge Base
```r
library(ragnar)
store <- ragnar_store_connect("pai_knowledge_base.duckdb")
results <- ragnar_retrieve(store, text = "your question here", top_k = 5L)
```

### Keyword Search (Immediately Available)
```r
library(arrow)
chunks <- read_parquet("pai_knowledge_base_chunks.parquet")

results <- chunks |>
  filter(str_detect(chunk_text, regex("mania", ignore_case = TRUE)))
```

### Process a New Patient

The workflow is illustrated by the processing of patient **Alessandra Snavely (AS)**, tested January 14, 2026, as documented in [[summaries/WORKFLOW_INSTRUCTIONS]]:

1. **Extract T-scores** from the patient's PAI report PDF (pages 3–4, bar graph profiles). If a Summary Table PDF (e.g., `AS_SummaryTable.pdf`) is available in `input/`, use it directly; otherwise read values manually from the graphical profiles.
2. **Populate** `input/AS_scores_template.json` with validity, clinical, treatment, and interpersonal scale scores.
3. **Run** `source("generate_as_interpretation.R")` to produce the interpretation report (output saved to `output/AS_PAI_Interpretation_YYYYMMDD.txt`).
4. Multiple patients can be processed independently by creating separate JSON score files.

The PAI scales extracted span four domains:

| Domain | Scales |
|---|---|
| **Validity** | ICN, INF, NIM, PIM |
| **Clinical** | SOM, ANX, ARD, DEP, MAN, PAR, SCZ, BOR, ANT, ALC, DRG |
| **Treatment** | AGG, SUI, STR, NON, RXR |
| **Interpersonal** | DOM, WRM |

See [[concepts/pdf-score-extraction]] for the broader methodology of pulling T-scores from PDF reports.

### Quick Start Commands
```r
# Step 1: Rebuild knowledge base
source("/Users/joey/rag/pai/rebuild_pai_ragnar.R")

# Step 2: Fill in scores in input/AS_scores_template.json

# Step 3: Generate interpretation
source("/Users/joey/rag/pai/generate_as_interpretation.R")
```

### Troubleshooting
- **KB not found:** `setwd("/Users/joey/rag/pai")`
- **Ollama not running:** `ollama serve` in terminal
- **Missing model:** `ollama pull snowflake-arctic-embed2:568m`
- **ragnar not installed:** `install.packages("ragnar")`
- **Can't read T-scores from graphs:** Look for a companion Summary Table PDF (e.g., `AS_SummaryTable.pdf`), or contact the testing source.
- **Store version mismatch:** Use `rebuild_pai_ragnar_v2.R` which employs the simplified ragnar v2 high-level API.
- **Semantic search not working:** Regenerate embeddings — old embeddings from January 7 are mismatched with the January 29 corpus.
- **Ollama 404 on `/api/generate`:** Model name mismatch or model not pulled; verify with `ollama list` and pull as needed.
- **All similarity scores identical:** Embedding vector serialization bug — ensure `embedding_matrix[i, ]` row indexing is used, not `i` list indexing.

See also: [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]], [[summaries/QUICK_REFERENCE]], [[summaries/FIX_EXPLANATION]]

---

## Search Architecture

The system supports two complementary retrieval modes:

- **BM25 / Full-Text Search (FTS):** Always available with no external dependencies. Functional as a standalone retrieval method. Works immediately with the updated 4,830-chunk corpus.
- **Vector Similarity Search (VSS) / semantic search:** Requires Ollama to be running locally and up-to-date embeddings matched to the current corpus. Adds semantic retrieval depth beyond exact keyword matching — e.g., finding passages about "elevated mania" when querying "high MAN score".

The system is fully operational with BM25 alone; Ollama enhances but is not required for basic use. This [[concepts/fallback-strategy]] design ensures robustness in clinical operator environments. See [[concepts/vector-search]] for the underlying similarity computation and [[concepts/hybrid-search-retrieval]] for the combined scoring approach.

---

## Scalability

The architecture scales gracefully as the corpus grows:

| Scale | Chunks | Approach |
|---|---|---|
| Current | 4,830 | DuckDB + Parquet |
| 10× | ~25,000 | No code changes needed |
| 100× | ~250,000 | Add approximate nearest neighbor (ANN) search (FAISS/Annoy) |
| 1,000×+ | 2.5M+ | Specialized vector database (Pinecone, Weaviate), sharding |

Optimization opportunities include query embedding caching, parallel embedding generation via `furrr`, and precomputed inverted indexes to replace SQL `LIKE` keyword search.

---

## Build History and Resolved Issues

The knowledge base has evolved through two major versions.

### OLD Version (Pre-January 29, 2026)
- **Created:** January 7–13, 2026
- **Storage:** `pai_knowledge_base_copy.duckdb` (17.5 MB) + `pai_embeddings.parquet` (272 KB) + `pai_embeddings_complete.parquet` (6.2 MB)
- **Documents:** ~50–81 PDFs
- **Chunks:** ~2,546
- **Embedding model:** nomic-embed-text (768-D)
- **Status:** Archive/backup — preserved as `pai_knowledge_base_chunks_OLD.parquet` (638 KB)

### NEW Version — Active (January 29, 2026)
- **Created:** January 29, 2026
- **Trigger:** File naming restructure (numbered system + `pai_source_` prefix); also prompted by onboarding new patient Alessandra Snavely (AS)
- **Documents:** 98 PDFs (79 reports + 19 sources)
- **Pages:** 1,663 total pages extracted
- **Chunks:** 4,830 (stored in `pai_knowledge_base_chunks.parquet`, 1.1 MB; +89.7% from old version)
- **Database size:** 18.4 MB (`pai_knowledge_base.duckdb`)
- **Embeddings:** ⚠️ Stale — old embeddings (2,546 chunks) do not match new corpus (4,830 chunks); regeneration required for semantic search
- **Improvement:** ~90% more source documents, better chunking strategy, full vector search capability, hybrid search (vector similarity + full-text)

**Technical issues resolved during the rebuild:**

| Problem | Solution Applied |
|---|---|
| Wrong embedding model (`embeddinggemma` not found) | Explicitly configured `snowflake-arctic-embed2:568m` |
| Incorrect ragnar API usage (wrong function arguments) | Switched to ragnar v2: `ragnar_store_ingest()`, `markdown_chunk()` |
| Store version mismatch errors | Simplified workflow with high-level functions |
| Interactive Shiny app blocking script completion | Replaced `ragnar_store_inspect()` with direct SQL queries |
| Assumed persistent .duckdb file | Confirmed architecture: Parquet is the real persistence layer; DuckDB is in-memory only |
| `bind_rows()` fails on ragnar S7 objects | Used `for` loop with manual `as.character()` extraction |
| Embedding vectors stored as single values | Fixed row indexing: `embedding_matrix[i, ]` not `i` |

**Test queries validated after rebuild:**
- Elevated Mania (MAN) scale interpretation
- High Dominance (DOM) scale meaning
- Validity scale assessment criteria
- Treatment planning considerations

**Clinical validation scenarios** (from original 81-document build, documented in [[summaries/conversation-export]]):
- ✅ Differential diagnosis (DEP vs ANX)
- ✅ Validity assessment (ICN/INF cutoffs)
- ✅ Treatment prognosis
- ✅ Scale interpretation (BOR)
- ✅ Complex comorbidity cases

See also: [[summaries/REBUILD_COMPLETE]], [[summaries/REBUILD_FINAL_STATUS]], [[summaries/FIX_EXPLANATION]], [[summaries/EMBEDDINGS_COMPLETE]]

---

## Embedding Generation

New chunks require embeddings generated via the `snowflake-arctic-embed2:568m` model through the Ollama [[concepts/local-llm-inference]] server, using the ragnar and arrow R libraries. This model was selected over `nomic-embed-text` for its superior speed while maintaining quality. The process typically takes 30–60 minutes for the full 4,830-chunk corpus.

**Current embedding state:** The embeddings on disk (`pai_embeddings_complete.parquet`, 6.2 MB) were generated for the old 2,546-chunk corpus (January 7). They are mismatched with the current 4,830-chunk corpus. Semantic search will not work correctly until new embeddings are generated. Text keyword search is unaffected and works immediately.

To regenerate embeddings:
```r
library(tidyverse)
library(arrow)
library(ragnar)

chunks <- read_parquet("pai_knowledge_base_chunks.parquet")

# Generate embeddings (takes 30-60 minutes)
embedded <- chunks |>
  mutate(
    embedding = embed_ollama(
      chunk_text, 
      model = "snowflake-arctic-embed2:568m"
    )
  )

write_parquet(embedded, "pai_embeddings_complete_NEW.parquet")
```

The use of locally-run embedding models supports privacy-first design — no clinical text is sent to external APIs during indexing or retrieval. This aligns with [[concepts/phi-data-handling]] requirements for patient assessment data.

---

## Security and Privacy

The knowledge base is designed for use with sensitive clinical assessment data:

- **Local processing (Ollama):** All data stays on the local machine; no external API calls; compatible with HIPAA requirements if properly configured; no usage tracking.
- **Cloud APIs (OpenAI/Anthropic):** Data is sent to third-party servers and is subject to provider terms of service. Institutional data policies must be verified before use.

**Recommendation:** Use Ollama for all processing involving real patient data. Cloud APIs are appropriate only for testing with de-identified content.

This privacy-first design is consistent with [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]] principles underlying the broader clinical neuropsychology toolchain.

---

## Integration with PAI Assessment Pipeline

The knowledge base feeds directly into the neuropsychological assessment pipeline and [[concepts/neuropsychological-reporting]] workflows. Retrieved chunks provide evidence-grounded context for interpreting PAI scale scores, supporting the generation of clinical report structure-compliant narrative text. The `ask_pai_expert()` function in `pai_rag_system.R` wraps retrieval and LLM generation into a single callable interface for clinical workflows.

Text-based keyword search is available immediately with the updated corpus; semantic vector search requires up-to-date embeddings and a running Ollama instance.

After generating a report, the recommended post-processing steps are: review and validate the AI-generated interpretation, add clinician notes, export to a preferred format (Word, PDF, etc.), and archive in the patient records system.

The system is designed for educational and research purposes in clinical neuropsychology, enabling clinicians and researchers to query PAI assessment literature efficiently and produce evidence-cited AI responses.

See also: [[summaries/README_AS_PROCESSING]], [[summaries/README_PIPELINE]], [[summaries/README_WORKFLOW]], [[summaries/TECHNICAL_DOCS]], [[summaries/WORKFLOW_INSTRUCTIONS]]

See also: [[summaries/processed_files]]

See also: [[summaries/pai_01]]

See also: [[summaries/pai_09]]

See also: [[summaries/pai_102]]

See also: [[summaries/pai_199]]

See also: [[summaries/pai_23]]

See also: [[summaries/pai_316]]

See also: [[summaries/pai_96]]