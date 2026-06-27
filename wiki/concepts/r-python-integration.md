---
sources: [summaries/File Folder Structure Rebuild.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/CLAUDE.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/report-template.md, summaries/README.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/OCR_PDF_GUIDE.md, summaries/FIX_EXPLANATION.md, summaries/report-generation.md, summaries/template-system.md, summaries/overview.md, summaries/001-choose-quarto-typst.md, summaries/brand-yml-spec.md, summaries/brand-yml-in-r.md, summaries/SKILL.md, summaries/project-setup-progress.md, summaries/deepagents_merged_mem_notes.md, summaries/SETUP_SUMMARY.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/PROJECT_SETUP_COMPLETE.md]
brief: Combining R's psychometric/statistical ecosystem with Python's engineering capabilities in a single project.
---

# R-Python Integration

R-Python integration refers to the practice of combining R — a language purpose-built for statistical computing and data analysis — with Python in a single project or pipeline. This approach lets teams leverage R's rich ecosystem of statistical and psychometric packages alongside Python's general-purpose capabilities for data engineering, machine learning, CLI tooling, and deployment.

## Why Combine R and Python?

| Concern | R Strengths | Python Strengths |
|---|---|---|
| Statistical modeling | Extensive CRAN ecosystem, mixed-effects models | scikit-learn, statsmodels |
| Neuropsychology | Specialized packages (e.g., cingulate, psych, lavaan) | Flexible data pipelines |
| Reporting | R Markdown, knitr, gt tables | Jinja2, Markdown, PDF generation |
| Deployment | Shiny apps | FastAPI, CLI tools, containerization |
| Scripting & automation | Limited | Strong (Makefile, shell, Python scripts) |

Neither language dominates every domain — integration allows each to do what it does best.

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

- **`scipy ≥1.10`**: Stats tests (`t-test`, `ANOVA`) via `scipy.stats` for normative comparisons — analogous to R's base statistical functions
- **`scikit-learn ≥1.3`**: ML classifiers and PCA for cognitive domain clustering
- **`pandas ≥2.0`**: DataFrame I/O in CSV/Excel/Parquet — the primary interchange format with R
- **`numpy ≥1.24`**: Array math and score matrices
- **`seaborn ≥0.12`** and **`matplotlib ≥3.7`**: Visualization layers paralleling R's `ggplot2`

These are all required for `import luria` and basic CLI operation, establishing the numeric/statistical foundation of the Python side.

## Integration Patterns

### 1. Subprocess / Shell Calls
Python scripts invoke R scripts via `subprocess.run(["Rscript", "analysis.R"])`. Simple but coarse-grained; data must be serialized to disk (CSV, RDS, JSON) between steps.

### 2. `rpy2` Library
The `rpy2` Python package embeds an R interpreter inside a Python process, allowing direct object exchange. Suitable for tight loops where passing files is too slow.

### 3. `reticulate` Library
The `reticulate` R package allows R to import and call Python modules directly, exchanging objects in memory. This is the pattern used in the Luria project for consuming the `luria` Python package from within an R session:

```r
library(reticulate)
luria <- import("luria")
results <- cingulate::analyze_neuropsych_data("data/processed/scores.parquet")
```

The `reticulate` package is a declared R dependency in the Luria installation (alongside `dplyr`, `tidyr`, `ggplot2`, `psych`, `lavaan`, `knitr`, `rmarkdown`, `jsonlite`, `DBI`, `RSQLite`, and the internal `cingulate` package). It is configured via the `R_HOME` environment variable set in `.env`.

### 4. Shared File Formats
Both languages read/write common formats (CSV, Parquet, JSON, HDF5), enabling loose coupling with clear boundaries. The Luria project uses Parquet as its primary interchange format — see [[concepts/parquet-as-knowledge-store]]. SQLite (`data/neuropsych.db`) is also shared: R reads it via `DBI`/`RSQLite`, and Python writes to it via the `index_node` pipeline stage. This is the most portable and debuggable integration pattern.

