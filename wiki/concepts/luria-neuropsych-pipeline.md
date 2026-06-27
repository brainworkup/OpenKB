---
sources: [summaries/agentic-workflows.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/SETUP_SUMMARY.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/RECOVERY_NOTES.md, summaries/README.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/neuropsych-pdf-parser.md, summaries/neuropsych-narrative-writer.md]
brief: Multi-stage LangGraph pipeline transforming neuropsych PDFs into structured clinical reports with local-first PHI safety.
---

# Luria Neuropsychological Assessment Pipeline

The **Luria neuropsychological assessment pipeline** is a multi-stage, agent-based system for producing clinical neuropsychological reports. It transforms raw assessment data — score sheets, PDFs, and structured tables — into polished, domain-organized reports rendered through a [[concepts/quarto]] and [[concepts/typst-typesetting]] toolchain. The pipeline is implemented both as a coordinated set of specialized agents and as the **Luria Python/R toolkit** (`luria`), an open-source package maintained by BrainWorkup Neuropsychology.

The toolkit is named in honor of A.R. Luria, a foundational figure in neuropsychology, and is designed for clinicians, researchers, and data scientists working with neuropsychological data.

## Toolkit Overview

The `luria` package (Python 3.10+, MIT License) provides:

- **Data Processing**: Ingestion of CSV, Excel, JSON, Parquet, and SQLite data; cleaning and validation of test scores.
- **Statistical Analysis**: Descriptive statistics, t-tests, ANOVA, correlation, regression, normative comparisons, and domain-level analysis. Powered by `numpy ≥1.24`, `pandas ≥2.0`, and `scipy ≥1.10`.
- **Report Generation**: Professional reports in HTML, PDF, DOCX, and Markdown via customizable templates (Adult, Pediatric, Forensic, Research).
- **R Integration**: Seamless Python–R interop via `reticulate` and the `cingulate` R package. See [[concepts/r-python-integration]].
- **CLI & Python API**: Both `luria init / process / analyze / report` commands (via `typer ≥0.9` and `click ≥8.0`) and a programmatic interface.
- **Privacy-First Design**: Local processing by default; optional cloud LLM integration (Anthropic, OpenAI) for advanced analysis.
- **Visualization**: Score profiles, domain heatmaps, longitudinal plots, and normative comparison charts via `matplotlib ≥3.7` and `seaborn ≥0.12`.

See [[concepts/neuropsychological-toolkit]], [[concepts/privacy-first-software]], and [[concepts/r-python-integration]] for related concepts.

## Standalone Streamlit App: `luria_streamlit_app`

A companion repository, `luria_streamlit_app`, packages the Streamlit pipeline (`streamlit ≥1.28`) as a self-contained, HIPAA-conscious desktop application for solo clinicians. It exposes four main tabs:

1. **Ingest** — Drag-and-drop PDFs through the 4-stage LangGraph pipeline (parse → extract → index → report).
2. **Ask** — Chat interface for RAG Q&A using SQL filtering over `TestScores` and semantic search over narrative chunks.
3. **Knowledge Base** — Browse indexed documents, clinical summaries, and test-score tables with domain filters.
4. **Audio** — Upload `.m4a`/`.mp3`/`.wav` files; transcribe via MacWhisper CLI, summarize via a local oMLX server.

The standalone app is documented fully in [[summaries/README]]. It complements the main `luria` workspace by providing a turnkey UI for clinicians who do not want to use the CLI.

### Key Design Decisions (Standalone App)

- **Local-first**: PDF parsing, embeddings, vector search, and structured storage all run on-device (macOS). Only the extraction step calls Anthropic's API — and only after PHI redaction. This is a core [[concepts/local-first-architecture]] principle.
- **Dual stores**: SQLite (`data/neuropsych.db`) for relational data (test scores, summaries); [[concepts/lancedb-vector-store]] for semantic retrieval of narrative chunks. A Soul exemplar index lives in a separate `report_soul_index.sqlite`. DuckDB is used for analytics queries in `kb/store/`.
- **Optional local LLM**: oMLX (OpenAI-compatible local server) for chat, summarization, and embeddings as an alternative to cloud providers. See [[concepts/local-llm-inference]] and [[concepts/openai-compatible-api]].
- **[[concepts/retrieval-augmented-generation]]**: The Ask tab queries only ingested data via hybrid SQL + semantic search.

