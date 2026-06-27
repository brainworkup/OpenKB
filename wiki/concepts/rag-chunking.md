---
sources: [summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/README.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/full-pipeline.md, summaries/customization.md]
brief: How text is divided into indexed segments to optimize retrieval quality in RAG systems.
---

# RAG Chunking Strategies

Chunking is the process of dividing source documents into discrete text segments before embedding and indexing them in a vector store. Chunk configuration directly affects retrieval quality in [[concepts/retrieval-augmented-generation]] systems: chunks that are too large dilute relevance signals, while chunks that are too small lose necessary context.

## Core Parameters

### Chunk Size
The number of tokens or characters in each indexed segment. Larger chunks preserve more contextual information per retrieval hit; smaller chunks improve precision when queries target narrow topics.

### Overlap
The number of tokens shared between consecutive chunks. Overlap prevents important information from being split across chunk boundaries and lost during retrieval.

## Chunking Strategies by Use Case

As documented in [[summaries/customization]], the neuro report style agent exposes chunking parameters directly via CLI flags:

| Report Type | `--chunk-size` | `--overlap` | Rationale |
|---|---|---|---|
| Long-form comprehensive reports | 2000 | 200 | Preserves extended clinical reasoning across paragraphs |
| Brief screening reports | 600 | 75 | Tighter focus on concise, modular language patterns |
| Default / general use | ~1000 | ~100 | Balanced precision and context |

## Section-Aware Chunking

For highly structured documents (e.g., neuropsychological reports with named sections), a pre-processing step can extract sections before indexing. Each section is then indexed with attached metadata, enabling retrieval that respects document structure rather than splitting across section boundaries. This is particularly relevant for [[concepts/clinical-report-structure]] where section identity carries semantic meaning.

## Relationship to Retrieval Quality

Chunk size interacts with the `--top-k` retrieval parameter: a smaller chunk size combined with a higher `--top-k` value can approximate the context of larger chunks while maintaining retrieval precision. This trade-off is discussed in [[summaries/customization]] under custom retrieval parameters.

Chunking strategy also affects [[concepts/style-profile-extraction]]: if training reports are chunked poorly, extracted style patterns may be fragmented, reducing the coherence of the learned [[concepts/style-profiles]].

## Chunking in the Vector Store Pipeline

Once chunked, segments are embedded and stored in a [[concepts/sqlite-as-vector-store]] backend (via the `build-index` command). The embedding model (configurable via `--omlx-embed-model`) encodes each chunk into a dense vector for similarity search. See [[concepts/vector-search]] for retrieval mechanics and [[concepts/text-chunking]] for broader discussion of chunking approaches.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — the broader framework within which chunking operates
- [[concepts/vector-search]] — how chunked embeddings are queried at inference time
- [[concepts/sqlite-as-vector-store]] — the storage backend for indexed chunks
- [[concepts/style-profile-extraction]] — downstream process that depends on chunk quality
- [[concepts/style-profiles]] — the learned output of training over indexed chunks
- [[concepts/text-chunking]] — general-purpose discussion of chunking methods
- [[concepts/hybrid-search-retrieval]] — retrieval strategies that operate over chunk indexes
- [[summaries/customization]] — source document detailing chunking parameter options
- [[summaries/soul-style-agent]] — the agent that consumes chunked indexes for report drafting


See also: [[summaries/full-pipeline]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]

See also: [[summaries/README]]

See also: [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]]