---
sources: [summaries/MIGRATION_GUIDE.md, summaries/File Folder Structure Rebuild.md]
brief: Reorganizing a codebase into clearer modules and service boundaries safely.
---

# Codebase Reorganization

## Definition
Codebase reorganization is the practice of restructuring a repository's files, directories, and module boundaries to better reflect system responsibilities without necessarily changing core behavior. It is usually done to improve maintainability, navigation, separation of concerns, and future development velocity.

This concept is illustrated by [[summaries/File Folder Structure Rebuild]], which proposes a cautious filesystem-level refactor of a project into clearer application, service, data, and external-system boundaries.

## Why it matters
A codebase often starts with pragmatic or rapid-growth structure, then accumulates mixed concerns over time. Reorganization helps when:

- UI code, backend logic, and storage concerns are intermingled
- experimental systems live beside production paths
- imports and responsibilities have become hard to trace
- one large package has grown into several distinct subsystems
- future work requires clearer ownership and dependency boundaries

In practice, codebase reorganization is often a precursor to deeper [[concepts/refactoring]] and broader [[concepts/software_architecture]] improvements.

## Core goals
Common goals of codebase reorganization include:

- clarifying system boundaries
- separating interfaces from implementation
- isolating legacy or experimental components
- reducing ambiguity about where new code should live
- making testing and deployment structure more predictable
- preparing the codebase for import cleanup, modularization, and service extraction

These goals align closely with [[concepts/python-project-structure]], [[concepts/monorepo-workspace-layout]], and [[concepts/knowledge-base-architecture]] as organizational disciplines.

## Example from the source document
In [[summaries/File Folder Structure Rebuild]], the repository is reorganized into a more explicit architecture:

- `app/streamlit` for the Streamlit user interface
- `services/` for ingest, retrieval, agent workflows, storage, voice, reporting, and integrations
- `api/fastapi/routes` for API endpoints
- `data/` for databases, vectors, and documents
- `voice_assets/` for non-code voice resources
- `external/` for preserved older or experimental systems
- `tests/` and `scripts/` for support infrastructure

The source also remaps older locations into these new buckets. For example:

- the Streamlit entrypoint is moved into `app/streamlit/app.py`
- `neuropsych_rag` is copied into `services/retrieval`
- `neuropsych_agent` is decomposed into agent orchestration, workflows, storage, voice, reporting, and integration files
- older or auxiliary systems such as `cingulate` and `rag` are moved into `external/`

This is a concrete example of reorganizing by responsibility rather than by historical origin.

## Design principles

### 1. Separate concerns first
A strong reorganization creates folders that represent stable responsibilities rather than temporary implementation details. The source document distinguishes:

- application UI
- services and workflows
- APIs
- data storage
- external or archived systems

This is a practical expression of [[concepts/software_architecture]] and supports later modularization.

### 2. Prefer deterministic mapping
The source document emphasizes moving known directories into predefined destinations. This makes the migration reproducible, reviewable, and easier to verify.

Deterministic mapping is especially important when the codebase has multiple overlapping pipelines or unclear ownership.

### 3. Preserve unknown material
A notable principle in [[summaries/File Folder Structure Rebuild]] is to leave unknown files untouched. This reduces the risk of accidental loss during restructuring and supports safer [[concepts/change_management]].

### 4. Reorganize structure before logic
The source explicitly does not attempt to fix imports, deduplicate pipelines, or fully clean internal modules during the initial move. Instead, it treats directory reorganization as an earlier stage in a longer migration. This staged approach aligns with [[concepts/migration-strategy]].

## Safe reorganization practices
The source document models several good practices for repository restructuring:

- create a git snapshot or backup before changes
- create the destination architecture first
- move or copy only known components
- avoid destructive deletion
- clean only obviously safe artifacts, such as empty directories
- verify the resulting tree after execution
- postpone semantic cleanup until after the filesystem layout is stable

These practices also support [[concepts/repository-hygiene]] and local operational safety.

## Relationship to refactoring
Codebase reorganization and [[concepts/refactoring]] overlap, but they are not identical.

- Codebase reorganization focuses on file placement, boundaries, and repository structure.
- Refactoring more broadly includes internal code changes that preserve behavior while improving design.

A reorganization often creates the conditions needed for successful refactoring by making modules easier to locate, reason about, and test.

## Common outcomes
When done well, codebase reorganization can lead to:

- clearer dependency paths
- more maintainable imports
- better isolation of experimental code
- easier onboarding for collaborators
- more obvious places for tests, scripts, and APIs
- reduced architectural drift over time

In the source example, the intended next steps after reorganization include import rewrites, selecting canonical pipelines, and archiving or deleting redundant implementations.

## Risks and tradeoffs
Reorganization still carries risks, even when it avoids deletion:

- imports can break after files move
- duplicate logic may temporarily exist in old and new locations
- unclear naming can recreate confusion in a new layout
- partial migration can leave the repository in a mixed state
- teams may mistake structural cleanup for completed architectural cleanup

The source addresses this by treating the script as a safe first pass rather than a complete transformation.

## Heuristics for good codebase reorganization
Useful heuristics include:

- organize around responsibilities, not accidents of history
- isolate legacy and experimental code explicitly
- separate UI, services, storage, and integrations
- keep the first migration reversible where possible
- verify structure before rewriting application logic
- document what the migration does not solve

These heuristics connect the concept to [[concepts/documentation-as-code]] and [[concepts/architecture-decision-records]] when teams want the rationale preserved.

## Related concepts
- [[concepts/refactoring]]
- [[concepts/software_architecture]]
- [[concepts/change_management]]
- [[concepts/migration-strategy]]
- [[concepts/repository-hygiene]]
- [[concepts/python-project-structure]]
- [[concepts/monorepo-workspace-layout]]
- [[concepts/local-first-architecture]]
- [[concepts/documentation-as-code]]
- [[concepts/architecture-decision-records]]

## Related source
- [[summaries/File Folder Structure Rebuild]]

See also: [[summaries/MIGRATION_GUIDE]]