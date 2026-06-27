---
sources: [summaries/redesign_20260623110910.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/agent-team.md, summaries/DEPENDENCIES.md, summaries/vector-store.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/report-generator.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/TECHNICAL_DOCS.md, summaries/index.md, summaries/0001‑choose‑local‑llm.md]
brief: Resilience pattern routing AI calls through a preferred backend with automatic degradation to a secondary option.
---

# Fallback Strategy in AI Service Layers

A **fallback strategy** is a resilience pattern in which a system attempts to use a preferred service or resource first, and automatically switches to a secondary option when the primary is unavailable, too slow, or fails. In the context of AI inference layers, this pattern is essential for maintaining service continuity across different deployment environments.

## Core Idea

Rather than hard-coding a single backend, the application wraps every outbound AI call in a helper function that:

1. **Attempts** the primary backend (e.g., a locally running OpenAI-compatible server).
2. **Catches** connection errors or timeout exceptions.
3. **Retries** the same request against the secondary backend (e.g., a lightweight alternative).
4. **Propagates** the error only if both backends fail.

This keeps the calling code agnostic to which backend is actually serving the request.

## Example: OMLX → Ollama

In [[summaries/0001‑choose‑local‑llm]] and [[summaries/0009-soul-local-llm-inference-with-omlx]], the project adopts a two-tier local inference stack:

- **Primary:** OMLX, an [[concepts/openai-compatible-api]] server (MLX-backed) running at `127.0.0.1:8000`.
- **Fallback:** Ollama, a lightweight workstation-friendly inference engine.

Two helper functions encode this logic across all three pipeline stages (build-index, train-style, write-report), both living in `soul/neuro_report_style_agent.py`:

```python
embed_with_fallback()    # Embeddings: OMLX first, then Ollama
generate_with_fallback() # Generation: OMLX first, then Ollama
```

These functions are the **single call-site** for all LLM inference in the system. Centralizing inference at this boundary means a future secondary provider can be added without widening changes to the rest of the codebase. Both backends are mocked in the test suite so tests never depend on live services.

## Implementation Detail: The Embedding Client

The concrete implementation of the primary (OMLX) backend is documented in [[summaries/embedding-client]]. Key implementation characteristics that affect fallback behavior:

- **HTTP layer**: Uses Python stdlib `urllib` exclusively — no third-party HTTP dependencies — keeping the client lightweight and portable.
- **Timeout**: Set to **3000 seconds (50 minutes)** to accommodate large model inference. This unusually long timeout means connection-refused errors (server not running) will fail fast, but slow inference will not prematurely trigger fallback.
- **Error surface**: All failures are surfaced as `RuntimeError`, giving the fallback wrapper a clean, consistent signal to catch.
- **Endpoints**: Embedding calls go to `POST {base_url}/embeddings`; generation calls go to `POST {base_url}/chat/completions`.

The fallback interface functions (`embed_with_fallback`, `generate_with_fallback`) are explicitly structured to reserve the secondary-backend slot — currently implemented as direct OMLX calls — making extension to a true two-backend pattern straightforward.

## Canonical Inference Defaults

As consolidated in [[summaries/0009-soul-local-llm-inference-with-omlx]] and [[summaries/embedding-client]], the default models and endpoint are:

| Task | Model |
|------|-------|
| Embedding | `nomicai-modernbert-embed-base-bf16` |
| Generation | `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` |
| Endpoint | `http://127.0.0.1:8000/v1` |

These can be overridden at runtime via CLI flags (`--omlx-url`, `--omlx-embed-model`, `--omlx-gen-model`), making the fallback boundary also a natural configuration point.

## Fallback Scope in a RAG Pipeline

In the neuropsychological report generation system, the fallback pattern applies at multiple points in the pipeline:

- **Embedding calls** in `build-index` (chunking & embedding historical reports) use `embed_with_fallback()`.
- **Generation calls** in `train-style` (producing the style profile JSON) use `generate_with_fallback()`.
- **Both embedding and generation** in `write-report` (similarity search + report drafting) use their respective fallback helpers.
- **Context summarization** (multi-turn memory) also routes through `generate_with_fallback()` to keep the stack minimal.

