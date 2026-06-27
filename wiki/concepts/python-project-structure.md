---
sources: [summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147.md, summaries/File Folder Structure Rebuild.md, summaries/table_API_readme.md, summaries/top_level.md, summaries/LICENSE.md, summaries/entry_points.md, summaries/requirements.md, summaries/installation.md, summaries/text-extraction.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/RECOVERY_NOTES.md, summaries/README.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/index.md, summaries/project-setup-progress.md, summaries/README_luria.md, summaries/SETUP_SUMMARY.md]
brief: How Python repos separate code, tooling, data, and authored project artifacts.
---

# Python Project Structure

Python project structure refers to the conventions, directory layouts, and tooling choices that organize a Python codebase into a maintainable, testable, and distributable package. A well-structured project reduces cognitive overhead, prevents circular imports, and enables smooth collaboration, deployment, and gradual architectural change. In practice, project structure is not just about where files live; it encodes boundaries between UI, services, data, tests, legacy systems, operational scripts, and writing or application artifacts.

A useful reminder from [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]] and [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]] is that many real Python repositories are not pure libraries. They often mix importable code, local environments, documentation, draft files, editor history, and project-specific working materials. In those snapshots, a `YC-2026` repository contained root-level files such as `application.md`, `README.md`, `pyproject.toml`, `CLAUDE.md`, and `GRAYMATTER.md`, plus `.history/` drafts, a `.remember/` temp directory, and a full `.venv/`. Good Python project structure must therefore distinguish between source code, tooling state, and human-authored working documents, not merely package modules.

## The `src/` Layout

The modern standard for Python packages is the **`src/` layout**, where all importable package code lives under a `src/<package_name>/` directory. This separates source code from project-level files (config, scripts, tests) and prevents accidental imports of the local directory during development.

Example from the Luria project (see [[summaries/SETUP_SUMMARY]] and [[summaries/project-setup-progress]]):

```
src/luria/
├── core/           # Configuration, logging, utilities
├── data/           # Data loading, cleaning, validation
├── analysis/       # Statistical analysis, visualization
├── reporting/      # Report generation, templates
└── cli.py          # Unified command-line interface
```

Supporting directories at the project root:

```
tests/          # Unit and integration tests
docs/           # Documentation
examples/       # Example scripts and apps
notebooks/      # Jupyter notebooks
scripts/        # Automation and setup scripts
data/raw/       # Raw input data (gitignored)
data/processed/ # Processed output data (gitignored)
```

When reorganizing an existing messy repository, it is advisable to create a dated backup directory (e.g., `backup_20260428`) before restructuring, preserving the original state while the new layout is established. The same principle applies to more aggressive folder rebuilds: snapshot first with git or a full filesystem copy, then perform deterministic moves. This aligns with [[concepts/migration-strategy]] and [[concepts/repository-hygiene]].

The `YC-2026` snapshots highlight an adjacent lesson: even when a repository is centered on writing rather than software delivery, project-root materials should remain clearly separated from importable code. Files like `application.md`, `CLAUDE.md`, and `GRAYMATTER.md` are legitimate top-level artifacts, but they should not be confused with package modules. If such a repo later grows automation, keeping code under `src/` or another explicit code directory prevents operational files, notebook support, and writing drafts from turning into an accidental application surface. This is especially relevant to [[concepts/personal-writing-workflows]] and [[concepts/plain-text-documentation]].

## RAG and Domain-Specific Project Layouts

Projects that incorporate a Retrieval-Augmented Generation (RAG) pipeline introduce additional directory conventions beyond the basic `src/` layout. The Autism RAG System (see [[summaries/README]] and [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]) illustrates a domain-specific variant:

```
project-root/
├── data/
│   ├── pdfs/               # Input PDF documents
│   ├── extracted_text/      # Extracted text from PDFs
│   ├── chunks/              # Text chunks
│   └── index/               # Vector store index
├── src/
│   ├── ingest.py            # Document ingestion pipeline
│   ├── query.py             # Query interface / REPL
│   ├── rag.py               # Core RAG logic
│   ├── pdf_parse.py         # PDF parsing utilities
│   ├── chunking.py          # Text chunking
│   ├── embeddings.py        # Embedding generation
│   ├── retrieval.py         # Similarity search
│   ├── prompts.py           # Prompt templates
│   └── citations.py         # Citation formatting
├── api/
│   └── server.py            # FastAPI server
└── eval/
    ├── questions.yaml        # Evaluation questions
    └── run_eval.py           # Evaluation runner
```

