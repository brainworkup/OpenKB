---
sources: [summaries/agentic-workflows.md, summaries/File Folder Structure Rebuild.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/CLAUDE.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/report-template.md, summaries/README.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/OCR_PDF_GUIDE.md, summaries/FIX_EXPLANATION.md, summaries/report-generation.md, summaries/template-system.md, summaries/overview.md, summaries/001-choose-quarto-typst.md, summaries/brand-yml-spec.md, summaries/brand-yml-in-r.md, summaries/SKILL.md, summaries/project-setup-progress.md, summaries/deepagents_merged_mem_notes.md, summaries/SETUP_SUMMARY.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/PROJECT_SETUP_COMPLETE.md]
brief: Combining R and Python in one pipeline to blend stats, tooling, and automation.
---

# R-Python Integration

R-Python integration is the practice of combining R and Python within a single project, application, or workflow so each language handles the work it is best suited for. In practice, this often means using R for statistical modeling, psychometrics, and mature domain-specific analysis packages, while Python manages data engineering, orchestration, CLI tooling, application logic, and agent-based automation.

In the Luria ecosystem, this is not just a convenience pattern but an architectural requirement: clinically relevant scoring and analysis may live in R, while surrounding ingestion, workflow control, reporting, and automation are Python-led. This makes R-Python integration central to [[concepts/luria-neuropsych-pipeline]], [[concepts/neuropsychological-assessment-pipeline]], and emerging [[concepts/multi-agent-orchestration]] workflows.

## Why Combine R and Python?

| Concern | R Strengths | Python Strengths |
|---|---|---|
| Statistical modeling | Extensive CRAN ecosystem, mixed-effects models | scikit-learn, statsmodels |
| Neuropsychology | Specialized packages (e.g., cingulate, psych, lavaan) | Flexible data pipelines |
| Reporting | R Markdown, knitr, gt tables | Jinja2, Markdown, PDF generation |
| Deployment | Shiny apps | FastAPI, CLI tools, containerization |
| Scripting & automation | Limited | Strong orchestration, shell, workflow tooling |
| Agent pipelines | Domain analysis tools callable from R | LLM orchestration, tool use, retries, state |

Neither language dominates every domain. Integration allows each to do what it does best, especially in workflows where Python coordinates a multi-step process and R performs specialized analytical work.

## R Packages in the Luria Ecosystem

The DEPENDENCIES document provides the full canonical list of R packages used across the Luria workspace:

| Package | Purpose | Notes |
|---|---|---|
| `dplyr` | Data manipulation | Tidyverse |
| `tidyr` | Data tidying | Wide ↔ long transforms |
| `ggplot2` | Visualization | Cognitive profile plots |
| `psych` | Psychometric functions | Score descriptives |
| `lavaan` | Structural equation modeling | Confirmatory factor analysis |
| `reticulate` | Python bridge | Call Python from R |
| `knitr` | Dynamic reports | R Markdown engine |
| `rmarkdown` | Report generation | HTML/PDF/DOCX output |
| `jsonlite` | JSON I/O | Read extracted records |
| `DBI` | Database interface | SQLite from R |
| `RSQLite` | SQLite driver | Read `neuropsych.db` |
| `cingulate` | Neuropsych analysis | Custom internal package at `agent/cingulate/` |

See [[concepts/r-neuropsych-packages]] for broader discussion of R packages in neuropsychological analysis, and [[concepts/cingulate-engine]] for the custom `cingulate` package architecture.

## Python Dependencies That Interface with R

On the Python side, several packages mediate interaction with R outputs or replicate R-style statistical functionality:

- **`scipy ≥1.10`**: statistical tests (`t-test`, `ANOVA`) via `scipy.stats` for normative comparisons
- **`scikit-learn ≥1.3`**: ML classifiers and PCA for cognitive domain clustering
- **`pandas ≥2.0`**: DataFrame I/O in CSV/Excel/Parquet, the primary interchange layer with R
- **`numpy ≥1.24`**: array math and score matrices
- **`seaborn ≥0.12`** and **`matplotlib ≥3.7`**: visualization layers paralleling R's `ggplot2`

