---
sources: [summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/README.md, summaries/agent-team.md, summaries/2026-04-26-cingulate-agent-team-design.md]
brief: Filesystem directory tree that serves as the sole inter-stage contract in the cingulate agent pipeline.
---

# Per-Patient Workspace Contract

In multi-agent neuropsychological pipelines, the **per-patient workspace** is a self-contained directory that acts as the sole inter-stage contract. Rather than passing context through shared memory, a message bus, or direct subagent-to-subagent communication, every stage reads its inputs from and writes its outputs to a single well-defined directory tree. This makes the system auditable, restartable, and easy to debug.

First formalized in [[summaries/2026-04-26-cingulate-agent-team-design]] and operationalized in [[summaries/agent-team]].

## Core Idea

Each patient gets an isolated workspace under `output/<patient_slug>/`. Stages are completely decoupled: a stage only needs to know the workspace path and its own slice of `state.json` to do its work. No stage reads another stage's in-memory state or receives the full conversation history.

This pattern sidesteps the need for a shared database or inter-process communication protocol during the scaffolding phase, while still enabling a clean hand-off sequence across five clinical stages. The workspace is the boundary between the generic luria skill layer (which describes *what* each stage does) and the cingulate-specific subagents (which bind those stages to this repo's R API, file conventions, and Quarto templates). See [[concepts/luria-skills]] and [[concepts/subagent-architecture]].

## Directory Layout

```
output/<patient_slug>/
├── state.json                        # orchestrator's source of truth
├── intake/
│   ├── packet.md                     # normalized referral + records + NSE summary
│   └── missing_data.md               # explicit list of unknowns
├── data-raw/csv/                     # raw scores (user-provided or copied in)
├── duckdb/staged.parquet             # scoring stage output
├── scoring/
│   └── <domain>_scored.csv           # one per active domain
├── interpretation/
│   └── _02-XX_<domain>_text.qmd      # narrative includes
├── report/
│   ├── template.qmd
│   └── <patient_slug>.pdf
├── qa/issue_list.md
└── logs/<stage>.log
```

## state.json as Orchestrator Source of Truth

The `state.json` file tracks stage lifecycle and is the only file the orchestrator writes to between dispatches. Each stage entry holds a `status` field with one of four values:

- `pending` — not yet started
- `in_progress` — subagent dispatched
- `done` — subagent returned success
- `error` — subagent failed; reason and stack trace written to `logs/<stage>.log`

Stages return richer status signals to the orchestrator:

- `DONE` — completed successfully
- `DONE_WITH_CONCERNS` — completed with warnings (e.g., some domains failed LLM generation)
- `NEEDS_CONTEXT` — missing required input
- `BLOCKED` — hard failure; orchestrator halts and surfaces the reason and log path

On `BLOCKED`, the orchestrator halts the chain rather than auto-retrying, surfacing the failing stage, a one-line reason, and the absolute path to the stage log. This makes failures visible and keeps a human in the loop for clinical safety. See [[concepts/multi-agent-orchestration]] for the broader dispatch pattern and [[concepts/agent-pipeline-state-management]] for the state machine design.

Example `state.json`:

```json
{
  "patient_slug": "example_patient",
  "age_group": "child",
  "stages": {
    "intake":         {"status": "done",        "updated": "2026-04-26T18:00:00Z"},
    "scoring":        {"status": "in_progress", "updated": "2026-04-26T18:05:00Z"},
    "interpretation": {"status": "pending"},
    "report":         {"status": "pending"},
    "qa":             {"status": "pending"}
  }
}
```

## Why a Filesystem Contract?

- **Auditability:** Every artifact from every stage is a file on disk, inspectable at any time.
- **Restartability:** Because each stage writes durable outputs, a failed run can resume from the last `done` stage without re-running earlier work. To resume after a fix, the orchestrator is re-dispatched with the same `patient_slug`; it reads `state.json` and continues from the first non-`done` stage.
- **Context isolation:** Each subagent receives only its workspace path and its stage slice — no full conversation history, reducing token cost and context confusion. See [[concepts/subagent-architecture]].
- **PHI containment:** The entire `output/` directory can be gitignored in one rule, keeping protected health information off version control. See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]].

## Stage Hand-off Sequence

The five cingulate stages consume and produce workspace artifacts in a strict chain:

| Stage | Reads | Writes | Key API Calls |
|---|---|---|---|
| intake | referral text, records, interview notes | `intake/packet.md`, `intake/missing_data.md` | optional `cingulate_quick_start()` |
| scoring | `data-raw/csv/*` | `duckdb/staged.parquet`, `scoring/<domain>_scored.csv` | `load_data_duckdb()`, `process_all_domains()`, `query_neuropsych()` |
| interpretation | `scoring/*`, `intake/packet.md` | `interpretation/_02-XX_<domain>_text.qmd` | `generate_domain_text_qmd()`, `process_domains_with_llm()`, `run_llm_for_all_domains()` |
| report | `interpretation/*`, `intake/packet.md` | `report/template.qmd`, `report/<slug>.pdf` | `generate_assessment_report()`, `quarto render` |
| qa | `report/<slug>.pdf`, all upstream outputs | `qa/issue_list.md` | `pdftotext` + heuristics; no R |