### Standalone App SQLite Schema

- **Documents** — one row per ingested PDF (`doc_id`, `filename`, `ingested_at`).
- **ClinicalSummaries** — narrative summaries per document.
- **TestScores** — structured rows: `test_name`, `subtest_name`, `scaled_score`, `standard_score`, `t_score`, `percentile_rank`, `classification`, `cognitive_domains_affected`.

## Dependency Stack

The pipeline spans three dependency groups:

### Python Core
| Package | Version | Role |
|---|---|---|
| `numpy` | ≥1.24 | Array math, score matrices |
| `pandas` | ≥2.0 | DataFrame I/O |
| `scipy` | ≥1.10 | Stats tests (t-test, ANOVA) |
| `scikit-learn` | ≥1.3 | PCA, cognitive domain clustering |
| `pydantic` | ≥2.0 | Strict validation of `PipelineState` |
| `typer` / `click` | ≥0.9 / ≥8.0 | CLI entry points |
| `rich` | ≥13.0 | Terminal formatting |
| `pyyaml` | ≥6.0 | `luria_config.yaml` loading |

### Application (Streamlit + Agent Pipeline)
| Package | Version | Role |
|---|---|---|
| `langgraph` | latest | Core StateGraph orchestration |
| `langchain` | ≥1.2.15 | LLM abstractions |
| `langchain-anthropic` | ≥1.4.1 | Claude API (cloud extraction fallback) |
| `langchain-huggingface` | ≥1.2.2 | Sentence-transformers bridge |
| `langchain-docling` | latest | Docling PDF loader |
| `lancedb` | latest | Local vector store (`data/vectors/`) |
| `openai` | latest | oMLX local server client (not cloud) |
| `sentence-transformers` | latest | Embeddings (`all-MiniLM-L6-v2`, 384-dim) |
| `streamlit` | ≥1.28 | 4-tab desktop UI |
| `deepagents` | ≥0.5.3 | Multi-agent orchestration (experimental) |
| `haystack-ai` | ≥2.28 | Alternative RAG framework (experimental) |
| `honcho-ai` | ≥2.1.1 | Peer-observation integration (optional) |

### R Packages
The [[concepts/r-neuropsych-packages]] layer includes `dplyr`, `tidyr`, `ggplot2`, `psych`, `lavaan` (SEM), `reticulate`, `knitr`, `rmarkdown`, `jsonlite`, `DBI`, `RSQLite`, and the internal `cingulate` package at `agent/cingulate/`.

See [[concepts/python-environment-management]] and [[concepts/uv-workspace-layout]] for package management via `uv ≥0.4`.

## oMLX Model Configuration

The pipeline supports fully local LLM inference via oMLX, which exposes an OpenAI-compatible `/v1` endpoint:

| Model | Purpose | Dim | Config Key |
|---|---|---|---|
| `medgemma-1.5-4b-it-bf16` | Chat / Extraction (default) | — | `OMLX_CHAT_MODEL` |
| `nomicai-modernbert-embed-base-bf16` | Embeddings + Reranking | 768 | `OMLX_EMBEDDING_MODEL` / `OMLX_RERANK_MODEL` |
| `sentence-transformers/all-MiniLM-L6-v2` | Fallback embeddings | 384 | `VECTOR_EMBEDDING_MODEL` |

The app uses `openai.OpenAI(base_url=...)` to call oMLX — **not** the OpenAI cloud service. See [[concepts/omlx-server]] and [[concepts/local-llm-inference]].

## External CLI Tools

