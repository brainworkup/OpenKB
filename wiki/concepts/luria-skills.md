---
sources: [summaries/2026-04-26-cingulate-agent-team-design.md]
brief: Thin, generic skill modules defining what each clinical neuropsych stage does, independent of any specific repo.
---

# Luria Skills Modules for Clinical Stage Definitions

Luria skills (`luria-*`) are thin, reusable modules that describe *what* each clinical stage of a neuropsychological assessment does, without binding that logic to any particular repository, file convention, or technology stack. They form the bottom layer of a two-layer agent architecture and are intended to be wrapped by repo-specific subagents that supply the implementation details.

See [[summaries/2026-04-26-cingulate-agent-team-design]] for the primary design document and [[summaries/AGENTS_luria]] and [[summaries/README_luria]] for additional context.

## Role in the Architecture

In the cingulate agent team design, the architecture separates concerns into two explicit layers:

```
luria-* skills          ← what each stage does (generic)
        ▲ wrapped by
cingulate-* subagents   ← how it is done in this repo (specific)
        ▲ dispatched by
cingulate-orchestrator  ← top-level coordinator
```

- **`luria-*` skills** are intentionally not cingulate-specific. They can be reused across different neuropsychological report systems or codebases.
- **`cingulate-*` subagents** bind the skill definitions to this repo's R API, DuckDB data layer, Quarto templates, and file conventions.

This separation means that if the underlying tooling changes (e.g., a different rendering engine or scoring library), the skill definitions remain stable while only the wrapping subagents need updating.

See [[concepts/subagent-architecture]] and [[concepts/multi-agent-orchestration]] for the broader pattern.

## The Five Clinical Stage Skills

Each `luria-*` skill corresponds to one stage of a neuropsychological assessment pipeline:

| Skill Module | Stage | Description |
|---|---|---|
| `luria-neuropsych-orchestrator` | Orchestration | Drives the full chain; manages state and dispatches stages |
| `luria-case-intake` | Intake | Normalize referral question, records, interview, NSE; track missing data |
| `luria-score-processing` | Scoring | Load raw scores, run domain processors, produce scored tables |
| `luria-interpretation` | Interpretation | Generate per-domain narrative text using an LLM router |
| `luria-report-writing` | Report | Assemble template and render to PDF |
| `luria-quality-review` | QA | PHI scan, completeness check, validity language, test-security review |

These stages align with the standard structure of a neuropsychological assessment: intake → scoring → interpretation → report → quality review. See [[concepts/neuropsychological-assessment-pipeline]] and [[concepts/luria-neuropsych-pipeline]] for related discussions.

## Design Principles

### Thinness
Skill modules are kept deliberately minimal — roughly one paragraph each. They describe the *goal and outputs* of a stage, not the implementation. This makes them portable and easy to reason about.

### Separation from Implementation
All repo-specific details (R API calls, DuckDB queries, Quarto rendering commands, file path conventions) live in the `cingulate-*` subagent definitions, not in the skill modules. This follows a clean interface-vs-implementation separation.

### Self-Contained Context
Each stage subagent that wraps a skill receives a self-contained brief at dispatch time: the workspace path, `patient_slug`, `age_group`, and the relevant slice of `state.json`. It never receives the full conversation history. The skill definition contributes the stage-level goal; the subagent contributes the execution context.

### Reusability Across Cases
Because skills are generic, the same `luria-case-intake` skill definition could in principle wrap a pediatric, adult, or forensic assessment workflow. The cingulate-specific subagent layer is where case-type branching occurs.

## Relationship to Other Concepts

- [[concepts/luria-skills]] — the overview concept for luria skill modules
- [[concepts/luria-overview]] — broader Luria framework context
- [[concepts/luria-neuropsych-pipeline]] — the pipeline these skills define
- [[concepts/skills-modules]] — general pattern for modular skill definitions
- [[concepts/subagent-architecture]] — how skills are wrapped by subagents
- [[concepts/multi-agent-orchestration]] — how the orchestrator dispatches skill-backed subagents
- [[concepts/agent-pipeline-state-management]] — the `state.json` mechanism that connects stages
- [[concepts/narrative-report-generation]] — what the interpretation and report stages produce
- [[concepts/report-review-qa]] — the quality review stage
- [[concepts/phi-data-handling]] — PHI concerns addressed in the QA skill stage
- [[concepts/neuropsychological-assessment-pipeline]] — the clinical workflow being modeled

## Open Questions

- **Portability in practice:** The skill definitions have not yet been tested outside the cingulate repo. True portability remains to be validated.
- **Granularity:** The current five-stage split matches common neuropsychological report workflows, but edge cases (e.g., multi-session evaluations, abbreviated batteries) may require additional or split skill modules.
- **Versioning:** As clinical best practices evolve, skill module definitions will need a versioning strategy separate from the wrapping subagents.
