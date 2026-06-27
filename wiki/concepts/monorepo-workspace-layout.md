---
sources: [summaries/File Folder Structure Rebuild.md, summaries/entry_points.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/SETUP_SUMMARY.md, summaries/RECOVERY_NOTES.md]
brief: Monorepo layout organizes related packages under one repo with shared tooling.
---

# Monorepo & Workspace Layout

A **monorepo** is a single version-controlled repository that houses multiple related projects or packages. A **workspace** extends this idea by letting a package manager such as uv for Python, Cargo for Rust, or npm/pnpm for JavaScript treat those sub-packages as first-class members with shared dependency resolution and unified tooling.

In practice, monorepo layout is not just about where files live. It defines how code is moved, how boundaries between UI and services are expressed, how shared assets are referenced, and how safely a project can be reorganized over time. In the Luria materials, this appears both as a workspace-path recovery problem and as a broader architectural reorganization effort documented in [[summaries/File Folder Structure Rebuild]] and [[summaries/RECOVERY_NOTES]].

## Why Use a Monorepo?

- **Atomic changes** — a single commit can update application code, shared libraries, scripts, and documentation together.
- **Unified dependency graph** — sub-packages share a lock file, preventing version skew between components.
- **Simplified CI** — one pipeline can build, test, and lint every member package.
- **Easier cross-package refactoring** — moving code between packages is a local filesystem operation, not a multi-repo dance.
- **Clear architectural zoning** — app, services, data, external systems, tests, and scripts can be separated without splitting repositories.

This makes monorepos especially useful for staged [[concepts/refactoring]] and larger-scale [[concepts/codebase-reorganization]] efforts.

## uv Workspaces (Python)

[[concepts/uv-workspace-layout]] describes the mechanics in more detail, but the key idea is that uv brings Cargo-style workspace semantics to Python. A workspace is declared in the root `pyproject.toml`:

```toml
[tool.uv.workspace]
members = ["app/streamlit", "scripts"]
```

Each member has its own `pyproject.toml` and can declare its own dependencies, but all members share a single `uv.lock` and a single virtual environment at the workspace root. A member package is installed and run with:

```bash
uv run --package app streamlit run app/streamlit/app.py
```

A workspace layout becomes more valuable as the repository grows beyond a single application entrypoint. The refactor plan in [[summaries/File Folder Structure Rebuild]] illustrates this: the UI remains under `app/streamlit`, while retrieval, ingest, storage, voice, reporting, and integrations are separated into `services/`, and legacy or experimental code is moved into `external/`.

## Reorganization as a Layout Strategy

A monorepo is most effective when the filesystem reflects architectural intent. The refactor plan in [[summaries/File Folder Structure Rebuild]] provides a useful pattern for this: create the target structure first, move known directories deterministically, and leave unknown files untouched.

The proposed target architecture included:

```text
app/streamlit
services/
  ingest
  retrieval
  agent/workflows
  storage
  voice
  reporting/templates
  integrations
api/fastapi/routes
data/
  db
  vectors
  documents
voice_assets/
external/
  cingulate
  experimental
tests/
scripts/
```

This shows an important monorepo principle: layout should communicate system boundaries. A Streamlit interface belongs in `app/`; orchestration and business logic belong in `services/`; preserved legacy systems belong in `external/`; generated or runtime data belong in `data/`. This kind of explicit partitioning supports [[concepts/software_architecture]], [[concepts/python-project-structure]], and [[concepts/knowledge-base-architecture]] at the repository level.

## Safe Reorganization Principles

A monorepo layout is often revised over time, so safe migration patterns matter. The reorganization plan in [[summaries/File Folder Structure Rebuild]] emphasizes three rules:

1. create the new structure
2. move known directories deterministically
3. leave unknown files untouched

Before any large move, it recommends either:

- a git snapshot commit, or
- a full backup copy of the repository

This is a practical example of [[concepts/change_management]], [[concepts/migration-strategy]], and [[concepts/repository-hygiene]]. The key idea is that layout refactors should be reversible and auditable. A good monorepo reorganization is not a destructive cleanup; it is a controlled remapping of code into clearer boundaries.

## Path Pitfalls After Reorganization

The most common failure mode when migrating from a flat layout to a workspace layout is **stale path assumptions** in source files. The Luria project provides a concrete case study (see [[summaries/RECOVERY_NOTES]]):

- The Streamlit app moved from `./streamlit_app.py` → `./app/streamlit/app.py`.
- LangGraph node code moved from `./neuropsych_agent/` → `./app/streamlit/neuropsych_agent/`.
- The four LangGraph node prompt files (`subagents/<name>/AGENTS.md`) remained at the repo root, where the README documented them.
- `nodes.py` computed `REPO_ROOT = Path(__file__).resolve().parent.parent`, which after the move resolved to `app/streamlit/` rather than the workspace root.
- Prompt files in `subagents/` were looked up relative to the wrong root, producing a `FileNotFoundError` that crashed the second pipeline node on every run.

