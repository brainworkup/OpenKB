---
sources: [summaries/entry_points.md, summaries/requirements.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/SETUP_SUMMARY.md, summaries/RECOVERY_NOTES.md]
brief: A uv workspace organizes multiple Python packages under a single root with shared lockfile and per-member pyproject.toml files.
---

# uv Workspace Layout

A **uv workspace** is a monorepo-style project structure for Python, managed by the [uv](https://github.com/astral-sh/uv) package manager. It allows multiple related packages to share a single lockfile and virtual environment while each maintaining its own `pyproject.toml`. This pattern is analogous to Cargo workspaces in Rust or npm/yarn workspaces in JavaScript.

## Core Concepts

### Workspace Root
The workspace root is the top-level directory containing a `pyproject.toml` with a `[tool.uv.workspace]` table that lists member packages. The root itself may or may not be an installable package.

### Member Packages
Each member is a subdirectory with its own `pyproject.toml`. Members can declare dependencies on each other using path references, and uv resolves them all into a single coherent lockfile (`uv.lock`).

### Shared Lockfile
All members share one `uv.lock` at the workspace root. This guarantees version consistency across the entire project without requiring separate lock files per package.

## Layout in the Luria Project

The Luria neuropsychology pipeline underwent a workspace reorg that is the central subject of [[summaries/RECOVERY_NOTES]]. The resulting layout is:

```
luria/                          ← workspace root
├── pyproject.toml              ← workspace manifest
├── uv.lock
├── subagents/                  ← shared prompt files (not a package)
│   ├── PDF_Ingestion_Parser/
│   ├── Neuropsych_Data_Extractor/
│   ├── Sheets_Data_Indexer/
│   └── Narrative_Report_Generator/
├── scripts/
│   └── smoke_test_paths.py
├── app/
│   └── streamlit/              ← workspace member: "app"
│       ├── pyproject.toml
│       ├── app.py
│       ├── neuropsych_agent/
│       │   └── nodes.py
│       ├── skills/
│       └── data/
└── voice/, rag/, kb/, ...      ← other project areas (not reorganized)
```

### Launch Command
Because the Streamlit app is a workspace member named `app`, it must be launched via:
```bash
uv run --package app streamlit run app/streamlit/app.py --server.address 127.0.0.1
```

## The Path-Resolution Problem

The reorg introduced a subtle but critical bug: code inside a member package can no longer assume `Path(__file__).parent.parent` reaches the workspace root. After the move, `nodes.py` computed:

```python
REPO_ROOT = Path(__file__).resolve().parent.parent
SUBAGENTS_DIR = REPO_ROOT / "subagents"
```

From `app/streamlit/neuropsych_agent/nodes.py`, `parent.parent` resolved to `app/streamlit/` — not the workspace root. The fix introduced explicit named constants:

```python
APP_ROOT       = Path(__file__).resolve().parent.parent  # app/streamlit/
WORKSPACE_ROOT = APP_ROOT.parent.parent                  # luria/ (workspace root)
SUBAGENTS_DIR  = WORKSPACE_ROOT / "subagents"
```

This makes the two levels of nesting explicit and self-documenting. `REPO_ROOT` is retained as a backwards-compatibility alias.

## Key Design Principles

1. **Shared resources at workspace root** — Files needed by multiple members (e.g., `subagents/`, `scripts/`) live at the workspace root, not inside any member package.
2. **Explicit path constants** — Member packages must not rely on relative `parent` traversal to find workspace-root resources; named constants with clear comments are safer.
3. **PHI isolation** — Runtime data directories (`data/uploads/`, `data/reports/`, `data/vectors/`, `data/neuropsych.db`) are kept inside the member package and excluded from version control via `.gitignore`.
4. **Smoke-test before deployment** — A lightweight script (`scripts/smoke_test_paths.py`) that validates all static paths and AST-parses all `.py` files should be run before every demo or funding push. See [[concepts/smoke-test-scripts]].

## Related Concepts

- [[concepts/luria-neuropsych-pipeline]] — the pipeline whose workspace reorg triggered this documentation
- [[concepts/monorepo-workspace-layout]] — broader monorepo patterns
- [[concepts/python-project-structure]] — Python packaging conventions
- [[concepts/smoke-test-scripts]] — path and import validation scripts
- [[concepts/phi-data-handling]] — why runtime data is excluded from the workspace VCS
- [[concepts/subagent-architecture]] — the four subagent prompt directories living at workspace root
- [[concepts/neuropsychological-assessment-pipeline]] — the clinical pipeline built on this workspace
- [[summaries/RECOVERY_NOTES]] — detailed account of the reorg bug and its fix
- [[summaries/README_luria]] — project overview and documented workspace structure


See also: [[summaries/SETUP_SUMMARY]]

See also: [[summaries/installation]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/requirements]]

See also: [[summaries/entry_points]]