In more automated systems, Python also commonly provides the orchestration layer for long-running workflows, including tool invocation, validation, retries, and state tracking. That makes R outputs natural inputs to Python-managed pipelines such as [[concepts/langgraph-agent-workflows]], [[concepts/agent-pipeline-state-management]], and [[concepts/subagent-architecture]].

## Integration Patterns

### 1. Subprocess / Shell Calls
Python scripts invoke R scripts via `subprocess.run(["Rscript", "analysis.R"])`. This is simple and robust, but coarse-grained; data must usually be serialized to disk between steps.

This pattern is especially useful in pipeline architectures where Python acts as the top-level orchestrator and R is treated as a specialized worker. In agentic systems, it maps cleanly onto [[concepts/orchestrator-worker-pattern]] and [[concepts/multi-agent-orchestration]]: Python delegates analysis to an R subprocess, captures outputs, and decides what to do next.

### 2. `rpy2` Library
The `rpy2` Python package embeds an R interpreter inside a Python process, allowing direct object exchange. This is suitable for tighter coupling, lower-latency calls, or cases where file-based handoff would be too slow or awkward.

This pattern is useful when R functions are being treated like callable tools inside a Python-led application, including [[concepts/clinical-ai-copilot]] or [[concepts/neuropsychological-assessment-automation]] workflows.

### 3. `reticulate` Library
The `reticulate` R package allows R to import and call Python modules directly, exchanging objects in memory. This is the pattern used in the Luria project for consuming the `luria` Python package from within an R session:

```r
library(reticulate)
luria <- import("luria")
results <- cingulate::analyze_neuropsych_data("data/processed/scores.parquet")
```

The `reticulate` package is a declared R dependency in the Luria installation. It is configured via the `R_HOME` environment variable set in `.env`.

### 4. Shared File Formats
Both languages read and write common formats such as CSV, Parquet, JSON, and HDF5, enabling loose coupling with clear boundaries. The Luria project uses Parquet as its primary interchange format — see [[concepts/parquet-as-knowledge-store]]. SQLite (`data/neuropsych.db`) is also shared: R reads it via `DBI`/`RSQLite`, and Python writes to it through pipeline stages. This is often the most portable and debuggable integration pattern.

In agentic workflows, shared files and databases are also a practical state boundary. They let one step finish cleanly before another begins, improve auditability, and support resumable execution. This relates to [[concepts/persistent-memory]], [[concepts/clinical-data-management]], and [[concepts/per-patient-workspace]].

### 5. Separate Microservices
R runs as an API, for example via Plumber or OpenCPU, and Python calls it over HTTP. This maximizes language independence and supports [[concepts/deployment-automation]]. It also fits evented or service-oriented systems where analytical capability is exposed as a remote tool.

### 6. Workspace / Monorepo Layout
Both language trees live in the same repository with a shared configuration layer. This is the pattern used in the Luria project, as documented in [[summaries/PROJECT_SETUP_COMPLETE]], [[summaries/SETUP_SUMMARY]], and [[summaries/project-setup-progress]].

This layout is especially helpful when Python and R components participate in one coordinated pipeline, because configuration, tests, scripts, and documentation can be managed together. See also [[concepts/monorepo-workspace-layout]] and [[concepts/uv-workspace-layout]].

## Installation and Environment Setup

A dual R-Python project requires both runtimes to be installed and discoverable. The Luria installation process, documented in [[summaries/installation]], illustrates the typical approach.

### Python Environment

Python 3.10+ is required. The preferred Python package manager is **uv**, with pip as an alternative:

```bash
# Recommended
uv add luria

# Or with pip
pip install luria
pip install luria[full]  # all optional dependencies
```

For development, a virtual environment is created and development dependencies installed:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

