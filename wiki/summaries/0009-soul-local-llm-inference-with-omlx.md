---
doc_type: short
full_text: sources/0009-soul-local-llm-inference-with-omlx.md
---

# ADR 0009: Local LLM Inference with OMLX

**Date:** 2026-04-22
**Status:** Accepted

## Overview

This Architecture Decision Record (ADR) consolidates the canonical decision to use [[concepts/local-llm-inference]] via **OMLX** for the style agent in the SOUL system. Because source text and outputs are PHI-adjacent (protected health information), all inference must remain on the clinician's local machine. See also [[concepts/architecture-decision-records]] for the broader ADR practice.

## Problem

The style agent (see [[concepts/style-profile-extraction]]) requires both:
- **Embedding** — vector representations of text
- **Text generation** — drafting/writing neuropsychological reports

Cloud APIs were ruled out due to privacy and data handling constraints (see [[concepts/phi-data-handling]]). Self-hosted [[concepts/local-first-architecture]] was selected as the solution.

## Decision

Use **OMLX** (MLX-backed, [[concepts/openai-compatible-api]]) running at `http://127.0.0.1:8000/v1`.

### Default Models

| Task | Model |
|------|-------|
| Embedding | `nomicai-modernbert-embed-base-bf16` |
| Generation | `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` |

### Key Implementation Details

- All LLM I/O is routed through two wrapper functions in `soul/neuro_report_style_agent.py`:
  - `embed_with_fallback`
  - `generate_with_fallback`
- These are the **single call-site** for inference, designed for extensibility via the [[concepts/fallback-strategy]] pattern.
- A future secondary provider (e.g., Ollama) can be added at the `*_with_fallback` boundary without changes to the wider codebase, supporting [[concepts/llm-provider-abstraction]].
- Model and endpoint overrides via CLI flags: `--omlx-url`, `--omlx-embed-model`, `--omlx-gen-model`.
- This design also reflects the [[concepts/single-file-agent-pattern]] described in [[summaries/0008-soul-single-file-style-agent-architecture]].

## Consequences

### Benefits
- **Privacy**: All inference stays on-device; no PHI leaves the machine (see [[concepts/privacy-first-software]]).
- **Cost**: No per-token spend on hosted APIs.
- **Latency**: No external network round-trips during drafting/training.
- **Control**: Model versions and behavior are fully local and user-controlled.
- **Apple Silicon fit**: [[concepts/mlx-framework]] leverages Metal GPU on Apple Silicon for faster inference.

### Trade-offs
- **Hardware dependency**: Requires Apple Silicon with sufficient RAM (~6 GB for the 9B-parameter [[concepts/model-quantization|quantized]] model).
- **Server dependency**: OMLX server must be running before `build-index`, `train-style`, or `write-report`.
- **Operational burden**: Users manage model downloads/updates and server health.
- **Fallback complexity**: Secondary backend support requires careful parity and error handling.

## Relationship to Other ADRs

This ADR consolidates two prior ADRs that described the same decision with slightly different emphasis, serving as the single canonical reference for [[concepts/local-llm-inference]] in the SOUL system. Related prior decisions are captured in [[summaries/0001‑choose‑local‑llm]] and [[summaries/0002-soul-sqlite-vector-storage]].

## References

- `soul/neuro_report_style_agent.py` — defaults, fallback wrappers, CLI flags
- [[concepts/phi-data-handling]] — data handling constraints driving this decision
- [[concepts/style-profile-extraction]] — the agent consuming these inference capabilities
- [[concepts/narrative-report-generation]] — downstream use of generated text

## Related Concepts
- [[concepts/clinical-data-privacy]]
- [[concepts/retrieval-augmented-generation]]
