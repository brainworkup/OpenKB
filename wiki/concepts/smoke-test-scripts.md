---
sources: [summaries/File Folder Structure Rebuild.md, summaries/RECOVERY_NOTES.md]
brief: Lightweight validation scripts that verify static paths resolve and source files parse before a deployment or demo.
---

# Smoke Test Scripts for Path and Parse Validation

A **smoke test script** is a lightweight, dependency-minimal validation tool run before a demo, deployment, or commit push to confirm that a project's static file paths resolve correctly and that all source files are syntactically valid. In the context of the [[concepts/luria-neuropsych-pipeline]], this pattern was formalized after a workspace reorganization introduced a silent path-resolution bug that crashed the pipeline at runtime.

## Purpose

Smoke tests of this kind serve three goals:

1. **Path resolution validation** — assert that every static file the pipeline depends on (prompt files, skill files, config files) actually exists at the path the code expects.
2. **AST parse validation** — confirm that every `.py` file in the project is syntactically valid Python, catching stale bytecode or encoding errors before they surface at runtime.
3. **Path-constant math verification** — explicitly test that workspace-root resolution logic (e.g., computing `WORKSPACE_ROOT` from `__file__`) produces the correct absolute path, independent of the current working directory.

These checks run *without* importing heavy runtime dependencies (LLM clients, database drivers, ML frameworks), making them fast and safe to run on a clean machine or CI environment.

## The Luria Implementation

After the recovery described in [[summaries/RECOVERY_NOTES]], a script `scripts/smoke_test_paths.py` was added to the [[concepts/luria-neuropsych-pipeline]] repo. It checks:

- All four [[concepts/subagent-architecture]] prompt directories under `subagents/`:
  - `subagents/PDF_Ingestion_Parser/AGENTS.md`
  - `subagents/Neuropsych_Data_Extractor/AGENTS.md`
  - `subagents/Sheets_Data_Indexer/AGENTS.md`
  - `subagents/Narrative_Report_Generator/AGENTS.md`
- That `WORKSPACE_ROOT` as computed in `app/streamlit/neuropsych_agent/nodes.py` resolves to the actual git/uv workspace root, not `app/streamlit/`
- That every `.py` file in the project AST-parses cleanly under the target Python version (3.13)

Expected terminal output on success:
```
PASS — all required paths resolve and all .py files parse.
```

Run command:
```bash
python3 scripts/smoke_test_paths.py
```

## Why Path-Constant Math Needs Explicit Testing

Path computation bugs are common after workspace reorganizations. The pattern:

```python
REPO_ROOT = Path(__file__).resolve().parent.parent
```

is sensitive to the depth of `__file__` within the project tree. When `nodes.py` moved from `./neuropsych_agent/nodes.py` (two levels from workspace root) to `./app/streamlit/neuropsych_agent/nodes.py` (four levels from workspace root), the formula silently produced the wrong answer. A smoke test that asserts the resolved path equals the known workspace root catches this class of bug immediately.

The fix replaced the single ambiguous constant with two named constants:
- `APP_ROOT` — the Streamlit project root (`app/streamlit/`)
- `WORKSPACE_ROOT` — the actual git/uv workspace root

See [[concepts/uv-workspace-layout]] for context on why multi-level workspace structures make this kind of path arithmetic error more likely.

## Relationship to Deployment Safety

Smoke test scripts complement broader [[concepts/deployment-automation]] practices by providing a fast pre-flight check that requires no network access, no credentials, and no running services. They are especially valuable in contexts involving [[concepts/phi-data-handling]], where a crashing pipeline could leave sensitive data in an inconsistent state.

## When to Run

- Before every demo or funding presentation
- Before committing after any file move or workspace restructure
- As the first step in a CI pipeline before heavier integration tests
- After updating Python version targets (to catch stale bytecode issues)

## Related Concepts

- [[concepts/luria-neuropsych-pipeline]] — the pipeline this script validates
- [[concepts/uv-workspace-layout]] — the workspace structure that motivated the path-constant fix
- [[concepts/subagent-architecture]] — the prompt files whose paths are verified
- [[concepts/python-project-structure]] — project layout conventions that affect path resolution
- [[concepts/deployment-automation]] — broader context for pre-deployment validation
- [[concepts/phi-data-handling]] — safety motivation for catching crashes before they affect sensitive data
- [[summaries/RECOVERY_NOTES]] — the recovery event that led to this script being created


See also: [[summaries/File Folder Structure Rebuild]]