Development tooling includes `pytest`, `black`, `ruff`, `mypy`, `pre-commit`, `ipython`, `jupyter`, and `jupyterlab`. See [[concepts/python-environment-management]] for broader discussion of virtual environments, uv, pip, and related tooling.

### R Environment

R ≥4.3 is required for integration features. Key R packages for the Luria pipeline:

```bash
Rscript -e "install.packages(c('dplyr', 'tidyr', 'ggplot2', 'psych', 'lavaan', 'reticulate', 'knitr', 'rmarkdown', 'jsonlite', 'DBI', 'RSQLite'), repos='https://cloud.r-project.org')"
```

The `R_HOME` path must be configured in the shared `.env` file so both the Python side and IDE tools can locate the R installation:

```bash
echo "R_HOME=$(R RHOME)" >> .env
```

### Unified Setup Scripts

The Luria project provides a unified `scripts/setup.sh` that initializes both environments, mirroring the broader principle of single-entrypoint development automation. The `Makefile` exposes common tasks across both languages:

```bash
make test      # Run tests
make format    # Format code
make build     # Build package
make check-all # Full quality check suite
```

### Environment Configuration

Luria uses `.env` for secrets and runtime paths shared across both languages:

```bash
# API Keys
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# Local LLM configuration
OMLX_BASE_URL=http://localhost:8000/v1
OMLX_CHAT_MODEL=medgemma-1.5-4b-it-bf16
OMLX_EMBEDDING_MODEL=nomicai-modernbert-embed-base-bf16

# Database paths
NEURUPSYCH_DB_PATH=./data/neuropsych.db

# R configuration
R_HOME=/usr/lib/R
```

This `.env`-based configuration approach is especially important in clinical settings where API keys, runtime paths, and patient-data locations must remain private. See [[concepts/yaml-configuration]] for broader configuration patterns, and [[concepts/clinical-data-privacy]] for the privacy considerations that motivate strict environment hygiene.

## Luria Project Example

The Luria neuropsychology project provides a concrete illustration of R-Python integration for neuropsychological assessment tooling. The major repository reorganization completed April 28, 2026 — tracked across [[summaries/PROJECT_SETUP_COMPLETE]], [[summaries/SETUP_SUMMARY]], and [[summaries/project-setup-progress]] — documents the full architecture. The README at [[summaries/README_luria]] and [[summaries/README]] describes the public-facing capabilities.

Just as importantly, the project motivates an agentic workflow model: Python can orchestrate ingestion, processing, redaction, scoring, and reporting, while R supplies domain-specific statistical and psychometric computation. This makes the Luria example relevant not only as a language-integration pattern but also as a foundation for [[concepts/neuropsychological-assessment-workflow]], [[concepts/neuropsychological-assessment-automation]], and [[concepts/clinical-narrative-generation]].

### Directory Structure

```
luria/
├── src/luria/          # Python package (core, data, analysis, reporting, cli)
├── R/                  # R integration (cingulate support, config.R)
├── engine/cingulate/   # Cingulate R package (migrated from agent/cingulate/)
├── tests/              # Test suite (pytest)
├── docs/               # Documentation
├── examples/           # Example scripts and notebooks
├── notebooks/          # Jupyter notebooks
├── scripts/            # Development automation scripts
└── data/               # Raw and processed data (gitignored)
    ├── raw/
    └── processed/
```

### Language Responsibilities

- **Python** handles the core library (`src/luria/`), CLI, configuration management, data pipelines, orchestration, validation, and reporting modules. It is structured using the modern `src/` layout pattern for Python packaging; see [[concepts/python-project-structure]].
- **R** handles specialized cingulate computations, psychometric scoring (`psych`, `lavaan`), visualization (`ggplot2`), and portions of report generation (`knitr`, `rmarkdown`). The `reticulate` bridge allows R sessions to directly invoke the `luria` Python package. The `RSQLite`/`DBI` stack allows R to query `data/neuropsych.db` directly.
- **Shared storage and configuration** provide the handoff boundary between steps, supporting resumability and auditability.
- **Development tooling** (Makefile, `scripts/`) orchestrates both languages in a unified workflow.

