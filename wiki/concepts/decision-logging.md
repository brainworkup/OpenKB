---
sources: [summaries/SKILL.md]
brief: Recording decisions with their rationale so future sessions and agents can understand past choices.
---

# Decision Logging

Decision logging is the practice of explicitly recording decisions — along with the reasoning, constraints, and context that produced them — so that future sessions, agents, or collaborators can understand *why* a choice was made, not just *what* was decided.

## Why It Matters

Without decision logging, rationale evaporates the moment a session ends. Future work then risks:

- Relitigating settled questions
- Reversing decisions whose constraints are no longer visible
- Losing the causal chain between problems and their chosen solutions

Decision logging converts ephemeral reasoning into durable, searchable institutional memory.

## Core Characteristics of a Good Decision Log Entry

| Property | Description |
|---|---|
| **Atomic** | One decision per entry, self-contained |
| **Rationale-bearing** | Captures *why*, not just *what* |
| **Titled strongly** | Findable by future search |
| **Importance-scored** | Prioritizes major decisions over minor notes |
| **Typed** | Classified as `decision`, `procedure`, `learning`, `preference`, or `event` |

## Decision Logging in Practice

The `distill-memory` skill (see [[summaries/SKILL]]) operationalizes decision logging inside AI agent sessions using the `nmem` CLI. Key behaviors:

- **Save proactively** — do not wait to be asked; capture the decision at the moment it is produced
- **Include rationale** — e.g., *"we chose PostgreSQL because ACID compliance is required"*
- **Add vs. Update** — use `nmem m add` for new decisions; use `nmem m update <id>` to refine an existing record rather than creating a duplicate
- **Assign importance scores** — 0.8–1.0 for major decisions, 0.5–0.7 for useful patterns, 0.3–0.4 for minor notes

## Relationship to Architecture Decision Records

Decision logging as practiced in agent memory is a lightweight, session-level analogue to formal [[concepts/architecture-decision-records]] (ADRs). ADRs are structured documents written for software architecture choices; decision logs are atomic memory units captured in real time during any kind of work session.

Both share the same underlying principle: decisions without rationale are liabilities.

## What to Log

**Good candidates:**
- Technology or tool selections with justification
- Chosen approaches after evaluating alternatives
- Constraints that ruled out other options
- Plans that future sessions must resume
- Preferences or policies that shape ongoing work

**Skip:**
- Routine fixes with no generalizable lesson
- Work in progress likely to change
- Widely known general information

## Related Concepts

- [[concepts/knowledge-capture]] — broader discipline of preserving insights, not just decisions
- [[concepts/knowledge-continuity]] — ensuring knowledge persists across sessions and agents
- [[concepts/persistent-memory]] — the storage layer that holds decision logs over time
- [[concepts/architecture-decision-records]] — formal structured variant of decision logging
- [[concepts/skills-modules]] — the agent skill system that triggers decision logging
- [[concepts/luria-skills]] — example of a skill ecosystem where decision logging is relevant
- [[summaries/SKILL]] — the `distill-memory` skill that implements decision logging in agent sessions
