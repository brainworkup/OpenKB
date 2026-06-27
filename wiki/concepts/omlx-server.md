---
sources: [summaries/LLM Benchmark Comparison.md, summaries/README.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/soul-style-agent.md, summaries/embedding-client.md]
brief: OMLX Server is a local OpenAI-compatible inference server for Apple Silicon workflows.
---

# OMLX Server

OMLX is a locally hosted inference server that exposes an **OpenAI-compatible REST API** for language model operations. It enables privacy-preserving, on-device LLM inference without reliance on external cloud services, making it a core component of [[concepts/local-first-architecture]] and [[concepts/local-llm-inference]] workflows.

A key practical lesson from current usage is that OMLX performance must be evaluated not only by peak model quality but by **stability under load**. In local Apple Silicon deployments, output quality and latency can degrade when multiple heavyweight models compete for memory and compute. For Luria-style clinical workflows, this makes [[concepts/local-inference-reliability]] and predictable behavior more important than occasional benchmark-best responses.

## What It Is

OMLX acts as a local drop-in replacement for the OpenAI API, serving two primary endpoints:

- `POST /v1/embeddings` — generates vector embeddings from text
- `POST /v1/chat/completions` — generates text completions from a prompt

Because it mirrors the OpenAI API contract, clients designed for [[concepts/openai-compatible-api]] patterns can communicate with OMLX with minimal modification. The Luria application uses the standard `openai` Python package pointed at the local server via `openai.OpenAI(base_url=OMLX_BASE_URL)` — **not** the OpenAI cloud service. Alternative backends such as Ollama can be substituted by overriding the base URL (e.g., `http://localhost:11434/v1`) via the `OMLX_BASE_URL` environment variable or CLI flags, demonstrating the server-agnostic nature of the [[concepts/llm-provider-abstraction]] layer.

In practice, OMLX should be understood as both an API surface and a resource management boundary. The server may expose clean OpenAI-compatible endpoints while still being sensitive to local hardware constraints such as unified memory pressure, model concurrency, and large-context inference behavior.

## Installation

OMLX is installed via the `mlx-lm` package and launched as a local server:

```bash
pip install mlx-lm
python -m mlx_lm.server
```

Server availability can be verified with:

```bash
curl http://127.0.0.1:8000/v1/models
```

## Model Configuration

The [[summaries/DEPENDENCIES]] document defines the canonical model assignments for the Luria workspace. The README for the Luria Streamlit App confirms these same models are used across the full ingest-and-RAG pipeline:

| Role | Model | Dim | Config Key |
|---|---|---|---|
| Chat/Extraction (default) | `medgemma-1.5-4b-it-bf16` | — | `OMLX_CHAT_MODEL` |
| Chat (high-capacity) | `Qwen3.6-35B-A3B-oQ4` | — | `OMLX_CHAT_MODEL` |
| Embeddings | `nomicai-modernbert-embed-base-bf16` | 768 | `OMLX_EMBEDDING_MODEL` |
| Reranking | `nomicai-modernbert-embed-base-bf16` | 768 | `OMLX_RERANK_MODEL` |
| Fallback embeddings | `sentence-transformers/all-MiniLM-L6-v2` | 384 | `VECTOR_EMBEDDING_MODEL` |

The README documents `Qwen3.6-35B-A3B-oQ4` as an example high-capacity chat model, reflecting the [[concepts/mixture-of-experts]] architecture common in modern quantized local models. The fallback embedding model (`all-MiniLM-L6-v2`, 384-dim) is served via the `sentence-transformers` Python package and used when the OMLX server is unavailable or when the application is configured for `LLM_PROVIDER=sentence-transformers`.

Earlier pipeline configurations (documented in [[summaries/0009-soul-local-llm-inference-with-omlx]]) used `nomicai-modernbert-embed-base-bf16` for embeddings and a quantized generation model such as `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit`. The current default chat model is `medgemma-1.5-4b-it-bf16`, optimized for medical and clinical extraction tasks.