This layout separates the data pipeline (`data/`), core logic (`src/`), serving layer (`api/`), and evaluation harness (`eval/`) into distinct top-level directories. The `src/` modules follow single-responsibility principles: parsing, chunking, embedding, retrieval, prompting, and citation formatting each have dedicated modules. The `api/` layer exposes a FastAPI server with endpoints for health checks, querying, and triggering ingestion. The `eval/` directory provides a structured test harness using YAML-defined questions, reflecting [[concepts/yaml-configuration]] practices.

A common inconsistency in RAG projects is divergence between documentation and actual directory names (e.g., architecture diagrams showing `data/pdfs/` while setup instructions reference `data/epub/`). Treat setup instructions as authoritative over structural diagrams when they conflict.

For a more detailed treatment of the RAG pipeline architecture, see [[concepts/autism-research-rag]] and [[concepts/retrieval-augmented-generation]].

## uv Workspace Layout

For projects with multiple interdependent subprojects, `uv` supports a **workspace layout** where a top-level `pyproject.toml` declares member packages. This is the pattern used in the Luria repo after its 2026-04-28 reorg, where `luria/` became a uv workspace with `app/streamlit/` as a member package (see [[summaries/RECOVERY_NOTES]]).

A critical pitfall of workspace layouts is **path-resolution ambiguity**: code that previously lived at the repo root and computed paths relative to `__file__` will resolve to a different base after being moved into a workspace member subdirectory. The Luria recovery illustrates this precisely — `nodes.py` computed:

```python
REPO_ROOT = Path(__file__).resolve().parent.parent
SUBAGENTS_DIR = REPO_ROOT / "subagents"
```

After the reorg, `REPO_ROOT` resolved to `app/streamlit/` rather than the workspace root, causing a `FileNotFoundError` for every AGENTS.md prompt file. The fix is to use **two named constants** with unambiguous semantics:

```python
APP_ROOT       = Path(__file__).resolve().parent.parent   # app/streamlit/
WORKSPACE_ROOT = APP_ROOT.parent.parent                  # repo/workspace root
SUBAGENTS_DIR  = WORKSPACE_ROOT / "subagents"
```

Preserving the old name as a backwards-compatibility alias (e.g., `REPO_ROOT = APP_ROOT`) prevents breakage in code that still references it. See [[concepts/uv-workspace-layout]] for the broader pattern.

The newer folder-rebuild guidance in [[summaries/File Folder Structure Rebuild]] shows a related but more service-oriented workspace pattern. Instead of centering everything under `src/`, it creates explicit top-level zones such as:

- `app/streamlit` for the Streamlit UI
- `services/` for ingest, retrieval, agent workflows, storage, voice, reporting, and integrations
- `api/fastapi/routes` for HTTP endpoints
- `data/` for databases, vectors, and documents
- `voice_assets/` for non-code brand/style assets
- `external/` for preserved legacy or experimental systems
- `tests/` and `scripts/` for validation and automation

This is still a valid Python project structure, especially in application-heavy or monorepo-style repositories. The key idea is the same: make boundaries explicit and keep importable logic separate from runtime data, assets, and archived subsystems.

## Configuration Files

A modern Python project uses a small set of standard configuration files:

- **`pyproject.toml`** — The canonical project metadata and build configuration file, replacing older `setup.cfg` / `setup.py` patterns. Defines dependencies, entry points, tool settings (pytest, mypy, ruff, etc.).
- **`requirements.txt`** — Pip-compatible dependency list for legacy tooling compatibility. RAG projects such as the Autism RAG System support both `uv sync` (via `pyproject.toml`) and `pip install -r requirements.txt` to accommodate different deployment environments.
- **`setup.py`** — Kept for legacy compatibility but no longer the primary config.
- **`.env` / `.env.example`** — Environment variable templates for secrets and local configuration (API keys such as Anthropic and OpenAI, database paths, R configuration).
- **`.pre-commit-config.yaml`** — Hooks for automated code quality checks on commit, including conventional commit enforcement.
- **`.gitignore` / `.gitattributes`** — Version control hygiene; `.gitattributes` is particularly useful for enforcing consistent line endings across platforms.

The `YC-2026` snapshots provide a minimal but realistic root-level example:

```
README.md
application.md
pyproject.toml
CLAUDE.md
GRAYMATTER.md
```

