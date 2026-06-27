---
sources: [summaries/SKILL.md, summaries/cognition.instructions.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/brand-yml-integration.md, summaries/brand-and-skills.md]
brief: Reusable, self-contained modules encoding decision logic, memory protocols, and retrieval guidance for AI agents.
---

# Skills Modules

Skills modules are structured, reusable knowledge units designed to guide AI assistants and developers through domain-specific tasks. Each module encapsulates decision logic, reference documentation, and integration patterns for a well-defined area of work. Skills can also encode **memory protocols** — instructing agents to proactively save decisions, procedures, and lessons to persistent storage, and to proactively *retrieve* relevant prior knowledge before beginning a task.

## Structure

Skills live in a `skills/` directory. Each skill is self-contained with either a `SKILL.md` or `README.md` that provides:

- **Decision trees** — step-by-step branching logic for common task flows
- **Reference documentation** — curated specs, API docs, and usage examples
- **Integration patterns** — how the skill connects to surrounding tools and workflows
- **Sub-skills** — nested modules for more granular concerns
- **Behavioral directives** — rules telling the agent when and how to act (e.g., save proactively, search before starting, check before creating duplicates)

This pattern makes skill knowledge portable and reusable across sessions, agents, and team members — essentially externalizing expert judgment into structured files.

## Known Skills

See [[summaries/brand-and-skills]] for the full inventory of brand and development skills, and [[summaries/SKILL]] for the `search-memory` and `distill-memory` skills.

### Development & Brand Skills

| Skill | Purpose | Sub-skills |
|---|---|---|
| `brand-yml` | Create, modify, troubleshoot `_brand.yml` files | — |
| `quarto` | Quarto authoring and alt-text | `quarto-alt-text`, `quarto-authoring` |
| `posit-dev` | Posit development patterns | `critical-code-reviewer`, `describe-design` |
| `shiny` | Shiny app development with bslib theming | `shiny-bslib`, `shiny-bslib-theming` |
| `tidyverse` | Tidyverse R coding patterns | — |

### Memory & Knowledge Skills

| Skill | Purpose |
|---|---|
| `distill-memory` | Proactively capture decisions, procedures, and lessons to persistent memory |
| `search-memory` | Proactively retrieve relevant prior knowledge before and during a task |
| `check-integration` | Verify whether a native Nowledge Mem plugin is available in the current agent |

## The `search-memory` Skill

The `search-memory` skill defines a protocol for retrieving knowledge from a personal knowledge base (powered by **Nowledge Mem**) without waiting for an explicit user request. Its core directive is **search proactively** — especially at the start of continuation-heavy engineering tasks — whenever context suggests prior knowledge is relevant.

### When to Search

**Strong signals:**
- The user references previous work, a prior fix, or an earlier decision
- The task resumes a named feature, bug, refactor, incident, or subsystem
- The task type is a review, regression, release, docs-alignment, or integration-behavior question
- A debugging pattern resembles something solved earlier
- The user asks for rationale, preferences, procedures, or recurring workflow details
- The user uses implicit recall language: *"that approach"*, *"like before"*, *"the pattern we used"*

**Contextual signals:**
- Complex debugging where prior context would narrow the search space
- Architecture discussions that may intersect with past decisions
- Domain-specific conventions established in earlier sessions
- Ambiguous current results that past context would sharpen

### Retrieval Routing

1. **Durable knowledge**: `nmem --json m search`
2. **Prior conversation / session history**: `nmem --json t search`
3. **Progressive thread inspection**: `nmem --json t show <thread_id> --limit 8 --offset 0 --content-limit 1200` when a result includes `source_thread`
4. Use the smallest retrieval surface that answers the question
5. For project-scoped queries, append `--space "<space name>"`

This skill connects directly to [[concepts/proactive-retrieval]] and [[concepts/agent-memory]], and complements the `distill-memory` skill to form a complete memory lifecycle: capture on session end, retrieve at session start.