The QMD narrative files produced by the interpretation stage are protected by the [[concepts/edit-protection-pattern]]: before regenerating any file, stages check for a `# manual-edit` marker on line 1 and skip if present, logging a warning.

## Subagent Roster and Models

The workspace is driven by six subagents defined under `.claude/agents/`. Each runs in its own isolated context and is dispatched by the orchestrator with a self-contained brief — never the full conversation history.

| Subagent | Wraps Skill | Stage | Model |
|---|---|---|---|
| `cingulate-orchestrator` | luria-neuropsych-orchestrator | (driver) | opus |
| `cingulate-intake` | luria-case-intake | 1 | sonnet |
| `cingulate-scoring` | luria-score-processing | 2 | sonnet |
| `cingulate-interpretation` | luria-interpretation | 3 | opus |
| `cingulate-report-writer` | luria-report-writing | 4 | sonnet |
| `cingulate-quality-reviewer` | luria-quality-review | 5 | opus |

Model selection reflects the relative complexity of each stage: interpretation and QA use opus for reasoning-heavy tasks, while intake, scoring, and report writing use sonnet.

## Rerunning and Skipping Stages

Because `state.json` is the authoritative record of stage progress, individual stages can be replayed or bypassed by editing it directly:

- To **force a rerun**: set the stage's status back to `pending`.
- To **skip a stage**: set the stage's status to `done` (use cautiously — downstream stages may depend on outputs).

This makes the workspace a recoverable, operator-controllable pipeline rather than a black box.

## Conventions Tied to the Workspace

These conventions are restated in each subagent definition (rather than centralized) because each subagent runs in its own isolated context:

1. Use `devtools::load_all('.')` from repo root — never `library(cingulate)`.
2. Default LLM mode: `development`; switch to `production` only when explicitly authorized by Joey.
3. Raw score files are never written directly to `data-raw/csv/` — inputs are always copied into the workspace first.
4. Domain numbering within the workspace is fixed (`01_iq`, `02_academics`, …) and must not be renumbered across runs.
5. Before regenerating any `_02-XX_*_text.qmd`, check for a `# manual-edit` marker on line 1 — if present, skip and log a warning.
6. All paths written to logs and `state.json` are **absolute**, not relative.
7. One git commit per stage on tracked branches: `feat(<patient_slug>): <stage> complete`.
8. `patient_slug` must be `lower_snake_case`, max 64 characters, ASCII only.

**CWD constraint (verified 2026-04-26):** Several cingulate R helpers — including `get_domains_with_data()`, `generate_domain_text_qmd()`, parts of `cingulate_llm.R`, and Quarto's `execute-dir: project` — require the working directory to be the patient workspace. Stage subagents (scoring, interpretation, report-writer) must call `setwd(workspace_path)` after `devtools::load_all()` and before any cingulate function. Threading a workspace-path argument through cleanly is deferred as a meaningful refactor.

## QA Output and Human Sign-off

The workspace culminates in two final artifacts:

- **PDF report**: `output/<slug>/report/<slug>.pdf`
- **QA issue list**: `output/<slug>/qa/issue_list.md`

Issues are classified by severity: `blocker | major | minor | nit`. All blockers must be resolved before sign-off. The agent team **never** approves a report — that decision is always a human responsibility. Joey reviews and re-dispatches any halted stage manually; there is no auto-retry.

## Relationship to Broader Patterns

The per-patient workspace contract is an instance of a more general pattern where a filesystem directory serves as a lightweight, human-readable database for a pipeline. It complements:

- [[concepts/neuropsychological-assessment-pipeline]] — the clinical workflow this workspace supports
- [[concepts/luria-neuropsych-pipeline]] — the upstream skill layer that stage subagents wrap
- [[concepts/luria-skills]] — the generic clinical stage definitions that cingulate subagents bind to repo conventions
- [[concepts/modular-report-architecture]] — the QMD include system that populates `interpretation/`
- [[concepts/clinical-data-management]] — broader principles for managing clinical data artifacts
- [[concepts/report-review-qa]] — the quality review stage that reads the completed workspace
- [[concepts/narrative-report-generation]] — how interpretation QMD files become the final PDF
- [[concepts/agent-pipeline-state-management]] — the state machine pattern underlying `state.json`
- [[concepts/edit-protection-pattern]] — the manual-edit marker convention protecting QMD files
- [[concepts/phi-data-handling]] — PHI containment via gitignore and workspace isolation

## Open Issues

- **CWD constraint:** Until cingulate helpers accept an explicit workspace-path argument, stage subagents must call `setwd(workspace_path)` before any R API call. This is documented in each affected stage definition.
- **PHI safeguard:** `output/` must be confirmed in `.gitignore` before any real case is run; verify with `git check-ignore output/`.
- **Fixture case:** A synthetic workspace under `output/_fixture/` is recommended as a smoke-test harness so the chain can be validated without real PHI. This is the recommended next step after scaffolding lands.
- **LLM mode:** The `llm_mode` parameter (`development`, `balanced`, etc.) controls which local model is used for interpretation; switching modes is a documented recovery path for garbled LLM output at the interpretation stage.
- **Subagent name collision:** Two agent files in `.claude/agents/` both declare `name: agent-of-cingulate` in their frontmatter; one should be renamed to `neuropsych-data-analyst`. Flagged as a follow-up, out of scope for this scaffolding.


See also: [[summaries/README]]

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/LLM_INTEGRATION]]