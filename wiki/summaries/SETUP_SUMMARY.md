---
doc_type: short
full_text: sources/SETUP_SUMMARY.md
---

# SETUP_SUMMARY: Luria Project Setup

## Overview

This document records the successful reorganization of the Luria project from a messy repository into a clean, modern Python/R project structure. The setup was completed on April 28, 2026, with all original files preserved in `backup_20260428/`.

## Project Purpose

Luria is a neuropsychology tooling project designed to support clinical neuropsychology workflows, including data processing, statistical analysis, report generation, and R/Python integration. See [[concepts/luria-neuropsych-pipeline]] for the broader pipeline context.

## Directory Structure

### Python Package (`src/luria/`)

| Module | Purpose |
|---|---|
| `core/` | Configuration, logging, utilities |
| `data/` | Data loading, cleaning, validation |
| `analysis/` | Statistical analysis, visualization |
| `reporting/` | Report generation, templates |
| `cli.py` | Unified command-line interface |

### Supporting Directories
- `R/` — R-specific code and `engine/cingulate/` for the cingulate package
- `data/raw/` and `data/processed/` — Data management
- `tests/`, `docs/`, `examples/`, `notebooks/`, `scripts/` — Standard project directories

## Configuration & Tooling

- **Package management**: `uv` (recommended), with `pyproject.toml` and `requirements.txt` — see [[concepts/uv-workspace-layout]]
- **Code quality**: `pre-commit` hooks, linting, formatting, type checking
- **Testing**: `pytest` with `tests/` directory; smoke test scripts follow the pattern described in [[concepts/smoke-test-scripts]]
- **Task automation**: `Makefile` for common workflows
- **Environment**: `.env.example` template covering API keys (Anthropic, OpenAI), database paths, R config — managed via [[concepts/yaml-configuration]]
- **Version control**: Comprehensive `.gitignore`, `.gitattributes`, conventional commit hooks

## R Integration

- `R/config.R` for R environment configuration
- `engine/cingulate/` prepared for the existing cingulate R package — see [[concepts/cingulate-engine]]
- Designed for [[concepts/r-python-integration]] workflows in neuropsychological assessment

## Key Architectural Decisions

1. **`src/` layout** — Avoids accidental imports from project root; follows [[concepts/python-project-structure]]
2. **Modular subpackages** — Separates concerns (data, analysis, reporting); relates to [[concepts/modular-report-architecture]]
3. **CLI entry point** — `luria` command for unified access
4. **Gradual migration** — `scripts/migrate_components.py` supports incremental refactoring

## Migration Path

Existing components are mapped to new locations:
- `agent/cingulate/` → `engine/cingulate/`
- `app/streamlit/` → `examples/streamlit_app/`
- `rag/` → `src/luria/rag/` (or kept separate)

## Development Workflow

```bash
# Setup
./scripts/setup.sh

# Daily
make test && make format && make lint

# Release
make check-all && make build
```

## Documentation Structure

- `docs/guides/` — User guides
- `docs/api/` — API reference
- `docs/tutorials/` — Step-by-step examples
- `docs/reference/` — Technical reference

## Related Concepts
- [[concepts/neuropsychological-toolkit]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/monorepo-workspace-layout]]

- [[concepts/luria-neuropsych-pipeline]] — Domain context for the Luria project
- [[concepts/python-project-structure]] — Standards applied here
- [[concepts/r-python-integration]] — Cross-language workflow pattern used
- [[concepts/uv-workspace-layout]] — Package management approach
- [[concepts/cingulate-engine]] — R integration target
- [[concepts/modular-report-architecture]] — Reporting module design pattern
- [[concepts/smoke-test-scripts]] — Testing strategy for verifying setup
- [[concepts/clinical-data-management]] — Data pipeline concerns