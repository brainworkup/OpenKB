---
sources: [summaries/agentic-workflows.md, summaries/File Folder Structure Rebuild.md, summaries/DEPENDENCIES.md, summaries/README.md, summaries/RECOVERY_NOTES.md, summaries/README_luria.md, summaries/deepagents_merged_mem_notes.md, summaries/SESSION_SUMMARY_2025-04-28.md]
brief: LangGraph-based stateful multi-step agent workflows for orchestrating neuropsychological data pipelines.
---

# LangGraph Agent Workflows

LangGraph is a Python library built on top of LangChain that enables developers to construct **stateful, multi-step agent workflows** as explicit directed graphs. Each node in the graph represents a discrete processing step (e.g., parsing, extraction, scoring, report generation), and edges define the control flow between steps — including conditional branching and cycles.

## Core Concepts

### Graph Structure
- **Nodes** — Individual processing units, typically Python functions or LLM calls
- **Edges** — Directed connections defining execution order and conditional routing
- **State** — A shared data structure (often a TypedDict or Pydantic model) passed between nodes and mutated as the workflow progresses
- **Checkpointing** — Built-in support for persisting state, enabling pause/resume and human-in-the-loop patterns

### Why Graphs Instead of Chains?
Traditional LLM chains are linear and stateless. LangGraph allows:
- **Cycles** — Re-running steps based on validation or quality checks
- **Branching** — Routing to different nodes based on intermediate results
- **Persistent state** — Maintaining context across multi-turn interactions (see [[concepts/persistent-memory]])
- **Parallelism** — Running independent nodes concurrently via the `Send` API or `asyncio.gather`

## Fan-Out Parallelism

LangGraph v0.2+ provides a `Send` API that enables true fan-out patterns: spawning N independent subgraph invocations and collecting results via a reducer. This is the recommended approach for production pipelines requiring per-task observability. An alternative — `asyncio.gather` inside a single node — is simpler to implement but sacrifices per-task tracing.

In practice, for a pipeline with 7 parallel domain subagents each running 3 concurrent sub-tasks (Text, Table, Figure generation), maximum real concurrency is approximately 21 simultaneous operations — not the theoretical maximum of 7×4=28, because DataPrep must complete serially before Text/Table/Figure begin.

## Two-Graph Architecture in the Luria Streamlit App

The Luria Streamlit App (see [[summaries/README]]) implements LangGraph as its core orchestration runtime, exposing two independently testable compiled graphs. The app provides four main capabilities through a Streamlit desktop UI: **Ingest**, **Ask**, **Knowledge Base**, and **Audio** — all orchestrated by LangGraph pipelines. The full dependency stack is documented in [[summaries/DEPENDENCIES]].

### Entry Points (defined in `neuropsych_agent/graph.py`)

| Function | Purpose |
|---|---|
| `build_ingest_graph()` | 4-stage pipeline: parse → extract → index → report |
| `build_rag_graph()` | Single-node retrieval for Q&A |
| `ingest_pdf(path, mode, **voice_kw)` | Convenience wrapper used by the Streamlit UI |
| `ask_rag(query)` | Convenience wrapper for RAG Q&A |

**IngestGraph** — Sequential document processing:
```
START → parse → extract → index → report → END
```
- **parse**: PDF → text via Docling (local, deterministic); PHI redacted locally before any network call — no patient data leaves the machine at this stage
- **extract**: LLM (oMLX `medgemma-1.5-4b-it-bf16` or Claude Sonnet via `langchain-anthropic`) → structured JSON records (test scores, clinical summaries)
- **index**: SQLite (`data/neuropsych.db`) + LanceDB (`data/vectors/`) writes; both stores are entirely local
- **report**: LLM → markdown narrative report; optional Typst or Quarto rendering for print-ready PDFs

**RAGGraph** — Retrieval-augmented question answering:
```
START → rag → END
```
- **rag**: SQL filtering over `TestScores` + semantic search over narrative chunks → LLM synthesis (see [[concepts/retrieval-augmented-generation]])

### Pipeline State (`neuropsych_agent/state.py`)

