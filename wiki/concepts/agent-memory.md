---
sources: [summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/README.md, summaries/SKILL.md]
brief: Agent memory enables AI agents to store, retrieve, and apply past knowledge within and across sessions.
---

# Agent Memory

Agent memory refers to the mechanisms by which AI agents persist, index, and retrieve knowledge across sessions, tasks, and conversations. Rather than treating each interaction as stateless, agent memory allows an agent to draw on prior decisions, solved problems, established preferences, and recorded workflows to deliver more accurate and contextually grounded responses.

## Core Idea

Without memory, an AI agent starts fresh on every task. With memory, it can recognize that a debugging pattern resembles something solved before, that an architecture decision was already made, or that the user has an established convention worth respecting. This transforms the agent from a single-shot responder into a **knowledge-accumulating collaborator**.

## Types of Memory

### Durable / Semantic Memory
Long-lived facts, decisions, breakthroughs, and procedures stored in a persistent knowledge base. Queried with tools like `nmem --json m search`. This is the primary target for project knowledge, rationale, and recurring workflow details.

### Episodic / Session Memory
Records of prior conversations or session transcripts. Queried with `nmem --json t search`. Useful when the user is asking about *what was said* in a prior session, not just *what was decided*. Threads can be inspected progressively using `nmem --json t show <thread_id>`.

## When Memory Should Be Activated

The [[summaries/SKILL]] document defines two tiers of retrieval signals:

**Strong signals** (always search):
- User references prior work, fixes, or decisions
- Task resumes a named feature, bug, incident, or subsystem
- Task type is review, regression, release, or integration-behavior question
- User uses implicit recall language: *"that approach"*, *"like before"*

**Contextual signals** (consider searching):
- Complex debugging where prior context narrows the search space
- Architecture discussions that may intersect with past decisions
- Domain-specific conventions previously established
- Ambiguous results that past context could sharpen

## Proactive vs. Reactive Retrieval

A key principle in agent memory design is **proactive retrieval** — the agent should search memory near the start of a continuation-heavy task, not wait for the user to explicitly request it. See [[concepts/proactive-retrieval]] for more on this pattern.

## Retrieval Routing

Effective agent memory systems route queries to the smallest retrieval surface that answers the question:

1. Durable knowledge store first (`nmem --json m search`)
2. Session/thread history second (`nmem --json t search`)
3. Progressive thread inspection when a `source_thread` reference is returned
4. Space-scoped queries (`--space "<space name>"`) for project-specific retrieval

This routing strategy connects to broader patterns in [[concepts/retrieval-augmented-generation]] and [[concepts/hybrid-search-retrieval]].

## Relationship to Persistent Memory

Agent memory at the session level is transient; at the knowledge-base level it becomes [[concepts/persistent-memory]]. The distinction matters for tool selection: episodic memory fades or is archived, while durable memory is explicitly curated and indexed for long-term reuse.

## Knowledge Capture and Continuity

For memory to be useful, knowledge must be captured consistently. This connects to [[concepts/knowledge-capture]] and [[concepts/knowledge-continuity]] — the practices of recording decisions, solutions, and context in a retrievable form so that future agent sessions can benefit from past work.

## Integration Considerations

Agent memory can be implemented via CLI tools (as in the `search-memory` skill described in [[summaries/SKILL]]) or via native plugins that provide auto-recall and auto-capture. The choice of backend — whether a vector store, graph, or hybrid index — affects retrieval quality and latency. See [[concepts/vector-search]] and [[concepts/lancedb-vector-store]] for storage-layer options.

## Related Concepts

- [[concepts/proactive-retrieval]]
- [[concepts/persistent-memory]]
- [[concepts/knowledge-capture]]
- [[concepts/knowledge-continuity]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/hybrid-search-retrieval]]
- [[concepts/vector-search]]
- [[concepts/lancedb-vector-store]]
- [[concepts/skills-modules]]
- [[concepts/multi-agent-orchestration]]


See also: [[summaries/README]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]]