In an agentic version of this architecture, Python functions as the planner and dispatcher while R acts as a specialized analytical tool. That division aligns naturally with [[concepts/orchestrator-worker-pattern]], [[concepts/subagent-architecture]], and [[concepts/silent-operation]] for low-risk automated steps.

### Python Submodules and Their Relation to R

The `src/luria/` package is organized to cleanly interface with R outputs:

| Submodule | Role |
|---|---|
| `core/` | Configuration (`config.py`) and shared utilities |
| `data/` | Loading, cleaning, and validating data produced or consumed by R scripts |
| `analysis/` | Statistical analysis and visualization, potentially wrapping R results |
| `reporting/` | Rendering R-computed scores into structured clinical reports |
| `cli.py` | CLI entry point exposing `luria init`, `luria process`, `luria analyze`, `luria report` |

### Key Integration Files

| File | Role |
|---|---|
| `R/config.R` | R-side configuration, paths, and package loading |
| `R/cingulate/` | Cingulate-specific R computations |
| `src/luria/core/config.py` | Python-side configuration, mirroring R config where needed |
| `src/luria/cli.py` | CLI entry point for the Luria package |
| `scripts/setup.sh` | Unified environment setup for both languages |
| `scripts/build.sh` | Build and packaging script |
| `scripts/migrate_components.py` | Migration helper for legacy code reorganization |
| `Makefile` | Common development tasks across both languages |

### Configuration and Environment

Luria uses `.env` for secrets and runtime paths and `luria_config.yaml` for reporting template and format configuration:

```yaml
# luria_config.yaml
reporting:
  templates:
    custom_adult:
      template: "templates/custom_adult.Rmd"
      styles: "styles/custom.css"
  formats: ["html", "pdf", "docx"]
```

The Python side uses uv for dependency management; R uses `renv` or direct package installation. Pre-commit hooks enforce Python code quality. See [[concepts/yaml-configuration]] for broader configuration patterns.

### Storage Backends Shared Across Languages

Both R and Python access the same storage layer:

| Backend | Path | R Access | Python Access |
|---|---|---|---|
| SQLite | `data/neuropsych.db` | `DBI` + `RSQLite` | pipeline writes and reads |
| LanceDB | `data/vectors/` | — | Semantic search vectors |
| SQLite (Soul) | `report_soul_index.sqlite` | — | Style exemplar vectors |
| DuckDB | `kb/store/` | — | Analytics / RAG column-store |

The `neuropsych.db` SQLite database is the primary structured data store accessible to both languages.

### Project Setup Phases

The reorganization of the Luria project followed a structured, phased approach:

1. **Analysis and Planning** — audited existing structure and identified the need for cleaner Python/R separation
2. **Directory Structure** — established `src/luria/` for Python and `R/cingulate/` for R components
3. **Configuration Files** — created `pyproject.toml`, `.env.example`, `.pre-commit-config.yaml`, and `R/config.R`
4. **Development Scripts** — added `Makefile`, `scripts/setup.sh`, `scripts/build.sh`, and `src/luria/cli.py`
5. **Git Setup** — produced a comprehensive `.gitignore`, `scripts/git_setup.sh`, `.gitattributes`, and `README.md`
6. **Documentation** — built `docs/README.md` and supporting guides
7. **Testing** — established basic verification for structure, file existence, and imports

### Migration Strategy

The Luria setup preserved original code in `backup_20260428/` and specified a gradual migration:

| Original Location | New Location |
|---|---|
| `agent/cingulate/` | `engine/cingulate/` |
| `app/streamlit/` | `examples/streamlit_app/` |
| `rag/` | `src/luria/rag/` |
| `kb/` | Integration TBD |
| `voice/` | Separate module |

### User Personas

The Luria project's dual-language structure accommodates multiple user types:

