---
sources: [summaries/SKILL.md, summaries/README.md, summaries/LLM_AGENT_MAP.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/agent-team.md]
brief: Maintaining a durable, queryable record of stage progress in a multi-step agent pipeline to enable halting, resuming, and recovery.
---

# Agent Pipeline State Management

Agent pipeline state management refers to the practice of maintaining a durable, queryable record of which stages in a multi-step agent workflow have completed, which are pending, and which have failed — so that the pipeline can be halted, resumed, and recovered without restarting from scratch.

## Core Problem

When a pipeline spans multiple sequential stages — each potentially running a different model, tool, or external service — failures are inevitable. Without explicit state tracking, a crash mid-run means losing all prior work and restarting the entire sequence. State management decouples *progress* from *process*, making runs resumable and auditable.

## The `state.json` Pattern

The Cingulate Agent Team design (see [[summaries/2026-04-26-cingulate-agent-team-design]] and [[summaries/agent-team]]) uses a single `state.json` file as the **single source of truth** for stage progress. Each stage in the pipeline has a recorded status:

| Status | Meaning |
|--------|----------|
| `pending` | Not yet started |
| `in_progress` | Currently being executed |
| `done` | Completed successfully |
| `DONE_WITH_CONCERNS` | Completed with warnings |
| `NEEDS_CONTEXT` | Blocked on missing input |
| `error` / `BLOCKED` | Hard failure; pipeline halts |

The orchestrator reads `state.json` at startup, identifies the first `pending` stage, sets it to `in_progress`, and dispatches the corresponding subagent. On return, it updates the status to `done` or `error`. This means re-dispatching the orchestrator after fixing a failure is safe and idempotent.

A concrete example of the schema used by the cingulate orchestrator:

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

Timestamps on completed stages provide an audit trail without requiring a separate logging layer for progress tracking.

## The Workspace as Inter-Stage Contract

In the Cingulate Agent Team design, `state.json` is one component of a broader **per-patient workspace** (see [[concepts/per-patient-workspace]]) that forms the only contract between stages. All stages read and write under `output/<patient_slug>/`, with well-defined subdirectory conventions:

- `intake/` — normalized referral packet and missing-data inventory
- `scoring/` — per-domain scored CSVs
- `interpretation/` — narrative QMD includes
- `report/` — assembled template and rendered PDF
- `qa/` — quality reviewer findings
- `logs/<stage>.log` — one log per stage invocation

This means stage subagents are fully decoupled: each reads its inputs from known paths and writes its outputs to known paths, with no shared in-memory state.

## Manual State Manipulation

Because state is stored in a plain file, operators can intervene directly:

- **Force a rerun**: Set a stage status back to `pending`.
- **Skip a stage**: Set a stage status to `done` (use cautiously).

This gives human operators fine-grained control without requiring code changes or special tooling — a key property of [[concepts/local-first-architecture]].

## Relationship to the Orchestrator Pattern

In the [[concepts/multi-agent-orchestration]] model, a central orchestrator subagent is responsible for reading state, dispatching stage subagents in order, and writing results back. The state file acts as the contract between the orchestrator and any external observer (human or tooling). This separation of concerns means:

1. Stage subagents are stateless — they receive inputs, do work, return a status.
2. The orchestrator is the only writer of `state.json`.
3. Log files (`logs/<stage>.log`) provide per-stage diagnostic detail without cluttering the state record.
4. Each stage subagent receives a self-contained brief (workspace path, patient_slug, age_group, relevant state slice) — never the full conversation history.

The cingulate orchestrator dispatch loop illustrates this cleanly:

```
orchestrator
  loop:
    state ← read state.json
    next  ← first stage with status == pending
    if next is None: break
    set state.stages[next].status = in_progress; write state.json
    dispatch cingulate-<next> subagent
    on return:
      if DONE → status = done
      if BLOCKED/error → status = error; surface to user; halt
  emit final summary
```

Critically, the orchestrator does **not** auto-retry. Failures halt the chain and bubble up for human review — a deliberate design choice that keeps error recovery auditable.

## Edit Protection as a State Variant

A related pattern is the [[concepts/edit-protection-pattern]]: adding `# manual-edit` as the first line of a generated file signals to the interpretation stage that this artifact should be treated as effectively "locked" — analogous to a `done` status for a file rather than a stage. Stage subagents must check for this marker before regenerating any QMD narrative file, and log a warning if they skip regeneration.

## Failure Surface and Recovery

When a stage returns `error` or `BLOCKED`, the orchestrator surfaces:
- The failing stage name
- A one-line `reason` field written into `state.json`
- A stack trace in the absolute path to the stage log

This triaging information allows the operator to fix the root cause (missing dependency, model unavailable, bad input data) and re-dispatch without guesswork. The [[concepts/smoke-test-scripts]] pattern complements this by catching common environment failures before a run begins — particularly important for the cingulate pipeline's CWD constraint (several helpers require `setwd(workspace_path)` after `devtools::load_all()`).

## Broader Applicability

The `state.json` pattern generalizes beyond neuropsychological reporting pipelines. Any [[concepts/subagent-architecture]] that chains discrete, potentially-failing stages benefits from:

- **Explicit status enums** rather than implicit success/failure flags
- **Append-only logs** per stage for diagnostics
- **Human-editable state files** for operator intervention
- **Idempotent orchestrator dispatch** (safe to re-run)
- **Workspace-as-contract** so subagents share no in-memory state

This connects closely to the design principles in [[summaries/2026-04-26-cingulate-agent-team-design]] and is a practical instantiation of the reliability requirements for the neuropsychological assessment pipeline.

## Related Concepts

- [[concepts/multi-agent-orchestration]] — the orchestrator pattern that consumes state
- [[concepts/subagent-architecture]] — stateless stage agents dispatched by the orchestrator
- [[concepts/per-patient-workspace]] — the broader workspace contract that `state.json` anchors
- [[concepts/edit-protection-pattern]] — file-level state locking for manually edited artifacts
- [[concepts/phi-data-handling]] — reason why pipeline artifacts must be carefully scoped and gitignored
- [[concepts/smoke-test-scripts]] — pre-run environment validation to reduce mid-run failures
- [[concepts/local-first-architecture]] — plain-file state as an operator-friendly design choice
- [[concepts/luria-skills]] — the generic clinical stage definitions that cingulate subagents wrap


See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/README]]

See also: [[summaries/SKILL]]