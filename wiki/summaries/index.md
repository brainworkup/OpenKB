---
doc_type: short
full_text: sources/index.md
---

# llama.cpp – Project Documentation Suite

## Overview
This document provides a complete documentation package for a local LLM-based neuropsychological report generation agent (`neuro_report_soul_agent.py`). The system runs entirely offline, uses [[concepts/retrieval-augmented-generation]] (RAG) for context, and learns a "soul profile" from historical reports to maintain consistent voice and tone.

## Architecture Decision Records (ADRs)

### ADR 0001 – Local LLM Backend
- **Decision**: Use **OMLX** ([[concepts/openai-compatible-api]] on `127.0.0.1:8000`) as primary inference server, with **Ollama** as fallback.
- **Rationale**: Single-endpoint OpenAI SDK compatibility; Ollama as lightweight fallback for local workstations.
- **Key helpers**: `embed_with_fallback()` and `generate_with_fallback()` implement the OMLX-then-Ollama pattern as a [[concepts/fallback-strategy]].
- See [[concepts/local-llm-inference]]

### ADR 0002 – Embeddings Backend
- **Decision**: Use `nomicai-modernbert-embed-base-bf16` via OMLX; store float32 embeddings as JSON strings in SQLite `chunks` table.
- **Rationale**: Minimal stack — no external vector DB; cosine similarity computed in-memory.
- **Trade-off**: Memory grows linearly with chunks (~4 bytes × dim × chunks); future migration to a vector DB would need a conversion script.
- See [[concepts/sqlite-as-vector-store]]

### ADR 0003 – JSON Serialization
- **Decision**: Embeddings stored as JSON number arrays; soul profile stored as a structured JSON document.
- **Soul profile keys**: `voice`, `tone`, `structure_patterns`, `typical_phrases`, `do_rules`, `avoid_rules`.
- **Rationale**: Human-readable, universally supported, directly injectable into LLM prompts.
- See [[concepts/yaml-configuration]] for related configuration serialization patterns.

## Component Documentation

### Build-Index
- **Purpose**: Extract text from PDFs/TXT/MD, chunk, embed, and store in SQLite.
- **Chunking**: `chunk_size=1200` chars, `overlap=150`.
- **Extraction**: PDFs via `fitz` (PyMuPDF); plain text via `open()`.
- **Error handling**: Non-text PDFs skipped; embedding failures retried up to 3 times.
- See [[concepts/document-chunking]]

### Train-Soul
- **Purpose**: Generate a **soul profile** capturing voice, tone, and structural patterns from historical reports.
- **Algorithm**: Seed RAG query → retrieve top-k chunks (default k=20) → LLM call → validate and persist JSON profile.
- **Output**: `report_soul_profile.json` — reusable across sessions.
- See [[concepts/style-profile-extraction]] and [[concepts/neuropsychological-reporting]]

### Write-Report
- **Purpose**: Generate new neuropsych report sections using the soul profile and RAG context.
- **Process**: Embed user prompt → cosine similarity search (top-k=10) → assemble prompt with system message, context, and style profile → LLM generation → write output.
- **Fallback**: If no chunks found, generate a soul-only minimal report.
- See [[concepts/clinical-report-structure]] and [[concepts/modular-report-architecture]]

## Integration Workflow
Three-stage pipeline:
1. **Build Index** → SQLite DB with chunks & embeddings
2. **Train Style Profile** → JSON soul profile
3. **Write Report** → Draft report file

Automation options: Makefile `pipeline` target, `entr`/`inotifywait` for file watching, CI/CD with cached DB/profile. See [[concepts/deployment-automation]] for orchestration patterns.

## Context Engineering Progress
Goal: Enable multi-turn memory without a stateful server by generating a **context summary** after each turn and prepending it to the next prompt. See [[concepts/persistent-memory]] for related approaches.

### Current State (2026-04-23)
| Feature | Status |
|---|---|
| Prompt template for context summary | ✅ Done |
| Persistent storage of context summary | ❌ Pending |
| Automatic retrieval of prior turns | ❌ Pending |
| Integration with write-report | ❌ Pending |

### Planned Steps
1. Define Context Summary Schema (JSON)
2. Implement Context Builder (last N turns → summary)
3. Persist to `context.json`
4. Add `--context-file` CLI flag to prepend context
5. Tests with mocked LLM
6. Optional integration with `write-report`

### Challenges
- **Token limits**: Aggressive truncation/summarization required.
- **Information loss**: Summaries may drop subtle soul cues.
- **Latency**: ~1s extra per turn for context generation LLM call.

### Success Criteria
- Agent recalls user preferences across turns.
- No degradation in report soul or coherence.
- Context generation < 1.5s average.

## Key Concepts
- [[concepts/retrieval-augmented-generation]]
- [[concepts/local-llm-inference]]
- [[concepts/openai-compatible-api]]
- [[concepts/sqlite-as-vector-store]]
- [[concepts/document-chunking]]
- [[concepts/style-profile-extraction]]
- [[concepts/persistent-memory]]
- [[concepts/fallback-strategy]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/modular-report-architecture]]

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/privacy-first-software]]
- [[concepts/python-project-structure]]
