---
sources: [summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/README.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/DEPENDENCIES.md, summaries/style-trainer.md, summaries/report-template.md, summaries/report-generator.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/conversation-export.md, summaries/TECHNICAL_DOCS.md]
brief: Architectural pattern insulating application code from specific LLM providers via a common dispatch interface with fallback and PHI safety.
---

# LLM Provider Abstraction

LLM Provider Abstraction is an architectural pattern that insulates application code from the specifics of any single AI model provider. Instead of calling OpenAI, Anthropic, or a local inference server directly, the application routes all LLM requests through a common interface that dispatches to the appropriate backend based on configuration. This makes provider switching, fallback, and testing dramatically simpler.

## Why It Matters

LLM APIs differ in authentication schemes, request body shapes, response formats, rate limits, and pricing. Encoding these details throughout an application creates brittle coupling: changing providers (or adding a new one) requires touching many call sites. Abstraction centralises that complexity in one place.

In clinical applications this pattern is especially valuable because **data-residency requirements may force different providers for different deployments**. A hospital deployment might mandate a fully local server, while a research team is comfortable using OpenAI. The same codebase satisfies both without modification at the call site.

## Implementations

### Luria App — LocalFallbackLLMClient (TypeScript)

The Luria redesign documents (see [[summaries/redesign_20260623110817]] and [[summaries/redesign_20260623110910]]) together describe the most fully-realized clinical implementation of this pattern. Luria's `LocalFallbackLLMClient` in `src/services/llmClient.ts` iterates a `DEFAULT_PROVIDER_ORDER` array, trying each provider in sequence until one succeeds. The `pickProvider()` function selects the first eligible provider, and `restrictToPreferredProviders` acts as a local-only gate.

**Fallback chain (priority order):**

| Priority | Provider | Mode | PHI-safe | Notes |
|---|---|---|---|---|
| 1 | **oMLX** | Local | ✓ | OpenAI-compatible API via `queryOpenAICompatible` |
| 2 | **vMLX** | Local | ✓ | Local Responses API via `queryVMLX` |
| 3 | **Ollama** | Local | ✓ | Native API via `queryOllamaNative` |
| 4 | **Cloud** | Remote | ✗ | Only when `restrictToPreferredProviders=false` |

When `restrictToPreferredProviders: true` (the default), inference is pinned to oMLX / vMLX / Ollama. Cloud is reachable only when a request carries no PHI — enforced at the dispatch layer, not the call site.

All providers are called through a uniform `.generate(request)` interface where `LLMGenerateRequest` carries `messages`, `temperature`, `maxTokens`, and the `restrictToPreferredProviders` flag. An `llmAbortContext` using `AsyncLocalStorage` threads an abort signal through `generate()` so a report job can be cancelled mid-fallback.

A separate **PHI guard** (`redactPhi()`) sits upstream of the LLM client in the orchestration layer, ensuring sensitive data is redacted before any provider call — including local ones. This mirrors the de-identification-before-dispatch design seen in the Python pipeline implementations.

The abstraction surfaces at three UI entry points in Luria's architecture:
- `IntakeDossier.tsx` → `handleCommitToDossier()` → `redactPhi()` → `llmClient.generate()` (local-only)
- `ReportJobStatus` → `POST /api/pipeline/orchestrate-report` → `runReportJob()` → `agentRunner.ts` → `generate()`
- `ConsoleChat` (prompt template lab) → direct `generate()` calls

Section agents (`nseCodSummary`, `ROCFT`, report-section agents) receive an `AgentContext` of raw clinical scores and produce structured report sections — all without knowing which provider handled the inference. The Console & Synthesis experience, where a clinician argues with Luria about a case and receives numbered source citations, is entirely provider-agnostic at the application layer.

The agent pipeline is structured into three layers (as documented in [[summaries/redesign_20260623110910]]):

