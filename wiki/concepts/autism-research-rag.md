---
sources: [summaries/autism_recommendations_for_adults_summary.md, summaries/README.md]
brief: A RAG pipeline purpose-built for autism research, enabling semantic question answering over clinical documents.
---

# Autism Research RAG System

An **Autism Research RAG System** is a domain-specific [[concepts/retrieval-augmented-generation]] application designed to answer natural-language questions about autism using a curated corpus of autism-related documents. Rather than relying on a language model's parametric knowledge alone, the system retrieves relevant passages from ingested documents and uses them as grounded context for answer generation — with source citations for transparency.

See also: [[summaries/README]], [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]].

## Core Pipeline

The system implements a standard RAG architecture adapted for clinical and research documents:

1. **Ingestion** — PDFs are parsed, text is extracted, and the content is segmented into chunks (see [[concepts/text-chunking]] and [[concepts/rag-chunking]])
2. **Embedding** — chunks are converted to vector representations (see [[concepts/sentence-transformer-embeddings]])
3. **Indexing** — embeddings are stored in a vector index for fast similarity search (see [[concepts/vector-search]] and [[concepts/faiss-vector-index]])
4. **Retrieval** — incoming queries are embedded and matched against the index to find the most relevant passages
5. **Generation** — a language model uses the retrieved passages and a prompt template to compose a grounded answer with citations

## Key Components

| Module | Role |
|---|---|
| `pdf_parse.py` | PDF text extraction (see [[concepts/pdf-data-extraction]]) |
| `chunking.py` | Text segmentation |
| `embeddings.py` | Embedding generation |
| `retrieval.py` | Similarity search |
| `rag.py` | Core RAG orchestration |
| `prompts.py` | Prompt templates |
| `citations.py` | Source citation formatting |
| `query.py` | Interactive query REPL |
| `api/server.py` | FastAPI REST interface |

## API Interface

The system exposes a RESTful API (FastAPI) with the following endpoints:

- `POST /query` — submit a question; accepts `top_k` to control the number of retrieved passages
- `GET /ingest` — trigger the document ingestion pipeline
- `GET /health` — health check

Example:
```bash
curl -X POST "http://localhost:8000/query" \
     -H "Content-Type: application/json" \
     -d '{"question": "What are early signs of autism?", "top_k": 5}'
```

## Domain Context

This system is designed around [[concepts/autism-spectrum-disorder-clinical-features]] as its primary knowledge domain. Source documents are expected to cover autism research, diagnostic criteria, and related clinical material. The retrieval-augmented approach is well-suited to this domain because:

- Autism research spans many overlapping diagnostic frameworks (e.g., [[concepts/autism-diagnostic-rating-scales]])
- Clinicians need precise, citable answers rather than hallucinated summaries
- The corpus can grow incrementally as new research is ingested

## Evaluation

The system includes a structured evaluation framework (`eval/questions.yaml` + `eval/run_eval.py`) for measuring retrieval and generation quality, supporting iterative improvement of the pipeline.

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — foundational architecture pattern
- [[concepts/text-chunking]] — document segmentation strategy
- [[concepts/rag-chunking]] — chunking considerations for RAG
- [[concepts/vector-search]] — similarity-based retrieval
- [[concepts/faiss-vector-index]] — vector indexing technology
- [[concepts/sentence-transformer-embeddings]] — embedding model approach
- [[concepts/pdf-data-extraction]] — source document parsing
- [[concepts/autism-spectrum-disorder-clinical-features]] — primary knowledge domain
- [[concepts/autism-diagnostic-rating-scales]] — diagnostic instruments in the corpus
- [[concepts/hybrid-search-retrieval]] — potential retrieval enhancement
- [[concepts/neuropsychological-assessment-pipeline]] — related clinical pipeline pattern


See also: [[summaries/autism_recommendations_for_adults_summary]]