This is a valid structure for a small Python-assisted writing project: `pyproject.toml` anchors tooling, `README.md` documents usage, and domain documents such as `application.md` remain separate from code. The newer snapshot also shows how a single top-level workflow can coexist with a substantial Python environment under `.venv/`, including Jupyter and IPython tools, without changing the fact that the repository's primary authored artifacts are markdown files. For projects integrating R alongside Python (see [[concepts/r-python-integration]]), an `R/config.R` file serves an analogous role for the R side of the stack. The Luria project's `R/` directory also reserves an `engine/cingulate/` subdirectory for the existing cingulate R package, keeping R-specific code cleanly separated from the Python source.

## Package Management

Modern Python projects benefit from fast, reliable package managers:

- **`uv`** — A fast Rust-based resolver and package manager, recommended for the Luria project and supported by the Autism RAG System. Supports `uv sync` for reproducible installs and `uv run --package <member>` for running commands scoped to a workspace member.
- **`pip` + `venv`** — The traditional baseline, still widely supported.
- **`poetry`** — An alternative with integrated lockfile management.

The `src/` layout works cleanly with all of these via an **editable install** (`pip install -e .`). Virtual environments are typically stored in a `.venv/` directory at the project root.

The `YC-2026` snapshots are a good example of conventional `.venv/` placement. They include Python 3.14 executables, Jupyter, IPython, pip, debug tooling, and supporting site-packages inside `.venv/`, while keeping the environment isolated from root-level authored files. This reflects [[concepts/python-environment-management]] and reinforces the distinction between environment state and project source.

## Task Automation

A `Makefile` provides a discoverable interface for common developer tasks:

```makefile
make install     # Set up environment
make test        # Run tests
make lint        # Check code style
make format      # Auto-format code
make typecheck   # Run type checking
make build       # Build distribution package
make publish     # Publish to PyPI
make check-all   # Run all quality checks
```

In addition to the `Makefile`, dedicated shell scripts serve specific purposes:
- `scripts/setup.sh` — One-click environment bootstrapping
- `scripts/build.sh` — Building and packaging
- `scripts/git_setup.sh` — Repository configuration automation
- `scripts/migrate_components.py` — Migration helper for moving legacy components to new locations

For RAG projects, the equivalent automation is often lighter — typically a pair of CLI commands:

```bash
# Build/rebuild the vector index
python -m src.ingest
# Interactively search the index
python -m src.query
# Start the API server
uvicorn api.server:app --reload
```

This pattern is especially valuable in projects that also integrate other languages — for example, [[concepts/r-python-integration]] setups where R scripts and Python code must be coordinated.

For large structural migrations, a dedicated shell rebuild script can also be appropriate. The folder-rebuild pattern in [[summaries/File Folder Structure Rebuild]] demonstrates a safe operational style:

- create target directories first
- move known paths deterministically
- copy selected code into new service buckets
- leave unknown files untouched
- avoid destructive deletion during the first pass

This kind of script is useful when repository structure itself is the object of change rather than package contents.

Small repositories with mixed writing and tooling may need a different automation layer. In the `YC-2026` snapshots, the visible structure suggests a lightweight workflow where Python tooling supports drafting rather than a large application runtime. In such cases, automation may focus on document checks, notebook utilities, export helpers, or small helper scripts rather than deployment pipelines. The structural lesson is the same: operational scripts belong in explicit locations rather than being scattered at the root.

## Smoke Test Scripts

For projects with complex path dependencies, a lightweight **smoke test script** is an essential addition to the `scripts/` directory. The Luria recovery introduced `scripts/smoke_test_paths.py`, which:

- Confirms every static path the pipeline relies on resolves correctly
- AST-parses every `.py` file to catch syntax errors
- Verifies workspace-root resolution math in files like `nodes.py`
- Runs without the heavy runtime dependencies installed

This pattern — verifying structural integrity before attempting a full run — is particularly valuable before demos or funding submissions. See [[concepts/smoke-test-scripts]] for the broader pattern.

```bash
python3 scripts/smoke_test_paths.py
# Expected: PASS — all required paths resolve and all .py files parse.
```

After any major file move, a simple tree inspection is also valuable. The rebuild guidance recommends using `tree -L 3` and then verifying that key destinations like `app/streamlit/app.py`, `services/`, and `external/` contain the expected content before attempting import rewrites or runtime execution.

The `YC-2026` snapshots suggest another practical structural check: confirm that directories like `.venv/`, `.history/`, and temporary state folders are present or ignored intentionally, and that the actual project artifacts of interest remain easy to identify at the root. In small projects, navigability is itself a structural quality metric.

## `.gitignore` and PHI/Data Hygiene