`PipelineState` is a `TypedDict` carrying:
- `messages` — accumulated LangChain messages (`SystemMessage`, `HumanMessage`, `AIMessage` from `langchain-core`)
- `pdf_path`, `doc_id` — document identity
- `parsed`, `records`, `indexed`, `report` — stage outputs
- `user_query`, `rag_answer` — RAG fields
- `voice_enabled`, `soul_db_path`, `soul_profile_path`, `quarto_format` — Luria Voice options

### Key Structural Patterns

| Pattern | Where | Why |
|---|---|---|
| `Annotated[list, operator.add]` | `state_luria.py` | Messages accumulate across nodes without overwriting |
| Frozen dataclass + `lru_cache` | `config_luria.py` | Singleton settings, testable |
| Markdown prompts loaded at runtime | `nodes_luria.py` | Decouple prompts from code |
| Separate compiled graphs | `graph_luria.py` | Test each pipeline independently |
| Placeholder tools in nodes | `nodes_luria.py` | Easy to swap real implementations |

Subagent prompts live in `subagents/` as standalone Markdown files — one per agent — making them editable without touching code. This reflects a broader [[concepts/plain-text-documentation]] approach to prompt management.

## SQLite Schema (managed by `store.py`)

The structured store mirrors the extraction outputs from the ingest pipeline:

- **`Documents`** — one row per ingested PDF (`doc_id`, `filename`, `ingested_at`)
- **`ClinicalSummaries`** — narrative summaries per document
- **`TestScores`** — structured rows: `test_name`, `subtest_name`, `scaled_score`, `standard_score`, `t_score`, `percentile_rank`, `classification`, `cognitive_domains_affected`

The `Ask` tab queries these tables directly via SQL filtering, combined with semantic search over LanceDB vector chunks for hybrid retrieval.

## Package Dependencies

The LangGraph pipeline depends on a specific set of versioned packages. Core orchestration relies on `langgraph` (latest) and `langchain ≥1.2.15`, with supporting packages from the LangChain ecosystem:

| Package | Version | Role |
|---|---|---|
| `langgraph` | latest | StateGraph engine — `build_ingest_graph()`, `build_rag_graph()` |
| `langchain` | ≥1.2.15 | Core abstractions |
| `langchain-core` | latest | `SystemMessage`, `HumanMessage`, `AIMessage`, runnables |
| `langchain-anthropic` | ≥1.4.1 | Claude API (cloud extraction fallback) |
| `langchain-huggingface` | ≥1.2.2 | Sentence-transformers bridge |
| `langchain-docling` | latest | Docling PDF parser integration |
| `langchain-milvus` | ≥0.3.3 | Alternative vector store (optional) |
| `openai` | latest | oMLX local server client (not OpenAI cloud) |
| `pydantic` | ≥2.0 | Strict schema validation for `PipelineState` |
| `lancedb` | latest | Local vector store at `data/vectors/` |
| `sentence-transformers` | latest | Default embeddings (`all-MiniLM-L6-v2`, 384-dim) |
| `honcho-ai` | ≥2.1.1 | Optional peer-observation layer |

The `openai` package is used exclusively to call the **oMLX local server** via `openai.OpenAI(base_url=...)` — not the OpenAI cloud service. See [[concepts/omlx-server]] and [[concepts/openai-compatible-api]] for details.

## Starter Kit: Minimal Project Template

The **Luria LangGraph Starter Kit** (see [[summaries/README_luria]]) distills the Luria pipeline into the fewest files needed to recreate the architecture in a fresh project. It provides two independently testable compiled graphs and a set of canonical design patterns applicable to any domain.

Minimal dependencies for a new project: `langchain-core`, `langgraph`, `openai`, `python-dotenv`. Optional additions include `langchain-docling` for PDF parsing, `lancedb` for vector storage, and `sentence-transformers` for local embeddings.

## Usage in Luria

In the Luria neuropsychological reporting system, LangGraph is the **confirmed orchestration runtime**, replacing an earlier aspirational `deepagents` SDK that did not exist as an installable package. As documented in [[summaries/deepagents_merged_mem_notes]] and [[summaries/SESSION_SUMMARY_2025-04-28]], the architecture centers on `neuropsych_agent/graph.py` extended with domain fan-out nodes.

