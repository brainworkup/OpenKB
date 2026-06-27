---
sources: [summaries/File Folder Structure Rebuild.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/SETUP_SUMMARY.md, summaries/README.md]
brief: Integrated Python/R software system for neuropsychology data ingestion, analysis, and clinical report generation.
---

# Neuropsychological Toolkit

A **neuropsychological toolkit** is an integrated software system that handles the full pipeline of clinical neuropsychology work: ingesting raw test scores, performing statistical analysis across [[concepts/cognitive-domains]], and producing professional clinical reports. The concept is embodied in the **Luria** project (see [[summaries/README]] and [[summaries/README_luria]]), which serves as a reference implementation of this pattern.

## What is Luria?

Luria is a neuropsychology data analysis and reporting toolkit designed to serve a broad audience:

| Audience | Primary Use Case |
|---|---|
| Clinicians | Clinical report generation from test data |
| Researchers | Statistical analysis of cognitive test scores |
| Developers | Python/R API integration and extension |
| Data Scientists | Batch processing and R integration |

The toolkit is accessible via CLI (`luria --help`) and Python API (`import luria`). GitHub: https://github.com/brainworkup/luria.

---

## Dependency Architecture

Luria's dependencies are organized into four tiers: core Python, application/RAG pipeline, development tooling, and external CLI binaries. Full details are in [[summaries/DEPENDENCIES]].

### Python Core

Required for `import luria` and basic CLI operation:

- **Numeric/stats**: `numpy ≥1.24`, `pandas ≥2.0`, `scipy ≥1.10` — foundation for all score matrices and normative comparisons
- **Visualization**: `matplotlib ≥3.7`, `seaborn ≥0.12`
- **ML**: `scikit-learn ≥1.3` — PCA and classifiers for cognitive domain clustering
- **Validation**: `pydantic ≥2.0` — strict mode for `PipelineState`
- **CLI**: `click ≥8.0`, `typer ≥0.9`, `rich ≥13.0`
- **Config**: `pyyaml ≥6.0`, `python-dotenv ≥1.0`, `tqdm ≥4.65`

### Application Dependencies (Streamlit + Agent Pipeline)

Required for the full ingestion/RAG app at `app/streamlit/`:

- **Orchestration**: `langgraph` (core — `build_ingest_graph()`, `build_rag_graph()`); see [[concepts/langgraph-agent-workflows]]
- **LangChain ecosystem**: `langchain ≥1.2.15`, `langchain-core`, `langchain-anthropic ≥1.4.1`, `langchain-huggingface ≥1.2.2`, `langchain-docling`, `langchain-milvus ≥0.3.3`
- **Vector storage**: `lancedb` (local default at `data/vectors/`); see [[concepts/lancedb-vector-store]]
- **Embeddings**: `sentence-transformers` (default: `all-MiniLM-L6-v2`, 384-dim); see [[concepts/multimodal-embeddings]]
- **LLM client**: `openai` package used for **oMLX local server** (not OpenAI cloud); see [[concepts/omlx-server]] and [[concepts/openai-compatible-api]]
- **UI**: `streamlit ≥1.28` — 4-tab interface (Ingest / Ask / KB / Audio)
- **Experimental**: `deepagents ≥0.5.3` (multi-agent orchestration), `honcho-ai ≥2.1.1` (session tracking), `haystack-ai ≥2.28` (alternative RAG framework)
- **PDF parsing**: `langchain-docling`; see [[concepts/docling-pdf-parsing]]

### Development Tooling

- **Testing**: `pytest ≥7.0`, `pytest-cov ≥4.0`
- **Code quality**: `black ≥23.0`, `ruff ≥0.1`, `mypy ≥1.0`, `pre-commit ≥3.0`
- **Notebooks**: `ipython ≥8.0`, `jupyter ≥1.0`, `jupyterlab ≥4.0`
- **Docs**: `sphinx ≥7.0`, `sphinx-rtd-theme`, `myst-parser ≥2.0`, `sphinx-autodoc-typehints ≥2.0`