For projects that handle sensitive data, `.gitignore` must be treated as a security control, not just a housekeeping file. The Luria recovery added explicit entries for:

- `.DS_Store` and `**/.DS_Store` — macOS metadata files
- `.history/` — IDE history directories
- Runtime data directories under `app/streamlit/data/` — to ensure PHI is never committed

The comprehensive `.gitignore` in the Luria setup covers Python build artifacts, virtual environments, data files, IDE metadata, and archived outputs. In application-oriented structures, this principle extends to directories such as `data/db`, `data/vectors`, `data/documents`, and any generated content under `external/` or `voice_assets/` when they contain local-only artifacts. See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]] for the broader clinical data context.

The `YC-2026` snapshots add a non-clinical but important example: local working directories such as `.history/`, `.remember/`, and `.venv/` should usually be excluded from version control. Root-level drafts may also contain highly personal material, which makes repository hygiene relevant even outside regulated data domains. This overlaps with [[concepts/application-preparation]] when a repository contains sensitive application writing.

## Modular Design Principles

- **Single responsibility per module**: each subdirectory (`core/`, `data/`, `analysis/`, `reporting/`) handles one concern. In RAG projects, this extends to separating ingestion, embedding, retrieval, prompting, and citation into dedicated modules.
- **Separate app, services, and assets**: in larger application repositories, put UI code in `app/`, backend logic in `services/`, API code in `api/`, data in `data/`, and non-code resources in dedicated asset directories.
- **Separate code from authored content**: if a repository includes markdown drafts, notes, or applications, keep them clearly distinct from importable Python modules.
- **Avoid circular imports**: the `src/` layout and clear module boundaries make this easier to enforce.
- **CLI as entry point**: a `cli.py` module or equivalent exposes functionality via a unified command-line interface, aiding both human use and scripting. The CLI should support at minimum `--help`, `--version`, and an `init` subcommand.
- **Core configuration module**: a dedicated `src/<package>/core/config.py` centralizes runtime configuration, keeping environment-specific values out of business logic.
- **Gradual migration**: when evolving a legacy structure, prefer moving components incrementally as they are refactored rather than all at once.
- **Deterministic migration**: when large moves are necessary, encode them in repeatable scripts that only touch known paths.
- **Explicit over implicit paths**: in workspace or multi-package layouts, always name path constants semantically (e.g., `APP_ROOT` vs `WORKSPACE_ROOT`) rather than relying on relative `..` traversal counts.
- **Isolate legacy systems**: preserve old or experimental components in clearly named directories such as `external/` rather than mixing them into active application code.
- **Structure first, semantic cleanup second**: large reorganizations should usually move files first, then fix imports, deduplicate logic, and choose canonical pipelines in a second pass.
- **Zero data loss**: before any major restructuring, create a dated backup (e.g., `backup_20260428/`) or commit a snapshot and verify all original files are preserved.
- **Treat local tooling as non-source state**: a full `.venv/`, notebook kernels, editor history, and temporary session files may be necessary to work, but they should not define the conceptual architecture of the repo.
- **Preserve root clarity**: when a project is writing-centered, the root should make the primary artifacts obvious at a glance, even if a large local Python environment also exists.

## Subproject and Module Migration

Larger projects often grow from smaller prototypes with ad hoc layouts. A common migration pattern maps legacy directories to new locations:

| Legacy Location | New Location |
|---|---|
| `agent/cingulate/` | `engine/cingulate/` or `external/cingulate/` |
| `app/streamlit/` | `examples/streamlit_app/` or `app/streamlit/` as a workspace member |
| `rag/` | `src/<package>/rag/` or `external/experimental/` |
| `kb/` | Integration TBD |
| `voice/` | Separate module or `voice_assets/` |

The guiding principle is to maintain separation between reusable library code and application-level code (e.g., Streamlit apps, voice integrations, legacy bridges), keeping the active Python surface area focused and coherent.

The folder rebuild plan in [[summaries/File Folder Structure Rebuild]] provides a concrete migration example:

