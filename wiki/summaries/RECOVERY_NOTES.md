---
doc_type: short
full_text: sources/RECOVERY_NOTES.md
---

# Recovery Notes — Luria Repo (2026-04-28)

## Overview

This document records the recovery work performed on the **Luria repo** on 2026-04-28 after a workspace reorganization introduced a critical path-resolution bug. The repo was not catastrophically broken; a single incorrect path computation in `nodes.py` was the sole blocker for the markdown-only demo pipeline.

## Root Cause

A **uv workspace reorg** moved the Streamlit app from the repo root into `app/streamlit/` and relocated the [[concepts/langgraph-agent-workflows]] code from `./neuropsych_agent/` to `./app/streamlit/neuropsych_agent/`. However, `nodes.py` still computed `REPO_ROOT` as two levels up from `__file__`, which after the move resolved to `app/streamlit/` rather than the workspace root. This caused a `FileNotFoundError` when loading the `Neuropsych_Data_Extractor/AGENTS.md` prompt, crashing the second pipeline node on every run.

## Fixes Applied

### 1. `nodes.py` — Path Constants
- Replaced the ambiguous `REPO_ROOT` with two named constants:
  - `APP_ROOT` → `app/streamlit/`
  - `WORKSPACE_ROOT` → the actual git/uv workspace root
- `SUBAGENTS_DIR` now correctly points to `WORKSPACE_ROOT / "subagents"`
- `REPO_ROOT` retained as a backwards-compatibility alias (used by the optional Typst skill path)

### 2. `subagents/` Directory Structure
Moved four [[concepts/subagent-architecture]] prompt directories to their documented location:
- `subagents/PDF_Ingestion_Parser/`
- `subagents/Neuropsych_Data_Extractor/`
- `subagents/Sheets_Data_Indexer/`
- `subagents/Narrative_Report_Generator/`

### 3. `README.md`
Updated Project Structure, Installation, and Usage sections to reflect the actual [[concepts/uv-workspace-layout]]. New launch command:
```
uv run --package app streamlit run app/streamlit/app.py ...
```

### 4. `.gitignore`
Added `.DS_Store`, `.history/`, and runtime data directories under `app/streamlit/data/` to prevent [[concepts/phi-data-handling]] violations from accidental commits.

### 5. Smoke Test Script
Added `scripts/smoke_test_paths.py` — a [[concepts/smoke-test-scripts]] implementation that runs without heavy dependencies, verifies all static paths resolve, AST-parses all `.py` files, and validates `WORKSPACE_ROOT` math. Expected output:
```
PASS — all required paths resolve and all .py files parse.
```

## Known Open Issues (Non-Blocking for Demo)

| # | Issue | Impact |
|---|-------|--------|
| 1 | Missing `app/streamlit/skills/typst-report-formatter/SKILL.md` | [[concepts/typst-typesetting]] rendering path only; markdown unaffected |
| 2 | Missing `app/streamlit/templates/quarto_report/` (`_quarto.yml`, `_brand.yml`, `_extensions/`) | [[concepts/quarto]] rendering path only |
| 3 | Three `voice/` flavors and three `rag/` flavors coexist; canonical choices deferred post-funding | Documentation debt |
| 4 | Stale `__pycache__` with `.cpython-310.pyc` files (project targets 3.13) | Cleared; will not regenerate |

Source files for Typst skill: `voice/.agents/skills/typst-report-formatter/SKILL.md`  
Source files for Quarto templates: `voice/brand/`, `voice/style/_extensions/`

## Verification Procedure

```bash
python3 scripts/smoke_test_paths.py
uv sync
cd app/streamlit
uv run streamlit run app.py --server.address 127.0.0.1
```

A successful run advances through all four [[concepts/luria-neuropsych-pipeline]] stages: `[parse]`, `[extract]`, `[index]`, `[report]` without a `FileNotFoundError`.

## What Was Not Changed

- No business logic modified beyond `SUBAGENTS_DIR` path fix
- No data files (`uploads/`, `reports/`, `neuropsych.db`, `vectors/`) touched
- No `voice/`, `rag/`, `kb/`, or `agent/cingulate/` files modified
- No `.agents/skills/` content changed
- Git history not rewritten; the four `AGENTS.md` files were previously untracked and are committed for the first time as part of this recovery

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-data-privacy]]

- [[concepts/langgraph-agent-workflows]] — four-node pipeline (parse → extract → index → report)
- [[concepts/subagent-architecture]] — AGENTS.md prompt files for each pipeline node
- [[concepts/uv-workspace-layout]] — uv workspace layout and multi-package structure
- [[concepts/luria-neuropsych-pipeline]] — the end-to-end neuropsychological report pipeline
- [[concepts/python-project-structure]] — [[concepts/monorepo-workspace-layout]] patterns relevant to this reorg
- [[concepts/phi-data-handling]] — PHI protection via `.gitignore` and data directory exclusions
- [[concepts/smoke-test-scripts]] — lightweight pre-demo path validation script
- [[concepts/typst-typesetting]] — optional Typst rendering path, not yet fully wired
- [[concepts/quarto]] — optional Quarto rendering path, templates not yet in place