---
doc_type: short
full_text: sources/0008-soul-single-file-style-agent-architecture.md
---

# Summary: 0008 — Single-File Style Agent Architecture

**Date:** 2026-04-22
**Status:** Accepted

## Overview

This ADR consolidates two earlier overlapping decisions about the neuropsych report style agent in the `soul/` subsystem. It canonizes a **single-file Python script** (`soul/neuro_report_style_agent.py`) as the implementation approach, prioritizing auditability, portability, and operational simplicity over modular packaging.

## Core Decision

The entire style-agent workflow lives in one file, organized into clear logical sections:

- **Text extraction and chunking**
- **Embedding and generation calls** (to a local inference endpoint)
- **SQLite-backed storage and retrieval**
- **CLI entry points**: `build-index`, `train-style`, `write-report`

State is kept in local SQLite and JSON files. HTTP calls use Python stdlib (`urllib`) to avoid adding `requests` or `httpx` as dependencies.

## Rationale

- A single clinician must be able to run, inspect, and relocate the agent without a heavyweight package structure or external services.
- The [[concepts/local-first-architecture]] posture of the project makes SQLite and JSON sufficient at the expected scale.
- [[concepts/single-file-agent-pattern]] trades enforced module boundaries for simplicity and traceability.

## Consequences

### Benefits
- **Auditability**: Full workflow visible in one file; no import chain tracing.
- **Portability**: Easy to copy or ship; `uv sync` + one script covers the runtime surface.
- **Debugging clarity**: Retrieval, generation, and CLI orchestration co-located.
- **Dependency restraint**: No extra HTTP client library introduced.

### Tradeoffs
- **Testing**: Fine-grained mocking and isolated reuse become harder over time.
- **Organization**: Logical boundaries are conventional, not enforced — internal discipline required.
- **Scalability ceiling**: Pure-Python cosine similarity over in-memory embeddings is practical only for small corpora; larger datasets will need a true [[concepts/vector-search]] index.
- **Reuse ceiling**: Subcomponent reuse is limited until the script is modularized.

## Revisit Signals

The decision should be revisited when the script:
- Enters the **600–1000 line range** and becomes harder to reason about
- Exceeds **five subcommands**
- Requires **independent imports** of subcomponents
- Needs **more sophisticated test isolation**

## Related Concepts
- [[concepts/architecture-decision-records]]
- [[concepts/local-llm-inference]]
- [[concepts/clinical-data-privacy]]
- [[concepts/openai-compatible-api]]

- [[concepts/local-first-architecture]]
- [[concepts/single-file-agent-pattern]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/vector-search]]
- [[concepts/sqlite-as-vector-store]]
- [[concepts/style-profile-extraction]]
- [[concepts/python-project-structure]]