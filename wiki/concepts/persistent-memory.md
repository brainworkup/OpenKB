---
sources: [summaries/SKILL.md, summaries/0004-soul-style-profile-json.md, summaries/CLAUDE.md, summaries/index.md, summaries/AGENTS_luria.md, summaries/deepagents_merged_mem_notes.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/DEMO_GUIDE.md, summaries/Hermes-Agent-Documentation-Hermes-Agent.md]
brief: How AI agents retain knowledge, skills, and context across sessions to compound learning over time.
---

# Persistent Memory in AI Agents

Persistent memory refers to an AI agent's ability to retain information, skills, and contextual knowledge **across multiple sessions** — so that each conversation builds on what came before, rather than starting from a blank slate. It is a foundational capability that separates autonomous agents from stateless chatbots.

## Why It Matters

Most language model interactions are ephemeral: the model has no recollection of past exchanges once a session ends. Persistent memory breaks this limitation, enabling agents to:

- Accumulate a growing understanding of the user's preferences, habits, and goals
- Reuse learned procedures and skills without relearning them from scratch
- Provide increasingly personalized and context-aware responses over time
- Preserve decisions, rationale, and hard-won lessons that would otherwise be lost when a session ends

## Capturing What Is Worth Keeping

Not everything generated in a session deserves to be stored. The `distill-memory` skill defines a protocol for **proactive, discriminating memory capture** using the `nmem` CLI tool. The core principle: save without waiting to be asked, but save selectively.

### Good Candidates for Persistent Memory

- **Decisions with rationale** — e.g., technology choices and the reasoning behind them (see also [[concepts/decision-logging]])
- **Repeatable procedures or workflows** — steps that will be reused across sessions
- **Lessons from debugging or incidents** — root cause analyses and post-mortems
- **Durable preferences or constraints** — standing rules that shape future work
- **Plans for future sessions** — context needed to resume work cleanly
- **Important session context** — information that would otherwise be lost

### What to Skip

- Routine fixes with no generalizable lesson
- Work in progress subject to change
- Simple Q&A answerable from documentation
- Generic, widely known information

## Structured Memory Saves

Memories should be saved with metadata that makes them searchable and meaningful over time. The `distill-memory` skill recommends:

| Field | Purpose | Examples |
|---|---|---|
| `--unit-type` | Classifies the memory | `decision`, `procedure`, `learning`, `preference`, `event` |
| `-l` | Labels for filtering | topic tags |
| `-i` | Importance score | 0.8–1.0 major decisions, 0.5–0.7 useful patterns, 0.3–0.4 minor notes |

Memories should be **atomic** and **standalone** — strong titles, clear meaning, focused on what was learned or decided. This aligns with the broader goal of [[concepts/knowledge-capture]]: preserving insight in a form that remains useful long after the originating session.

## Add vs. Update: Memory Hygiene

A key discipline in persistent memory management is avoiding duplication:

- Use `nmem --json m add` for **genuinely new** insights.
- Use `nmem m update <id>` when an existing memory captures the same topic and new information **refines** it.
- At the end of a substantial task, explicitly check whether one durable memory should be added or updated.

This mirrors the principle of [[concepts/knowledge-continuity]]: memory stores should grow in quality, not just quantity.

## Proactive Retrieval: The `search-memory` Skill

Persistent memory is only valuable if it is retrieved at the right moment. The `search-memory` skill (see [[summaries/SKILL]]) defines when and how agents should query stored knowledge — without waiting for the user to ask.

### When to Search

**Strong signals:**
- The user references previous work, a prior fix, or an earlier decision
- The task resumes a named feature, bug, refactor, incident, or subsystem
- The task is a review, regression, release, docs-alignment, or integration-behavior question
- A debugging pattern resembles something solved earlier
- The user asks for rationale, preferences, procedures, or recurring workflow details
- The user uses implicit recall language: *"that approach"*, *"like before"*, *"the pattern we used"*

**Contextual signals:**
- Complex debugging where prior context narrows the search space
- Architecture discussions that may intersect with past decisions
- Domain-specific conventions the user has established before
- Ambiguous current results that past context would sharpen

### Retrieval Routing

