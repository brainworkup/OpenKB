---
sources: [summaries/agentic-workflows.md, summaries/agent-team.md, summaries/DEPENDENCIES.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/README.md, summaries/RECOVERY_NOTES.md, summaries/neuropsych-pdf-parser.md, summaries/clinical-validity-reviewer.md, summaries/CLAUDE.md, summaries/responses_to_claude.md]
brief: Architectural pattern distinguishing static agent specs from runtime-dispatchable subagents in Claude-based pipelines.
---

# Subagent Architecture: Specs vs. Dispatchable Agents

## Overview
In Claude-based agentic systems, there is an important distinction between **agent specification documents** (static documentation describing what an agent does and what tools it uses) and **dispatchable subagents** (runtime-invokable agents that can be launched via the Task tool). Conflating these two forms is a common structural pitfall.

See [[summaries/responses_to_claude]] for a concrete example of this tension in a neuropsychological reporting pipeline. See [[summaries/2026-04-26-cingulate-agent-team-design]] for a fully worked multi-stage design that applies these principles concretely.

## Agent Specs: Documentation-as-Spec
Agent specs typically consist of:
- An `AGENTS.md` file describing the agent's role, behavior, and constraints
- A `tools.json` file enumerating the tools the agent may call

These live in a `subagents/` directory and serve as **design documentation**. They are human-readable references, not automatically invokable by the Claude Task tool.

Examples from the `luria` project:
- `subagents/Neuropsych_Data_Extractor/`
- `subagents/PDF_Ingestion_Parser/`
- `subagents/Narrative_Report_Generator/`
- `subagents/Sheets_Data_Indexer/`

See also [[summaries/AGENTS_luria]] for the project-level agent documentation.

## Dispatchable Subagents: Runtime Agents
To make an agent dispatchable by the Task tool, its specification must be **promoted** into `.claude/agents/<name>.md` with appropriate frontmatter. Only then can Claude invoke the agent in parallel as part of a multi-step pipeline.

The cingulate agent team (documented in [[summaries/2026-04-26-cingulate-agent-team-design]]) provides a concrete, fully specified example of this promotion in practice. Its six dispatchable subagents are:

1. `cingulate-orchestrator` — drives the chain, manages `state.json`, dispatches stages
2. `cingulate-intake` — normalizes referral question, records, interview, NSE
3. `cingulate-scoring` — loads CSVs via DuckDB, runs domain processors
4. `cingulate-interpretation` — generates per-domain narrative QMD files via LLM router
5. `cingulate-report-writer` — assembles Quarto/Typst template and renders PDF
6. `cingulate-quality-reviewer` — PHI scan, completeness, validity language, test-security review

This promotion step is a deliberate architectural action, not automatic.

## Two-Layer Skill/Subagent Separation
The cingulate design introduces a reusable pattern for separating *what* a stage does from *how* it is bound to a specific codebase:

```
luria-* skills (clinical reasoning, ~1 paragraph each)
        ▲ uses
cingulate-* subagents (.claude/agents/, project-local)
        ▲ dispatched by
cingulate-orchestrator (top-level subagent)
```

- **`luria-*` skills** describe the clinical reasoning logic and are intentionally generic and reusable across projects.
- **`cingulate-*` subagents** bind those skills to a specific repo's R API, file conventions, and Quarto templates — this is where most project-specific work lands.

This two-layer model is a scalable pattern for any domain where reusable reasoning skills need to be adapted to project-local tooling. See [[concepts/skills-modules]] for the broader skills pattern, and [[concepts/luria-skills]] for the neuropsychological-specific skill layer.

## Per-Patient Workspace as the Inter-Stage Contract
A key architectural feature of the cingulate design is that all subagents communicate exclusively through a shared per-patient workspace directory (`output/<patient_slug>/`), never through direct agent-to-agent calls or shared conversation history. The orchestrator maintains `state.json` as the single source of truth, tracking each stage's status (`pending` | `in_progress` | `done` | `error`).

The workspace tree includes:
- `state.json` — orchestrator's source of truth
- `intake/packet.md`, `intake/missing_data.md`
- `data-raw/csv/` — raw score inputs (read-only; inputs are copied in)
- `duckdb/staged.parquet` — scoring stage output
- `scoring/<domain>_scored.csv`
- `interpretation/_02-XX_<domain>_text.qmd`
- `report/template.qmd`, `report/<slug>.pdf`
- `qa/issue_list.md`
- `logs/<stage>.log`

This workspace-as-contract pattern:
- Keeps each subagent's context slice small and focused
- Makes the pipeline resumable and inspectable at any point
- Isolates failures to a single stage without cascading

See [[concepts/per-patient-workspace]] for a dedicated treatment of this pattern, and [[concepts/agent-pipeline-state-management]] for the broader state management approach.

## Orchestration and Dispatch Flow
The orchestrator loops over `state.json`, dispatches each `pending` stage in sequence, and **halts on any error — no auto-retry**. Failures bubble up for manual human review. This is a deliberate safety choice: in clinical contexts, silent retries could mask data errors that affect patient care.

Key dispatch properties:
- Each stage subagent receives only the workspace path and its relevant context slice
- Stage subagents never receive the full conversation history
- One git commit per stage is recorded on tracked branches
- An explicit human approval gate exists before any subagent is invoked against a real case

See [[concepts/multi-agent-orchestration]] for the broader orchestration pattern.

