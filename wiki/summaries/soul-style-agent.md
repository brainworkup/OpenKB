---
doc_type: short
full_text: sources/soul-style-agent.md
---

# Soul — Style Agent

A single-file local agent (`neuro_report_style_agent.py`) that learns the writing style of existing neuropsychological reports and uses that style to generate new draft sections. Powered by a local [[concepts/omlx-server]] (OMLX) and a lightweight [[concepts/retrieval-augmented-generation]] (RAG) pipeline backed by [[concepts/sqlite-as-vector-store]].

## Purpose

Automate styled neuropsychological report drafting by:
1. Indexing existing reports into a local vector store
2. Extracting a reusable JSON **style profile** from those reports
3. Generating new draft sections that match the learned style

## Three-Stage Pipeline

```
PDF/TXT/MD reports ──► build-index ──► SQLite vector store
                                              │
                                      train-style ──► JSON style profile
                                              │
                              write-report ◄──┘
```

### Stage 1: `build-index`
- Extracts text from PDF (PyPDF2), TXT, and MD files
- Chunks text with fixed-size overlap (default 1200 chars, 150 overlap)
- Embeds chunks via OMLX `/embeddings` endpoint
- Stores in SQLite `chunks` table: `id`, `source_path`, `chunk_id`, `content`, `embedding`

### Stage 2: `train-style`
- Seeds a RAG query to retrieve style-exemplar chunks (default top-12)
- Prompts the LLM to produce a structured [[concepts/style-profile-extraction]] JSON:
  - `voice`, `tone`, `structure_patterns`, `typical_phrases`, `do_rules`, `avoid_rules`
- Saves profile to a JSON file for reuse

### Stage 3: `write-report`
- Embeds the user's task prompt and retrieves top-k relevant chunks (default 6)
- Assembles a prompt combining style profile + RAG context + user task
- Enforces rules: professional language, no fabrication, `[NEEDS DATA]` for missing info
- Outputs a styled draft to a text file

## Key Design Decisions

- **Single-file architecture**: All logic in one Python file, minimising dependencies — see [[concepts/single-file-agent-pattern]]
- **Stdlib-only runtime**: Uses `urllib`, `sqlite3`, `json`, `argparse` — no heavy frameworks
- **Fallback hooks**: `embed_with_fallback` and `generate_with_fallback` provide single call-sites for swapping backends, embodying a [[concepts/fallback-strategy]]
- **Pure-Python cosine similarity**: No numpy required for similarity search — see [[concepts/vector-search]]
- **Local-first**: All data stays on-device; OMLX runs at `http://127.0.0.1:8000/v1` — see [[concepts/local-first-architecture]]

## Key Functions

| Function | Role |
|---|---|
| `extract_text(path)` | PDF/TXT/MD text extraction |
| `chunk_text(text, chunk_size, overlap)` | Overlapping chunking |
| `omlx_embed / omlx_generate` | OMLX API calls |
| `retrieve_top_k(db_path, query_embedding, k)` | SQLite cosine search |
| `init_db(db_path)` | Schema initialisation |

## Dependencies

- Python ≥ 3.13 (stdlib only)
- Optional: `PyPDF2` for PDF support
- External: OMLX local inference server

## Related Concepts
- [[concepts/openai-compatible-api]]
- [[concepts/clinical-report-structure]]
- [[concepts/clinical-data-privacy]]

- [[concepts/retrieval-augmented-generation]] — core retrieval mechanism
- [[concepts/omlx-server]] — generation and embedding backend
- [[concepts/style-profile-extraction]] — JSON style profile extraction and application
- [[concepts/sqlite-as-vector-store]] — SQLite-based embedding storage and search
- [[concepts/single-file-agent-pattern]] — architectural pattern used throughout
- [[concepts/local-llm-inference]] — local model inference approach
- [[concepts/narrative-report-generation]] — the downstream output goal
- [[concepts/neuropsychological-reporting]] — domain context for report drafting