### External CLI Tools

| Tool | Purpose | Notes |
|---|---|---|
| **uv ≥0.4** | Python package manager | Workspace resolver; see [[concepts/uv-workspace-layout]] |
| **Typst ≥0.11** | PDF typesetting | Forensic template fallback; see [[concepts/typst-typesetting]] |
| **Quarto ≥1.5** | Report rendering | `neurotyp-*-typst` formats; see [[concepts/quarto]] |
| **MacWhisper** | Audio transcription | `mw` binary on PATH |
| **oMLX** | Local LLM server | OpenAI-compatible `/v1` endpoint; see [[concepts/local-llm-inference]] |
| **R ≥4.3** | Statistical computing | `R_HOME` env var |
| **Git ≥2.40** | Version control | Conventional commits hook |

---

## Installation

Luria supports multiple installation methods to suit different workflows. Full details are documented in [[summaries/installation]].

### Prerequisites
- **Python 3.10+** required (3.11+ recommended)
- Optional package managers: **uv** (recommended) or **pip**
- Linux may require `python3-dev` and `build-essential`

### Quick Installation

```bash
# Recommended: using uv
uv add luria

# Standard: using pip
pip install luria

# With all optional dependencies
pip install luria[full]

# From source (development)
git clone https://github.com/brainworkup/luria.git
cd luria
pip install -e .
```

Alternative package manager support includes **Poetry** (`poetry add luria`), **conda/mamba**, and **Docker** (`python:3.10-slim` base image). See [[concepts/python-environment-management]] for broader context.

### System Requirements

| Level | Python | RAM | Disk |
|---|---|---|---|
| Minimum | 3.10+ | 4GB | 1GB |
| Recommended | 3.11+ | 8GB+ | 2GB+ |

---

## Configuration

Configuration is managed through `.env` files (API keys, database paths) and YAML files (template selection, output formats). A `.env.example` template is provided:

```bash
cp .env.example .env
luria init
```

Key environment variables:

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Claude extraction (optional cloud fallback) |
| `OPENAI_API_KEY` | GPT fallback (optional) |
| `OMLX_BASE_URL` | Local LLM endpoint (e.g., `http://localhost:8000/v1`) |
| `OMLX_CHAT_MODEL` | Default: `medgemma-1.5-4b-it-bf16` |
| `OMLX_EMBEDDING_MODEL` | Default: `nomicai-modernbert-embed-base-bf16` (768-dim) |
| `VECTOR_EMBEDDING_MODEL` | Fallback: `all-MiniLM-L6-v2` (384-dim) |
| `NEUROPSYCH_DB_PATH` | SQLite database path |
| `LANGSMITH_API_KEY` | LangSmith tracing (optional) |
| `PHI_REDACTION_ENABLED` | Strip identifiers before cloud calls |

See [[concepts/yaml-configuration]] and [[concepts/local-llm-inference]] for related configuration patterns.

---

## oMLX Model Configuration

Luria defaults to a fully local LLM stack via oMLX, which exposes an OpenAI-compatible `/v1` endpoint. The app uses `openai.OpenAI(base_url=...)` to call it — not the OpenAI cloud service.

| Model | Purpose | Dim |
|---|---|---|
| `medgemma-1.5-4b-it-bf16` | Chat/Extraction (default) | — |
| `nomicai-modernbert-embed-base-bf16` | Embeddings + Reranking | 768 |
| `sentence-transformers/all-MiniLM-L6-v2` | Fallback embeddings | 384 |

See [[concepts/omlx-server]], [[concepts/mlx-framework]], and [[concepts/local-first-architecture]] for architectural context.

---

## Core Pipeline Stages

### 1. Data Ingestion and Processing
The toolkit accepts multiple data formats (CSV, Excel, JSON, Parquet, SQLite) and normalizes them into a consistent long-format structure. Each record captures subject identifier, test name, cognitive domain, and score variants: raw, scaled, T-score, and percentile. This aligns with the [[concepts/long-format-clinical-data]] pattern.

