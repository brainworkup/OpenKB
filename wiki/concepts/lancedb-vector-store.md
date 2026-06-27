---
sources: [summaries/DEPENDENCIES.md, summaries/vector-store.md, summaries/README.md]
brief: LanceDB is an embedded, file-local vector database used for semantic retrieval in the Luria neuropsych pipeline.
---

# LanceDB as Local Vector Store

LanceDB is an embedded, serverless vector database that stores data as files on disk, making it a natural fit for privacy-sensitive, local-first applications. Unlike cloud-hosted vector services, LanceDB runs entirely in-process, requiring no separate server and leaving data under the operator's direct control.

## Role in the Luria Neuropsych Pipeline

In the Luria Streamlit app, LanceDB serves as the semantic retrieval layer in a dual-store architecture:

- **Path**: `data/vectors/` — created at runtime and gitignored.
- **Table**: `narrative_chunks` — stores vector embeddings of narrative text extracted from neuropsychological PDF reports.
- **Purpose**: Enables semantic (similarity) search over unstructured clinical narrative text during RAG Q&A.

This complements the structured relational store (SQLite at `data/neuropsych.db`), which holds `Documents`, `TestScores`, and `ClinicalSummaries`. Together they form a [[concepts/hybrid-search-retrieval]] pattern: SQL filtering for precise score lookup, semantic search for open-ended narrative queries.

An additional LanceDB-backed SQLite index (`report_soul_index.sqlite`, `chunks` table) serves the **Voice / Soul** subsystem, storing style exemplar vectors for report tone matching — a distinct use case from the main narrative retrieval store.

## Integration with the Streamlit App

The README confirms that the Streamlit desktop UI exposes a four-tab interface: Ingest, Ask, Knowledge Base, and Audio. Within this architecture:

- **Ingest tab**: The 4-stage LangGraph pipeline (parse → extract → index → report) writes vector embeddings into LanceDB during the **Index** stage. The `Sheets_Data_Indexer` subagent (`index_node`) is responsible for all SQLite and LanceDB writes, operating without an LLM call — writes are deterministic and data-driven.
- **Ask tab**: The RAG Q&A interface queries LanceDB for semantically similar narrative chunks alongside SQL filtering over `TestScores`, then passes retrieved context to the LLM for answer generation.
- **Knowledge Base tab**: Browsing is handled primarily through SQLite structured tables; LanceDB underpins the semantic search powering the Ask interface.

This mirrors the standard [[concepts/retrieval-augmented-generation]] retrieval pattern and is orchestrated via [[concepts/langgraph-agent-workflows]].

## Why LanceDB for Local-First Clinical Software

### Privacy & PHI Compliance
Because LanceDB is embedded and file-local, no vector data ever leaves the machine. This directly supports [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]] goals — critical when the underlying documents contain real patient health information. PHI redaction happens locally (via Docling) before any text is sent to Anthropic, and neither vector embeddings nor their source texts are transmitted to cloud vector services. The README explicitly confirms: "No cloud vector store: LanceDB and SQLite are entirely local."

### No Infrastructure Overhead
There is no server process to manage, no cloud subscription, and no network dependency for search queries. This aligns with the [[concepts/local-first-architecture]] philosophy of the broader stack, where embeddings, summarization, and retrieval all run on the clinician's own hardware.

### Concurrency Considerations
LanceDB uses file-level locking. A known failure mode — **LanceDB lock errors** — can occur when multiple processes attempt concurrent access to `data/vectors/`. The README troubleshooting table identifies this explicitly: the fix is to close competing processes using `data/vectors/` before restarting the app.

## Embedding Models

Two configured embedding backends feed vector data into LanceDB:

| Model | Dim | Config Key | Notes |
|---|---|---|---|
| `nomicai-modernbert-embed-base-bf16` | 768 | `OMLX_EMBEDDING_MODEL` | Local oMLX server — preferred for PHI safety |
| `sentence-transformers/all-MiniLM-L6-v2` | 384 | `VECTOR_EMBEDDING_MODEL` | Fallback via `sentence-transformers` package |

The higher-dimensional oMLX model (768-dim) is the default when the local [[concepts/omlx-server]] is running. The `sentence-transformers` package provides the fallback path when oMLX is unavailable. Both routes support [[concepts/llm-provider-abstraction]] across local and cloud backends, and both feed their output into the `narrative_chunks` LanceDB table.

## Optional Alternative: Milvus

The `langchain-milvus ≥0.3.3` package is listed as an **optional** dependency, providing an alternative vector store integration via LangChain. LanceDB remains the default local store; Milvus would be relevant for deployments requiring a dedicated vector database server.

## Comparison to Alternatives

| Store | Type | Location | Use in App |
|-------|------|----------|------------|
| LanceDB | Vector (semantic) | Local files | Narrative chunk retrieval; Soul style exemplars |
| SQLite | Relational | Local file | Test scores, summaries, documents |
| [[concepts/duckdb-as-vector-store]] | Analytical + vector | Local (`kb/store/`) | Analytics queries for RAG |
| [[concepts/sqlite-as-vector-store]] | Relational + vector ext | Local | Alternative pattern |

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — the RAG pattern LanceDB enables
- [[concepts/hybrid-search-retrieval]] — combining LanceDB semantic search with SQLite SQL filtering
- [[concepts/phi-data-handling]] — privacy rationale for local-only vector storage
- [[concepts/privacy-first-software]] — design philosophy of the Luria stack
- [[concepts/local-first-architecture]] — the broader principle of keeping all data and computation on-device
- [[concepts/local-llm-inference]] — local embedding generation feeding LanceDB
- [[concepts/neuropsychological-assessment-pipeline]] — the broader pipeline context
- [[concepts/vector-search]] — the underlying search mechanism
- [[concepts/clinical-data-management]] — structured + vector storage together
- [[concepts/llm-provider-abstraction]] — switching between oMLX and fallback embedding providers
- [[concepts/omlx-server]] — the local LLM server providing 768-dim embeddings
- [[concepts/subagent-architecture]] — the Sheets_Data_Indexer subagent that performs LanceDB writes
- [[concepts/docling-pdf-parsing]] — the parse stage that precedes LanceDB indexing
- [[summaries/README_PIPELINE]] — pipeline architecture details
- [[summaries/README_luria]] — related Luria project documentation
- [[summaries/vector-store]] — vector store implementation notes
- [[summaries/DEPENDENCIES]] — full dependency and configuration reference
- [[summaries/README]] — Luria Streamlit App overview and architecture
