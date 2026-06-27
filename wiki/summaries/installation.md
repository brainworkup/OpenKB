---
doc_type: short
full_text: sources/installation.md
---

# Installation Guide — Summary

This document provides comprehensive instructions for installing and configuring Luria on various systems and environments.

## Prerequisites

- **Python 3.10+** is required (3.11+ recommended)
- Optional but recommended package managers: **uv** (fast, preferred) or **pip**
- System-specific dependencies: Linux may need `python3-dev` and `build-essential`; macOS and Windows have minimal requirements

## Installation Methods

### Standard Installation
- **uv** (recommended): `uv add luria`
- **pip**: `pip install luria` or `pip install luria[full]` for all optional dependencies
- **From source**: clone the GitHub repo and use `pip install -e .` or `uv sync`

### Alternative Package Managers
- **Poetry**: `poetry add luria`
- **conda/mamba**: create environment with Python 3.10, then install via pip (not yet on conda-forge)
- **Docker**: use `python:3.10-slim` base image with `pip install luria`

### Development Installation
Clone the repo and run `scripts/setup.sh` or manually create a virtual environment, install with `pip install -e ".[dev]"`, and set up pre-commit hooks.

## Verification

```bash
luria --version
luria --help
python -c "import luria; print(luria.__version__)"
```

## Configuration

- Copy `.env.example` to `.env` and set API keys and paths
- Key environment variables:
  - `ANTHROPIC_API_KEY`, `OPENAI_API_KEY` — LLM API access
  - `OMLX_BASE_URL` — local LLM endpoint (see [[concepts/omlx-server]])
  - `NEUROPSYCH_DB_PATH` — database path
- Initialize project structure with `luria init`

## R Integration (Optional)

- Requires R 4.0+ and packages: `dplyr`, `tidyr`, `ggplot2`, `psych`, `reticulate`
- Configure `R_HOME` in `.env`
- See [[concepts/r-python-integration]] for details on using R alongside Python in Luria

## System Requirements

| Level | Python | RAM | Disk |
|-------|--------|-----|------|
| Minimum | 3.10+ | 4GB | 1GB |
| Recommended | 3.11+ | 8GB+ | 2GB+ |

## Upgrading & Uninstalling

- Upgrade: `pip install --upgrade luria` or `uv add luria@latest`
- Uninstall: `pip uninstall luria` or `uv remove luria`
- Clean config/cache: `rm -rf .luria/ .cache/luria/`

## Troubleshooting

- **"Command not found"**: add Python scripts directory to PATH
- **Permission errors**: use `--user` flag or a virtual environment
- **Version conflicts**: create a fresh virtual environment; use `pip check`
- **R integration issues**: verify R is in PATH and `R_HOME` is set

## Security Notes

- Never commit `.env` to version control
- Use virtual environments for isolation
- Audit dependencies with `pip-audit` or `safety`

## Related Concepts
- [[concepts/neuropsychological-toolkit]]
- [[concepts/local-first-architecture]]
- [[concepts/openai-compatible-api]]
- [[concepts/local-llm-inference]]
- [[concepts/clinical-data-privacy]]
- [[concepts/luria-neuropsych-pipeline]]

- [[concepts/luria-overview]] — What Luria is and its purpose
- [[concepts/python-environment-management]] — Virtual environments, uv, pip, Poetry
- [[concepts/r-python-integration]] — Using R alongside Python in Luria
- [[concepts/omlx-server]] — Setting up and configuring the local LLM endpoint
- [[concepts/python-project-structure]] — How the Luria project is organized
- [[concepts/uv-workspace-layout]] — uv-based dependency and workspace management
- [[concepts/yaml-configuration]] — Environment and configuration file conventions