### 5. Separate Microservices
R runs as an API (e.g., via Plumber or OpenCPU) and Python calls it over HTTP. Maximizes language independence and supports [[concepts/deployment-automation]].

### 6. Workspace / Monorepo Layout
Both language trees live in the same repository with a shared configuration layer. This is the pattern used in the Luria project, as documented in [[summaries/PROJECT_SETUP_COMPLETE]], [[summaries/SETUP_SUMMARY]], and [[summaries/project-setup-progress]].

## Installation and Environment Setup

A dual R-Python project requires both runtimes to be installed and discoverable. The Luria installation process (documented in [[summaries/installation]]) illustrates the typical approach.

### Python Environment

Python 3.10+ is required (3.11+ recommended). The preferred Python package manager is **uv** (≥0.4, the fastest modern resolver), with pip as an alternative:

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

Development tooling includes `pytest ≥7.0`, `pytest-cov ≥4.0`, `black ≥23.0`, `ruff ≥0.1`, `mypy ≥1.0`, `pre-commit ≥3.0`, `ipython ≥8.0`, `jupyter ≥1.0`, and `jupyterlab ≥4.0`. See [[concepts/python-environment-management]] for broader discussion of virtual environments, uv, pip, and Poetry in Python projects.

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

This `.env`-based configuration approach (never committed to version control) is critical in clinical settings where API keys and data paths must remain private. See [[concepts/yaml-configuration]] for broader configuration patterns, and [[concepts/clinical-data-privacy]] for the privacy considerations that motivate strict environment hygiene.

## Luria Project Example

The Luria neuropsychology project provides a detailed, concrete illustration of R-Python integration for neuropsychological assessment tooling. The major repository reorganization completed April 28, 2026 — tracked across [[summaries/PROJECT_SETUP_COMPLETE]], [[summaries/SETUP_SUMMARY]], and [[summaries/project-setup-progress]] — documents the full architecture. The README at [[summaries/README_luria]] and [[summaries/README]] describe the public-facing capabilities.

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

- **Python** handles the core library (`src/luria/`), CLI (`luria` command via `typer`/`click`), configuration management, data pipelines, statistical analysis, and reporting modules. It is structured using the modern `src/` layout pattern for Python packaging (see [[concepts/python-project-structure]]).
- **R** handles specialized cingulate computations via `R/cingulate/` and `engine/cingulate/`, psychometric scoring (`psych`, `lavaan`), visualization (`ggplot2`), and report generation (`knitr`, `rmarkdown`). The `reticulate` bridge allows R sessions to directly invoke the `luria` Python package. The `RSQLite`/`DBI` stack allows R to query `data/neuropsych.db` directly.
- Development tooling (Makefile, `scripts/`) orchestrates both languages in a unified workflow.

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

Luria uses `.env` for secrets and runtime paths (API keys, database paths) and `luria_config.yaml` for reporting template and format configuration:

```yaml
# luria_config.yaml
reporting:
  templates:
    custom_adult:
      template: "templates/custom_adult.Rmd"
      styles: "styles/custom.css"
  formats: ["html", "pdf", "docx"]
```

The Python side uses `uv` for dependency management; R uses `renv` or direct package installation. Pre-commit hooks enforce Python code quality. See [[concepts/yaml-configuration]] for broader configuration patterns.

### Storage Backends Shared Across Languages

Both R and Python access the same storage layer:

| Backend | Path | R Access | Python Access |
|---|---|---|---|
| SQLite | `data/neuropsych.db` | `DBI` + `RSQLite` | `index_node` (SQLite writes) |
| LanceDB | `data/vectors/` | — | Semantic search vectors |
| SQLite (Soul) | `report_soul_index.sqlite` | — | Style exemplar vectors |
| DuckDB | `kb/store/` | — | Analytics / RAG column-store |

