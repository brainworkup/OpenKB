---
sources: [summaries/File Folder Structure Rebuild.md]
brief: A staged, low-risk approach for reorganizing systems while preserving recoverability.
---

# Migration Strategy

## Definition
Migration strategy is a planned approach for changing a system's structure, location, or architecture while minimizing operational risk, preserving recoverability, and keeping the transition understandable and auditable.

## Context
In this wiki, migration strategy is illustrated by [[summaries/File Folder Structure Rebuild]], which describes a cautious local refactor of a project directory tree. The document frames migration not as a single destructive rewrite, but as a staged reorganization with backups, deterministic file movement, and explicit follow-up work.

## Core principles

### 1. Protect the pre-migration state
Before making structural changes, preserve a rollback point:

- create a git snapshot
- or create a full backup copy

This makes migration reversible and supports [[concepts/change_management]] and [[concepts/repository-hygiene]].

### 2. Prefer local, controlled execution
The source document explicitly advises against granting external access and instead recommends running a local script. This aligns with [[concepts/local-first-architecture]] and reduces unnecessary exposure during sensitive changes.

### 3. Create the destination before moving content
A good migration strategy defines the target architecture first, then creates the needed folders and boundaries before relocating code. This reduces ambiguity and supports clearer [[concepts/software_architecture]] and [[concepts/python-project-structure]].

### 4. Move known items deterministically
The source emphasizes deterministic mapping of known directories into predefined destinations. This matters because migrations are easier to review, repeat, and debug when each source path has a deliberate target.

Examples from [[summaries/File Folder Structure Rebuild]] include:

- Streamlit app moved into `app/streamlit`
- retrieval code moved into `services/retrieval`
- agent graph moved into `services/agent/orchestrator.py`
- legacy or experimental systems moved into `external/`

This overlaps with [[concepts/codebase-reorganization]] and [[concepts/monorepo-workspace-layout]].

### 5. Leave unknown material untouched
A low-risk migration should avoid broad deletion or aggressive cleanup when the contents are not fully understood. The source document explicitly preserves unknown files rather than trying to normalize everything at once. This is a practical risk-control tactic.

### 6. Separate structural migration from semantic cleanup
The source document intentionally does not fix imports, deduplicate logic, or resolve pipeline conflicts during the filesystem migration. This reflects a key migration pattern:

- first stabilize structure
- then repair references
- then simplify and consolidate logic

This staged approach reduces the chance that many kinds of failures happen at once. It also complements [[concepts/refactoring]] and [[concepts/architecture-decision-records]].

## Migration strategy in the source document
[[summaries/File Folder Structure Rebuild]] presents a practical migration sequence:

1. snapshot the current repository
2. create a new folder structure
3. move or copy known code into target buckets
4. isolate legacy or experimental code in external holding areas
5. avoid deleting unknown files
6. remove only empty directories
7. inspect the resulting tree
8. perform import rewrites and pipeline decisions afterward

This is a strong example of staged migration because it narrows each step's purpose and keeps judgment-heavy changes until after the physical reorganization is complete.

## Why this strategy is effective

### Risk reduction
By using backup-first execution and non-destructive moves/copies, the migration lowers the chance of irreversible loss.

### Auditability
Deterministic path mappings make it easier to inspect what changed and why.

### Idempotence and repeatability
A migration script that safely tolerates missing items and preexisting folders is easier to rerun during iterative cleanup.

### Better architectural boundaries
The target structure in the source document separates UI, services, API, data, assets, external systems, and tests. That improves system legibility and supports future modularization.

## Common migration tactics
A migration strategy often includes some combination of:

- backups or version-control checkpoints
- explicit target architecture creation
- scripted moves for known components
- temporary archival zones for legacy code
- post-migration verification steps
- later import, dependency, and workflow cleanup

The source document uses all of these except automated semantic cleanup, which it deliberately postpones.

## Related architectural ideas
Migration strategy frequently appears alongside:

- [[concepts/change_management]] for controlled transitions
- [[concepts/refactoring]] for improving structure without changing core intent
- [[concepts/codebase-reorganization]] for directory and module reshaping
- [[concepts/software_architecture]] for defining target boundaries
- [[concepts/repository-hygiene]] for maintaining recoverability and clarity
- [[concepts/python-project-structure]] for organizing code into coherent layers

## Practical guidance
When applying migration strategy to a codebase:

1. define the destination structure clearly
2. create a rollback point
3. automate only the low-ambiguity moves
4. preserve anything not yet understood
5. verify the result before deeper cleanup
6. defer judgment-heavy consolidation until the new structure is stable

## See also
- [[summaries/File Folder Structure Rebuild]]
- [[concepts/refactoring]]
- [[concepts/change_management]]
- [[concepts/codebase-reorganization]]
- [[concepts/software_architecture]]
- [[concepts/repository-hygiene]]
- [[concepts/local-first-architecture]]
- [[concepts/python-project-structure]]
- [[concepts/architecture-decision-records]]