Operationally, model choice should be matched to concurrency expectations. Smaller domain-specialized models such as MedGemma are often a better default for extraction and drafting when reliability matters. Larger reasoning models can be used selectively, but they are more likely to exhibit unstable latency or degraded output quality if another large process is running concurrently.

## How It Is Used

OMLX underpins multiple stages of the [[concepts/luria-neuropsych-pipeline]]:

### Streamlit App — Ingest Pipeline

The Luria Streamlit App's 4-stage LangGraph ingest pipeline calls OMLX at multiple points (see [[concepts/langgraph-agent-workflows]]):

1. **Parse stage** — Docling parses the PDF locally and redacts PHI before any network call
2. **Extract stage** — LLM structures narrative into JSON (test scores, clinical summaries) via `POST /v1/chat/completions`; Claude Sonnet is the default cloud fallback when OMLX is unavailable
3. **Index stage** — text chunks are embedded via `POST /v1/embeddings` and written to [[concepts/lancedb-vector-store]] at `data/vectors/`
4. **Report stage** — LLM synthesizes a markdown narrative report via `POST /v1/chat/completions`

### Streamlit App — RAG Pipeline

The RAG graph performs semantic search over LanceDB using embeddings from OMLX, then passes retrieved narrative chunks plus SQL results from `TestScores` to the LLM for synthesis with citations. This enables the **Ask** tab Q&A interface to answer questions using only locally ingested data.

### Streamlit App — Audio Summarization

The **Audio** tab uses OMLX for summarization after MacWhisper transcribes audio files locally. This is part of the broader [[concepts/audio-transcription-pipeline]] where both transcription and summarization remain on-device.

### Soul Style Agent

The [[summaries/embedding-client]] module (`soul/neuro_report_style_agent.py`) communicates with OMLX at `http://127.0.0.1:8000/v1` by default. It underpins the full three-stage style pipeline described in [[summaries/full-pipeline]] and [[summaries/style-training-to-report-drafting]]:

1. **Build Index (Stage 1)** — historical report chunks are embedded via `POST /v1/embeddings` and stored in a SQLite vector index.
2. **Train Style (Stage 2)** — the index is queried to synthesize a style profile JSON using `POST /v1/chat/completions`.
3. **Write Report (Stage 3)** — the style profile is injected as a system prompt and top-k retrieved chunks are prepended as context for `POST /v1/chat/completions`.

### Embedding Requests

```json
{
  "model": "nomicai-modernbert-embed-base-bf16",
  "input": "text to embed"
}
```

The server returns a response from which `data[0]["embedding"]` is extracted as a `List[float]`.

### Generation Requests

```json
{
  "model": "medgemma-1.5-4b-it-bf16",
  "messages": [{"role": "user", "content": "prompt"}],
  "temperature": 0.2,
  "stream": false
}
```

The server returns a response from which `choices[0]["message"]["content"]` is extracted.

## Default Configuration

| Parameter | Default |
|---|---|
| Base URL | `http://127.0.0.1:8000/v1` (env: `OMLX_BASE_URL`) |
| Chat Model | `medgemma-1.5-4b-it-bf16` (env: `OMLX_CHAT_MODEL`) |
| Embedding Model | `nomicai-modernbert-embed-base-bf16` (env: `OMLX_EMBEDDING_MODEL`) |
| Rerank Model | `nomicai-modernbert-embed-base-bf16` (env: `OMLX_RERANK_MODEL`) |
| Fallback Embed Model | `all-MiniLM-L6-v2` (env: `VECTOR_EMBEDDING_MODEL`) |

All parameters are overridable via CLI flags (`--omlx-url`, `--omlx-embed-model`, `--omlx-gen-model`), enabling substitution of alternative providers:

```bash
python neuro_report_style_agent.py write-report \
    --omlx-url http://localhost:11434/v1 \     # Use Ollama instead
    --omlx-embed-model nomic-embed-text \      # Different embed model
    --omlx-gen-model llama3.1 \                # Different gen model
    --top-k 10 \
    --temperature 0.3
```

