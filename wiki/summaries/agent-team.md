---
doc_type: short
full_text: sources/agent-team.md
---

# Cingulate Agent Team — Operator Runbook

## Overview

This runbook describes how to operate the **Cingulate Agent Team**, a multi-stage AI pipeline that processes a neuropsychological evaluation case from raw CSV data through to a draft PDF report with a QA issue list. It is intended as a one-page operational guide for running the team for a single patient.

## Pipeline Stages

The pipeline consists of five sequential stages, each driven by a dedicated subagent:

```
intake → scoring → interpretation → report → qa
```

| Subagent | Stage | Model |
|----------|-------|-------|
| `cingulate-orchestrator` | (driver) | opus |
| `cingulate-intake` | 1 | sonnet |
| `cingulate-scoring` | 2 | sonnet |
| `cingulate-interpretation` | 3 | opus |
| `cingulate-report-writer` | 4 | sonnet |
| `cingulate-quality-reviewer` | 5 | opus |

All artifacts are written to `output/<patient_slug>/`. The orchestrator reads/writes `state.json` as the single source of truth for stage progress, following the [[concepts/agent-pipeline-state-management]] pattern. Each subagent is defined as a single markdown file under `.claude/agents/`, consistent with the [[concepts/single-file-agent-pattern]].

## Prerequisites

- `output/` directory must be gitignored (verify with `git check-ignore output/`) to protect PHI, per [[concepts/phi-data-handling]] requirements.
- [[concepts/local-llm-inference]] must be running with required models pulled (configured via [[concepts/yaml-configuration]] in `inst/config/ollama_models.yml`).
- `pdftotext` (from Poppler) must be installed for the QA stage.
- Smoke-test the LLM integration with `cingulate_llm_smoke_test()` before a real run — see [[concepts/smoke-test-scripts]].

## Launching a Run

The orchestrator is invoked via a natural-language prompt in Claude Code, specifying:
- `patient_slug` and `age_group`
- Referral question, records, interview/NSE notes
- Path to raw CSVs
- LLM mode (`development`, `balanced`, etc.)

The orchestrator creates the output directory, initializes `state.json`, dispatches each stage in order, and emits a final summary. This follows the [[concepts/multi-agent-orchestration]] pattern described in [[summaries/2026-04-26-cingulate-agent-team-design]].

## Status Model

Each stage returns one of four statuses:
- `DONE` — completed successfully
- `DONE_WITH_CONCERNS` — completed with warnings
- `NEEDS_CONTEXT` — missing input
- `BLOCKED` — hard failure; orchestrator halts and surfaces the reason and log path

## Common Failure Modes

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `BLOCKED` at scoring: "JEP 66 handshake failed" | R startup timeout | `.rs.restartR()`, avoid auto-loading `library(cingulate)` |
| `BLOCKED` at interpretation: "model not available" | Local LLM not running | `bash setup_ollama.sh` |
| `BLOCKED` at report: "extension not found" | Missing [[concepts/quarto-extensions]] | `quarto add` or symlink extensions |
| `BLOCKED` at qa: "pdftotext not found" | Poppler missing | `brew install poppler` |
| `DONE_WITH_CONCERNS` at interpretation | LLM empty/garbled output | Re-dispatch with failed domain list, switch `llm_mode` |

## Resuming and Rerunning

- To **resume** after a fix: re-dispatch the orchestrator with the same `patient_slug`; it reads `state.json` and continues from the first non-`done` stage.
- To **force a rerun** of a stage: set its status back to `pending` in `state.json`.
- To **skip** a stage: set its status to `done` (use cautiously).

## Edit Protection

Manually edited `_02-XX_<domain>_text.qmd` files can be protected by adding `# manual-edit` as the first line. The interpretation stage will skip such files rather than overwriting them. This implements the [[concepts/edit-protection-pattern]] described in the broader [[concepts/cingulate-engine]] design.

## Outputs

- **PDF report**: `output/<slug>/report/<slug>.pdf`
- **QA issue list**: `output/<slug>/qa/issue_list.md` (severities: `blocker | major | minor | nit`)

Blockers must be resolved before sign-off. **Human approval is always required** — the agent team never auto-approves a report. This is consistent with [[concepts/report-review-qa]] practices and the [[concepts/neuropsychological-assessment-pipeline]] this team implements.

## Related Concepts
- [[concepts/narrative-report-generation]]
- [[concepts/typst-typesetting]]
- [[concepts/fallback-strategy]]

- [[concepts/multi-agent-orchestration]] — multi-agent pipeline coordination patterns
- [[concepts/subagent-architecture]] — design of individual stage agents
- [[concepts/agent-pipeline-state-management]] — state.json-based progress tracking
- [[concepts/phi-data-handling]] — PHI protection in AI pipelines
- [[concepts/neuropsychological-reporting]] — automated report generation for neuropsychological evaluations
- [[concepts/llm-provider-abstraction]] — development vs. balanced LLM operation modes
- [[concepts/per-patient-workspace]] — output directory structure per patient slug