---
sources: [summaries/LLM Benchmark Comparison.md, summaries/top_level.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/customization.md, summaries/soul-style-agent.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/README.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/index.md, summaries/0001‑choose‑local‑llm.md]
brief: An inference server interface mirroring OpenAI's REST API, enabling drop-in local or third-party LLM backends.
---

# OpenAI-Compatible API

An **OpenAI-compatible API** is a local or third-party inference server that exposes the same HTTP endpoint structure and request/response schema as the official OpenAI REST API. Because the interface is identical, client code written against the OpenAI SDK can redirect requests to a different backend simply by changing the base URL — no other code changes are required.

## Why It Matters

The OpenAI SDK has become a de facto standard for interacting with large language models. By adopting its interface, alternative inference servers gain immediate compatibility with a vast ecosystem of tools, libraries, and applications. This dramatically reduces integration friction when switching between cloud and local deployments.

## Key Characteristics

- **Endpoint parity** — exposes routes such as `/v1/chat/completions`, `/v1/embeddings`, and `/v1/models` matching OpenAI's specification.
- **Drop-in substitution** — changing only the `base_url` parameter in the SDK client is sufficient to redirect traffic.
- **Vendor-agnostic client code** — application logic does not need to know whether it is talking to a cloud model or a local one.
- **Testability** — mock servers can implement the same interface, making unit tests backend-independent.
- **Stdlib-compatible** — because the interface is plain HTTP+JSON, clients can consume it with nothing more than Python's built-in `urllib`, as demonstrated in [[summaries/embedding-client]].

## Usage in This Project

As documented in [[summaries/0001‑choose‑local‑llm]], the project selects **OMLX** as its primary local inference server precisely because it exposes an OpenAI-compatible API. This allows inference to run locally at `http://127.0.0.1:8000/v1` with no external SDK dependency.

The embedding client (`soul/neuro_report_style_agent.py`) consumes two key endpoints:

| Endpoint | Purpose | Response Field |
|---|---|---|
| `POST /v1/embeddings` | Vector embedding | `data[0]["embedding"]` |
| `POST /v1/chat/completions` | Text generation | `choices[0]["message"]["content"]` |

The client deliberately uses only Python stdlib `urllib` for HTTP — zero third-party dependencies — with a 3000-second timeout to accommodate large local model inference. See [[summaries/embedding-client]] for full implementation details.

The fallback server, **Ollama**, also supports an OpenAI-compatible interface, meaning the helper functions `embed_with_fallback()` and `generate_with_fallback()` can target either backend through the same call pattern. This is detailed in [[summaries/0009-soul-local-llm-inference-with-omlx]].

## Endpoint Details

### Embeddings (`/v1/embeddings`)

```json
{
  "model": "nomicai-modernbert-embed-base-bf16",
  "input": "text to embed"
}
```

Returns a `List[float]` vector. Missing or malformed responses raise `RuntimeError`.

### Chat Completions (`/v1/chat/completions`)

```json
{
  "model": "Qwopus3.5-9B-v3-PolarQuant-MLX-4bit",
  "messages": [{"role": "user", "content": "prompt"}],
  "temperature": 0.2,
  "stream": false
}
```

Returns a plain string. Empty or missing `choices` raise `RuntimeError`.

## Relationship to Other Concepts

- [[concepts/local-llm-inference]] — OpenAI-compatible APIs are the primary integration surface for local models.
- [[concepts/omlx-server]] — the specific OpenAI-compatible server used as the primary backend in this project.
- [[concepts/fallback-strategy]] — because both primary and fallback servers share the same interface, fallback logic is simplified to a URL swap.
- [[concepts/retrieval-augmented-generation]] — embedding endpoints (`/v1/embeddings`) are commonly consumed via this interface in RAG pipelines.
- [[concepts/model-context-protocol]] — complements OpenAI-compatible APIs by providing a structured tool/context layer on top of the inference interface.
- [[concepts/llm-provider-abstraction]] — the shared interface is the mechanism that makes provider abstraction practical.
- [[concepts/mlx-framework]] — quantized MLX models are served locally via OMLX, which exposes this API.

## Examples of OpenAI-Compatible Servers

| Server | Notes |
|---|---|
| OMLX | Used as primary backend in this project |
| Ollama | Lightweight fallback; also OpenAI-compatible |
| LM Studio | Desktop GUI with compatible server mode |
| vLLM | High-throughput serving for GPU clusters |
| llama.cpp server | Minimal C++ implementation |

## Error Handling Conventions

Clients consuming this interface should handle:
- **Connection refused** — server not running
- **400/404 responses** — model not loaded or endpoint mismatch
- **Timeout** — long inference on large local models (mitigated by the 3000-second timeout in the embedding client)

All such errors are surfaced as `RuntimeError` in this project's client layer.

## Summary

The OpenAI-compatible API pattern is a pragmatic standardisation strategy: it lets teams move between cloud and local inference, or between different local backends, with minimal code changes. It supports both high-level SDK usage and zero-dependency stdlib HTTP clients. It is a cornerstone of local-first AI architectures and [[concepts/local-llm-inference]] deployments alike.

See also: [[summaries/embedding-client]] · [[summaries/0009-soul-local-llm-inference-with-omlx]] · [[summaries/0008-soul-single-file-style-agent-architecture]] · [[summaries/0001‑choose‑local‑llm]] · [[summaries/README]]

See also: [[summaries/soul-style-agent]]

See also: [[summaries/customization]]

See also: [[summaries/installation]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/top_level]]

See also: [[summaries/LLM Benchmark Comparison]]