---
sources: [summaries/autonomous-execution.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/vector-store.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/report-generator.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md]
brief: A design pattern placing an entire agent workflow in one script for auditability, portability, and low overhead.
---

# Single-File Agent Pattern

The **single-file agent pattern** is a software design approach in which an entire agent workflow — including data access, model calls, storage, and CLI orchestration — is implemented in a single script rather than distributed across a package of modules. It trades enforced architectural boundaries for auditability, portability, and low operational overhead.

## Core Idea

Instead of splitting concerns across importable modules, the single-file pattern places all logic in one well-organized script. Internal organization is achieved through **conventional sections** (constants, data classes, text processing, model calls, storage helpers, similarity search, pipeline commands, CLI parsing) rather than through the Python module system.

This is particularly suited to tools that:
- Are run by a single operator rather than a team
- Need to be inspected, copied, or relocated easily
- Have a bounded feature surface (a small number of subcommands)
- Operate on local data at modest scale

## Canonical Example

The `soul/neuro_report_style_agent.py` script implements this pattern for a neuropsychological report style agent. See [[summaries/0008-soul-single-file-style-agent-architecture]] for the full architectural rationale and [[summaries/soul-style-agent]] for an overview of the agent's design.

The script owns the complete three-stage workflow:

```
PDF/TXT/MD reports ──► build-index ──► SQLite vector store
                                              │
                                      train-style ──► JSON style profile
                                              │
                              write-report ◄──┘
```

Concretely, this means the single file contains:
- Text extraction from PDF, TXT, and MD sources
- Fixed-size overlapping chunking (default 1200 chars, 150 overlap)
- Embedding and generation calls to a local [[concepts/omlx-server]] inference endpoint
- [[concepts/sqlite-as-vector-store|SQLite]]-backed storage and cosine-similarity retrieval
- Style profile extraction producing a structured JSON document
- CLI entry points: `build-index`, `train-style`, `write-report`
- Fallback hooks (`embed_with_fallback`, `generate_with_fallback`) for swapping backends without restructuring

## Benefits

### Auditability
The full workflow is visible without tracing import chains across a package. A non-engineer (e.g., a clinician or reviewer) can open one file and understand what the tool does, including exactly how style profiles are built and how RAG context is assembled.

### Portability
`uv sync` plus one script is sufficient to understand and run the agent. This aligns with a [[concepts/local-first-architecture]] posture where the tool must move between environments without a deployment pipeline.

### Debugging Clarity
Because retrieval, generation, and CLI orchestration are co-located, failures are easier to trace. There are no cross-module import surprises — the cosine similarity function, the SQLite schema initialiser, and the prompt assembly logic are all visible in one place.

### Dependency Restraint
A single-file agent resists dependency creep. In the `soul/` implementation, Python's stdlib `urllib` is used for HTTP calls rather than adding external HTTP libraries. The runtime requires only Python ≥ 3.13 stdlib, with `PyPDF2` as the sole optional dependency.

### Fallback Hooks
The pattern pairs well with single call-site abstractions — `embed_with_fallback` and `generate_with_fallback` — that allow backend substitution without restructuring the file. This is a lightweight form of the [[concepts/llm-provider-abstraction]] pattern kept entirely internal to the script.

## Tradeoffs

### Testing
With one module, fine-grained mocking and isolated unit testing become harder over time. Tests must import the whole file even when testing a small helper.

### Organization Discipline
Module boundaries enforce separation in a package. In a single file, logical section boundaries are conventional — discipline inside the file is essential.

### Scalability Ceiling
Pure-Python cosine similarity over in-memory embeddings is practical for small corpora. Larger datasets will eventually require a proper [[concepts/vector-search]] index or a dedicated vector store.

### Reuse Ceiling
Subcomponents cannot be imported independently until the script is refactored into modules. The pattern prioritizes initial simplicity over future composability.

## Revisit Signals

The pattern should be reconsidered when:
- The script enters the **600–1000 line range** and becomes hard to reason about
- The agent exceeds **five subcommands**
- Subcomponents need to be **imported independently** by other tools
- **Test isolation** requirements grow beyond what a single-module structure supports
- The embedding corpus grows large enough that in-memory cosine search becomes a bottleneck

## Relationship to Other Patterns

- [[concepts/local-first-architecture]] — the pattern is a natural fit for local-first tools with a single operator
- [[concepts/retrieval-augmented-generation]] — the `soul/` implementation uses RAG internally, all within the single file
- [[concepts/style-profile-extraction]] — one of the pipeline stages housed in the single-file agent
- [[concepts/sqlite-as-vector-store]] — the storage backend that keeps the single-file agent self-contained
- [[concepts/python-project-structure]] — contrasts with conventional package-based Python project layouts
- [[concepts/narrative-report-generation]] — the end goal of the style agent implemented with this pattern
- [[concepts/omlx-server]] — the local inference server called by the agent for both embeddings and text generation
- [[concepts/fallback-strategy]] — the pattern of wrapping model calls in fallback-aware single call-sites

See also: [[summaries/0009-soul-local-llm-inference-with-omlx]]

See also: [[summaries/embedding-client]]

See also: [[summaries/report-generator]]

See also: [[summaries/style-trainer]]

See also: [[summaries/vector-store]]

See also: [[summaries/style-training-to-report-drafting]]

See also: [[summaries/customization]]

See also: [[summaries/full-pipeline]]

See also: [[summaries/autonomous-execution]]