### Architecture Evolution (2025-04-28)

As of the April 2025 refactor, the Luria codebase underwent significant restructuring:

**Removed:**
- `app/cli.py` and `app/orchestrator.py` — CLI-based orchestration eliminated
- `app/agents/nse_agent.py`, `nt_agent.py` — Visit-scoped agents removed
- `app/intake/` directory — CSV extraction moved to the `cingulate` R package

**Active entry points and services:**
- `streamlit_app.py` — Primary UI and application entry point (4 tabs: Ingest, Ask, Knowledge Base, Audio)
- `neuropsych_agent/nodes.py` — LangGraph node implementations
- `neuropsych_agent/graph.py` — Graph wiring and edge definitions
- `neuropsych_agent/tools/store.py` — SQLite state persistence
- `neuropsych_agent/tools/soul_context.py` — Soul/persona context injection
- `neuropsych_rag/` — Standalone RAG library (reusable outside Streamlit)
- `rag/page-index/app/service.py` — PageIndex document service
- `rag/docling/detect_pii.py` — PII obfuscation layer

### Current 4-Stage Ingest Graph

| Node Role | Responsibility |
|---|---|
| PDF parse | Ingest and extract raw text from neuropsych reports via [[concepts/docling-pdf-parsing]] |
| Extract | Pull structured test scores and patient data |
| Index | Store content in SQLite + [[concepts/lancedb-vector-store]] |
| Report | Generate clinical narrative output |

### Extended Pipeline (A→E Phases)

The corrected Luria orchestration plan extends `graph.py` with nodes covering five clinical phases:

```
# Phase A — Intake
"nse_admin", "nse_clinical_past", "nse_clinical_current",
"nse_stt_transcript", "nse_cod_summary", "nse_report"

# Phase B — Testing (with fan-out)
"nt_tests", "nt_behav_obs",
"nt_neurocog_fanout",    # spawns 7 domain subgraphs
"nt_neurobehav_fanout",  # spawns 2 domain subgraphs

# Per-domain (reusable)
"domain_dataprep", "domain_text", "domain_table",
"domain_figure", "domain_typst"

# Phase C — SIRF
"sirf_summary", "sirf_impression", "sirf_recs"

# Phase D — Assembly
"report_assemble", "quality_review"
```

**Phase B fan-out verified:** All 7 neurocognitive domains generated clinical narratives sequentially in ~19.8s total (avg 2.8s/domain) using `gpt-oss-20b-MXFP4-Q8` via a local oMLX endpoint. Parallel execution is projected to reduce this to ~7s.

### Subagents

Four canonical subagents map directly to LangGraph nodes, sourcing their system prompts from `subagents/*/AGENTS.md` files — no prompt rewriting is required when the graph is extended:

| Subagent | Path | Node | LLM? |
|---|---|---|---|
| PDF_Ingestion_Parser | `subagents/PDF_Ingestion_Parser/AGENTS.md` | `parse_node` | No (deterministic) |
| Neuropsych_Data_Extractor | `subagents/Neuropsych_Data_Extractor/AGENTS.md` | `extract_node` | Yes (JSON extraction) |
| Sheets_Data_Indexer | `subagents/Sheets_Data_Indexer/AGENTS.md` | `index_node` | No (SQLite/LanceDB) |
| Narrative_Report_Generator | `subagents/Narrative_Report_Generator/AGENTS.md` | `report_node` | Yes (markdown) |

### Key Integration Points

