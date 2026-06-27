---
sources: [summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/README.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/vector-store.md, summaries/text-extraction.md]
brief: Splitting documents into overlapping segments before embedding, balancing retrieval quality with storage cost.
---

# Text Chunking Strategy

Text chunking is the process of splitting a long extracted document into smaller, overlapping segments before generating vector embeddings. This boundary condition between raw text extraction and semantic search is critical to the quality of [[concepts/retrieval-augmented-generation]] systems.

## Why Chunking Matters

Embedding models have fixed context windows, and embedding an entire document as a single unit loses fine-grained semantic resolution. Chunking ensures that:

- Each segment is semantically coherent and independently embeddable
- Retrieval returns focused, relevant passages rather than entire documents
- Context is preserved across boundaries via overlap

## Core Parameters Across Systems

Different pipelines in this knowledge base apply chunking with different parameter choices depending on their domain and retrieval goals.

### Neuropsychological Report Pipeline

As documented in [[summaries/text-extraction]], the chunking function used in the neuropsychological report pipeline applies these defaults:

```python
chunks = chunk_text(text, chunk_size=1200, overlap=150)
```

| Parameter | Value | Rationale |
|-----------|-------|----------|
| `chunk_size` | 1200 characters | Balances context preservation with embedding quality |
| `overlap` | 150 characters | Maintains continuity across chunk boundaries |

A chunk size of 1200 characters (~200–250 words) is a practical middle ground:
- Large enough to contain a complete clinical observation or test result summary
- Small enough for an embedding model to encode with high fidelity
- Avoids the semantic dilution of very large chunks

### Autism Research Document Pipeline

The Autism RAG system (see [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]) uses a word-based chunking scheme rather than a character-based one:

| Parameter | Value | Rationale |
|-----------|-------|----------|
| `chunk_size` | 800 words | Suited to research document prose and FLAN-T5 context limits |
| `overlap` | 120 words | Prevents boundary gaps in dense scientific text |

This word-count approach is well-matched to research literature, where paragraphs tend to be longer and more semantically self-contained than in structured clinical reports. The 800-word chunks feed directly into 384-dimensional embeddings produced by `all-MiniLM-L6-v2` (SentenceTransformers), which are then stored in a FAISS IndexFlatIP. The clinical recommendation pipeline in the same system reuses this chunking infrastructure for de-identified neuropsychological reports, applying the same approach to different source material.

## Overlap Strategy

Overlapping adjacent chunks — whether measured in characters or words — ensures that sentences or phrases near a boundary are represented in both neighboring chunks. This prevents retrieval gaps where a key phrase falls exactly on a split point and would otherwise be poorly represented in either chunk. Both pipelines above demonstrate this principle: the neuropsychological pipeline uses 150-character overlap, while the autism research pipeline uses 120-word overlap.

## Customizing Chunk Parameters

The default configurations are not universal. The [[summaries/customization]] guide documents how chunk size and overlap should be adjusted for different report types:

| Use Case | `--chunk-size` | `--overlap` |
|---|---|---|
| Long-form comprehensive reports | 2000 | 200 |
| Brief screening reports | 600 | 75 |
| Default (balanced) | 1200 | 150 |

Larger chunks preserve more context within each segment but may dilute embedding specificity; smaller chunks improve precision but fragment clinical narratives and increase the total number of embeddings to store and scan.

### Section-Aware Chunking

For structured reports with clearly delineated sections, a more sophisticated approach involves pre-extracting named sections before indexing and attaching section metadata to each chunk. This allows retrieval to be scoped to specific report regions (e.g., background, test results, recommendations) rather than treating the document as a flat stream of text.

```python
# Pre-extract sections before indexing
sections = extract_sections(report_text)
for section_name, section_text in sections.items():
    # Index with section metadata
    ...
```

The Autism RAG system extends this idea further with **metadata-filtered search**: the vector store retrieves a 3× over-fetch of candidates, then post-filters on structured metadata fields (e.g., diagnoses as list membership, age_group as equality check). Chunking parameters therefore interact with filter design — smaller chunks mean more candidates to over-fetch and filter, while larger chunks reduce candidate volume but may span multiple metadata contexts.

## Preprocessing Before Chunking

