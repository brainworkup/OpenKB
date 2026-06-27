---
doc_type: short
full_text: sources/SKILL.md
---

# Summary: search-memory Skill

## Overview
This document defines the `search-memory` skill for AI agents, providing structured guidance on when to proactively search a personal knowledge base (powered by **Nowledge Mem**) and how to route retrieval queries effectively.

## Purpose
The skill enables AI agents to recognize when stored knowledge — past breakthroughs, decisions, fixes, or workflows — is relevant to the current task, and to retrieve it without waiting for an explicit user request.

## When to Search

### Strong Signals
- User references previous work, prior fixes, or earlier decisions
- Task resumes a named feature, bug, refactor, incident, or subsystem
- Task type is: review, regression, release, docs-alignment, or integration-behavior question
- Debugging pattern resembles a previously solved problem
- User asks for rationale, preferences, procedures, or recurring workflow details
- User uses implicit recall language: *"that approach"*, *"like before"*, *"the pattern we used"*

### Contextual Signals
- Complex debugging where prior context narrows the search space
- Architecture discussions intersecting with past decisions
- Domain-specific conventions established in earlier sessions
- Ambiguous current results that past context could sharpen

## Retrieval Routing
1. **Durable knowledge**: `nmem --json m search`
2. **Prior conversation / session history**: `nmem --json t search`
3. **Progressive thread inspection**: `nmem --json t show <thread_id> --limit 8 --offset 0 --content-limit 1200` when a result includes `source_thread`
4. Use the smallest retrieval surface that answers the question
5. For project-scoped queries, append `--space "<space name>"`

## Key Principle
For continuation-heavy engineering work, search **near the start of the task** — do not wait for an explicit user request.

## Integration
The skill works via CLI in any agent. For enhanced capabilities (auto-recall, auto-capture, graph tools), a native Nowledge Mem plugin may be available — verified via the `check-integration` skill or the [integrations docs](https://mem.nowledge.co/docs/integrations).

## Related Concepts
- [[concepts/hybrid-search-retrieval]]
- [[concepts/decision-logging]]
- [[concepts/knowledge-continuity]]
- [[concepts/vector-search]]
- [[concepts/agent-memory]]
- [[concepts/proactive-retrieval]]
- [[concepts/persistent-memory]]
- [[concepts/knowledge-capture]]
- [[concepts/skills-modules]]
- [[concepts/retrieval-augmented-generation]]