The graph integrates with:
- **R6 Domain Processors** (via [[concepts/r-python-integration]]) for domain-specific scoring (IQ, Memory, Attention, Language, Executive, Motor)
- **PageIndex RAG service** for document retrieval (see [[concepts/retrieval-augmented-generation]])
- **PII redaction gate** at node A5 using Presidio, ensuring all downstream nodes receive only de-identified content (see [[concepts/pii-redaction-pipelines]])
- **SQLite store** at `neuropsych_agent/tools/store.py` for state persistence
- **LanceDB vector index** at `data/vectors/` for semantic retrieval (see [[concepts/lancedb-vector-store]])
- **Local LLM inference** via oMLX endpoint at port 8000 (see [[concepts/local-llm-inference]])
- **Docling PII obfuscation** at `rag/docling/detect_pii.py` (see [[concepts/docling-pdf-parsing]])
- **Typst/Quarto rendering** for print-ready PDF reports (see [[concepts/typst-typesetting]] and [[concepts/quarto]])
- **MacWhisper CLI** and local oMLX server for audio transcription and summarization (see [[concepts/audio-transcription-pipeline]])
- **Honcho AI** peer observation layer (optional, via `honcho-ai ≥2.1.1`) — see [[concepts/honcho-ai-peer-observation]]
- **DuckDB** analytics store in `kb/store/` for column-store RAG queries (see [[concepts/duckdb-as-vector-store]])

## Skill-Based Orchestration

Luria wraps LangGraph workflows inside a **skill layer** — the `luria-neuropsych-orchestrator` — which coordinates five sub-skills:

```
luria-neuropsych-orchestrator
├── luria-case-intake
├── luria-score-processing
├── luria-interpretation
├── luria-report-writing
└── luria-quality-review
```

This pattern separates *orchestration logic* (skill routing) from *execution logic* (LangGraph graph), making each layer independently testable and replaceable. The April 2025 refactor confirmed this as the canonical Luria pattern, replacing the earlier CLI-driven model.

See [[concepts/multi-agent-orchestration]] for the broader architectural context and [[summaries/SKILL]] for skill specification details.

## Model Routing Within the Graph

Each node in the graph is assigned a specific model based on task requirements — a role-based routing pattern verified against a local oMLX endpoint. oMLX configuration is managed via environment variables:

| Config Key | Default Model | Dim | Purpose |
|---|---|---|---|
| `OMLX_CHAT_MODEL` | `medgemma-1.5-4b-it-bf16` | — | Chat/Extraction (default) |
| `OMLX_EMBEDDING_MODEL` | `nomicai-modernbert-embed-base-bf16` | 768 | Embeddings |
| `OMLX_RERANK_MODEL` | `nomicai-modernbert-embed-base-bf16` | 768 | Reranking |
| `VECTOR_EMBEDDING_MODEL` | `sentence-transformers/all-MiniLM-L6-v2` | 384 | Fallback embeddings |

Per-phase model routing for the extended pipeline:

| Graph Phase | Model | Rationale |
|---|---|---|
| Deterministic/DataPrep nodes | `granite-4.1-8b-nvfp4` (temp=0) | Fast, cheap, reproducible |
| Domain narrative nodes | `gpt-oss-20b-MXFP4-Q8` | Clinical prose quality |
| Summary/diagnosis/review nodes | `Qwen3.6-35B-A3B-oQ4` | Strong reasoning |
| Orchestrator/patient-facing nodes | `MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit` | Best output quality |

This leverages [[concepts/mixture-of-experts]] and [[concepts/model-quantization]] models for efficient local inference without cloud dependency.

## Failure Handling

A production LangGraph pipeline for clinical reporting requires explicit failure states at every domain node. The Luria implementation defines four outcomes:

| State | Orchestrator Behavior |
|---|---|
| `COMPLETED` | All artifacts produced; continue |
| `DEGRADED` | Include available artifacts, flag the gap |
| `SKIPPED` | Insert "Not Administered" placeholder |
| `FAILED` (retryable) | Retry up to 2× with backoff |
| `FAILED` (non-retryable) | Skip domain, add to issue list; escalate to human |

The quality review node (`luria-quality-review`) performs a final consistency pass — checking for PHI leaks, score↔narrative consistency, and completeness — with up to 3 correction cycles before human escalation.

## Security and PHI Design Within the Graph