- `app/luria_streamlit_app/streamlit_app.py` → `app/streamlit/app.py`
- `app/luria_streamlit_app/neuropsych_rag/*` → `services/retrieval/`
- `app/luria_streamlit_app/neuropsych_agent/graph.py` → `services/agent/orchestrator.py`
- `app/luria_streamlit_app/neuropsych_agent/nodes.py` → `services/agent/workflows/ingest_flow.py`
- `app/luria_streamlit_app/neuropsych_agent/tools/store.py` → `services/storage/sqlite_store.py`
- `app/luria_streamlit_app/neuropsych_agent/tools/soul_context.py` → `services/voice/soul_context.py`
- `app/luria_streamlit_app/neuropsych_agent/tools/voice_quarto.py` → `services/reporting/quarto.py`
- `app/luria_streamlit_app/neuropsych_agent/tools/r_bridge.py` → `services/integrations/cingulate_bridge.py`
- `voice/` → `voice_assets/`
- `agent/cingulate` → `external/cingulate`
- `rag/*` → `external/experimental/`

This illustrates a common reality: not all Python projects evolve toward a single clean package. Some evolve toward an application workspace with multiple bounded areas and preserved legacy components.

When migrating, note that **untracked files** (files that existed locally but were never committed) will not appear in git history. These must be explicitly staged and committed as part of the migration, as was the case with the four `AGENTS.md` subagent prompt files in the Luria recovery.

The `YC-2026` snapshots suggest a softer migration case: a repository may begin as a document-centric workspace with files like `application.md` and later accumulate Python tooling. In that case, the first structural migration may simply be introducing `src/`, `scripts/`, or `notebooks/` rather than performing a full service extraction. The important move is making the transition from “folder with some Python in it” to “Python project with clearly bounded non-code artifacts.”

## Testing Strategy

- **`tests/`** at the project root, mirroring the `src/` structure or active service structure.
- **`pytest`** as the standard test runner.
- Tests cover unit logic, integration between components, CLI behavior, and data pipeline correctness.
- A basic test suite (`tests/test_basic.py`) can verify directory structure, file existence, and import functionality even before all dependencies are installed; the full suite requires the declared dependencies.
- For RAG projects, an `eval/` directory with YAML-defined questions and a dedicated runner (`eval/run_eval.py`) provides domain-specific evaluation separate from unit tests.
- A path-validation smoke test (see above) should be run before every demo or deployment.
- After major refactors, verify the new entrypoint imports explicitly; for example, replacing imports from an old monolithic module with imports from a new orchestrator module.
- For mixed-language projects, integration tests should cover the [[concepts/r-python-integration]] boundary explicitly.

For small repositories, structure tests may be simpler: assert that root documents exist, code directories import correctly, and ignored local-state directories are not treated as production artifacts. The `YC-2026` snapshots are a good example of a repo where identifying the intended system boundary is itself part of testing.

## Documentation Structure

Project documentation is typically organized as:

```
docs/guides/      # User guides and tutorials
docs/api/         # API reference (auto-generated)
docs/tutorials/   # Step-by-step examples
docs/reference/   # Technical reference
```

This aligns with [[concepts/documentation-as-code]] practices, treating docs as first-class versioned artifacts. A `docs/README.md` serves as a navigation hub linking into the subdirectories.

For structural migrations, documentation should also state what the reorganization does **not** yet accomplish. The folder rebuild example is explicit that file movement does not automatically fix imports, deduplicate logic, resolve conflicting pipelines, or fully modularize transitional files like `nodes.py`. Capturing these boundaries in docs prevents false assumptions during handoff.

The `YC-2026` snapshots also demonstrate a lighter documentation pattern: `README.md` plus domain-specific markdown files at the root. This can be sufficient for small projects, especially when the repository supports a concrete workflow rather than a public package. As the code surface grows, however, it becomes useful to distinguish operational docs from project content, local history, and archival drafts.

## Git Setup

Proper version control hygiene is a first-class concern in a well-structured project:

- A comprehensive `.gitignore` covers Python build artifacts, virtual environments, data files, IDE metadata, and (critically for clinical projects) PHI-containing runtime directories.
- `.gitattributes` enforces consistent line endings across operating systems.
- A `scripts/git_setup.sh` script can automate repository configuration for new contributors.
- Git hooks (via `.pre-commit-config.yaml`) enforce conventional commit message formats.
- Git workflow documentation (branch strategy, commit conventions) should live in `docs/` alongside other project documentation.
- Before large restructures, create an explicit pre-refactor snapshot commit or a full backup directory.
- Avoid rewriting git history when fixing path bugs; prefer additive commits that move files and document the rationale.

The `YC-2026` snapshots reinforce the importance of excluding local environment and editor-history material from commits. A repository with `.venv/`, `.history/`, and temporary state folders becomes much easier to maintain when git tracks only intended source, documentation, and stable configuration.

## Relation to Monorepo Layouts

