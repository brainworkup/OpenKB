---
doc_type: short
full_text: sources/2026-04-26-cingulate-agent-team-design.md
---

# Cingulate Agent Team — Design

**Source:** 2026-04-26-cingulate-agent-team-design
**Date:** 2026-04-26
**Status:** Design / scaffolding only — no live PHI runs yet
**Author:** Claude (autonomous draft, `agent-team-scaffolding` branch)
**Approval gate:** Joey reviews before any subagent is invoked against a real case

## Purpose

Provides reusable scaffolding so any future cingulate neuropsychology case can be processed by spawning a single **orchestrator subagent**, which fans work out to five specialized stage subagents and produces a draft PDF report plus a QA issue list under a per-patient workspace.

Out of scope: running real cases, modifying R exports, pushing to GitHub.

## Two-Layer Architecture

The design separates generic clinical reasoning from repo-specific implementation:

- **`luria-*` skills** — thin, generic descriptions of *what* each clinical stage does (not cingulate-specific)
- **`cingulate-*` subagents** — bind those stages to this repo's R API, file conventions, and Quarto templates
- **`cingulate-orchestrator`** — top-level subagent that dispatches the five stage subagents

See [[concepts/multi-agent-orchestration]] and [[concepts/luria-skills]].

## Agent Roster

| Subagent | Wraps Skill | Responsibility |
|---|---|---|
| `cingulate-orchestrator` | luria-neuropsych-orchestrator | Drives chain, manages `state.json`, dispatches stages, surfaces failures |
| `cingulate-intake` | luria-case-intake | Normalize referral question, records, interview, NSE; track missing data |
| `cingulate-scoring` | luria-score-processing | Load CSVs via DuckDB, run domain processors, produce scored tables |
| `cingulate-interpretation` | luria-interpretation | Generate per-domain narrative QMD files via LLM router |
| `cingulate-report-writer` | luria-report-writing | Assemble template, render Quarto/Typst → PDF |
| `cingulate-quality-reviewer` | luria-quality-review | PHI scan, completeness, validity language, test-security review |

## Per-Patient Workspace

All stages read/write under `output/<patient_slug>/`. This directory tree is the only inter-stage contract. See [[concepts/per-patient-workspace]].

Key paths:
- `state.json` — orchestrator's source of truth (stage statuses, timestamps)
- `intake/packet.md`, `intake/missing_data.md`
- `data-raw/csv/` — raw score inputs
- `duckdb/staged.parquet` — scoring stage output
- `scoring/<domain>_scored.csv`
- `interpretation/_02-XX_<domain>_text.qmd`
- `report/template.qmd`, `report/<slug>.pdf`
- `qa/issue_list.md`
- `logs/<stage>.log`

`state.json` tracks status per stage: `pending` | `in_progress` | `done` | `error`. This pattern is described in [[concepts/agent-pipeline-state-management]].

## Dispatch Flow

The orchestrator loops over `state.json`, finds the first `pending` stage, sets it to `in_progress`, dispatches the corresponding subagent, and updates status on return. Key policy: **no auto-retry** — failures halt the chain and surface to Joey for manual review and re-dispatch.

Each stage subagent receives a self-contained brief (workspace path, patient_slug, age_group, relevant state slice) — never the full conversation history. This isolation approach reflects the [[concepts/subagent-architecture]] pattern.

## Stage Contracts

| Stage | Key R/API Calls |
|---|---|
| intake | optional `cingulate_quick_start()` |
| scoring | `load_data_duckdb()`, `process_all_domains()`, `query_neuropsych()` |
| interpretation | `generate_domain_text_qmd()`, `process_domains_with_llm()`, `run_llm_for_all_domains()` |
| report | `generate_assessment_report()`, `quarto render` |
| qa | `pdftotext` + heuristics; no R |

## Conventions Every Stage Inherits

1. Use `devtools::load_all('.')` — not `library(cingulate)`
2. Default LLM mode: `development`; switch to `production` only when explicitly authorized
3. Never write directly to `data-raw/csv/` — copy inputs to workspace first
4. Domain numbering (`01_iq`, `02_academics`, …) is fixed; do not renumber
5. Check for `# manual-edit` marker before regenerating any QMD file — see [[concepts/edit-protection-pattern]]
6. Output paths are absolute in all logs and `state.json`
7. One git commit per stage on tracked branches
8. `patient_slug` format: `lower_snake_case`, max 64 chars, ASCII only

See [[concepts/cingulate-engine]] and [[concepts/quarto]].

## Open Questions / Follow-ups

- **CWD constraint:** Several cingulate helpers require `setwd(workspace_path)` after `devtools::load_all()` because they do not accept a workspace-path argument. A clean refactor is noted but out of scope.
- **Subagent name collision:** Two agent files both declare `name: agent-of-cingulate`; one should be renamed to `neuropsych-data-analyst`.
- **LLM mode policy:** Joey decides when to flip from `development` to `production`. See [[concepts/llm-provider-abstraction]].
- **PHI handling:** `output/` must be gitignored before any real run. See [[concepts/phi-data-handling]].
- **Test fixture:** A synthetic case under `output/_fixture/` is recommended for smoke-testing the chain without real PHI. See [[concepts/smoke-test-scripts]].

## Relationship to Other Documents

- Depends on [[concepts/luria-skills]] for stage definitions; see also [[concepts/luria-neuropsych-pipeline]] and [[concepts/luria-overview]]
- Uses [[concepts/r-neuropsych-packages]] for data loading and domain processing
- Produces output via [[concepts/quarto]] and [[concepts/typst-typesetting]] (Quarto/Typst rendering)
- Implements the [[concepts/multi-agent-orchestration]] pattern with [[concepts/narrative-report-generation]] as its primary output
- QA stage enforces [[concepts/validity-language]] and [[concepts/report-review-qa]]

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/skills-modules]]
- [[concepts/local-first-architecture]]