The Luria graph architecture enforces a strict local-first, HIPAA-conscious design:
- PHI redaction occurs **locally** in the parse node (`PHI_REDACTION_ENABLED=true`) before any text reaches Anthropic's API
- LanceDB and SQLite are entirely local; no cloud vector or relational store
- Audio transcription (MacWhisper) and summarization (oMLX) are also local
- The `.env` file is gitignored; API keys must not be committed
- oMLX enables fully offline operation via `LLM_PROVIDER=omlx`
- **Action required**: Rotate any API keys that may have been committed before `.gitignore` was finalized

See [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]] for broader treatment of these patterns.

## Cloud LLM Fallback

When local inference is insufficient, the graph falls back to cloud providers via a [[concepts/llm-provider-abstraction]] pattern:
- **Anthropic Claude** (via `langchain-anthropic ≥1.4.1`, `ANTHROPIC_API_KEY`): cloud extraction fallback — PHI must be redacted before any call
- **OpenAI GPT** (via `OPENAI_API_KEY`): optional secondary fallback
- **LangSmith** (`LANGSMITH_API_KEY`, `LANGSMITH_TRACING=true`): pipeline tracing and observability

See [[concepts/fallback-strategy]] for architectural guidance on local-to-cloud fallback patterns.

## Luria Voice Integration Within the Graph

The report node accepts optional Luria Voice parameters passed through `PipelineState`, enabling clinician-specific output via three layers:
- **SOUL** — style profile + exemplar retrieval from de-identified prior reports (configured via `soul_db_path`, `soul_profile_path` state fields)
- **STYLE** — Quarto `neurotyp-*-typst` format selection (configured via `quarto_format` state field)
- **BRAND** — `_brand.yml` visual theming applied during Typst/Quarto rendering

The Streamlit sidebar exposes these options as toggles; the graph passes them transparently through `PipelineState` to the report node.

## Adapting the Pattern to Other Domains

The starter kit's design makes it straightforward to apply LangGraph's pipeline architecture beyond neuropsychology:

1. **Replace PipelineState fields** — swap neuropsych-specific fields for domain fields in the TypedDict
2. **Edit subagent prompts** — each `AGENTS.md` is a standalone Markdown prompt file
3. **Swap the LLM** — replace the `_llm()` factory with `ChatOpenAI` or `ChatAnthropic`
4. **Add nodes** — extend `build_ingest_graph()` with new edges for additional processing steps
5. **Add a router** — create a top-level classifier node that dispatches to subgraphs

## Relationship to Clinical NLP

LangGraph workflows are particularly well-suited to [[concepts/clinical-nlp-pipelines]] because clinical document processing requires:
- Multi-step extraction with validation loops
- Conditional routing based on document type or completeness
- Stateful accumulation of findings across multiple source documents
- Human review checkpoints before final report generation
- HIPAA-aware permission scoping (raw PHI paths denied to subagents)

The combination of LangGraph's explicit graph structure with domain-specific R processors and role-based model routing reflects a practical pattern for [[concepts/neuropsychological-reporting]] systems that must balance flexibility with clinical rigor.

## Related Concepts
- [[concepts/multi-agent-orchestration]]
- [[concepts/subagent-architecture]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/r-python-integration]]
- [[concepts/persistent-memory]]
- [[concepts/phi-data-handling]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/local-llm-inference]]
- [[concepts/llm-provider-abstraction]]
- [[concepts/fallback-strategy]]
- [[concepts/mixture-of-experts]]
- [[concepts/model-quantization]]
- [[concepts/plain-text-documentation]]
- [[concepts/python-project-structure]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/docling-pdf-parsing]]
- [[concepts/lancedb-vector-store]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/typst-typesetting]]
- [[concepts/quarto]]
- [[concepts/omlx-server]]
- [[concepts/openai-compatible-api]]
- [[concepts/yaml-configuration]]
- [[concepts/audio-transcription-pipeline]]
- [[concepts/honcho-ai-peer-observation]]
- [[concepts/local-first-architecture]]

See also: [[summaries/RECOVERY_NOTES]], [[summaries/SESSION_SUMMARY_2025-04-28]], [[summaries/SKILL]], [[summaries/README]], [[summaries/DEPENDENCIES]]

See also: [[summaries/File Folder Structure Rebuild]]

See also: [[summaries/agentic-workflows]]