| Tool | Version | Purpose |
|---|---|---|
| **uv** | ≥0.4 | Python package manager (workspace resolver) |
| **Typst** | ≥0.11 | PDF typesetting (forensic template fallback) |
| **Quarto** | ≥1.5 | Report rendering (`neurotyp-*-typst` formats) |
| **MacWhisper** | CLI | Audio transcription (`mw` binary on PATH) |
| **oMLX** | latest | Local LLM server (`pip install mlx-lm`) |
| **R** | ≥4.3 | Statistical computing (`R_HOME` env var) |
| **Git** | ≥2.40 | Version control (conventional commits hook) |

## Architecture Overview (2025-04-28 Refactor)

Following the 2025-04-28 refactor, the Luria architecture was significantly simplified. The old CLI-based orchestration (`app/cli.py`, `app/orchestrator.py`) and visit-scoped agents (`nse_agent.py`, `nt_agent.py`) were removed. The `app/intake/` directory was eliminated and CSV extraction moved to the `cingulate` R package. The **Streamlit app** (`app/streamlit/`) is now the primary entry point.

### Multi-Agent Orchestration Pattern

Instead of explicit CLI agents, the system now uses a skill-based [[concepts/multi-agent-orchestration]] pattern via a `luria-neuropsych-orchestrator` skill:

```
luria-neuropsych-orchestrator
├── luria-case-intake          → Patient data normalization
├── luria-score-processing     → Test score extraction
├── luria-interpretation       → Domain-level analysis
├── luria-report-writing       → Report generation
└── luria-quality-review       → Final validation
```

Each skill delegates to one or more agent layers:
- **Python [[concepts/langgraph-agent-workflows]] nodes** — PDF parse, extract, index, report
- **R6 Domain Processors** — IQ, Memory, Attention, Language, etc.
- **PageIndex service agents** — Document upload, chat, export

