---
sources: [summaries/README.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md]
brief: FAISS high-performance vector index for nearest-neighbor search, used as the retrieval core in clinical RAG pipelines.
---

# FAISS Vector Index

FAISS (Facebook AI Similarity Search) is a high-performance library for similarity search and clustering of dense vectors. In RAG and clinical NLP systems, FAISS serves as the core [[concepts/vector-search]] engine, enabling fast nearest-neighbor lookup over large collections of embedded text chunks.

## How FAISS Works

FAISS operates by storing high-dimensional vectors in an index structure optimized for similarity queries. Given a query vector, it returns the k most similar stored vectors according to a distance or similarity metric. The primary variant used in these systems is **IndexFlatIP** — a flat (brute-force) index using **inner product (IP)** similarity, which is equivalent to cosine similarity when vectors are L2-normalized.

### Core Operations

| Operation | Method | Description |
|-----------|--------|-------------|
| Insert | `index.add()` | Add embedding vectors to the index |
| Search | `index.search()` | Retrieve top-k nearest neighbors |
| Persist | `vs.save()` | Serialize index to disk |

## Use in the Autism RAG System

See [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] for full architectural detail. FAISS underpins the research document pipeline:

- Ingested PDF/EPUB documents are chunked into 800-word segments with 120-word overlap (see [[concepts/text-chunking]] and [[concepts/rag-chunking]])
- Each chunk is embedded into a **384-dimensional vector** using the `all-MiniLM-L6-v2` SentenceTransformer model
- Vectors are inserted via `VectorStore.add_embeddings()` → `index.add()` at `src/retrieval.py:49`
- At query time, `FAISS IndexFlatIP.search()` is called at `src/retrieval.py:77`

## Use in the Neuropsychological Report RAG Pipeline

See [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]] and [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] for architectural detail. FAISS serves as the vector index for the clinical recommendation retrieval system, described across six pipeline traces.

### Ingestion (Trace 1)

The full ingestion workflow runs: PDF text extraction via PyMuPDF → report structure parsing → PHI de-identification → embedding generation → FAISS index population → persistence.

- De-identified neuropsychological report recommendation chunks are embedded by a SentenceTransformer model and added to the FAISS index via `VectorStore.add_embeddings()` at `ingest_recommendations.py:202`
- The FAISS index is populated at `retrieval.py:49` and persisted at `ingest_recommendations.py:205`
- Stored vectors carry rich metadata: DSM-5 diagnoses (with ICD-10 codes and canonical names from [[concepts/icd10-diagnosis-extraction]]), age groups, clinical context, and recommendation subsection headers

### Retrieval (Trace 5)

At query time:

1. The user's search query is encoded with the same SentenceTransformer model (`app_recommendations.py:214`)
2. The persisted FAISS index is loaded from disk at `app_recommendations.py:213`
3. `VectorStore.search_filtered()` at `retrieval.py:89` runs the filtered search
4. Additional post-filtering by disorder diagnosis list occurs at `app_recommendations.py:230`

This pipeline is tightly integrated with the [[concepts/phi-deidentification-pipeline]] (Trace 3), which must complete before any recommendation chunks enter the FAISS index. It is also connected to [[concepts/recommendation-rag-pipeline]], the overarching clinical recommendation retrieval and generation system.

### Diagnosis Metadata (Trace 2)

A distinguishing feature of this pipeline is the richness of metadata stored alongside FAISS vectors. The diagnosis parser in `src/report_parser.py` handles six different code format patterns (name-first, codes-first, ICD-10 only, etc.) and canonicalizes diagnosis names across 12 DSM-5 categories. This normalized diagnosis metadata — including ICD-10 codes and canonical aliases — is attached to each FAISS-indexed recommendation chunk and used at retrieval time for metadata filtering. See [[concepts/dsm5-diagnosis-normalization]] for the normalization approach.

## Filtered Search (Over-Fetch Strategy)

A key design pattern shared across both pipelines is `search_filtered()` at `src/retrieval.py:89`. Because FAISS itself does not natively support metadata filtering, the system uses an **over-fetch strategy**:

1. Retrieve **k × 3** candidate results from FAISS (`retrieval.py:105`)
2. Apply post-hoc metadata filters: list membership checks (e.g., diagnoses at `retrieval.py:122`) and equality checks (e.g., age group at `retrieval.py:128`)
3. Return the top-k results that pass all filters

In the neuropsychological report pipeline, an additional post-filter step is applied in `app_recommendations.py:230` for disorder selection. This approach avoids under-retrieval that would occur if filters were applied to only k initial results, and relates to the broader pattern of [[concepts/hybrid-search-retrieval]], where dense retrieval is combined with structured filtering.

## Embedding Compatibility

FAISS indexes are tightly coupled to the embedding model that produced the stored vectors. In both the autism RAG and neuropsychological report pipelines, all vectors are **384-dimensional outputs** from `all-MiniLM-L6-v2`. The query must be encoded with the same model as the indexed chunks; the index must be rebuilt if the embedding model changes. See [[concepts/retrieval-augmented-generation]] for how the index fits into the broader RAG pipeline, and [[concepts/sentence-transformer-embeddings]] for model details.

## Persistence and Decoupling

The FAISS index is serialized to disk after ingestion (`ingest_recommendations.py:205`, `retrieval.py:221`), enabling the index to persist across sessions without re-ingesting all documents. At search time, the persisted index is reloaded on demand (`app_recommendations.py:213`). This separation of ingestion and retrieval is critical for clinical pipelines where ingestion is a slow, batch-oriented process and recommendations must be surfaced with low latency at query time.

## Comparison to Alternatives

FAISS is one of several vector store backends used across related systems in this knowledge base:

- [[concepts/lancedb-vector-store]] — columnar, supports native metadata filtering
- [[concepts/duckdb-as-vector-store]] — SQL-native with vector extensions
- [[concepts/sqlite-as-vector-store]] — lightweight, file-based option

FAISS IndexFlatIP is optimal for moderate-scale collections where brute-force search is fast enough and maximum recall is required, but it lacks native metadata filtering and requires custom over-fetch workarounds as described above.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — FAISS is the retrieval backbone of RAG
- [[concepts/vector-search]] — Broader concept of similarity-based lookup
- [[concepts/text-chunking]] — Determines the granularity of indexed units
- [[concepts/rag-chunking]] — Chunking strategy specifics (size, overlap)
- [[concepts/hybrid-search-retrieval]] — Combining dense retrieval with metadata filters
- [[concepts/phi-deidentification-pipeline]] — Applied before clinical chunks enter the index
- [[concepts/clinical-data-privacy]] — Motivation for de-identification before indexing
- [[concepts/neuropsychological-assessment-pipeline]] — Clinical context for the indexed reports
- [[concepts/recommendation-rag-pipeline]] — Full pipeline for recommendation retrieval and generation
- [[concepts/icd10-diagnosis-extraction]] — Diagnosis metadata stored alongside FAISS vectors
- [[concepts/dsm5-diagnosis-normalization]] — Canonicalization of diagnosis names and DSM-5 category assignment
- [[concepts/sentence-transformer-embeddings]] — Embedding model used for all indexed vectors

See also: [[summaries/README]]