- **Clinicians**: focus on reporting outputs and the CLI; customize report templates and cognitive domain mappings
- **Researchers**: extend `analysis/` with statistical methods and research database integration
- **Developers**: add data formats in `data/`, new CLI commands, and additional analysis modules
- **Workflow architects**: design end-to-end pipelines where Python manages sequencing and R handles specialized scoring steps

## IDE Support: Positron

The [[concepts/positron-ide]] is an IDE built specifically with R-Python integration in mind. It provides strong support for both languages in a single environment.

### Positron Databot

Positron includes an AI assistant feature called Databot. Setting up Databot in an R-Python project requires attention to several configuration details, documented in [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]]:

- **R Runtime**: Databot operates on R-based projects, so R must be installed and on the system PATH
- **Python Version**: the workspace `.python-version` file must point to an installed Python version
- **API Key**: Databot uses an Anthropic API key set via `ANTHROPIC_API_KEY`
- **Model Selection**: Claude Sonnet 4 is recommended
- **Ollama Integration**: local providers require at least one pulled model before the provider is usable

### Workspace Configuration for Dual-Language Projects

A `.vscode/settings.json` in a Positron project should configure interpreters for both languages explicitly:

```json
{
  "positron.interpreter.r": {
    "enabled": true,
    "path": "/usr/bin/R"
  },
  "positron.interpreter.python": {
    "enabled": true,
    "path": "${workspaceFolder}/.venv/bin/python"
  }
}
```

This mirrors the broader principle of explicit, version-pinned interpreter configuration, avoiding ambiguity when both runtimes coexist in the same workspace.

## Verification

After installing both runtimes, verifying the integration is working correctly requires checking both sides:

```bash
# Python side
luria --version
python -c "import luria; print(luria.__version__)"

# R side
Rscript -e "library(reticulate); luria <- import('luria'); print('R-Python bridge OK')"
```

For workflow automation, verification should also test the handoff boundary: can Python call R, can R read shared inputs, and can outputs return in a validated format for downstream stages such as reporting or redaction review.

## Challenges

- **Environment management**: R uses `renv` or `packrat`; Python uses `venv` and uv. Keeping both in sync requires discipline and unified setup scripts.
- **Runtime availability**: both R and Python must be installed and discoverable by the IDE and CI environment. As shown in [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]], missing or mismatched runtimes are a common failure point.
- **Data serialization**: agreeing on interchange formats is critical; Parquet and SQLite are practical bridges. See [[concepts/parquet-as-knowledge-store]].
- **Dependency installation**: CI/CD pipelines must install both R and Python dependencies, relevant to [[concepts/deployment-automation]].
- **Type safety**: there is no shared type system, so schemas must be documented or validated explicitly.
- **Debugging**: stack traces do not cross the language boundary cleanly.
- **Migration complexity**: moving legacy mixed-language codebases into clean structures requires careful planning.
- **Testing**: full test suites spanning both languages require both environments to be installed.
- **State management**: long-running pipelines need clear handoff points, checkpoints, and recovery behavior across language boundaries; see [[concepts/agent-pipeline-state-management]] and [[concepts/persistent-memory]].
- **Retry semantics**: a Python orchestrator may need to distinguish transient failures from deterministic R-side errors before retrying.
- **Human review boundaries**: in clinical settings, some outputs can be automated silently while others, such as sensitive redaction or final report sign-off, should surface for review. See [[concepts/silent-operation]] and [[concepts/phi-deidentification-pipeline]].
- **Security**: `.env` files containing API keys and data paths must never be committed to version control.

## Relation to Neuropsychological Reporting

In clinical and research neuropsychology, R's psychometric packages (`psych`, `lavaan`, `cingulate`) provide validated implementations of scoring and analysis methods. Python handles the surrounding infrastructure: ingesting raw test data, orchestrating multistep processing, coordinating calls into R, managing state, and rendering outputs into structured reports.