See [[concepts/subagent-architecture]] for the decomposition pattern and [[concepts/multi-agent-orchestration]] for broader context.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│  Streamlit Desktop UI  (http://127.0.0.1:8501)               │
│    • Ingest  → drag-and-drop PDFs, watch pipeline stages     │
│    • Ask     → chat panel for RAG Q&A                        │
│    • Knowledge → browse SQLite tables                         │
│    • Audio   → transcribe + summarize                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  LangGraph Orchestrator  (local Python)                       │
│    Ingest:  parse → extract → index → report                 │
│    RAG:     single retrieval node                            │
└─────────────────────────────────────────────────────────────┘
       │           │              │              │
       ▼           ▼              ▼              ▼
┌──────────┐ ┌──────────┐  ┌──────────┐  ┌──────────────┐
│ Docling  │ │ Claude   │  │ SQLite   │  │ Typst CLI    │
│ (local)  │ │ (cloud)  │  │ LanceDB  │  │ Quarto + R   │
│ PDF parse│ │ extract  │  │ (local)  │  │ (optional)   │
└──────────┘ └──────────┘  └──────────┘  └──────────────┘
```

### Active Components

| Component | Path |
|-----------|------|
| Streamlit App | `app/streamlit/app.py` |
| LangGraph Nodes | `app/streamlit/neuropsych_agent/nodes.py` |
| Graph Wiring | `app/streamlit/neuropsych_agent/graph.py` |
| SQLite Store | `app/streamlit/neuropsych_agent/tools/store.py` |
| Soul Context | `app/streamlit/neuropsych_agent/tools/soul_context.py` |
| R Workflow | `agent/cingulate/R/WorkflowRunnerR6.R` |
| Domain Processors | `agent/cingulate/R/DomainProcessorR6.R` |
| PageIndex Service | `rag/page-index/app/service.py` |
| Docling PII | `rag/docling/detect_pii.py` |
| Voice Soul | `voice/soul/neuro_report_soul_agent.py` |

The `rag/docling/` component handles PII obfuscation (renamed from `luria_docling/`). See [[concepts/pii-redaction-pipelines]] and [[concepts/clinical-data-privacy]].

## Repository Structure and Workspace Layout

The `luria` repository is organized as a **uv workspace** with the Streamlit app as a member package. The standalone `luria_streamlit_app` repository mirrors the `app/streamlit/` structure with its own `neuropsych_agent/`, `neuropsych_rag/`, `subagents/`, and `skills/` directories — as documented in [[summaries/README]].

```
luria/                              # workspace root
├── app/
│   └── streamlit/                  # uv workspace member (the app)
│       ├── app.py                  # Streamlit entrypoint
│       ├── neuropsych_agent/       # LangGraph pipeline code
│       │   └── nodes.py
│       ├── skills/                 # skill files (e.g., typst-report-formatter)
│       └── templates/              # Quarto templates
├── agent/
│   └── cingulate/                  # R6-based domain processing
│       └── R/
├── rag/
│   ├── page-index/                 # Document processing service
│   └── docling/                    # PII obfuscation
├── subagents/                      # prompt directories (at workspace root)
│   ├── PDF_Ingestion_Parser/
│   ├── Neuropsych_Data_Extractor/
│   ├── Sheets_Data_Indexer/
│   └── Narrative_Report_Generator/
├── scripts/
│   └── smoke_test_paths.py         # path verification script
├── src/luria/                      # Python package (core, data, analysis, reporting, cli)
├── R/                              # R integration (cingulate package, config)
├── tests/
├── docs/
├── examples/
├── notebooks/
└── data/                           # gitignored
```

See [[concepts/uv-workspace-layout]] and [[concepts/python-project-structure]] for structural conventions.

## Streamlit App: Launch & Management

The Streamlit app is the primary interface and exposes a **4-tab interface**: Ingest, Reference, Ask, Knowledge.

### Start (foreground)
```bash
cd /Users/joey/luria/app/streamlit
uv run streamlit run app.py \
    --server.address 127.0.0.1 \
    --server.port 8501 \
    --browser.gatherUsageStats false
```

### Start (background)
```bash
nohup uv run streamlit run app.py \
    --server.address 127.0.0.1 \
    --server.port 8501 \
    --browser.gatherUsageStats false \
    --server.headless true \
    > /tmp/streamlit.log 2>&1 &
```

### Status & Stop
```bash
curl http://localhost:8501/_stcore/health   # Health check
lsof -i :8501                              # Check process
pkill -f "streamlit.*8501"                # Stop server
tail -f /tmp/streamlit.log                # View logs
```

### Performance Notes
- **First run** is slow: UV creates `.venv` and installs 175+ packages
- **Docling models** download ~500MB on first PDF parse
- **Watchdog** improves file-watching: `pip install watchdog`
- **Local LLM** (MLX/oMLX) may have model load time on startup

See [[concepts/local-llm-inference]] for local model configuration.

## LangGraph Runtime and Path Resolution

The pipeline's four nodes are implemented in `neuropsych_agent/nodes.py` and coordinated by the [[concepts/langgraph-agent-workflows]] runtime. Each node loads its system prompt from a corresponding `AGENTS.md` file under `subagents/`.

### Critical Path Constants (post-recovery)

After the 2025-04-28 recovery, `nodes.py` defines two named constants to avoid ambiguity:

- `APP_ROOT` — resolves to `app/streamlit/` (the Streamlit project root)
- `WORKSPACE_ROOT` — resolves to the actual git/uv workspace root
- `SUBAGENTS_DIR` = `WORKSPACE_ROOT / "subagents"`
- `REPO_ROOT` — retained as a backwards-compatibility alias (used by the optional Typst skill path)

Prior to the fix, `REPO_ROOT` was computed as `Path(__file__).resolve().parent.parent`, which after the workspace reorg resolved to `app/streamlit/` instead of the workspace root. This caused a `FileNotFoundError` on `app/streamlit/subagents/Neuropsych_Data_Extractor/AGENTS.md` and crashed the second pipeline node on every run.

### Smoke Test

A script `scripts/smoke_test_paths.py` verifies all static paths the pipeline relies on, AST-parses every `.py` file, and validates `WORKSPACE_ROOT` math — without requiring heavy dependencies. Run before every demo or funding push:

```bash
python3 scripts/smoke_test_paths.py
# Expected: PASS — all required paths resolve and all .py files parse.
```

See [[concepts/smoke-test-scripts]] for the broader pattern.

## Pipeline Stages

The pipeline advances through four stages, each corresponding to a subagent and its `AGENTS.md` prompt:

| Stage | Tag | Subagent Directory |
|---|---|---|
| 1 | `[parse]` | `PDF_Ingestion_Parser` |
| 2 | `[extract]` | `Neuropsych_Data_Extractor` |
| 3 | `[index]` | `Sheets_Data_Indexer` |
| 4 | `[report]` | `Narrative_Report_Generator` |

### Stage 1: PDF Parse

Docling extracts text and layout from the PDF locally. PHI is redacted before any network call. See [[concepts/docling-pdf-parsing]], [[concepts/pii-redaction-pipelines]], and [[summaries/neuropsych-pdf-parser]].

### Stage 2: Structured Extraction

Claude Sonnet (Anthropic cloud, via `langchain-anthropic ≥1.4.1`) structures the parsed narrative into JSON — test scores, clinical summaries, cognitive domain mappings. The [[summaries/neuropsych-data-extractor]] agent performs this work, producing normalized records that downstream stages consume. oMLX (`medgemma-1.5-4b-it-bf16`) can serve as a local fallback. See also [[concepts/pdf-score-extraction]], [[concepts/long-format-clinical-data]], [[concepts/neuropsychological-test-scores]].

### Stage 3: Index

SQLite stores structured relational data (test scores, summaries); [[concepts/lancedb-vector-store]] stores semantic vector embeddings of narrative chunks using `sentence-transformers` (`all-MiniLM-L6-v2`, 384-dim, or `nomicai-modernbert-embed-base-bf16`, 768-dim via oMLX). Filtering is available by `doc_id` and cognitive domain. DuckDB provides column-store analytics in `kb/store/`.

### Stage 4: Narrative Report Generation

The [[summaries/neuropsych-narrative-writer]] agent consumes the extracted data and produces **per-domain prose narratives** as Quarto include files (`.qmd`). It writes only prose — no tables, no plots. For each cognitive domain present in the data, it emits a file following the stable `_NN-XX_<domain>_text.qmd` naming convention. Multi-rater domains (ADHD, Emotion/Behavior) produce one file per rater. The agent also enforces an [[concepts/edit-protection-pattern]] to avoid overwriting clinician hand-edits.

Optional Typst or Quarto rendering produces print-ready PDFs. The [[concepts/cingulate-engine]] R6 layer assembles all include files — prose, score tables, and plots — and renders via neurotyp-{adult,pediatric,forensic} Quarto extensions.

See also: [[concepts/narrative-report-generation]], [[concepts/modular-report-architecture]], [[concepts/quarto-extensions]], [[concepts/typst-typesetting]].

## LangGraph Entry Points

Defined in `neuropsych_agent/graph.py`:

| Function | Purpose |
|----------|----------|
| `build_ingest_graph()` | 4-stage pipeline: parse → extract → index → report |
| `build_rag_graph()` | Single-node retrieval for Q&A |
| `ingest_pdf(path, mode="ingest", **voice_kw)` | Convenience wrapper used by the UI |
| `ask_rag(query)` | Convenience wrapper for RAG Q&A |

### Pipeline State

`neuropsych_agent/state.py` defines `PipelineState`, a `TypedDict` (validated via `pydantic ≥2.0` strict mode) carrying:

- `messages` — accumulated LangChain messages
- `pdf_path`, `doc_id` — document identity
- `parsed`, `records`, `indexed`, `report` — stage outputs
- `user_query`, `rag_answer` — RAG fields
- `voice_enabled`, `soul_db_path`, `soul_profile_path`, `quarto_format` — Luria Voice options

## Subagent Prompt Map

| Subagent | Path | Node | LLM? |
|---|---|---|---|
| PDF_Ingestion_Parser | `subagents/PDF_Ingestion_Parser/AGENTS.md` | `parse_node` | No (deterministic) |
| Neuropsych_Data_Extractor | `subagents/Neuropsych_Data_Extractor/AGENTS.md` | `extract_node` | Yes (JSON extraction) |
| Sheets_Data_Indexer | `subagents/Sheets_Data_Indexer/AGENTS.md` | `index_node` | No (SQLite/LanceDB) |
| Narrative_Report_Generator | `subagents/Narrative_Report_Generator/AGENTS.md` | `report_node` | Yes (markdown) |

## Honcho AI Integration (Optional)

The standalone app includes an optional peer-observation layer via [[concepts/honcho-ai-peer-observation]]. The `honcho-luria-app.py` example demonstrates creating sessions and enabling `observe_me` peer-chat patterns using `honcho-ai ≥2.1.1`. This allows the system to learn user preferences over time while keeping clinical data local.

## Luria Voice Integration

Luria Voice provides clinician-specific reporting through three layers:

- **BRAND** — `_brand.yml` for logos, colors, typography. See [[concepts/brand-theming]] and [[concepts/brand-typography]].
- **SOUL** — style profile + exemplar RAG from de-identified prior reports (`report_soul_profile.json` + `report_soul_index.sqlite`). The standalone CLI `voice/soul/neuro_report_soul_agent.py` builds the exemplar index. See [[concepts/style-profile-extraction]].
- **STYLE** — Quarto `neurotyp-*-typst` formats (adult, pediatric, forensic). See [[concepts/quarto-extensions]].

Configure via environment variables:
```fish
set -gx VOICE_ROOT ~/voice
set -gx NEUROPSYCH_SOUL_DB ~/path/to/report_style_index.sqlite
set -gx NEUROPSYCH_SOUL_PROFILE ~/path/to/report_style_profile.json
```

## Cloud LLM Providers (Optional)

| Provider | Package | Env Key | Notes |
|---|---|---|---|
| Anthropic | `langchain-anthropic` | `ANTHROPIC_API_KEY` | PHI redacted before call |
| OpenAI | `openai` | `OPENAI_API_KEY` | GPT fallback (optional) |
| LangSmith | built-in | `LANGSMITH_API_KEY` | Tracing (`LANGSMITH_TRACING=true`) |

## Configuration

All runtime configuration is environment-driven. Key variables:

| Variable | Purpose | Default |
|----------|---------|----------|
| `ANTHROPIC_API_KEY` | Claude Sonnet extraction | *(required)* |
| `OMLX_BASE_URL` | Local OpenAI-compatible server | `http://localhost:8000/v1` |
| `OMLX_CHAT_MODEL` | Model ID for local chat | `medgemma-1.5-4b-it-bf16` |
| `OMLX_EMBEDDING_MODEL` | Model ID for local embeddings | `nomicai-modernbert-embed-base-bf16` |
| `OMLX_RERANK_MODEL` | Model ID for reranking | `nomicai-modernbert-embed-base-bf16` |
| `VECTOR_EMBEDDING_MODEL` | Fallback embeddings | `sentence-transformers/all-MiniLM-L6-v2` |
| `NEUROPSYCH_SOUL_DB` | Path to Luria Soul SQLite index | *(optional)* |
| `NEUROPSYCH_SOUL_PROFILE` | Path to style profile JSON | *(optional)* |
| `VOICE_ROOT` | Path to Luria Voice checkout | *(optional)* |
| `PHI_REDACTION_ENABLED` | Strip identifiers before cloud calls | `true` |
| `LLM_PROVIDER` | `omlx` for fully local operation | *(optional)* |
| `LANGSMITH_PROJECT` | LangSmith tracing project name | `neuropsych_rag` |

The toolkit is also configured via `luria_config.yaml` (templates, output formats, CSS styles). PHI safety is enforced via `.gitignore` rules that exclude `data/uploads/`, `data/reports/`, `data/neuropsych.db`, and `data/vectors/` from version control. See [[concepts/yaml-configuration]], [[concepts/local-llm-inference]], and [[concepts/phi-data-handling]].

## Storage Backends

| Backend | Path/URL | Purpose | Schema |
|---|---|---|---|
| SQLite | `data/neuropsych.db` | Structured test scores + summaries | `Documents`, `TestScores`, `ClinicalSummaries` |
| LanceDB | `data/vectors/` | Semantic search vectors | `narrative_chunks` table |
| SQLite (Soul) | `report_soul_index.sqlite` | Style exemplar vectors | `chunks` table |
| DuckDB | `kb/store/` | Analytics queries | Column-store for RAG |

## Known Open Issues

These issues do not break the markdown-only happy path but must be resolved for Typst and Quarto rendering:

1. **Missing Typst skill file** — `skills/typst-report-formatter/SKILL.md` is referenced from `nodes.py` when `state['mode'] == 'typst'`. The code falls back to an empty system prompt if absent, so markdown is unaffected. Source copies exist at `voice/.agents/skills/typst-report-formatter/SKILL.md`. See [[concepts/typst-typesetting]] and [[concepts/fallback-strategy]].

2. **Missing Quarto templates** — `templates/quarto_report/` needs `_quarto.yml`, `_brand.yml`, and `_extensions/`. These can be sourced from `voice/brand/` and `voice/style/_extensions/`. See [[concepts/quarto-extensions]].

3. **Multiple voice/ and rag/ flavors** — Three `voice/` flavors (`brand`, `soul`, `style`) and three `rag/` flavors (`page-index`, `docling`, `openmed`) coexist. Canonical choices deferred post-funding submission.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `ANTHROPIC_API_KEY is not set` warning | Missing env var | Add key to `.env` and re-source |
| Docling parse hangs | First run downloads models | Wait; or pre-download with `docling` CLI |
| Typst/Quarto buttons missing | Binaries not on `PATH` | `brew install typst quarto` |
| MacWhisper transcription fails | `mw` not installed | Install MacWhisper from App Store / Gumroad |
| LanceDB lock errors | Concurrent access | Close other processes using `data/vectors/` |
| oMLX connection refused | Server not running | Start `mlx_lm.server` or verify `OMLX_BASE_URL` |

## Data Model

The pipeline operates on a tidy long-format data structure. The expected tabular schema per row is:

| Field | Description |
|---|---|
| `subject_id` | Participant identifier |
| `test_name` | Name of neuropsychological test (e.g., WAIS-IV_VCI) |
| `domain` | Cognitive domain (e.g., memory, attention, language) |
| `raw_score` | Raw test score |
| `scaled_score` | Scaled score |
| `t_score` | T-score |
| `percentile` | Percentile rank |

Downstream agent stages extend this schema with columns such as `subdomain`, `score_type`, `range`, `rater`, and `age_group`. See [[concepts/long-format-clinical-data]] and [[concepts/neuropsychological-test-scores]].

## Cognitive Domain Taxonomy

The pipeline organizes assessment data into a fixed set of numbered domains:

| Prefix | Domain |
|---|---|
| `_02-01_iq` | General Cognitive Ability / IQ |
| `_02-02_academics` | Academic / Achievement |
| `_02-03_verbal` | Verbal / Language |
| `_02-04_spatial` | Visuospatial / Visual-Construction |
| `_02-05_memory` | Memory & Learning |
| `_02-06_executive` | Executive Function |
| `_02-07_motor` | Sensorimotor |
| `_02-08_social` | Social Cognition |
| `_02-09_adhd` | ADHD (multi-rater) |
| `_02-10_emotion` | Emotion/Behavior (multi-rater) |
| `_02-11_adaptive` | Adaptive Functioning |
| `_03-00_sirf` | Summary, Impressions, Recommendations & Formulation |
| `_03-01_recs` | Recommendations |

This numbering is **stable by design** — `_quarto.yml` and `template.qmd` in the cingulate package depend on it. See [[concepts/cognitive-domains]] and [[concepts/neuropsychological-assessment-pipeline]] for broader context.

## Security & PHI Handling

- **PHI redaction** happens locally (Docling parse stage) before any text is sent to Anthropic (`PHI_REDACTION_ENABLED=true`).
- **No cloud vector store**: [[concepts/lancedb-vector-store]] and SQLite are entirely local.
- **No cloud spreadsheet**: All structured data stays in `data/neuropsych.db`.
- **oMLX local LLM**: All chat/embeddings can run without internet (`LLM_PROVIDER=omlx`).
- **.env isolation**: Secrets are gitignored by default — never commit `.env`.
- **Audio**: MacWhisper runs locally; oMLX summarization runs locally.
- **Action required**: Rotate any API keys that may have been committed before `.gitignore` was finalized.

See [[concepts/phi-data-handling]], [[concepts/clinical-data-privacy]], and [[concepts/pii-redaction-pipelines]].

## Key Design Principles

### Separation of Concerns
Each stage handles one responsibility: extraction, narration, or rendering. The narrative writer never produces tables or scores; the R6 layer never writes prose. This mirrors the [[concepts/modular-report-architecture]] philosophy.

### Data Fidelity
The narrative writer uses the `range` column verbatim from the CSV and never invents score classifications. Raw scores and percentiles are excluded from prose — the R6 layer renders those in tables. See [[concepts/neuropsychological-report-variables]].

### PHI Safety
Upstream stages scrub protected health information before the narrative stage. If real names appear, the narrative writer replaces them with `[PATIENT]` and emits a warning. The toolkit defaults to local processing, with cloud LLM use remaining optional. See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]].

