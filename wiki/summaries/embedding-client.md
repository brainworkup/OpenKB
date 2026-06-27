---
doc_type: short
full_text: sources/embedding-client.md
---

# Embedding Client Component

## Overview

The Embedding Client is an HTTP client module located in `soul/neuro_report_style_agent.py` (lines 87–144). It provides embedding and text generation capabilities by communicating with a locally running [[concepts/omlx-server]] that exposes an [[concepts/openai-compatible-api]].

## Core Functions

### Embedding: `omlx_embed`

- **Endpoint**: `POST {base_url}/embeddings`
- **Default model**: `nomicai-modernbert-embed-base-bf16`
- Returns a `List[float]` extracted from `data[0]["embedding"]` in the response
- Raises `RuntimeError` if `data` or `embedding` fields are missing

### Text Generation: `omlx_generate`

- **Endpoint**: `POST {base_url}/chat/completions`
- **Default model**: `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit`
- Uses a chat-completion format with `stream: false` and configurable `temperature` (default `0.2`)
- Extracts `choices[0]["message"]["content"]` and strips whitespace
- Raises `RuntimeError` if `choices` is missing or content is empty

## Entry Points with Fallback Pattern

- `embed_with_fallback(args, text)` — currently a direct OMLX call; structured for future [[concepts/fallback-strategy]] extension
- `generate_with_fallback(args, prompt, temperature)` — same pattern for generation

## HTTP Implementation

Uses **Python stdlib `urllib`** exclusively — no third-party HTTP libraries. The internal `_post_json` helper:
- Encodes payload as UTF-8 JSON
- Sets `Content-Type: application/json`
- Has a **3000-second (50-minute) timeout** to accommodate large model inference

## Configuration

| Parameter | Default | CLI Flag |
|---|---|---|
| Base URL | `http://127.0.0.1:8000/v1` | `--omlx-url` |
| Embedding Model | `nomicai-modernbert-embed-base-bf16` | `--omlx-embed-model` |
| Generation Model | `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` | `--omlx-gen-model` |

## Error Handling

All errors surface as `RuntimeError`. Callers must handle:
- Connection refused (server not running)
- 404/400 responses (model not loaded)
- Timeout on large generation requests

## Key Design Notes

- **Zero external dependencies** for HTTP — only stdlib used
- Assumes the [[concepts/omlx-server]] is already running before any calls are made
- The fallback interface design anticipates future [[concepts/llm-provider-abstraction]] support across multiple backends
- Extremely long timeout reflects [[concepts/local-llm-inference]] workload characteristics, consistent with the [[concepts/local-first-architecture]] approach used throughout the project

## Related Concepts
- [[concepts/clinical-data-privacy]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/vector-search]]
- [[concepts/mlx-framework]]
- [[concepts/single-file-agent-pattern]]