This division is especially important in agentic workflows. A Python-led system can automate stages such as data ingestion, data processing, PHI/PII redaction, table creation, figure generation, and report assembly, while R remains the execution environment for domain-specific statistical logic. That makes R-Python integration a key enabling layer for [[concepts/neuropsychological-assessment-automation]], [[concepts/clinical-narrative-generation]], [[concepts/pii-redaction-pipelines]], and [[concepts/phi-data-handling]].

The Luria toolkit exemplifies this division: R computes cingulate domain scores; Python packages them into HTML, PDF, or DOCX reports via customizable templates rendered by Quarto and Typst. See [[concepts/neuropsychological-reporting]] and [[concepts/narrative-report-generation]] for the reporting side of this pipeline.

## Related Concepts

- [[concepts/neuropsychological-reporting]] — end-to-end reporting pipelines that consume R/Python integration outputs
- [[concepts/narrative-report-generation]] — generating clinical narrative text from structured neuropsychological scores
- [[concepts/luria-neuropsych-pipeline]] — the broader Luria pipeline that R-Python integration underpins
- [[concepts/cingulate-engine]] — the R-based cingulate computation component central to the Luria project's R side
- [[concepts/r-neuropsych-packages]] — R packages for psychometric and neuropsychological analysis
- [[concepts/r-visualization-theming]] — ggplot2 and R-based visualization in clinical reporting
- [[concepts/deployment-automation]] — automating environments that must install and run both R and Python
- [[concepts/python-project-structure]] — modern Python packaging conventions used in hybrid projects
- [[concepts/python-environment-management]] — virtual environments and dependency management for Python
- [[concepts/positron-ide]] — IDE with first-class support for both R and Python
- [[concepts/ide-ai-assistant-configuration]] — configuring AI coding assistants in dual-language workspaces
- [[concepts/local-llm-inference]] — local model providers that may be configured alongside dual-language pipelines
- [[concepts/yaml-configuration]] — workspace and environment configuration patterns used across both ecosystems
- [[concepts/parquet-as-knowledge-store]] — Parquet as a primary interchange format between R and Python components
- [[concepts/privacy-first-software]] — clinical data handled by these pipelines carries strong privacy requirements
- [[concepts/clinical-data-privacy]] — privacy requirements specific to neuropsychological and clinical data
- [[concepts/retrieval-augmented-generation]] — Python-native pattern sometimes used alongside R analysis in hybrid research tools
- [[concepts/omlx-server]] — local LLM server configured via shared `.env`
- [[concepts/langgraph-agent-workflows]] — Python orchestration layers that may call R outputs via shared storage or tools
- [[concepts/orchestrator-worker-pattern]] — a useful model for Python-led coordination of R analysis steps
- [[concepts/multi-agent-orchestration]] — broader agent pipeline pattern relevant when R functions are treated as specialized tools
- [[concepts/subagent-architecture]] — decomposition of workflows into specialized executors
- [[concepts/agent-pipeline-state-management]] — tracking intermediate state across Python/R boundaries
- [[concepts/persistent-memory]] — durable storage of workflow state and intermediate outputs
- [[concepts/phi-deidentification-pipeline]] — deidentification workflows that often coexist with scoring and reporting

See also: [[summaries/installation]] · [[summaries/SESSION_SUMMARY_2025-04-28]] · [[summaries/SETUP_SUMMARY]] · [[summaries/PROJECT_SETUP_COMPLETE]] · [[summaries/project-setup-progress]] · [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]] · [[summaries/README_luria]] · [[summaries/README]] · [[summaries/DEPENDENCIES]]

See also: [[summaries/deepagents_merged_mem_notes]] · [[summaries/SKILL]] · [[summaries/brand-yml-in-r]] · [[summaries/brand-yml-spec]] · [[summaries/001-choose-quarto-typst]] · [[summaries/overview]] · [[summaries/template-system]] · [[summaries/report-generation]] · [[summaries/FIX_EXPLANATION]] · [[summaries/OCR_PDF_GUIDE]]

See also: [[summaries/report-template]] · [[summaries/requirements]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/File Folder Structure Rebuild]]

See also: [[summaries/agentic-workflows]]