The [[summaries/customization]] guide documents several recommended model substitutions, including higher-quality embedding models such as `nomic-embed-text-v1.5` and larger generation models for demanding cases.

## Cloud LLM Fallback

When `LLM_PROVIDER` is not set to `omlx`, the pipeline falls back to cloud providers:

- **Anthropic Claude Sonnet** (via `langchain-anthropic`) — the primary extraction engine in the Streamlit app; PHI is redacted by Docling before any text is sent to Anthropic
- **OpenAI GPT** — optional secondary fallback

This fallback strategy is part of the broader [[concepts/fallback-strategy]] pattern and [[concepts/phi-data-handling]] security model. See [[concepts/llm-provider-abstraction]] for how the client layer abstracts across providers.

## Retrieval and Generation Tuning

Beyond model selection, two parameters directly shape the quality and character of OMLX-powered generation (see [[concepts/rag-chunking]] and [[concepts/retrieval-augmented-generation]]):

| Mode | `--top-k` | `--temperature` |
|---|---|---|
| High-context (complex cases) | 12 | 0.15 |
| Balanced (default) | 6 | 0.2 |
| Quick drafting (standard cases) | 4 | 0.25 |

The [[summaries/full-pipeline]] document also illustrates iterative drafting at multiple temperatures (0.1, 0.2, 0.3) to compare conservative versus creative output variants. Higher `--top-k` values pull more retrieved chunks into the generation context, useful for complex neuropsychological cases. Lower temperatures produce more conservative, style-consistent output.

Chunking parameters set during the `build-index` stage also affect what context OMLX receives. The default is **1200 characters per chunk** with **150 characters of overlap**, tunable via `--chunk-size` and `--overlap` flags.

The new benchmark comparison also suggests an additional tuning principle: generation quality is partly a systems property, not just a prompt or model property. If a response becomes less coherent during a heavy concurrent workload, the cause may be resource contention rather than prompt design alone. For this reason, tuning should consider both generation parameters and machine load.

## Operational Requirements

- The OMLX server must be **started separately** before any client calls are made (`mlx_lm.server`). The client does not manage server lifecycle.
- A typical startup ensures the server is accessible at `http://127.0.0.1:8000/v1` with the required embedding and generation models loaded.
- The client uses a **3000-second (50-minute) timeout** to accommodate large model inference times on local hardware.
- Connection errors during any pipeline stage indicate the server is not running; all pipeline stages that use OMLX require an active instance.
- The `LLM_PROVIDER=omlx` environment variable enables fully offline operation — no internet required for chat or embeddings.

The README troubleshooting table notes: if `oMLX connection refused`, start `mlx_lm.server` or verify `OMLX_BASE_URL`.

A practical operating rule for Apple Silicon systems is to avoid concurrent heavyweight reasoning models when latency consistency matters. Observed degradation under concurrent large-model use is plausibly driven by:

- memory pressure
- KV cache contention
- scheduling instability
- speculative decoding interference
- thermal or resource saturation
- unified memory contention
- context allocation pressure
- concurrent prefill
- cache fragmentation

On hardware such as an M3 Max 48GB, one large reasoning model plus several small utility models may be realistic, while multiple 30B+ reasoning models at once are often unstable or slower overall. This is especially relevant during PDF ingestion, long context windows, simultaneous embeddings, or speculative decoding workloads. In other words, OMLX reliability depends not just on model size, but on what else is running at the same time.

## Relationship to MLX

OMLX is closely related to the [[concepts/mlx-framework]] ecosystem. The quantized model name convention (e.g., `PolarQuant-MLX-4bit`) indicates [[concepts/model-quantization]] via MLX, Apple's machine learning framework optimized for Apple Silicon. The `bf16` suffix on current default models (e.g., `medgemma-1.5-4b-it-bf16`) similarly reflects MLX-native quantization formats. This positions OMLX as a natural complement to MLX-based model serving on Apple hardware.

The benchmark comparison reinforces that MLX-native serving on Apple Silicon can be highly capable, but also sensitive to system-level interference. Good local model performance therefore depends on orchestration discipline as much as on the serving stack itself.