1. **Durable knowledge**: `nmem --json m search`
2. **Prior conversation / session history**: `nmem --json t search`
3. **Progressive thread inspection**: `nmem --json t show <thread_id> --limit 8 --offset 0 --content-limit 1200` when a result includes `source_thread`
4. Use the smallest retrieval surface that answers the question
5. For project-scoped queries, append `--space "<space name>"`

For continuation-heavy engineering work, search **near the start of the task**. This pairs naturally with the capture discipline above: memory is only as useful as the retrieval that surfaces it. See [[concepts/proactive-retrieval]] for a broader treatment of this principle.

## How Hermes Agent Implements Persistent Memory

The [[summaries/Hermes-Agent-Documentation-Hermes-Agent]] describes a multi-layered memory architecture at the core of Hermes Agent's closed learning loop:

### 1. Episodic / Cross-Session Recall
Hermes uses **FTS5 full-text search** combined with **LLM summarization** to index and retrieve relevant memories from past sessions. This allows the agent to surface pertinent context even from much earlier interactions.

### 2. Skill Memory (Procedural)
The agent autonomously **creates skills from experience** and stores them for future reuse. Crucially, these skills are **self-improved during use** — the agent refines a skill each time it applies it, so procedural knowledge deepens over time. Skills conform to the open standard at agentskills.io, making them portable and shareable across users and deployments. The `distill-memory` and `search-memory` skills are examples of this pattern: reusable, structured procedures for knowledge capture and retrieval, stored and invoked by any compatible agent. See [[concepts/skills-modules]] for more on how modular skills are structured.

### 3. User Modeling
Hermes integrates with **Honcho** (by Plastic Labs) for dialectic user modeling — a continuous process of building and updating a structured model of who the user is, what they value, and how they communicate. This model persists and grows across sessions. See [[concepts/honcho-ai-peer-observation]] for more on this approach.

### 4. Memory Nudges
The agent periodically **nudges itself** to consolidate and persist knowledge it might otherwise lose — an internal mechanism that treats memory maintenance as an active, ongoing task rather than a passive side effect.

## Relationship to the Learning Loop

Persistent memory is the substrate of Hermes Agent's defining feature: its **closed learning loop**. Without durable memory, there can be no compounding improvement. The loop works as follows:

1. The agent acts and observes outcomes
2. It creates or updates skills based on experience
3. Memory nudges and explicit capture protocols ensure key knowledge is written to persistent storage
4. On future sessions, FTS5 recall and user modeling retrieve the right context — guided by retrieval skills like `search-memory`
5. The agent acts with richer context → the loop continues

## Storage and Infrastructure

Because Hermes Agent is designed to run on persistent infrastructure — VPS servers, GPU clusters, or serverless platforms — its memory stores are not tied to a local machine or a single session's process. This infrastructure flexibility is a prerequisite for persistent memory to function reliably. See [[concepts/multi-platform-messaging]] for how this memory-backed agent surfaces across different communication channels.

For local-first deployments, see [[concepts/local-first-architecture]] and [[concepts/sqlite-as-vector-store]] for storage strategies that keep memory under user control.

## Contrast with Stateless Models

Traditional large language models (see [[summaries/The-Complete-Guide-to-AI-Architectures-From-Neural-Networks-to-Foundation-Models]]) operate without session-level memory by default. Persistent memory is an **architectural addition** layered on top of the base model, implemented through retrieval systems, structured storage, and agent-side logic — not baked into model weights.

## Key Takeaways

- Persistent memory transforms a stateless model into a continuously improving agent
- It encompasses episodic recall, procedural skill memory, and user modeling
- Proactive, discriminating **capture** — guided by protocols like `distill-memory` — is as important as the storage infrastructure itself
- Proactive, signal-driven **retrieval** — guided by protocols like `search-memory` — ensures stored knowledge is actually used
- Hermes Agent implements it through FTS5 search, Honcho integration, autonomous skill creation, and self-directed memory nudges
- It is the foundation of the agent's closed learning loop and long-term utility

See also: [[summaries/SKILL]]

See also: [[summaries/DEMO_GUIDE]]

See also: [[summaries/SESSION_SUMMARY_2025-04-28]]

See also: [[summaries/deepagents_merged_mem_notes]]

See also: [[summaries/CLAUDE]]