- **UI / Entry**: `IntakeDossier.tsx`, `ReportJobStatus`, `ConsoleChat`
- **Orchestration**: `reportJobs.ts` (createReportJob / runReportJob), `redactPhi()` PHI guard, `llmAbortContext`
- **Agents**: `agentRunner.ts` → section agents → `LocalFallbackLLMClient` → `pickProvider()` → fallback chain

Data flow for a report job:
1. `ReportJobStatus` → `POST /api/pipeline/orchestrate-report`
2. `createReportJob()` → `runReportJob()` → `run(signal | runner)`
3. `agentRunner.ts` → section agents with `AgentContext` (raw clinical data + patient scores)
4. `generate(request)` → `LocalFallbackLLMClient` → `llmAbortContext`
5. Structured report section output → fallback chain resolution

### Neuropsychological Report RAG Pipeline (Python / LangChain)

The [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] documents a production-grade implementation in `src/llm.py` that powers the recommendation generation workflow for de-identified neuropsychological reports. The pipeline supports three providers via **dynamic `importlib` loading** — a more explicit lazy-import mechanism than relative import deferral:

| Provider | Mode | Notes |
|---|---|---|
| **Anthropic Claude** | Cloud | Primary production LLM; subject to institutional data policies |
| **OpenAI GPT** | Cloud | Cloud alternative; usage tracked and billed |
| **Ollama** | Local | No data leaves the machine; HIPAA-compatible |

The abstraction is implemented through `get_llm(provider)` in `src/llm.py:85`:

```python
# Dynamic provider import — no SDK required unless that provider is selected
mod = importlib.import_module(provider_module)   # llm.py:85
cls = getattr(mod, class_name)                   # llm.py:86
llm = cls(**params)                              # llm.py:101
```

The calling function `generate_recommendations()` at `src/llm.py:124` is entirely provider-agnostic:

```python
llm = get_llm(provider)          # resolves to Claude, GPT, or Ollama
response = llm.invoke(messages)  # identical call regardless of backend
return response.content
```

Prompts are normalised into a two-message structure before dispatch:
- `SystemMessage` — clinical guidelines loaded from markdown + patient context (age, diagnoses)
- `HumanMessage` — patient description query + formatted retrieved examples + generation instructions

Retrieved recommendation chunks are formatted with metadata (diagnoses, age group, context, subsection headers) as numbered examples before being injected into the human message via `_format_chunks()`.

### Autism RAG System (Python / LangChain)

The [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] illustrates a closely related multi-provider abstraction using the same LangChain-based approach. The clinical recommendation pipeline supports the same three providers (Anthropic, OpenAI, Ollama) through an equivalent `get_llm(provider)` factory pattern.

Notably, the Autism RAG system also uses **FLAN-T5** (`google/flan-t5-base`) for a separate research Q&A pipeline, instantiated as a Hugging Face `text2text-generation` pipeline rather than through the LangChain provider abstraction. This illustrates a practical boundary: the abstraction covers cloud and local chat LLMs, while small task-specific models are loaded directly.

### PAI RAG System (R / ragnar)

The [[summaries/conversation-export]] document illustrates this pattern in R using the `ragnar` package, `httr2` for HTTP, and Ollama for local embeddings. Three providers are supported:

| Provider | Mode | Default Model | Notes |
|---|---|---|---|
| **Ollama** | Local | `llama3.2` | No data leaves the machine; HIPAA-compatible |
| **OpenAI** | Cloud | `gpt-4o-mini` | Fastest cloud option; usage tracked and billed |
| **Anthropic Claude** | Cloud | `claude-3-5-sonnet-20241022` | Cloud; subject to institutional data policies |

The abstraction layer is implemented through a set of R functions:

```r
format_context_for_llm()  # Prepare retrieved chunks as numbered sources
build_pai_prompt()        # Construct system + user message pair
call_ollama()             # Provider-specific handler: local Ollama
call_openai()             # Provider-specific handler: OpenAI REST API
call_anthropic()          # Provider-specific handler: Anthropic REST API
call_llm()                # Master router dispatching by provider argument
ask_pai_expert()          # Complete end-to-end RAG + LLM pipeline
```

