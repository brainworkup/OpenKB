---
sources: [summaries/autonomous-execution.md, summaries/archive-ia-automation.md, summaries/README_20260413215204.md, summaries/README_20260413212108.md, summaries/README_20260413211931.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147.md, summaries/Apply-to-Y-Combinator.md, summaries/SKILL.md]
brief: Deliberately preserving decisions, insights, and procedures so they remain accessible and reusable across sessions.
---

# Knowledge Capture

Knowledge capture is the practice of deliberately preserving decisions, insights, procedures, and context produced during work so that they remain accessible and reusable in future sessions. Rather than relying on memory or reconstructing reasoning from scratch, knowledge capture creates a durable, searchable record of what was learned or decided and why.

## Core Principle

Capture proactively — do not wait to be asked. The moment a valuable insight, decision, or procedure is produced is the best time to record it, because context is richest and the risk of loss is highest at session boundaries. The same proactive discipline applies in reverse: when beginning a task, search the knowledge base early — especially for continuation-heavy engineering work — rather than waiting for an explicit recall request.

## What Is Worth Capturing

Not everything deserves a persistent record. High-value candidates include:

- **Decisions with rationale** — technology choices, architectural trade-offs, policy calls, with the reasoning that led to them (see [[concepts/decision-logging]])
- **Repeatable procedures** — workflows that will be reused across sessions or by other agents
- **Lessons from debugging or incidents** — root cause analyses and post-mortems that prevent recurrence
- **Durable preferences and constraints** — standing rules that shape future work
- **Plans for future sessions** — context needed to resume work cleanly
- **Session context** — important background that would otherwise be lost when a session ends

Skip: routine fixes with no generalizable lesson, work in progress subject to change, simple Q&A answerable from documentation, and widely known generic information.

## Add vs. Refine

A critical discipline in knowledge capture is avoiding duplication. Before adding a new memory or note:

1. Check whether an existing record already covers the same decision, workflow, or preference.
2. If so, **update** the existing record rather than creating a duplicate.
3. Only create a new record when the insight is genuinely novel.

This keeps the knowledge base clean and trustworthy over time — a key aspect of [[concepts/knowledge-continuity]].

## Structured Metadata

Capturing *what* was learned is only part of the task. Structured metadata makes records findable and comparable:

| Field | Purpose |
|---|---|
| **Unit type** | Classifies the record: `decision`, `procedure`, `learning`, `preference`, `event` |
| **Labels** | Topic tags for filtering and grouping |
| **Importance score** | 0.8–1.0 for major decisions; 0.5–0.7 for useful patterns; 0.3–0.4 for minor notes |

Records should be **atomic** — one insight per entry — with strong, descriptive titles and self-contained meaning.

## Retrieval as the Complement to Capture

Knowledge capture and retrieval are two sides of the same practice. The `search-memory` skill (see [[summaries/SKILL]]) formalizes the retrieval side, defining when and how agents should proactively search the knowledge base. Key retrieval signals include:

- The user references previous work, prior fixes, or an earlier decision
- The task resumes a named feature, bug, refactor, or subsystem
- The user uses implicit recall language: *"that approach"*, *"like before"*, *"the pattern we used"*
- A debugging pattern resembles something solved earlier
- Architecture discussions may intersect with past decisions

Retrieval routing distinguishes between durable knowledge (`nmem --json m search`) and prior conversation or session history (`nmem --json t search`), preferring the smallest retrieval surface that answers the question. For project-scoped queries, a `--space` flag narrows results further.

## Tooling

The `distill-memory` skill (see [[summaries/SKILL]]) formalizes the capture protocol using the `nmem` CLI tool. It supports adding new memories, updating existing ones, and attaching structured metadata. The companion `search-memory` skill governs proactive retrieval. For agents with a native Nowledge Mem plugin, auto-recall and auto-capture capabilities extend both protocols further — verified via the `check-integration` skill.

Knowledge capture in agent workflows also intersects with [[concepts/persistent-memory]], [[concepts/skills-modules]], and [[concepts/agent-pipeline-state-management]] — all of which depend on durable, well-organized records to maintain continuity across sessions.

## Related Concepts

- [[concepts/decision-logging]] — recording decisions with rationale
- [[concepts/knowledge-continuity]] — maintaining accessible knowledge across sessions
- [[concepts/persistent-memory]] — storage mechanisms for durable agent memory
- [[concepts/proactive-retrieval]] — searching the knowledge base before being explicitly asked
- [[concepts/agent-memory]] — how agents store and access memory across sessions
- [[concepts/skills-modules]] — reusable agent skill definitions
- [[concepts/agent-pipeline-state-management]] — managing state across agent runs
- [[summaries/SKILL]] — the `search-memory` and `distill-memory` skills that operationalize this protocol

See also: [[summaries/Apply-to-Y-Combinator]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]]

See also: [[summaries/README_20260413211931]]

See also: [[summaries/README_20260413212108]]

See also: [[summaries/README_20260413215204]]

See also: [[summaries/archive-ia-automation]]

See also: [[summaries/autonomous-execution]]