Before chunking, whitespace normalization is applied — collapsing multiple spaces and newlines into single spaces. This prevents artificially short chunks caused by formatting artifacts, which is especially important when processing PDFs that may introduce irregular whitespace. Both pipelines use PyMuPDF (fitz) or similar tools for initial text extraction before chunking begins.

## Pipeline Position

Chunking sits between extraction and embedding in the ingestion pipeline. In the neuropsychological report pipeline:

```
Report Files → extract_text() → chunk_text() → embed_with_fallback() → SQLite
```

In the Autism RAG research pipeline:

```
PDF/EPUB → parse_document() → chunk_text() → generate_embeddings() → FAISS Index
```

In both cases, chunks are stored with positional metadata (source path, chunk index) enabling citation tracking. The Autism RAG system additionally stores similarity scores alongside source document names at query time.

In the [[concepts/sqlite-as-vector-store]] implementation, each chunk produced by the neuropsychological pipeline is stored with:
- Its source file path and positional index (`chunk_id`)
- The raw text content
- A JSON-serialized embedding vector (~768 dimensions from ModernBERT)

The `INSERT OR REPLACE` strategy means re-indexing a file is safe and idempotent.

See [[concepts/neuropsychological-assessment-pipeline]] for broader pipeline context and [[concepts/sqlite-as-vector-store]] for how chunks are stored after embedding.

## Storage Implications of Chunk Size

Chunk size directly affects storage footprint. At ~768 embedding dimensions, each JSON-serialized vector occupies roughly 5–10 KB. With 1200-character chunks:

| Scale | Approximate DB Size |
|-------|-----------|
| 1,000 chunks | 5–10 MB |
| 10,000 chunks | 50–100 MB |

This is manageable within the SQLite-based approach up to approximately 10,000 chunks, after which migration to a dedicated vector database (such as FAISS, as used in the Autism RAG system) becomes advisable. Adjusting chunk size upward reduces the total chunk count and the per-query memory footprint, while downward adjustment increases granularity at the cost of more embeddings.

For very large corpora (1000+ reports), the customization guide also recommends batch indexing by year and merging resulting databases, which compounds the importance of choosing a chunk size that keeps total chunk count manageable across the merged index.

## Relationship to Retrieval Quality

Chunk boundaries directly affect [[concepts/vector-search]] and [[concepts/hybrid-search-retrieval]] quality. Poorly sized chunks produce either:
- **Too-small chunks**: Loss of clinical context, fragmented sentences, and more embeddings to scan
- **Too-large chunks**: Diluted embeddings that match broadly but not precisely

The 1200/150 default configuration is tuned for clinical neuropsychological reports, where individual paragraphs describe distinct cognitive domains (see [[concepts/cognitive-domains]]) and must be retrievable independently. The 800-word/120-word configuration in the Autism RAG system is tuned for research literature, where longer prose passages carry more coherent semantic units.

The `--top-k` retrieval parameter (number of chunks returned at query time) interacts with chunk size: larger chunks may warrant a lower top-k because each chunk carries more context, while smaller chunks may require a higher top-k to assemble sufficient context for generation.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — downstream consumer of chunked embeddings
- [[concepts/vector-search]] — retrieval over chunk embeddings
- [[concepts/sqlite-as-vector-store]] — storage layer for embedded chunks in clinical pipelines
- [[concepts/faiss-vector-index]] — storage layer for embedded chunks in the Autism RAG system
- [[concepts/hybrid-search-retrieval]] — combines vector and keyword search over chunks
- [[concepts/neuropsychological-assessment-pipeline]] — full pipeline this chunking strategy supports
- [[concepts/style-profiles]] — style profiles trained on chunk-indexed report corpora
- [[concepts/rag-chunking]] — closely related concept covering chunking design patterns
- [[summaries/text-extraction]] — source implementation details
- [[summaries/vector-store]] — vector store implementation that consumes chunked embeddings
- [[summaries/customization]] — per-use-case chunk size and overlap recommendations
- [[summaries/style-training-to-report-drafting]] — end-to-end workflow from indexed chunks to drafted reports
- [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] — second pipeline demonstrating word-based chunking

See also: [[summaries/full-pipeline]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]

See also: [[summaries/README]]

See also: [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]]