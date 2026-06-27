---
doc_type: short
full_text: sources/0001‑choose‑local‑llm.md
---

# Summary: ADR 0001 – Choose a Local LLM Backend

**Date:** 2026-04-23 | **Status:** ✅ Adopted | **Approver:** Jane Doe

## Overview

This Architecture Decision Record (ADR) documents the selection of a local LLM inference stack to allow the project to operate fully offline and avoid external API costs.

## Decision

- **Primary backend:** OMLX — an OpenAI-compatible local inference server exposed at `127.0.0.1:8000`.
- **Fallback backend:** Ollama — a lightweight, easy-to-install inference server for local workstations.

## Rationale

- OMLX provides a single-endpoint interface compatible with the **OpenAI SDK**, minimising integration friction.
- Ollama is simple to install and ensures the test suite can run without a live OMLX instance.
- The dual-backend strategy improves resilience across different environments.

## Consequences & Implementation Notes

- `omlx` must be running on `127.0.0.1:8000` in production environments.
- Two helper functions must be shipped with the codebase:
  - `embed_with_fallback()` — attempts OMLX first, then Ollama for embeddings.
  - `generate_with_fallback()` — attempts OMLX first, then Ollama for text generation.
- All tests must mock both backends to maintain independence from live services.

## Key Concepts

- [[concepts/local-llm-inference]] — running language models entirely on local hardware.
- [[concepts/openai-compatible-api]] — standardised API surface enabling drop-in backend swaps.
- [[concepts/fallback-strategy]] — graceful degradation when the primary service is unavailable.
- [[concepts/offline-ai]] — designing AI-powered systems with no external network dependency.

## Related Concepts
- [[concepts/privacy-first-software]]