The `neuropsych.db` SQLite database (tables: `Documents`, `TestScores`, `ClinicalSummaries`) is the primary structured data store accessible to both languages.

### Project Setup Phases

The reorganization of the Luria project followed a structured, phased approach:

1. **Analysis and Planning** — Audited existing structure; identified need for cleaner Python/R separation.
2. **Directory Structure** — Established `src/luria/` Python package and `R/cingulate/` for R components.
3. **Configuration Files** — Created `pyproject.toml`, `.env.example`, `.pre-commit-config.yaml`, and `R/config.R`.
4. **Development Scripts** — Added `Makefile`, `scripts/setup.sh`, `scripts/build.sh`, and `src/luria/cli.py`.
5. **Git Setup** — Produced a comprehensive `.gitignore`, `scripts/git_setup.sh`, `.gitattributes`, and `README.md`.
6. **Documentation** — Built `docs/README.md` navigation hub and subdirectories for API, guides, tutorials.
7. **Testing** — Established `tests/test_basic.py`; verified directory structure, file existence, and imports.

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

- **Clinicians**: Focus on `reporting/` and the `luria` CLI; customize report templates and cognitive domain mappings.
- **Researchers**: Extend `analysis/` with statistical methods and research database integration.
- **Developers**: Add data formats in `data/`, new CLI commands, and additional analysis modules.

## IDE Support: Positron

The [[concepts/positron-ide]] is an IDE built specifically with R-Python integration in mind, developed by Posit (the company behind RStudio). It provides first-class support for both languages in a single environment.

### Positron Databot

Positron includes an AI assistant feature called **Databot** (available from Positron 2025.08.0+), which can answer questions about code and data across both R and Python files. Setting up Databot in an R-Python project requires attention to several configuration details, documented in [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]]:

- **R Runtime**: Databot operates on R-based projects, so R must be installed and on the system PATH.
- **Python Version**: The workspace `.python-version` file must point to an installed Python version (3.11 or 3.12 recommended).
- **API Key**: Databot uses an Anthropic API key, set via `ANTHROPIC_API_KEY` environment variable.
- **Model Selection**: Claude Sonnet 4 is recommended.
- **Ollama Integration**: Local LLM providers require at least one model to be pulled before the provider is usable.

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

This mirrors the broader principle of explicit, version-pinned interpreter configuration — avoiding ambiguity when both runtimes coexist in the same workspace.

## Verification

After installing both runtimes, verifying the integration is working correctly requires checking both sides:

```bash
# Python side
luria --version
python -c "import luria; print(luria.__version__)"

# R side
Rscript -e "library(reticulate); luria <- import('luria'); print('R-Python bridge OK')"
```

## Challenges

- **Environment management**: R uses `renv` or `packrat`; Python uses `venv`/`uv`. Keeping both in sync requires discipline and unified setup scripts.
- **Runtime availability**: Both R (≥4.3) and Python must be installed and discoverable by the IDE and CI environment. As shown in [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]], missing or mismatched runtimes are a common failure point — especially when `.python-version` specifies a nonexistent Python release.
- **Data serialization**: Agreeing on interchange formats (Arrow/Parquet is increasingly standard; see [[concepts/parquet-as-knowledge-store]]). SQLite bridges both via `RSQLite` (R) and direct Python writes.
- **Dependency installation**: CI/CD pipelines must install both R packages and Python packages — relevant to [[concepts/deployment-automation]].
- **Installation complexity**: Optional extras like R integration are not always available through standard package managers, requiring pip-based installation even in conda environments.
- **Type safety**: No shared type system; schemas must be documented or enforced via validation (Luria uses `pydantic ≥2.0` on the Python side).
- **Debugging**: Stack traces don't cross the language boundary cleanly.
- **Migration complexity**: Moving legacy mixed-language codebases into clean structures requires careful planning.
- **Testing**: Full test suites spanning both languages require both environments to be installed.
- **IDE AI assistant configuration**: Tools like Positron Databot that assist across both languages require correct configuration of both runtimes, API credentials, and language model providers. See [[concepts/ide-ai-assistant-configuration]].
- **Security**: `.env` files containing API keys and data paths must never be committed to version control. Dependencies should be audited regularly.