### 2. Statistical Analysis
Analysis capabilities include descriptive statistics, t-tests, ANOVA, correlation, regression, and normative comparisons via `scipy.stats`. Domain-level profiling supports impairment classification — a key step in the [[concepts/neuropsychological-assessment-pipeline]].

### 3. Agent Pipeline (LangGraph)

Two `StateGraph` pipelines orchestrate the full workflow:

**IngestGraph**: `START → parse → extract → index → report → END`
- `parse_node`: Docling PDF → text (local, deterministic) — see [[concepts/docling-pdf-parsing]]
- `extract_node`: LLM → JSON records (oMLX or Claude)
- `index_node`: SQLite + LanceDB writes
- `report_node`: LLM → markdown + optional Typst/Quarto

**RAGGraph**: `START → rag → END`
- Semantic search (LanceDB) + SQL (`TestScores`) → LLM synthesis with citations

**State object**: `PipelineState(TypedDict)` carries messages, pdf_path, doc_id, parsed, records, indexed, report, user_query, rag_answer, plus Voice keys.

See [[concepts/langgraph-agent-workflows]], [[concepts/retrieval-augmented-generation]], and [[concepts/subagent-architecture]] for related patterns.

### 4. Subagents

| Subagent | Node | LLM? |
|---|---|---|
| PDF_Ingestion_Parser | `parse_node` | No (deterministic) |
| Neuropsych_Data_Extractor | `extract_node` | Yes (JSON extraction) |
| Sheets_Data_Indexer | `index_node` | No (SQLite/LanceDB) |
| Narrative_Report_Generator | `report_node` | Yes (markdown) |

### 5. Report Generation
The toolkit generates clinical reports in HTML, PDF, DOCX, and Markdown using template-driven rendering. Multiple report types are supported (Adult, Pediatric, Forensic, Research), reflecting the [[concepts/modular-report-architecture]] approach. Forensic-specific templates connect to the [[concepts/forensic-neuropsychological-evaluation]] use case. See [[summaries/report-generation]] and [[summaries/template-system]].

### 6. Visualization
Visualizations include score profiles, domain heatmaps, longitudinal plots, and normative comparisons — critical for conveying findings in the [[concepts/clinical-report-structure]].

---

## Storage Backends

| Backend | Path | Purpose |
|---|---|---|
| SQLite | `data/neuropsych.db` | Structured test scores + summaries (`Documents`, `TestScores`, `ClinicalSummaries`) |
| LanceDB | `data/vectors/` | Semantic search vectors (`narrative_chunks` table) |
| SQLite (Soul) | `report_soul_index.sqlite` | Style exemplar vectors |
| DuckDB | `kb/store/` | Analytics / RAG column-store |

See [[concepts/lancedb-vector-store]], [[concepts/sqlite-as-vector-store]], and [[concepts/duckdb-as-vector-store]].

---

## R Dependencies

Luria bridges Python and R via `reticulate` and the `cingulate` R package:

- **Tidyverse**: `dplyr`, `tidyr`, `ggplot2`
- **Psychometrics**: `psych`, `lavaan` (SEM / confirmatory factor analysis)
- **Interop**: `reticulate`, `jsonlite`, `DBI`, `RSQLite` (reads `neuropsych.db`)
- **Reporting**: `knitr`, `rmarkdown`
- **Internal**: `cingulate` — custom neuropsych analysis package at `agent/cingulate/`

See [[concepts/r-python-integration]], [[concepts/cingulate-engine]], and [[concepts/r-neuropsych-packages]].

---

## Voice Stack

Luria includes a layered "Voice" system for report style consistency:

- **BRAND** (`voice/brand/`): Logos, colors, `_brand.yml` (Quarto brand extension)
- **SOUL** (`voice/soul/`): Style profile + exemplar RAG (`report_soul_profile.json` + SQLite index)
- **STYLE** (`voice/style/`): Quarto Typst formats (`neurotyp-adult-typst`, etc.)

