---
sources: [summaries/autonomous-execution.md, summaries/agentic-workflows.md, summaries/SKILL.md, summaries/README.md, summaries/LLM_AGENT_MAP.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/agent-team.md]
brief: Durable stage tracking that makes agent workflows resumable and auditable.
---

# Agent Pipeline State Management

Agent pipeline state management is the practice of maintaining a durable, queryable record of which stages in a multi-step agent workflow have completed, which are pending, and which have failed, so the workflow can be halted, resumed, reviewed, and recovered without restarting from scratch. In agentic data pipelines, this state layer is what allows a planning or orchestrating agent to delegate work across tools and subagents while preserving progress across long-running tasks.

## Core Problem

When a pipeline spans multiple sequential stages, each potentially using a different model, runtime, tool, or external service, failures are inevitable. This is especially true in end-to-end workflows that include ingestion, transformation, redaction, analysis, and report generation. Without explicit state tracking, a crash in the middle of the run can erase the practical distinction between completed work and unfinished work, forcing a full restart.

State management decouples *progress* from *process*. The agent may stop, but the workflow state persists. This makes runs resumable, auditable, and easier to supervise in environments where some steps can proceed silently while higher-risk steps require human review. In this sense, state management is a core reliability layer for [[concepts/multi-agent-orchestration]] and related agentic workflow designs.

## The `state.json` Pattern

The Cingulate Agent Team design (see [[summaries/2026-04-26-cingulate-agent-team-design]] and [[summaries/agent-team]]) uses a single `state.json` file as the single source of truth for pipeline progress. Each stage has an explicit recorded status:

| Status | Meaning |
|--------|----------|
| `pending` | Not yet started |
| `in_progress` | Currently being executed |
| `done` | Completed successfully |
| `DONE_WITH_CONCERNS` | Completed with warnings |
| `NEEDS_CONTEXT` | Blocked on missing input |
| `error` / `BLOCKED` | Hard failure; pipeline halts |

The orchestrator reads `state.json` at startup, identifies the first runnable stage, marks it `in_progress`, dispatches the corresponding tool or subagent, and then updates the stage to `done`, `DONE_WITH_CONCERNS`, or `error` on return. This makes re-dispatch after interruption safe and largely idempotent.

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

Timestamps on completed stages provide a lightweight audit trail. This is particularly useful in agentic workflows where an orchestrator may span long horizons, invoke multiple runtimes, and aggregate outputs over time rather than in a single synchronous execution.

## The Workspace as Inter-Stage Contract

In the Cingulate Agent Team design, `state.json` is one component of a broader [[concepts/per-patient-workspace]] that acts as the contract between stages. All stages read and write under `output/<patient_slug>/`, with well-defined subdirectory conventions:

- `intake/` — normalized referral packet and missing-data inventory
- `scoring/` — per-domain scored CSVs
- `interpretation/` — narrative QMD includes
- `report/` — assembled template and rendered PDF
- `qa/` — quality reviewer findings
- `logs/<stage>.log` — one log per stage invocation

This design keeps stage agents decoupled. Each stage reads inputs from known paths and writes outputs to known paths, rather than depending on shared in-memory context. That matters even more in hybrid agent systems where some stages may call Python code, others may call R, and others may invoke external tools. The workspace becomes the stable boundary across runtimes, while the state file records what has happened at that boundary.

## Manual State Manipulation

Because state is stored in a plain file, operators can intervene directly:

- **Force a rerun**: set a stage status back to `pending`
- **Skip a stage**: set a stage status to `done` or another terminal status, with caution
- **Pause for review**: leave downstream stages pending after a high-stakes step

This gives human operators fine-grained control without code changes or specialized orchestration software, which is aligned with [[concepts/local-first-architecture]]. It also supports a practical autonomy model: deterministic or low-risk stages may run unattended, while sensitive decisions such as PII handling or final report approval can be surfaced for manual review before the next stage is released.

## Relationship to the Orchestrator Pattern

In [[concepts/multi-agent-orchestration]], a central orchestrator is responsible for reading state, dispatching stage subagents in order, and writing results back. The state file is the contract between the orchestrator, the stage agents, and any external observer.

This separation of concerns implies:

1. Stage subagents are effectively stateless at dispatch time: they receive inputs, perform work, and return a status.
2. The orchestrator is the authoritative writer of `state.json`.
3. Log files (`logs/<stage>.log`) provide diagnostic detail without cluttering the state record.
4. Each stage subagent receives a focused brief such as workspace path, patient slug, age group, and relevant state slice rather than the full conversation history.
5. State, not prompt history, is what allows the pipeline to continue safely across interruptions.

The cingulate orchestrator dispatch loop illustrates this cleanly:

```text
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

This is a concrete implementation of an orchestrator-worker design. The orchestrator plans and routes work; the stage agents execute specialized tasks; the state file preserves shared progress across the whole chain.

## Edit Protection as a State Variant

A related pattern is [[concepts/edit-protection-pattern]]: adding `# manual-edit` as the first line of a generated file signals that the artifact should be treated as locked. This behaves like file-level state, analogous to marking a stage effectively complete for that artifact.

Stage subagents should check for this marker before regenerating narrative files and log a warning when they skip regeneration. This is useful in workflows where human-authored corrections must take precedence over automated regeneration.

## Failure Surface and Recovery

When a stage returns `error`, `BLOCKED`, or `NEEDS_CONTEXT`, the orchestrator should surface:

- the failing stage name
- a one-line `reason` field written into `state.json`
- a stack trace or diagnostic trace in the stage log
- any missing prerequisite artifact or context required to resume

This makes triage faster: the operator can fix the root cause, such as missing dependencies, unavailable models, malformed data, or absent inputs, and then re-dispatch without guesswork.

This pattern is especially important in hybrid Python and R pipelines, where a failure may occur at runtime boundaries rather than within a single language environment. If one stage invokes R from a Python orchestrator, the state file allows the system to record that the boundary-crossing step failed without invalidating prior ingestion or preprocessing work. The [[concepts/r-python-integration]] concerns are therefore not separate from state management; they increase the need for explicit durable progress tracking.

The [[concepts/smoke-test-scripts]] pattern complements this approach by catching environment failures before a run begins.

## Broader Applicability

The `state.json` pattern generalizes well beyond neuropsychological reporting. Any [[concepts/subagent-architecture]] that chains discrete, failure-prone stages benefits from:

- **Explicit status enums** instead of implicit success/failure
- **Persistent workflow memory** separate from agent prompt history
- **Append-only or per-stage logs** for diagnosis
- **Human-editable state files** for operator intervention
- **Idempotent orchestrator dispatch** so reruns are safe
- **Workspace-as-contract** so agents share artifacts, not volatile in-memory state
- **Risk-gated advancement** so some stages run automatically and others pause for review

This is especially relevant for agentic data science workflows that combine ingestion, processing, deidentification, report generation, tables, and figures under one orchestrating agent. In those systems, state management is what turns a sequence of tool calls into a recoverable pipeline rather than a fragile one-shot script.

The concept also aligns closely with [[concepts/agent-memory]] and [[concepts/persistent-memory]], but with a narrower focus: not general long-term memory, but operational workflow state that determines what the system should do next.

## Related Concepts

- [[concepts/multi-agent-orchestration]] — the orchestrator pattern that consumes and updates state
- [[concepts/subagent-architecture]] — stateless stage agents dispatched by the orchestrator
- [[concepts/per-patient-workspace]] — the broader artifact contract that `state.json` anchors
- [[concepts/edit-protection-pattern]] — file-level locking for manually edited artifacts
- [[concepts/phi-data-handling]] — why pipeline outputs must be scoped and protected carefully
- [[concepts/pii-redaction-pipelines]] — a high-stakes stage type that often warrants explicit review gates
- [[concepts/smoke-test-scripts]] — pre-run environment validation to reduce mid-run failures
- [[concepts/local-first-architecture]] — plain-file state as an operator-friendly design choice
- [[concepts/agent-memory]] — broader agent memory patterns related to workflow continuity
- [[concepts/persistent-memory]] — durable memory mechanisms adjacent to pipeline state
- [[concepts/r-python-integration]] — cross-runtime execution that increases the need for durable stage tracking
- [[concepts/luria-skills]] — the generic clinical stage definitions that cingulate subagents wrap

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/README]]

See also: [[summaries/SKILL]]

## Related Documents
- [[summaries/agentic-workflows]]


See also: [[summaries/autonomous-execution]]