Critically, **no business logic was broken** — only a single path constant was wrong. The markdown-only demo pipeline was entirely unblocked by fixing that one line.

The broader refactor plan in [[summaries/File Folder Structure Rebuild]] shows how these problems can compound during larger migrations. It does not just move one entrypoint; it redistributes `neuropsych_agent`, `neuropsych_rag`, `voice`, `agent/cingulate`, and `rag/` into new top-level zones. Every such move creates opportunities for stale imports and stale path assumptions.

### The Fix: Explicit Named Constants

Instead of a single ambiguous `REPO_ROOT`, introduce clearly named constants that make the directory hierarchy explicit:

```python
# nodes.py
APP_ROOT       = Path(__file__).resolve().parent.parent   # app/streamlit/
WORKSPACE_ROOT = APP_ROOT.parent.parent                   # repo root
SUBAGENTS_DIR  = WORKSPACE_ROOT / "subagents"
REPO_ROOT      = APP_ROOT   # backwards-compat alias
```

This makes the intent unambiguous and survives future reorganizations as long as the constants are updated together. Backwards-compatibility aliases (`REPO_ROOT`) allow gradual migration of call sites that have not yet been updated.

For larger reorganizations, the same naming discipline should apply beyond root constants: directories like `SERVICES_DIR`, `DATA_DIR`, `VOICE_ASSETS_DIR`, and `EXTERNAL_DIR` can reduce ambiguity when code must reference shared resources across workspace members.

## Subagent Prompt Files and Workspace Roots

When a workspace member deep in the directory tree needs to load files that live at the workspace root, such as shared prompt files or configuration, the distinction between `APP_ROOT` and `WORKSPACE_ROOT` is critical. In the Luria layout:

```text
<workspace_root>/
  subagents/
    PDF_Ingestion_Parser/AGENTS.md
    Neuropsych_Data_Extractor/AGENTS.md
    Sheets_Data_Indexer/AGENTS.md
    Narrative_Report_Generator/AGENTS.md
  app/
    streamlit/
      app.py
      neuropsych_agent/
        nodes.py          ← must reach back to workspace root
```

The rule of thumb: any file that is **shared across workspace members** belongs at the workspace root and must be referenced via `WORKSPACE_ROOT`, not via a path relative to the calling module.

This principle becomes even more important after decomposition of a formerly single package into multiple zones such as `services/agent`, `services/retrieval`, `services/storage`, and `services/reporting`, as proposed in [[summaries/File Folder Structure Rebuild]]. Shared artifacts should either live in a clearly shared root location or be given an explicit package boundary, rather than being reached through fragile relative paths.

## From Monolith to Service Buckets

One practical use of monorepo layout is decomposing a single large package into domain-specific buckets without splitting the repository. The refactor plan in [[summaries/File Folder Structure Rebuild]] illustrates this by breaking apart `neuropsych_agent` and `neuropsych_rag`:

- `graph.py` → `services/agent/orchestrator.py`
- `nodes.py` → `services/agent/workflows/ingest_flow.py`
- retrieval code copied into `services/retrieval/`
- tool modules redistributed into storage, voice, reporting, and integrations

This is a useful intermediate state between a single-file or single-package system and a fully modular architecture. In monorepo terms, the repository stays unified while the internals become more legible. This pattern connects closely with [[concepts/langgraph-agent-workflows]], [[concepts/subagent-architecture]], [[concepts/multi-agent-orchestration]], and [[concepts/luria-neuropsych-pipeline]].

It also demonstrates an important caution: filesystem decomposition does not automatically fix application semantics. The reorganization plan explicitly leaves import rewiring, deduplication, and canonical pipeline selection for later. Layout changes create structure; they do not by themselves resolve design ambiguity.

## Documentation Must Track the Layout

A monorepo reorganization is incomplete until the `README.md`, installation instructions, and launch commands reflect the new structure. Stale documentation is a silent bug — contributors and automation scripts follow the docs, not the actual filesystem. The Luria recovery updated the README's Project Structure, Installation, and Usage sections as part of the same commit that fixed the path bug.

The same principle applies to planned reorganizations like the one in [[summaries/File Folder Structure Rebuild]]: if entrypoints move from `neuropsych_agent.graph` to `services.agent.orchestrator`, or from a flat app layout to `app/streamlit/app.py`, the docs must change at the same time. Keeping documentation synchronized with code structure is a principle shared with [[concepts/documentation-as-code]] and [[concepts/plain-text-documentation]].

## Protecting Sensitive Data in a Monorepo