### Clinical Voice
Narratives follow APA-style neuropsychological report conventions with appropriately hedged language. Register adapts to the `age_group` field: pediatric, adult, or forensic. See [[concepts/clinical-communication-register]] and [[concepts/validity-language]].

### Edit Protection
Before overwriting any existing `.qmd` file, the agent checks for clinician hand-edits. If found, the new draft is appended as a comment block rather than replacing the existing content. See [[concepts/edit-protection-pattern]].

## Related Pages

- [[summaries/README_luria]] — pipeline README
- [[summaries/README]] — Luria Streamlit App README (standalone app)
- [[summaries/AGENTS_luria]] — overview of all Luria pipeline agents
- [[summaries/RECOVERY_NOTES]] — path fix, workspace reorg, open issues
- [[summaries/SESSION_SUMMARY_2025-04-28]] — 2025-04-28 architecture refactor session notes
- [[summaries/DEPENDENCIES]] — full dependency and tool reference
- [[summaries/neuropsych-narrative-writer]] — stage 4 narrative agent specification
- [[summaries/neuropsych-data-extractor]] — stage 2 extraction agent
- [[summaries/neuropsych-pdf-parser]] — PDF parsing component
- [[summaries/PROJECT_SETUP_COMPLETE]] — project setup notes
- [[concepts/neuropsychological-reporting]] — broader reporting context
- [[concepts/neuropsychological-assessment-pipeline]] — related pipeline concept
- [[concepts/multi-agent-orchestration]] — how pipeline stages are coordinated
- [[concepts/subagent-architecture]] — agent decomposition pattern used here
- [[concepts/clinical-report-structure]] — report organization principles
- [[concepts/forensic-neuropsychological-evaluation]] — forensic register variant
- [[concepts/clinical-data-management]] — data handling practices
- [[concepts/langgraph-agent-workflows]] — LangGraph runtime used for node orchestration
- [[concepts/uv-workspace-layout]] — uv workspace structure
- [[concepts/smoke-test-scripts]] — path verification before demos
- [[concepts/fallback-strategy]] — graceful degradation when skill files are absent
- [[concepts/pii-redaction-pipelines]] — PII handling via docling component
- [[concepts/retrieval-augmented-generation]] — RAG Q&A layer
- [[concepts/lancedb-vector-store]] — vector store used for semantic retrieval
- [[concepts/docling-pdf-parsing]] — PDF parsing component
- [[concepts/honcho-ai-peer-observation]] — optional peer-observation integration
- [[concepts/local-first-architecture]] — core privacy-first design principle
- [[summaries/SETUP_SUMMARY]] — setup notes

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/installation]]

See also: [[summaries/AGE_OVERRIDE_GUIDE]]

See also: [[summaries/DIAGNOSIS_FIX_SUMMARY]]

See also: [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/LLM Benchmark Comparison]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/agentic-workflows]]