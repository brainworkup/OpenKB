---
doc_type: short
full_text: sources/File Folder Structure Rebuild.md
---

# File Folder Structure Rebuild

## Summary
This document proposes a **safe, local-only, idempotent shell script** to reorganize a project folder structure without granting external access. The script creates a new architecture, moves known directories deterministically, copies selected components into new service buckets, and leaves unknown files untouched to reduce risk of data loss.

## Core recommendation
The document explicitly advises **against external access** and instead recommends running a local refactor script after creating a backup or git snapshot. The guiding principle is a cautious [[concepts/refactoring]] approach:

- create the target structure first
- move or copy only known directories
- avoid deleting unknown files
- preserve reversibility through backup/version control

## Safety and operational guidance
Before running the script, the document recommends:

- committing the current state with git
- or creating a full filesystem backup copy if git is unavailable

This frames the refactor as a controlled migration with rollback protection, aligned with [[concepts/change_management]] and [[concepts/codebase-reorganization]].

## What the script does
The provided shell script performs these actions:

1. **Creates a new directory architecture** for app, services, API, data, voice assets, external systems, tests, and scripts.
2. **Moves the Streamlit app** into `app/streamlit`.
3. **Copies `neuropsych_rag` into `services/retrieval`**, while preparing `services/ingest`.
4. **Decomposes `neuropsych_agent`** into more specific service areas:
   - `graph.py` → `services/agent/orchestrator.py`
   - `nodes.py` → `services/agent/workflows/ingest_flow.py`
   - selected tools redistributed into storage, voice, reporting, and integrations
5. **Extracts voice assets** from `voice/` into `voice_assets/`.
6. **Moves `agent/cingulate`** into `external/cingulate`.
7. **Archives experimental RAG content** from `rag/*` into `external/experimental/`.
8. **Deletes empty directories only**, as a limited cleanup step.

The script is described as non-destructive in intent: it uses moves and copies, avoids broad deletion, and tolerates missing files with `|| true` in several places.

## Target architecture
The intended structure includes these major areas:

- `app/streamlit` for the UI
- `services/` for ingest, retrieval, agent workflows, storage, voice, reporting, and integrations
- `api/fastapi/routes` for API endpoints
- `data/` for databases, vectors, and documents
- `voice_assets/` for brand/style/soul resources
- `external/` for preserved legacy or experimental systems
- `tests/` and `scripts/` for support infrastructure

This reflects a shift toward clearer [[concepts/software_architecture]] boundaries and separation of concerns.

## Intended outcomes
The refactor aims to:

- separate UI from backend services
- isolate legacy or experimental systems into `external/`
- break a monolithic agent package into domain-specific modules
- prepare the codebase for later import rewrites and pipeline cleanup

## Explicit limitations
The document is careful to state what the script **does not** do:

- fix imports
- deduplicate logic
- resolve competing pipelines
- fully clean or modularize `nodes.py`

These omissions are deliberate and positioned as tasks requiring human judgment after the filesystem reorganization. This highlights a staged refactor process: structure first, logic cleanup second.

## Required follow-up work
After running the script, the document recommends:

- inspecting the resulting tree
- confirming key destinations such as `app/streamlit/app.py`, `services/`, and `external/`
- rewiring imports in the Streamlit entrypoint, especially replacing imports from `neuropsych_agent.graph` with `services.agent.orchestrator`
- selecting a single canonical ingestion pipeline
- selecting a single canonical retrieval pipeline
- deleting or archiving the rest

## Key ideas

### Safe refactor strategy
The document emphasizes a conservative migration strategy: preserve data, avoid destructive cleanup, and treat architectural changes as reversible. This is a useful pattern for [[concepts/migration-strategy]] and [[concepts/repository-hygiene]].

### Deterministic directory mapping
Known components are mapped into predetermined destinations. This makes the reorganization reproducible and easier to audit.

### Structural refactor before semantic cleanup
The approach separates file movement from deeper code changes such as imports, module boundaries, and pipeline choice. This staged approach reduces risk and simplifies review.

## Notable file mappings
Examples of explicit mappings include:

- `app/luria_streamlit_app/streamlit_app.py` → `app/streamlit/app.py`
- `app/luria_streamlit_app/neuropsych_rag/*` → `services/retrieval/`
- `app/luria_streamlit_app/neuropsych_agent/graph.py` → `services/agent/orchestrator.py`
- `app/luria_streamlit_app/neuropsych_agent/nodes.py` → `services/agent/workflows/ingest_flow.py`
- `agent/cingulate` → `external/cingulate`
- `rag/*` → `external/experimental/`

## Overall significance
This document is a practical migration plan for reorganizing a codebase into a cleaner service-oriented structure while minimizing operational risk. Its main contribution is a cautious, auditable filesystem refactor procedure that prepares the project for later architectural cleanup and import rewiring.

## Related concepts
- [[concepts/refactoring]]
- [[concepts/software_architecture]]
- [[concepts/codebase-reorganization]]
- [[concepts/migration-strategy]]
- [[concepts/change_management]]
- [[concepts/repository-hygiene]]

## Related Concepts
- [[concepts/python-project-structure]]
- [[concepts/monorepo-workspace-layout]]
- [[concepts/luria-overview]]
- [[concepts/documentation-as-code]]
- [[concepts/local-first-architecture]]
- [[concepts/clinical-data-privacy]]
- [[concepts/smoke-test-scripts]]
- [[concepts/r-python-integration]]
- [[concepts/langgraph-agent-workflows]]
- [[concepts/neuropsychological-toolkit]]