With multiple packages sharing a single `.gitignore` at the workspace root, it is easy to miss runtime-data directories created by individual packages. The Luria recovery added explicit `.gitignore` entries for:

- `.DS_Store` and `**/.DS_Store`
- `.history/`
- All runtime-data directories under `app/streamlit/data/`

This ensures PHI-containing files such as uploads, reports, database files, and vector stores could never be accidentally committed — a concern covered in depth under [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]].

As repositories are reorganized, data locations often change too. A clean monorepo layout should make durable distinctions between source code, runtime data, generated artifacts, and external systems. Explicit top-level areas like `data/` and `external/` help support that separation.

## Smoke-Testing the Layout

After any workspace reorganization, a lightweight path-validation script is invaluable. The `scripts/smoke_test_paths.py` added during the Luria recovery (see [[summaries/RECOVERY_NOTES]]) checks:

1. Every static path the pipeline depends on resolves to an existing file or directory.
2. Every `.py` file in the workspace AST-parses without error.
3. The `WORKSPACE_ROOT` resolution math in `nodes.py` produces the correct directory.

This script runs without heavy ML dependencies installed, making it fast enough to run before every demo or deployment push. See [[concepts/smoke-test-scripts]] for the broader pattern. The recommended workflow:

```bash
python3 scripts/smoke_test_paths.py
uv sync
cd app/streamlit
uv run streamlit run app.py --server.address 127.0.0.1
```

Expected smoke-test output: `PASS — all required paths resolve and all .py files parse.`

For larger refactors like the one in [[summaries/File Folder Structure Rebuild]], smoke tests should also verify that:

- `app/streamlit/app.py` exists
- `services/` contains expected content
- `external/` received archived legacy systems
- imports and runtime entrypoints match the new layout

## Deferred Decisions and Known Open Items

A workspace reorganization often surfaces decisions that can be deferred without blocking the immediate goal. In the Luria case, the following were explicitly left open post-recovery:

- **Multiple voice/ and rag/ flavors** coexisting (`voice/brand`, `voice/soul`, `voice/style`; `rag/page-index`, `rag/docling`, `rag/openmed`) — canonical choices deferred until after the funding submission.
- **Typst skill file** (`app/streamlit/skills/typst-report-formatter/SKILL.md`) — needed for the [[concepts/typst-typesetting]] rendering path but not the markdown-only demo path; source copies exist in `voice/.agents/skills/`.
- **Quarto report templates** (`_quarto.yml`, `_brand.yml`, `_extensions/`) — needed for [[concepts/quarto]] rendering paths; sources in `voice/brand/` and `voice/style/_extensions/`.

[[summaries/File Folder Structure Rebuild]] adds a second kind of deferred decision: after structural relocation, teams still need to choose a **single canonical ingestion pipeline** and **single canonical retrieval pipeline**, then archive or remove alternatives. This is a common monorepo reality: layout clarity often exposes duplicated implementations that were previously hidden by a messy tree. Documenting these deferrals explicitly prevents them from being forgotten without blocking the immediate milestone.

## Related Concepts

- [[concepts/uv-workspace-layout]] — detailed mechanics of uv workspace configuration
- [[concepts/documentation-as-code]] — keeping docs in sync with code structure
- [[concepts/plain-text-documentation]] — using text files such as README and AGENTS.md as the source of truth
- [[concepts/phi-data-handling]] — protecting sensitive data in multi-package layouts
- [[concepts/privacy-first-software]] — design choices that prevent data leaks at the tooling level
- [[concepts/deployment-automation]] — CI/CD implications of workspace-aware build commands
- [[concepts/smoke-test-scripts]] — lightweight pre-demo validation scripts
- [[concepts/luria-neuropsych-pipeline]] — the pipeline whose path bug motivated this case study
- [[concepts/python-project-structure]] — broader Python project organization patterns
- [[concepts/software_architecture]] — structural boundaries expressed through repository layout
- [[concepts/refactoring]] — staged restructuring without immediate semantic cleanup
- [[concepts/codebase-reorganization]] — deliberate remapping of a repository into clearer zones
- [[concepts/migration-strategy]] — safe and reversible approaches to layout changes
- [[concepts/change_management]] — operational discipline around repo-wide reorganization
- [[concepts/repository-hygiene]] — cleanup, safety, and predictable repository organization
- [[summaries/RECOVERY_NOTES]] — concrete case study of a uv workspace path bug and its resolution
- [[summaries/SETUP_SUMMARY]] — setup context for the same project
- [[summaries/PROJECT_SETUP_COMPLETE]] — project completion status after recovery
- [[summaries/File Folder Structure Rebuild]] — planned directory reorganization and deterministic migration script

See also: [[summaries/entry_points]]