## Stage Contracts
Each stage reads from and writes to specific locations in the workspace, and calls specific R/API functions:

| Stage | Key API Calls |
|---|---|
| intake | optional `cingulate_quick_start()` |
| scoring | `load_data_duckdb()`, `process_all_domains()`, `query_neuropsych()` |
| interpretation | `generate_domain_text_qmd()`, `process_domains_with_llm()`, `run_llm_for_all_domains()` |
| report | `generate_assessment_report()`, `quarto render` |
| qa | `pdftotext` + heuristics; no R |

## Inherited Conventions Across Subagents
Because each subagent runs in its own isolated context, shared conventions must be **restated in each subagent definition** rather than centralized. The cingulate design formalizes these as:

1. Use `devtools::load_all('.')` — not `library(cingulate)` at startup
2. Default LLM mode is `development`; switch to `production` only when explicitly told
3. Never write to `data-raw/csv/`; copy inputs into the patient workspace first
4. Domain numbering is fixed (`01_iq`, `02_academics`, …) — do not renumber
5. Check for a `# manual-edit` marker before regenerating any narrative QMD file (see [[concepts/edit-protection-pattern]])
6. All paths in logs and `state.json` are absolute
7. One git commit per stage on tracked branches
8. `patient_slug` is `lower_snake_case`, max 64 chars, ASCII only

## CWD Constraints: A Hidden Coupling Risk
Several cingulate helpers — including `get_domains_with_data()`, `generate_domain_text_qmd()`, parts of `cingulate_llm.R`, and Quarto's `execute-dir: project` — require the working directory to be the patient workspace, with `data/<type>.parquet` directly underneath. Stage subagents must call `setwd(workspace_path)` after `devtools::load_all()` and before any cingulate function.

This is a significant hidden coupling: the package does not currently accept a workspace-path argument on these helpers. Threading one through cleanly would be a meaningful refactor. Until then, this constraint must be explicitly documented in each affected stage subagent definition — it cannot be assumed.

## Why the Distinction Matters
- **Specs without promotion** are inert — they inform developers but do not enable automation.
- **Parallel dispatch** via the Task tool requires the `.claude/agents/` location and frontmatter contract.
- Keeping specs in `subagents/` is valuable for [[concepts/documentation-as-code]] practices, but must be paired with promotion to achieve [[concepts/multi-agent-orchestration]].
- **CWD constraints** can be a hidden coupling: implementation-level working-directory assumptions must be documented in each stage subagent definition, not assumed.
- **Name collisions**: two agent files sharing the same `name:` frontmatter field (as flagged in the cingulate design for `agent-of-cingulate` / `neuropsych-data-analyst`) create ambiguity at dispatch time and must be resolved before live runs.

## Relationship to the Neuropsychological Pipeline
In the `luria`/`brainworkup` system, the subagent specs map directly onto the stages of the [[concepts/neuropsychological-assessment-pipeline]]:
- `PDF_Ingestion_Parser` / `cingulate-intake` → [[concepts/clinical-pdf-assessment]] and [[concepts/ocr-pipeline]]
- `Neuropsych_Data_Extractor` / `cingulate-scoring` → [[concepts/pdf-score-extraction]]
- `Narrative_Report_Generator` / `cingulate-interpretation` → [[concepts/narrative-report-generation]]
- `cingulate-quality-reviewer` → [[concepts/report-review-qa]] and [[concepts/phi-data-handling]]
- `Sheets_Data_Indexer` → now deprecated in favor of [[concepts/parquet-as-knowledge-store]] and [[concepts/cingulate-engine]]

## Key Risks
- **Structural drift**: reorganizations can cause old specs or code to resurface unexpectedly (noted in [[summaries/responses_to_claude]]).
- **Spec/runtime mismatch**: agents that are documented but never promoted may be assumed to be active when they are not.
- **Stale specs**: retiring integrations (e.g., Google Sheets) requires updating both the spec and the dispatchable agent, or deprecating the spec entirely.
- **Name collisions**: two agent files sharing the same `name:` frontmatter field create ambiguity at dispatch time.
- **CWD coupling**: undocumented working-directory assumptions in helper libraries create hidden stage dependencies that must be explicitly called out in each subagent definition.
- **PHI exposure**: `output/` must be gitignored before any real case run; a synthetic fixture case is recommended for smoke-testing the chain. See [[concepts/phi-data-handling]].
- **LLM mode drift**: defaulting to `development` mode is a safety measure; promotion to `production` requires explicit human authorization.

## Related Concepts
- [[concepts/multi-agent-orchestration]]
- [[concepts/documentation-as-code]]
- [[concepts/langgraph-agent-workflows]]
- [[concepts/model-context-protocol]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/cingulate-engine]]
- [[concepts/per-patient-workspace]]
- [[concepts/skills-modules]]
- [[concepts/luria-skills]]
- [[concepts/edit-protection-pattern]]
- [[concepts/phi-data-handling]]
- [[concepts/report-review-qa]]
- [[concepts/agent-pipeline-state-management]]
- [[concepts/smoke-test-scripts]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/clinical-validity-reviewer]]

See also: [[summaries/neuropsych-pdf-parser]]

See also: [[summaries/RECOVERY_NOTES]]

See also: [[summaries/README]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/agent-team]]

See also: [[summaries/agentic-workflows]]