This pervasive application of the pattern ensures that no single component creates a hard dependency on a running OMLX instance.

## Retry Logic Within a Single Backend

Fallback strategies are often combined with retry logic at the component level. For example, in the `build-index` component, embedding failures trigger up to **3 retries** before a chunk is skipped. Only after exhausting retries does the system consider falling back or abandoning the chunk. This layered approach — retry first, then fall back — avoids unnecessary backend switches due to transient errors.

## Why It Matters for Local AI

Fallback strategies are especially valuable in [[concepts/local-llm-inference]] and offline-first contexts, where:

- There is no cloud safety net to absorb infrastructure failures.
- Different machines in a team may have different software installed.
- CI/CD pipelines must be decoupled from long-running local services.
- Privacy requirements prevent routing data to external APIs (see [[concepts/privacy-first-software]] and [[concepts/phi-data-handling]]).
- The inference server (e.g., OMLX) must be manually started before pipeline commands can run, making graceful degradation important for developer experience.

## Design Considerations

| Concern | Guidance |
|---|---|
| **Transparency** | Log which backend was used so debugging is straightforward. |
| **Latency** | Set short connection timeouts so fallback triggers quickly (note: the OMLX client uses a long timeout for inference, not connection, phases). |
| **Parity** | Ensure both backends accept the same request schema (easier with [[concepts/openai-compatible-api]]). |
| **Testing** | Mock both backends independently; never let tests depend on live services. |
| **Ordering** | Choose the fallback order by capability, not just availability. |
| **Retry first** | Retry transient failures within the primary before falling back to the secondary. |
| **Single call-site** | Centralize fallback logic in one module to avoid scattered error handling. |
| **Minimal dependencies** | Using stdlib HTTP (as in the OMLX client) reduces the risk of dependency conflicts affecting fallback paths. |

## Relationship to Other Concepts

- [[concepts/openai-compatible-api]] — a shared API surface makes swapping backends seamless.
- [[concepts/local-llm-inference]] — fallback strategies are a primary resilience tool when running models locally.
- [[concepts/retrieval-augmented-generation]] — RAG pipelines need fallback strategies for both the retriever and the generator components.
- [[concepts/multi-agent-orchestration]] — agent frameworks benefit from backend fallbacks to avoid cascading failures.
- [[concepts/sqlite-as-vector-store]] — minimal storage backends pair naturally with fallback inference layers to keep the entire stack self-contained.
- [[concepts/style-profile-extraction]] — style profile generation relies on `generate_with_fallback()` to remain environment-agnostic.
- [[concepts/single-file-agent-pattern]] — the style agent consolidates all fallback logic in a single module, keeping the boundary clean.
- [[concepts/mlx-framework]] — OMLX is backed by MLX, making Apple Silicon the preferred hardware target for the primary backend.
- [[concepts/local-first-architecture]] — fallback strategies are a core enabler of local-first systems that must function without network dependencies.
- [[concepts/omlx-server]] — the primary backend target; fallback logic exists precisely because this server must be manually started.

## Summary

A fallback strategy is a lightweight but powerful pattern for making AI service layers robust without adding complex infrastructure. By trying the best available backend first and degrading gracefully to a simpler one, teams gain flexibility across environments while keeping code clean and testable. The concrete embedding client implementation — using only stdlib `urllib`, surfacing all errors as `RuntimeError`, and exposing named fallback entry points — demonstrates how the pattern is designed for extension from day one. When applied consistently across embedding, generation, and context summarization calls, and centralized at a single call-site boundary (as in `embed_with_fallback` / `generate_with_fallback`), the pattern ensures that an entire multi-stage pipeline remains functional regardless of which local inference server is available.

See also: [[summaries/TECHNICAL_DOCS]], [[summaries/0009-soul-local-llm-inference-with-omlx]], [[summaries/embedding-client]]

See also: [[summaries/report-generator]]

See also: [[summaries/soul-style-agent]]

See also: [[summaries/style-trainer]]

See also: [[summaries/vector-store]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/agent-team]]

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/redesign_20260623110910]]