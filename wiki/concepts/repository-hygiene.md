---
sources: [summaries/README_20260413204228.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147.md, summaries/File Folder Structure Rebuild.md]
brief: Practices that keep a repository safe, organized, and maintainable over time.
---

# Repository Hygiene

## Definition
Repository hygiene is the set of practices used to keep a code repository safe, understandable, reversible, and maintainable as it evolves. It includes careful file organization, predictable structure, backup and rollback safeguards, conservative cleanup, and explicit separation between active, legacy, and experimental code.

## Why it matters
Good repository hygiene reduces the risk of:

- accidental data loss
- hard-to-review structural changes
- hidden duplication
- confusing project layout
- unsafe cleanup during refactors
- long-term architectural drift

It supports healthy [[concepts/codebase-reorganization]], more reliable [[concepts/change_management]], and clearer [[concepts/software_architecture]].

## Repository hygiene in the source document
[[summaries/File Folder Structure Rebuild]] presents repository hygiene as a practical refactor discipline rather than a purely stylistic concern. The document recommends reorganizing a project with a safe, idempotent local shell script that:

- creates the target directory structure first
- moves known directories deterministically
- copies selected components into new homes where appropriate
- leaves unknown files untouched
- avoids destructive deletion
- preserves rollback via git snapshot or full backup

This reflects a conservative approach to [[concepts/refactoring]] and [[concepts/migration-strategy]].

## Core principles

### 1. Preserve recoverability
Before structural changes, create a git commit or a full backup. This ensures the repository can be restored if the refactor introduces breakage.

In practice, this means repository hygiene is not only about cleanliness; it is also about recoverability and operational safety.

### 2. Prefer deterministic moves
The source document maps known components into explicit destinations, such as moving the Streamlit app into `app/streamlit` and relocating legacy systems into `external/`. Deterministic mapping makes changes easier to audit, test, and review.

### 3. Do not disturb unknown files
A key hygiene principle in the source is to leave unrecognized files alone. This minimizes the chance of deleting important but poorly understood assets.

### 4. Separate active, legacy, and experimental code
The document distinguishes current application structure from preserved old systems by moving older or experimental components into `external/`. This improves navigability and prevents architectural ambiguity.

### 5. Clean conservatively
The script only removes empty directories and avoids broad destructive cleanup. This is a hallmark of good repository hygiene: clean up enough to reduce clutter, but not so aggressively that context or assets are lost.

### 6. Stage structural and semantic changes separately
The source document explicitly does not fix imports, deduplicate pipelines, or fully modularize certain files during the filesystem move. Instead, it performs structure first and logic cleanup second. This staged method makes large changes safer and easier to validate.

## Signals of good repository hygiene
A repository with good hygiene often shows the following traits:

- clear top-level directory purpose
- separation of UI, services, data, tests, and external dependencies
- explicit handling of legacy artifacts
- reversible changes through version control
- minimal destructive operations during migration
- documented next steps after automated reorganization

These traits align closely with [[concepts/python-project-structure]], [[concepts/monorepo-workspace-layout]], and [[concepts/documentation-as-code]].

## Example from the source document
In [[summaries/File Folder Structure Rebuild]], repository hygiene appears in several concrete decisions:

- snapshot the repo before refactoring
- create directories before moving files into them
- move known modules into specific service buckets
- archive experimental RAG content instead of deleting it
- place older systems in `external/` rather than mixing them with core code
- postpone import rewrites and pipeline decisions until after the structure is stable

This demonstrates that repository hygiene is closely tied to disciplined [[concepts/architecture-decision-records]] style thinking even when the decisions are expressed in operational instructions rather than formal ADRs.

## Relationship to architecture work
Repository hygiene is not identical to architecture, but it helps architecture remain legible in the filesystem. A clean repository makes service boundaries, ownership, and workflow responsibilities easier to understand.

In this sense, repository hygiene supports:

- [[concepts/software_architecture]] by making structure visible
- [[concepts/local-first-architecture]] by favoring local, controlled operations
- [[concepts/knowledge-base-architecture]] by emphasizing organization and retrievability
- [[concepts/security-policy]] by discouraging unnecessary external access

## Common anti-patterns
Poor repository hygiene often looks like:

- mixing production, experimental, and archived code in the same directories
- deleting files during refactors before understanding their role
- moving files without backups
- combining structural reorganization with logic rewrites in one step
- leaving duplicate pipelines unresolved without clear staging
- using ambiguous folder names that hide system boundaries

The source document addresses these anti-patterns directly by advocating a safe, staged reorganization process.

## Practical guidance
When applying repository hygiene in similar situations:

1. snapshot the current state
2. define the target structure clearly
3. move only what is understood
4. quarantine legacy or experimental components instead of deleting them
5. verify the resulting tree after migration
6. only then fix imports, simplify modules, and remove duplication

## Related pages
- [[summaries/File Folder Structure Rebuild]]
- [[concepts/codebase-reorganization]]
- [[concepts/refactoring]]
- [[concepts/migration-strategy]]
- [[concepts/change_management]]
- [[concepts/software_architecture]]
- [[concepts/python-project-structure]]
- [[concepts/monorepo-workspace-layout]]
- [[concepts/documentation-as-code]]
- [[concepts/security-policy]]
- [[concepts/local-first-architecture]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]]

See also: [[summaries/README_20260413204228]]