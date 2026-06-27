---
sources: [summaries/README.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md]
brief: Using SentenceTransformer models to encode clinical text into dense vectors for semantic retrieval.
---

# Sentence Transformer Embeddings for Clinical NLP

Sentence Transformer embeddings convert natural language text into fixed-length dense vectors that encode semantic meaning. In clinical NLP systems, these embeddings enable retrieval of thematically related content — such as neuropsychological recommendations — even when exact keyword matches are absent. This approach is central to [[concepts/retrieval-augmented-generation]] workflows for clinical documents.

## What Are Sentence Transformer Embeddings?

Sentence Transformers are a family of transformer-based models fine-tuned with contrastive objectives (e.g., siamese/triplet networks) to produce semantically meaningful sentence-level embeddings. Unlike token-level embeddings from base transformers, sentence-level representations allow direct comparison between full passages via cosine or inner product similarity.

Key properties relevant to clinical NLP:
- **Semantic density**: Similar clinical concepts (e.g., "working memory deficit" and "executive function impairment") cluster in vector space even without shared vocabulary.
- **Fixed-length output**: Enables storage in flat vector indexes like [[concepts/faiss-vector-index]].
- **Batch encodability**: Entire corpora of recommendation chunks can be encoded in a single pass.

## Role in the Neuropsychological RAG Pipeline

In the system described in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]], Sentence Transformer embeddings are used at two stages:

### 1. Ingestion-Time Encoding

After de-identified recommendation chunks are extracted from neuropsychological PDF reports, each chunk is passed to `generate_embeddings()` in `src/embeddings.py`:

```python
model.encode(texts)  # embeddings.py:19
```

The resulting vectors are normalized and stored in a FAISS index via `VectorStore.add_embeddings()`. This index is persisted to disk for repeated retrieval without re-encoding.

### 2. Query-Time Encoding

When a clinician submits a search query in the Shiny app, the same SentenceTransformer model encodes the query string:

```python
SentenceTransformer.encode(query)  # app_recommendations.py:214
```

Crucially, the **same model** must be used for both ingestion and query encoding to ensure vector space alignment. Mismatched models would produce incomparable representations.

## Similarity Search Mechanism

The system uses **cosine similarity via inner product on L2-normalized vectors** — a standard optimization that converts cosine similarity computation into efficient inner product operations supported natively by FAISS.

The retrieval flow:
1. Query vector is compared against all stored chunk vectors.
2. Top-k×3 candidates are over-fetched to allow post-filtering by metadata (age group, context, diagnosis).
3. Filtered results are returned ranked by semantic similarity.

See [[concepts/faiss-vector-index]] and [[concepts/vector-search]] for details on the indexing and search infrastructure.

## Why Sentence Transformers for Clinical Recommendations?

Clinical recommendations in neuropsychological reports exhibit high lexical variation — the same intervention may be described differently across clinicians, report formats, and diagnostic contexts. Sentence Transformer embeddings handle this gracefully:

- A query for "reading support strategies" can retrieve chunks mentioning "literacy interventions," "phonological decoding accommodations," or "specialized reading instruction."
- Diagnosis-specific language (e.g., ASD, ADHD, [[concepts/executive-function-deficits]]) clusters semantically, enabling retrieval even when exact diagnostic terms differ.

This is especially valuable given the multi-format nature of [[concepts/neuropsych-report-parsing]] inputs, where surface form varies considerably.

## Integration with Metadata Filtering

Pure semantic similarity is insufficient for clinical recommendation retrieval. A semantically similar chunk may be inappropriate if it targets a different age group or diagnostic context. The system combines embedding-based similarity with structured metadata filters:

- **List membership checks**: Does the chunk's diagnosis list include the queried disorder?
- **Equality filters**: Does the chunk's age group match the patient being queried for?

This hybrid approach — semantic retrieval followed by metadata post-filtering — is a core pattern in [[concepts/recommendation-rag-pipeline]] systems for clinical use.

## Relationship to Broader Pipeline

| Stage | Component | Embedding Role |
|---|---|---|
| Ingestion | `src/embeddings.py` | Encode all recommendation chunks |
| Storage | `src/retrieval.py` + FAISS | Store normalized vectors |
| Query | `app_recommendations.py` | Encode user search query |
| Generation | `src/llm.py` | Retrieved chunks passed to LLM |

See also:
- [[concepts/clinical-nlp-pipelines]] — broader context for NLP in clinical settings
- [[concepts/phi-deidentification-pipeline]] — de-identification that precedes embedding
- [[concepts/rag-chunking]] — how recommendation text is split before encoding
- [[concepts/icd10-diagnosis-extraction]] — diagnosis metadata attached to chunks
- [[concepts/neuropsychological-assessment-pipeline]] — the full system context
- [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] — related RAG system for the same domain


See also: [[summaries/README]]