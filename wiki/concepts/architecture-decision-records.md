---
sources: [summaries/MIGRATION_GUIDE.md, summaries/SKILL.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md, summaries/0004-soul-style-profile-json.md, summaries/0002-soul-sqlite-vector-storage.md, summaries/0001-voice-record-architecture-decisions.md]
brief: Structured documents capturing the why behind architectural choices, kept immutable alongside code.
---

# Architecture Decision Records (ADRs)

Architecture Decision Records (ADRs) are short, structured documents that capture the **why** behind significant technical or architectural choices. Each record is written at the moment a decision is made and remains immutable thereafter — preserving the original reasoning even as a project evolves.

## Why ADRs Matter

In long-running or multi-contributor projects, rationale decays faster than code. A future contributor (or a future AI session) looking at source files can reconstruct *what* was built but rarely *why* one path was chosen over another. ADRs close this gap by making reasoning a first-class artifact alongside code.

This is especially important in systems that integrate multiple non-obvious subsystems — see [[summaries/0001-voice-record-architecture-decisions]] for the original adoption decision in the voice project.

## Standard Template (Michael Nygard Format)

The most widely used ADR template contains four sections:

| Section | Purpose |
|---|---|
| **Status** | Current state: Proposed, Accepted, Deprecated, Superseded |
| **Context** | The forces and constraints driving the decision |
| **Decision** | The choice that was made, stated plainly |
| **Consequences** | Trade-offs, downstream effects, and what becomes easier or harder |

## Key Properties

- **Immutability** — Once accepted, an ADR is never edited. If a decision changes, a new ADR is written that references and supersedes the old one.
- **Sequential numbering** — ADRs are numbered (e.g., `0001`, `0002`) for stable references across documents and conversations.
- **Colocation with code** — Stored in a dedicated directory (typically `docs/adr/`) so they travel with the repository.
- **Lightweight** — Each ADR is short enough to write in minutes but valuable enough to read years later.

## Relationship to Knowledge Continuity

ADRs are a direct implementation of [[concepts/knowledge-continuity]]. They ensure that the reasoning behind design choices survives across:

- Contributor turnover
- Long gaps between work sessions
- AI-assisted workflows where context windows reset

## ADRs in Practice: Examples

### Voice Project (0001) — Adopting ADRs

The voice project adopted ADRs in [[summaries/0001-voice-record-architecture-decisions]] specifically because it integrates several complex, interacting subsystems:

- [[concepts/typst-typesetting]] — document typesetting
- [[concepts/quarto]] — rendering pipeline
- [[concepts/retrieval-augmented-generation]] — style agents
- [[concepts/brand-theming]] — visual and brand identity

Each of these carries non-obvious trade-offs that benefit from explicit, durable documentation.

### Soul Project (0002) — SQLite for Vector Storage

[[summaries/0002-soul-sqlite-vector-storage]] demonstrates the ADR format applied to an infrastructure trade-off: choosing [[concepts/sqlite-as-vector-store]] over dedicated vector databases (Chroma, Weaviate, Pinecone) for a [[concepts/retrieval-augmented-generation]] pipeline.

The ADR records:
- **Context**: Need to store text chunks with embeddings for semantic search, under a [[concepts/single-file-agent-pattern]] constraint
- **Decision**: SQLite with JSON-serialized embeddings
- **Rationale**: Zero dependencies, ACID compliance, portability, sufficient performance at the expected scale (<10,000 chunks)
- **Consequences**: O(n) cosine similarity scan and no approximate nearest neighbor support are accepted trade-offs for simplicity

This illustrates how ADRs make scaling limits and accepted trade-offs explicit — a future maintainer knows exactly when (>10k chunks) to reconsider the decision.

### Soul Project (0008) — Single-File Style Agent Architecture

[[summaries/0008-soul-single-file-style-agent-architecture]] records the canonical decision to implement the neuropsych report style agent as a **single Python script** at `soul/neuro_report_style_agent.py`. This ADR is notable for consolidating two prior overlapping records — itself an example of ADRs needing to be superseded when context or numbering schemes diverge.

The ADR captures:
- **Context**: A single clinician must be able to run, inspect, and relocate the agent without a heavyweight package structure or external services.
- **Decision**: One file owns the full workflow — text extraction, embedding and generation calls, SQLite-backed storage, and CLI entry points.
- **Consequences**: Strong auditability, portability, and debugging clarity; accepted trade-offs include testing complexity, lack of enforced module boundaries, and a scalability ceiling for large corpora.
- **Revisit signals**: Script entering the 600–1000 line range, exceeding five subcommands, or requiring independent imports of subcomponents.

This ADR exemplifies the [[concepts/local-first-architecture]] posture of the project and illustrates how [[concepts/single-file-agent-pattern]] decisions benefit from explicit documentation of when the pattern should be abandoned.

## ADRs and Architectural Trade-offs

A recurring theme across ADR examples is the **explicit acknowledgment of negative consequences**. Good ADRs do not just justify a choice — they document what is sacrificed. This intellectual honesty is what makes the format durable: when the stated limits are eventually hit, the ADR itself points toward the migration path.

The 0008 ADR is a strong example of this principle: it names specific thresholds (line count, subcommand count) and patterns (need for fine-grained mocking) that should trigger a revisit, giving future maintainers concrete guidance rather than vague unease.

## Related Concepts

- [[concepts/knowledge-continuity]] — the broader goal ADRs serve
- [[concepts/documentation-as-code]] — the philosophy of treating docs as versioned artifacts
- [[concepts/plain-text-documentation]] — ADRs are typically plain Markdown, fitting naturally into code repositories
- [[concepts/modular-report-architecture]] — an example domain where ADR-style decisions arise frequently
- [[concepts/single-file-agent-pattern]] — a constraint that often surfaces in ADR context sections
- [[concepts/sqlite-as-vector-store]] — a concrete decision captured in ADR 0002
- [[concepts/local-first-architecture]] — the posture that shapes many soul-project ADRs
- [[concepts/style-profile-extraction]] — domain context for the soul agent decisions

See also: [[summaries/0004-soul-style-profile-json]]

See also: [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]]

See also: [[summaries/0006-brand-yml-for-cross-platform-theming]]

See also: [[summaries/0007-style-modular-report-sections-via-quarto-includes]]

See also: [[summaries/0008-soul-single-file-style-agent-architecture]]

See also: [[summaries/0009-soul-local-llm-inference-with-omlx]]

See also: [[summaries/0010-voice-quarto-typst-reporting]]

See also: [[summaries/0007-voice-modular-report-sections-via-quarto-includes]]

See also: [[summaries/SKILL]]

See also: [[summaries/MIGRATION_GUIDE]]