## The `distill-memory` Skill

The `distill-memory` skill defines a protocol for saving breakthrough moments and insights to a persistent knowledge base using the `nmem` CLI tool. Its core directive is **save proactively** — without waiting to be asked — whenever a conversation produces:

- Decisions with rationale (e.g., technology choices and their reasoning)
- Repeatable procedures or workflows
- Lessons from debugging, incidents, or root cause analysis
- Durable preferences or constraints
- Plans that future sessions will need to resume
- Important context that would otherwise be lost

### Add vs. Update Logic

The skill enforces memory hygiene through an explicit branching rule:

- Use `nmem --json m add` for **genuinely new** insights
- Use `nmem m update <id>` when an existing memory captures the same topic and new information **refines** it
- At the end of a substantial task, explicitly check whether one durable memory should be added or updated

### Structured Saves

Memories are saved with metadata for searchability and importance weighting:

| Field | Purpose | Examples |
|---|---|---|
| `--unit-type` | Classifies the memory | `decision`, `procedure`, `learning`, `preference`, `event` |
| `-l` | Labels for filtering | topic tags |
| `-i` | Importance score | 0.8–1.0 major decisions, 0.5–0.7 useful patterns, 0.3–0.4 minor notes |

This connects directly to [[concepts/decision-logging]] and [[concepts/knowledge-capture]] — structured saves make memories atomic, searchable, and meaningful across sessions.

## The `brand-yml` Skill in Detail

The `brand-yml` skill is the most elaborated development example. Its decision tree follows this flow:

```
Creating → Shiny R → Shiny Python → Quarto → Modifying → Troubleshooting
```

It references a `references/` directory containing:
- The `_brand.yml` spec
- Shiny R integration docs
- Shiny Python integration docs
- Quarto integration docs
- `brand-yml-in-r` usage guide (see [[summaries/brand-yml-in-r]])

The skill is also tied to a `brand-yml.prompt` file, making it directly usable as an AI prompt template.

## Relationship to Other Patterns

Skills modules share design DNA with several related concepts:

- [[concepts/ide-ai-assistant-configuration]] — Skills are often loaded as context into AI coding assistants
- [[concepts/documentation-as-code]] — Skills are documentation artifacts version-controlled alongside code
- [[concepts/yaml-configuration]] — Skills frequently reference or produce YAML-based configuration (e.g., `_brand.yml`)
- [[concepts/knowledge-continuity]] — Skills solve the problem of knowledge lost between sessions or team members
- [[concepts/persistent-memory]] — The `distill-memory` skill integrates directly with persistent memory stores
- [[concepts/plain-text-documentation]] — Structuring knowledge as files makes it composable, auditable, and improvable
- [[concepts/knowledge-capture]] — Skills encode when and how to capture insights, not just what to do
- [[concepts/proactive-retrieval]] — The `search-memory` skill encodes when and how to retrieve prior knowledge
- [[concepts/agent-memory]] — Skills operate on top of agent memory infrastructure (Nowledge Mem / `nmem` CLI)

## Why This Pattern Matters

Skills modules solve two related problems:

1. **Knowledge continuity** — ensuring that hard-won integration knowledge (e.g., exactly how `bs_theme(brand = TRUE)` connects to `_brand.yml`) is not lost between sessions or team members.
2. **Behavioral consistency** — encoding agent decision rules (e.g., search before you start, save before you forget, check for duplicates before adding) as durable, inspectable files rather than ad-hoc prompts.

Together, `search-memory` and `distill-memory` form a complete memory lifecycle: retrieve relevant context at the start of a task, capture new knowledge at its end. By structuring this as skills — plain files rather than ephemeral instructions — the pattern is composable, auditable, and improvable over time, applicable to both human developers and AI agents operating across multiple sessions.

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/cognition.instructions]]

See also: [[summaries/SKILL]]