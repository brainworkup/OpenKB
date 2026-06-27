---
doc_type: short
full_text: sources/README_luria.md
---

# Luria LangGraph Starter Kit

## Overview

The Luria LangGraph Starter Kit is a minimal framework extracted from the Luria neuropsychology project, providing the fewest files needed to recreate a [[concepts/multi-agent-orchestration]] pipeline using LangChain and [[concepts/langgraph-agent-workflows]]. It demonstrates how to structure a real-world LangGraph project with clean separation of concerns.

## Core Architecture

The kit defines two independent StateGraph pipelines:

### IngestGraph
A sequential pipeline: `START → parse → extract → index → report → END`
- **parse**: PDF → text (local, deterministic)
- **extract**: LLM → JSON records
- **index**: SQLite + vector store writes
- **report**: LLM → markdown narrative report

### RAGGraph
A simple retrieval pipeline: `START → rag → END`
- **rag**: semantic search + SQL → LLM synthesis

## File Structure

| File | Purpose |
|------|---------|
| `state_luria.py` | `PipelineState` TypedDict (shared state) |
| `graph_luria.py` | StateGraph wiring for both graphs |
| `nodes_luria.py` | Node implementations (4 agents + RAG) |
| `config_luria.py` | Environment-driven settings |
| `subagents_luria/` | Markdown system prompts for each agent |

## Key Design Patterns

- **`Annotated[list, operator.add]`** in state — messages accumulate across nodes without overwriting
- **Frozen dataclass + `lru_cache`** for config — singleton settings, easy to test
- **Markdown prompts loaded at runtime** — decouples prompts from code, editable without code changes
- **Separate compiled graphs** — each pipeline can be tested independently
- **Placeholder tools in nodes** — easy to swap real implementations later

## Four Subagents

1. **PDF Ingestion Parser** — converts PDF documents to text
2. **Neuropsych Data Extractor** — extracts structured JSON records via LLM
3. **Sheets Data Indexer** — writes to SQLite and a vector store
4. **Narrative Report Generator** — produces markdown narrative reports via LLM

## Adapting for Other Domains

1. Replace `PipelineState` fields with domain-specific fields
2. Edit subagent prompts (each `AGENTS_luria.md` is standalone)
3. Swap the LLM — replace `_llm()` with `ChatOpenAI` or `ChatAnthropic`
4. Add nodes by extending `build_ingest_graph()` with new edges
5. Add a top-level classifier/router node to dispatch between subgraphs

## Dependencies

**Minimal:** `langchain-core`, `langgraph`, `openai`, `python-dotenv`

**Optional (full):** `langchain-anthropic`, `langchain-docling` (PDF), `lancedb` (vector storage), `sentence-transformers` (local embeddings)

## Naming Convention

All files use the `_luria` suffix to avoid naming conflicts on platforms that reject duplicate filenames.

## License

MIT

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/phi-data-handling]]
- [[concepts/langgraph-agent-workflows]] — StateGraph orchestration framework used for both pipelines
- [[concepts/multi-agent-orchestration]] — Pattern of coordinating multiple LLM agents
- [[concepts/retrieval-augmented-generation]] — Used in the RAGGraph for semantic search + LLM synthesis
- [[concepts/neuropsychological-reporting]] — Domain context from which this starter kit was extracted
- [[concepts/python-project-structure]] — The kit exemplifies minimal, well-separated Python project layout
- [[concepts/local-llm-inference]] — Supported via oMLX local server client as the default LLM backend