`call_llm()` inspects the `provider` argument and dispatches to the appropriate handler, so callers such as `ask_pai_expert()` never contain provider-specific logic. Default models are set inside `call_llm()` when `model = NULL`:

```r
if (is.null(model)) {
  model <- switch(
    provider,
    "ollama"    = "llama3.2",
    "openai"    = "gpt-4o-mini",
    "anthropic" = "claude-3-5-sonnet-20241022"
  )
}
```

### OMLX Embedding Client (Python / stdlib)

The [[summaries/embedding-client]] document shows a leaner implementation of the same concept in Python, inside `soul/neuro_report_style_agent.py`. Rather than supporting multiple cloud providers, it abstracts over a local omlx server that exposes an [[concepts/openai-compatible-api]], with a fallback interface reserved for future extension.

Two primary operations are exposed through `*_with_fallback` entry points:

| Entry Point | Underlying Call | Endpoint |
|---|---|---|
| `embed_with_fallback(args, text)` | `omlx_embed(base_url, model, text)` | `POST /embeddings` |
| `generate_with_fallback(args, prompt)` | `omlx_generate(base_url, model, prompt)` | `POST /chat/completions` |

A key implementation detail is the use of **Python stdlib `urllib` exclusively** — no third-party HTTP libraries — keeping the dependency footprint minimal:

```python
def _post_json(url: str, payload: dict) -> dict:
    data = json.dumps(payload).encode("utf-8")
    req = request.Request(
        url,
        data=data,
        headers={"Content-Type": "application/json"},
        method="POST",
    )
    with request.urlopen(req, timeout=3000) as resp:
        return json.loads(resp.read().decode("utf-8"))
```

The 3000-second timeout (50 minutes) reflects the reality of [[concepts/local-llm-inference]] workloads where large model inference can be slow.

## Prompt Construction

A key part of the abstraction is normalising the prompt structure across providers. Across implementations, a two-part message structure (system + user/human) is universal:

- **Neuropsychological Report RAG (Python/LangChain):** `SystemMessage` carries clinical guidelines and patient context (age group, diagnoses extracted via DSM-5/ICD-10 parsing); `HumanMessage` carries the query, formatted retrieved recommendation chunks with subsection metadata, and generation instructions. `_format_chunks()` renders each chunk as a numbered example with diagnoses, age, context, and header.
- **Autism RAG (Python/LangChain):** Identical architecture — `SystemMessage` with guidelines and patient context; `HumanMessage` with query, chunks, and instructions. LangChain serialises this to each provider's wire format automatically.
- **PAI RAG (R):** `build_pai_prompt()` accepts a `system_role` argument selecting a pre-written system message (`"expert neuropsychologist"`, `"clinical supervisor"`, or `"researcher"`). The resulting `list(system = ..., user = ...)` is translated by each provider handler.
- **OMLX client (Python):** The prompt is passed directly as a user message in the `messages` array, with `stream: false` and a configurable `temperature` (default `0.2`).
- **Luria ConsoleChat (TypeScript):** The clinician's question and the assembled evidence rail (cited test scores and source documents) are passed as a structured message array to `LocalFallbackLLMClient.generate()`. The provider handles wire-format translation; the Console UI receives a response with numbered citations regardless of which backend fired. In the Console, Luria's response includes source citations (e.g., Conners-4 T72, NEPSY-II SS76, Clinical Interview) rendered uniformly regardless of whether oMLX, vMLX, Ollama, or Cloud processed the request.

## Key Design Decisions