## Relation to Neuropsychological Reporting

In clinical and research neuropsychology, R's psychometric packages (`psych`, `lavaan`, `cingulate`) provide validated implementations of cognitive scoring algorithms. Python handles the surrounding infrastructure — ingesting raw test data, orchestrating scoring via [[concepts/langgraph-agent-workflows]], and rendering output into structured reports. The Luria toolkit exemplifies this division: R computes cingulate domain scores; Python packages them into HTML, PDF, or DOCX reports via customizable templates rendered by Quarto and Typst. See [[concepts/neuropsychological-reporting]] and [[concepts/narrative-report-generation]] for the reporting side of this pipeline.

## Related Concepts

- [[concepts/neuropsychological-reporting]] — End-to-end reporting pipelines that consume R/Python integration outputs
- [[concepts/narrative-report-generation]] — Generating clinical narrative text from structured neuropsychological scores
- [[concepts/luria-neuropsych-pipeline]] — The broader Luria pipeline that R-Python integration underpins
- [[concepts/cingulate-engine]] — The R-based cingulate computation component central to the Luria project's R side
- [[concepts/r-neuropsych-packages]] — R packages for psychometric and neuropsychological analysis
- [[concepts/r-visualization-theming]] — ggplot2 and R-based visualization in clinical reporting
- [[concepts/deployment-automation]] — Automating environments that must install and run both R and Python
- [[concepts/python-project-structure]] — Modern Python packaging conventions (src/ layout, pyproject.toml, uv) used in hybrid projects
- [[concepts/python-environment-management]] — Virtual environments, uv, pip, Poetry, and conda for managing Python dependencies
- [[concepts/positron-ide]] — IDE with first-class support for both R and Python, including the Databot AI assistant
- [[concepts/ide-ai-assistant-configuration]] — Configuring AI coding assistants (including Databot) in dual-language workspaces
- [[concepts/local-llm-inference]] — Local model providers such as Ollama that can serve as Databot backends
- [[concepts/yaml-configuration]] — Workspace and environment configuration patterns used across both language ecosystems
- [[concepts/parquet-as-knowledge-store]] — Parquet as the primary interchange format between R and Python components
- [[concepts/privacy-first-software]] — Clinical data handled by these pipelines carries strong privacy requirements
- [[concepts/clinical-data-privacy]] — Privacy requirements specific to neuropsychological and clinical data
- [[concepts/retrieval-augmented-generation]] — Python-native pattern sometimes used alongside R analysis in hybrid research tools
- [[concepts/omlx-server]] — Local LLM server providing OpenAI-compatible API, configured via shared `.env`
- [[concepts/langgraph-agent-workflows]] — The Python orchestration layer that calls R outputs via shared storage

See also: [[summaries/installation]] · [[summaries/SESSION_SUMMARY_2025-04-28]] · [[summaries/SETUP_SUMMARY]] · [[summaries/PROJECT_SETUP_COMPLETE]] · [[summaries/project-setup-progress]] · [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]] · [[summaries/README_luria]] · [[summaries/README]] · [[summaries/DEPENDENCIES]]

See also: [[summaries/deepagents_merged_mem_notes]] · [[summaries/SKILL]] · [[summaries/brand-yml-in-r]] · [[summaries/brand-yml-spec]] · [[summaries/001-choose-quarto-typst]] · [[summaries/overview]] · [[summaries/template-system]] · [[summaries/report-generation]] · [[summaries/FIX_EXPLANATION]] · [[summaries/OCR_PDF_GUIDE]]

See also: [[summaries/report-template]] · [[summaries/requirements]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/File Folder Structure Rebuild]]