The Soul agent (`voice/soul/neuro_report_soul_agent.py`) is a standalone CLI for building the exemplar index from de-identified prior reports. See [[concepts/style-profiles]] and [[concepts/narrative-report-generation]].

---

## Project Structure

As of the April 2026 reorganization (see [[summaries/SETUP_SUMMARY]]), the Luria project follows a clean, modern layout:

```
src/luria/
├── core/           # Configuration, logging, utilities
├── data/           # Data loading, cleaning, validation
├── analysis/       # Statistical analysis, visualization
├── reporting/      # Report generation, templates
└── cli.py          # Unified command-line interface
```

Supporting directories: `R/` for R-specific code, `data/raw/` and `data/processed/`, `tests/`, `docs/`, `examples/`, `notebooks/`, `scripts/`, and `voice/` for the brand/soul/style stack. This matches the [[concepts/python-project-structure]] and [[concepts/monorepo-workspace-layout]] patterns.

---

## Architectural Characteristics

### Privacy-First Design
Processing is local by default. PHI redaction occurs at the Docling parse stage — identifiers are stripped before any cloud call (`PHI_REDACTION_ENABLED=true`). SQLite and LanceDB remain on local disk only. All chat/embeddings can run without internet via oMLX (`LLM_PROVIDER=omlx`). API keys are managed via `.env` files (gitignored). This reflects the [[concepts/privacy-first-software]] principle and [[concepts/phi-data-handling]] practices. See also [[concepts/pii-redaction-pipelines]].

### Local-First Architecture
The default model stack (oMLX + LanceDB + SQLite) requires no cloud connectivity, supporting airgapped clinical environments. See [[concepts/local-first-architecture]].

### LLM Provider Abstraction
Cloud providers (Anthropic Claude, OpenAI GPT) are optional fallbacks. LangSmith tracing is available when `LANGSMISH_TRACING=true`. See [[concepts/llm-provider-abstraction]] and [[concepts/fallback-strategy]].

### Development Tooling
- **Package management**: `uv` (recommended), `pyproject.toml`
- **Task automation**: `Makefile` (`make test`, `make format`, `make lint`, `make build`)
- **Code quality**: `pre-commit`, `black`, `ruff`, `mypy --strict`
- **Testing**: `pytest` with coverage
- **Scripts**: `scripts/setup.sh`, `scripts/build.sh`, `scripts/migrate_components.py`

---

## Quality Assurance

- Code formatting: `black`, `ruff`
- Static type checking: `mypy --strict`
- Testing: `pytest` with `pytest-cov`
- Pre-commit hooks for continuous quality enforcement
- Smoke tests via `tests/test_basic.py`

See [[concepts/report-review-qa]] and [[concepts/smoke-test-scripts]].

---

## Troubleshooting

Common issues and resolutions:
- **"Command not found: luria"**: Add Python scripts directory to PATH
- **Permission errors**: Use `--user` flag or a virtual environment
- **Version conflicts**: Create a fresh virtual environment; use `pip check`
- **R integration issues**: Ensure R is in PATH, `R_HOME` is set in `.env`, and R packages are installed

Support: https://github.com/brainworkup/luria/issues

---

## Related Pages
- [[concepts/neuropsychological-reporting]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/narrative-report-generation]]
- [[concepts/clinical-data-management]]
- [[concepts/python-environment-management]]
- [[concepts/r-python-integration]]
- [[concepts/langgraph-agent-workflows]]
- [[concepts/local-first-architecture]]
- [[concepts/phi-data-handling]]
- [[concepts/subagent-architecture]]
- [[summaries/DEPENDENCIES]]
- [[summaries/installation]]
- [[summaries/README]]
- [[summaries/README_luria]]
- [[summaries/SETUP_SUMMARY]]
- [[summaries/report-generation]]
- [[summaries/template-system]]

See also: [[summaries/File Folder Structure Rebuild]]