- **Ordered fallback chain**: Luria's `DEFAULT_PROVIDER_ORDER` makes the fallback sequence explicit and configurable — the system degrades gracefully from preferred local providers to cloud rather than failing outright. See [[concepts/fallback-strategy]].
- **Local-only gate**: `restrictToPreferredProviders: true` in Luria provides a hard PHI safety boundary at the dispatch layer, not scattered across call sites.
- **Abort propagation**: Luria's `llmAbortContext` (`AsyncLocalStorage`) threads cancellation signals through multi-step fallback chains — critical for long-running report generation jobs.
- **Structured prompts**: All providers receive a normalised system + user message structure, regardless of backend wire format differences.
- **Lazy / dynamic imports**: Both the Neuropsychological Report RAG and Autism RAG systems use lazy loading (via `importlib` or deferred imports) of provider-specific LangChain classes, avoiding import errors when a provider's SDK is not installed.
- **Error handling**: Each provider call surfaces errors as `RuntimeError` (Python) or returns `NULL` via `tryCatch` (R), enabling graceful degradation.
- **Metadata passthrough**: The R implementation returns the full response object including provider name, model, retrieved sources, and prompt, supporting debugging and audit trails.
- **Provider selection by configuration**: Switching providers is a single argument or flag change, not a code change.
- **API key management**: Cloud provider keys are read from environment variables by default, with explicit override parameters.
- **Zero external HTTP dependencies** (Python OMLX client): Using only stdlib `urllib` eliminates a category of dependency conflicts.
- **Chunk metadata in prompts**: The neuropsychological pipeline formats retrieved chunks with their diagnoses, age group, context, and subsection headers — ensuring the LLM receives clinically structured context regardless of which provider processes it.

## Relationship to Privacy and Security

Provider abstraction directly enables privacy-first software design and [[concepts/phi-data-handling]] compliance. By making [[concepts/local-llm-inference]] a first-class provider option — whether through Ollama, a local omlx server, or Luria's vMLX/oMLX stack — the abstraction layer allows sensitive clinical data to remain on-premises while the same codebase can leverage cloud APIs for non-sensitive workloads.

Critically, both Luria and the neuropsychological report pipeline apply PHI redaction/de-identification before any data reaches the LLM provider. In Luria, `redactPhi()` sits in the orchestration layer upstream of `LocalFallbackLLMClient`; in the Python pipelines, de-identification runs before chunks are formatted into prompts. This means provider switching does not alter the de-identification guarantees — the abstraction operates downstream of privacy protections.

The PAI RAG system explicitly recommends Ollama for protected health information. The OMLX client is local-only by design. All implementations share the same principle: there is no code penalty for choosing the privacy-preserving option.

## Relationship to Other Patterns

- [[concepts/retrieval-augmented-generation]] — the broader pipeline within which provider abstraction operates
- [[concepts/recommendation-rag-pipeline]] — the clinical recommendation workflow built on top of this abstraction
- [[concepts/local-llm-inference]] — the on-premises provider option enabled by this pattern
- [[concepts/openai-compatible-api]] — a common interface standard that simplifies multi-provider support
- [[concepts/hybrid-search-retrieval]] — the retrieval layer that feeds context into the abstracted LLM call
- [[concepts/fallback-strategy]] — error handling and graceful degradation when a provider is unavailable
- [[concepts/neuropsychological-assessment-pipeline]] — the clinical application context driving these design choices
- [[concepts/pai-knowledge-base]] — the specific knowledge store queried before the LLM call
- [[concepts/pai-assessment]] — the clinical instrument whose documents populate the knowledge base
- [[concepts/faiss-vector-index]] — the vector store used in the RAG systems alongside the LLM abstraction
- [[concepts/phi-deidentification-pipeline]] — de-identification applied before data reaches any LLM provider
- [[concepts/icd10-diagnosis-extraction]] — the diagnosis parsing layer that populates chunk metadata used in prompt construction
- [[concepts/dsm5-diagnosis-normalization]] — normalization of diagnosis names injected into LLM context
- [[concepts/luria-neuropsych-pipeline]] — the broader Luria pipeline within which the TypeScript implementation operates
- [[concepts/multi-agent-orchestration]] — section agents in Luria that call through the abstraction layer
- [[concepts/clinical-ai-reasoning]] — the Console & Synthesis experience enabled by provider-agnostic inference
- [[concepts/omlx-server]] — the primary local inference backend in Luria's fallback chain
- [[concepts/clinical-ai-copilot]] — the Luria Console experience that delivers source-cited clinical reasoning over the abstracted provider layer
- [[concepts/agent-pipeline-state-management]] — `llmAbortContext` and `AsyncLocalStorage` abort propagation through the fallback chain
- [[concepts/luria-overview]] — the broader Luria application context