For larger projects with multiple interdependent packages, the single-package `src/` layout can evolve into a [[concepts/monorepo-workspace-layout]], where `pyproject.toml` workspace support manages several packages in one repository. This is particularly relevant when subprojects like a RAG module, a knowledge base module, or a voice integration grow into independently releasable packages. The [[concepts/uv-workspace-layout]] pattern extends this with uv-specific tooling.

At the application level, a monorepo may combine Python package conventions with service directories such as `app/`, `services/`, `api/`, `external/`, and `voice_assets/`. The important distinction is whether a directory exists for packaging, runtime deployment, archival separation, or human navigation; good project structure makes that purpose obvious from the layout itself.

Not every repository needs to become a monorepo. The `YC-2026` snapshots are a counterexample: a small, focused workspace can remain root-oriented so long as its boundaries are clear. Project structure should fit the scale and purpose of the repository rather than imposing unnecessary complexity.

## User Persona Customization

A well-structured project can be navigated differently depending on the user's role:

- **Clinicians**: focus on the `reporting/` module and CLI for generating and customizing clinical reports.
- **Researchers**: extend the `analysis/` module with new statistical methods and database integrations.
- **Developers**: add data format handlers in `data/`, new CLI subcommands, additional analysis modules, new service modules, or new RAG pipeline components.
- **Writers or applicants**: focus on root-level markdown artifacts, revision history, and any lightweight automation supporting drafting workflows.

This mirrors the [[concepts/dual-audience-design]] principle of building software that serves both technical and non-technical users from the same codebase.

## References

- [[summaries/README]] — Autism RAG System README, illustrating a domain-specific RAG project layout with ingestion, API, and evaluation layers.
- [[summaries/RECOVERY_NOTES]] — Documents the 2026-04-28 Luria repo recovery, illustrating path-resolution bugs in uv workspace layouts and the smoke test pattern.
- [[summaries/SETUP_SUMMARY]] — Complete setup record for the Luria neuropsychology project, illustrating these conventions in practice.
- [[summaries/project-setup-progress]] — Phase-by-phase progress log of the Luria project reorganization.
- [[summaries/PROJECT_SETUP_COMPLETE]] — Detailed setup overview including migration plan and success criteria.
- [[summaries/File Folder Structure Rebuild]] — Safe, idempotent local script for rebuilding a repo into app, services, data, external, and asset directories.
- [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]] — Snapshot of a small Python-assisted writing repository mixing markdown drafts, local tooling, history folders, and `.venv/`.
- [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]] — Expanded snapshot of the same `YC-2026` repository, emphasizing root authored files, revision history, and a large local Python 3.14 environment.
- [[concepts/autism-research-rag]] — The RAG system built on top of these structural conventions.
- [[concepts/retrieval-augmented-generation]] — Core RAG architecture pattern implemented in the Autism RAG System.
- [[concepts/r-python-integration]] — Relevant when a Python project must coordinate with R codebases.
- [[concepts/monorepo-workspace-layout]] — Evolution path for multi-package projects.
- [[concepts/uv-workspace-layout]] — uv-specific workspace and multi-package patterns.
- [[concepts/documentation-as-code]] — Treating documentation with the same rigor as source code.
- [[concepts/dual-audience-design]] — Designing for both technical and non-technical users.
- [[concepts/smoke-test-scripts]] — Lightweight path and syntax validation before demos or deployments.
- [[concepts/phi-data-handling]] — Safe handling of protected health information in clinical software.
- [[concepts/clinical-data-privacy]] — Broader clinical data privacy context.
- [[concepts/yaml-configuration]] — YAML-based configuration patterns used in evaluation harnesses.
- [[concepts/repository-hygiene]] — Practices that keep repositories safe, clear, and recoverable.
- [[concepts/python-environment-management]] — Managing `.venv/`, installs, and local interpreter state.
- [[concepts/plain-text-documentation]] — Markdown-first project documentation and authored content.
- [[concepts/personal-writing-workflows]] — Repositories that mix writing drafts with lightweight tooling.
- [[concepts/application-preparation]] — Sensitive writing workflows that may shape repository hygiene.

See also: [[summaries/README_luria]]

See also: [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]]

See also: [[summaries/0008-soul-single-file-style-agent-architecture]]

See also: [[summaries/text-extraction]]

See also: [[summaries/installation]]

See also: [[summaries/requirements]]

See also: [[summaries/entry_points]]

See also: [[summaries/LICENSE]]

See also: [[summaries/top_level]]

See also: [[summaries/table_API_readme]]