## Security & PHI

OMLX is central to the Luria security model:

- All chat and embeddings can run **without internet** via OMLX (`LLM_PROVIDER=omlx`)
- When using cloud fallback (Claude Sonnet), PHI is redacted at the Docling parse stage **before** any text leaves the machine; OMLX avoids this risk entirely by keeping all inference local
- Storage backends (SQLite + LanceDB) remain on local disk only
- Audio transcription (MacWhisper) and summarization (OMLX) both remain on-device
- See [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]] for broader security context

## Error Handling

Clients must handle the following failure modes propagated as `RuntimeError`:

- **Connection refused** — server not running; verify with `curl http://127.0.0.1:8000/v1/models`
- **404/400 responses** — requested model not loaded
- **Timeout** — inference on large models exceeded wait window
- **JSON parse errors** — generation model returned malformed output; retry with reduced `--style-examples` count
- **Quality collapse under concurrency** — responses become slower, less coherent, or less stable because another large model process is consuming memory or compute

The last category is operational rather than protocol-level, but it matters in practice. If a previously poor run improves after another large model stops, that is a diagnostic clue pointing to shared-resource interference rather than purely stochastic output variation.

## Related Concepts

- [[concepts/local-llm-inference]] — the broader pattern of running LLMs on local hardware
- [[concepts/openai-compatible-api]] — the API contract OMLX implements
- [[concepts/mlx-framework]] — the inference backend powering OMLX models
- [[concepts/model-quantization]] — applied to generation models (bf16, 4-bit quant)
- [[concepts/retrieval-augmented-generation]] — downstream use of embeddings produced via OMLX
- [[concepts/vector-search]] — embeddings from OMLX feed vector search pipelines
- [[concepts/llm-provider-abstraction]] — the fallback pattern anticipates multi-provider support
- [[concepts/sqlite-as-vector-store]] — the vector index populated by OMLX embeddings
- [[concepts/lancedb-vector-store]] — primary vector store for the RAG pipeline
- [[concepts/style-profile-extraction]] — style profiles generated using OMLX completions
- [[concepts/rag-chunking]] — chunking strategy affects what context OMLX receives
- [[concepts/text-chunking]] — chunking parameters set at index-build time shape OMLX input context
- [[concepts/fallback-strategy]] — cloud LLM fallback when OMLX is unavailable
- [[concepts/phi-data-handling]] — PHI redaction policy governing when cloud calls are permissible
- [[concepts/langgraph-agent-workflows]] — the StateGraph pipeline that calls OMLX nodes
- [[concepts/audio-transcription-pipeline]] — OMLX provides local summarization after MacWhisper transcription
- [[concepts/mixture-of-experts]] — architecture of high-capacity models like Qwen3.6-35B served via OMLX
- [[concepts/concurrent-model-serving]] — concurrent heavyweight inference can degrade OMLX quality and latency
- [[concepts/local-inference-reliability]] — consistency under load is a primary deployment concern
- [[concepts/llm-evaluation]] — local model assessment should include operational stability, not just peak output quality
- [[summaries/0009-soul-local-llm-inference-with-omlx]] — architecture decision for choosing local LLM inference
- [[summaries/embedding-client]] — the HTTP client that calls the OMLX server
- [[summaries/full-pipeline]] — end-to-end three-stage pipeline relying on OMLX for embedding and generation
- [[summaries/style-training-to-report-drafting]] — workflow relying on OMLX for both embedding and generation
- [[summaries/customization]] — guide to overriding OMLX models and tuning retrieval parameters
- [[summaries/DEPENDENCIES]] — authoritative list of model assignments and config keys
- [[summaries/soul-style-agent]] — soul agent that uses OMLX for exemplar indexing
- [[summaries/installation]] — setup instructions including OMLX server startup
- [[summaries/README]] — Luria Streamlit App README documenting OMLX integration across all four pipeline stages
- [[summaries/LLM Benchmark Comparison]] — notes on concurrency effects, answer quality, and Apple Silicon local inference behavior