## Scalability Considerations

As the number of supported providers grows, the abstraction layer may itself become complex. Common mitigation strategies include:

1. **Adapter objects** — one class or environment per provider, each implementing a common interface
2. **Configuration-driven dispatch** — a lookup table mapping provider names to handler functions (Luria's `DEFAULT_PROVIDER_ORDER` array)
3. **Middleware hooks** — pre/post processing (logging, redaction, caching) applied uniformly before dispatching

The PAI RAG system currently uses a lightweight functional approach sufficient for three providers. Both RAG systems delegate wire-format complexity to LangChain, which effectively acts as an adapter layer. The OMLX client uses an even lighter pattern — two entry points wrapping a single backend — but names those entry points with `_with_fallback` to signal the intended extension point.

Luria's `LocalFallbackLLMClient` represents the most mature implementation: an ordered array of providers, a `pickProvider()` selector, a boolean gate for PHI-safety, and `AsyncLocalStorage`-threaded abort signals — all composable without modifying call sites.

The `importlib`-based dynamic dispatch in the neuropsychological report pipeline represents a middle ground: it avoids a formal adapter class hierarchy while still keeping provider-specific code isolated and load-time dependencies optional.

## Troubleshooting Notes

Common failure modes across implementations include:

- **Ollama 404**: The model name does not match a pulled model. Verify with `ollama list` and pull with `ollama pull <model>`.
- **OMLX / vMLX connection refused**: The local server is not running. Start it before invoking the client.
- **Cloud blocked by PHI gate**: `restrictToPreferredProviders: true` prevents cloud fallback. Intended behaviour for PHI-containing requests; verify the flag is set correctly for non-PHI workloads.
- **OpenAI authentication**: Ensure `OPENAI_API_KEY` is set; the function raises a descriptive error rather than sending an empty key.
- **Anthropic version header**: The `anthropic-version` header is required; omitting it returns a 400 error.
- **Generation timeout**: The Python OMLX client uses a 3000-second timeout; local model inference on large prompts can be slow by design.
- **Missing LangChain provider SDK**: Both RAG systems use lazy/dynamic imports; if a provider's package (e.g., `langchain-anthropic`) is not installed, `get_llm()` will raise an `ImportError` only when that provider is selected.
- **`importlib` module path errors**: The neuropsychological report pipeline resolves provider modules dynamically; incorrect module path strings will raise `ModuleNotFoundError` rather than a descriptive provider error.
- **Mid-fallback cancellation**: In Luria, if an abort signal fires while the client is mid-fallback (e.g., oMLX failed and vMLX is in-flight), `llmAbortContext` ensures the signal propagates correctly through `AsyncLocalStorage`.

All handlers propagate errors in a consistent, catchable form (`RuntimeError` in Python, `NULL` via `tryCatch` in R, thrown errors in TypeScript), allowing calling code to detect failure and potentially retry with a different provider.

See also: [[summaries/redesign_20260623110817]], [[summaries/redesign_20260623110910]], [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]], [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]], [[summaries/0009-soul-local-llm-inference-with-omlx]], [[summaries/embedding-client]], [[summaries/LLM_INTEGRATION]], [[summaries/LLM_AGENT_MAP]], [[summaries/2026-04-26-cingulate-agent-team-design]], [[summaries/CLAUDE]], [[summaries/README]], [[summaries/DEPENDENCIES]], [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]