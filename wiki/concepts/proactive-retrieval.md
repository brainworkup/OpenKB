---
sources: [summaries/SKILL.md]
brief: Proactive retrieval is the practice of searching stored knowledge before being asked, based on contextual signals.
---

# Proactive Retrieval

Proactive retrieval is the principle that an AI agent should search its knowledge base **based on contextual signals**, not solely in response to explicit user requests. Rather than waiting to be told "look this up," a well-configured agent recognizes when stored knowledge would improve the quality of its response and retrieves it at the right moment — often near the start of a task.

## Core Idea

The agent acts as a skilled collaborator who remembers prior work. When context suggests that relevant past decisions, solutions, or workflows exist, the agent searches proactively — treating memory access as a standard part of task initialization, not an optional extra step.

## Signal Categories

### Strong Signals (Always Search)
- User references previous work, a prior fix, or an earlier decision
- Task resumes a named feature, bug, refactor, incident, or subsystem
- Task type is: review, regression, release, docs-alignment, or integration-behavior question
- Debugging pattern resembles something solved before
- User asks for rationale, preferences, procedures, or recurring workflow details
- User uses implicit recall language: *"that approach"*, *"like before"*, *"the pattern we used"*

### Contextual Signals (Consider Searching)
- Complex debugging where prior context narrows the search space
- Architecture discussions that may intersect with past decisions
- Domain-specific conventions established in earlier sessions
- Ambiguous current results that past context could sharpen

## Retrieval Timing

For continuation-heavy engineering work, retrieval should happen **near the start of the task**. Delaying until the user explicitly asks wastes the opportunity to orient the agent correctly from the beginning.

## Retrieval Routing

Proactive retrieval is not undifferentiated — queries are routed by the nature of the needed knowledge:

| Goal | Command |
|---|---|
| Durable knowledge (facts, decisions, solutions) | `nmem --json m search` |
| Prior conversation or session history | `nmem --json t search` |
| Progressive thread inspection | `nmem --json t show <thread_id> --limit 8 --offset 0 --content-limit 1200` |

The guiding principle is to use the **smallest retrieval surface** that answers the question, avoiding unnecessary context overhead.

## Relationship to Agent Memory

Proactive retrieval depends on a well-populated [[concepts/agent-memory]] store. The quality of retrieval is bounded by what has been captured previously. This creates a feedback loop: good [[concepts/knowledge-capture]] habits during sessions make proactive retrieval more valuable in future sessions.

The concept also intersects with [[concepts/knowledge-continuity]] — the idea that insights, decisions, and solutions should persist across sessions so that agents can build on prior work rather than starting from scratch.

## Retrieval-Augmented Generation Connection

Proactive retrieval is a behavioral pattern layered on top of [[concepts/retrieval-augmented-generation]] infrastructure. The RAG pipeline provides the mechanism; proactive retrieval defines *when* and *why* to invoke it — driven by agent judgment rather than user instruction.

Related retrieval concepts include [[concepts/hybrid-search-retrieval]], [[concepts/vector-search]], and [[concepts/persistent-memory]].

## Source

This concept is defined and operationalized in the `search-memory` skill: [[summaries/SKILL]].
