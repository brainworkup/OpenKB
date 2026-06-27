---
sources: [summaries/README_20260414001057.md, summaries/README_20260413235533.md, summaries/README_20260413235353.md, summaries/README_20260413235148.md, summaries/README_20260413235016.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/top_level.md, summaries/entry_points.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/neurocog.prompt.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/agent-team.md, summaries/installation.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/soul-style-agent.md, summaries/report-template.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/conversation-export.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/README_AS_PROCESSING.md, summaries/README.md, summaries/QUICK_REFERENCE.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/COMPLETE_STATUS.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/mlx_embeddings.md, summaries/local_models.md, summaries/index.md, summaries/0001‑choose‑local‑llm.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/template-system.md, summaries/overview.md, summaries/002-mcp-llm-integration.md, summaries/README_luria.md, summaries/deepagents_merged_mem_notes.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/A-Mac-Studio-for-Local-AI-6-Months-Later.md]
brief: Running LLMs on local hardware for privacy, control, and workflow reliability.
---

# Local LLM Inference

Local LLM inference refers to running large language model (LLM) computations on personal or on-premises hardware rather than sending requests to cloud-based APIs. This approach trades setup complexity for privacy, autonomy, and freedom from rate limits or vendor lock-in.

A central practical lesson from recent benchmark observations is that local inference should be evaluated not only by peak model quality, but by stability under real workload conditions. In deployment settings such as Luria, consistency often matters more than occasional flashes of higher intelligence. See [[concepts/llm-evaluation]], [[concepts/local-inference-reliability]], and [[summaries/LLM Benchmark Comparison]].

Local inference has become especially important in clinical environments, where the value proposition is not just lower cost or offline capability, but the ability to support privacy-preserving, end-to-end workflows. In Luria's neuropsychology systems, local models are used not only for drafting text, but for structured reasoning, score interpretation, workflow orchestration, and clinician-facing report generation within a [[concepts/neuropsychological-assessment-workflow]]. The Q4 2026 investor memo frames this as part of a broader push toward [[concepts/healthcare-workforce-automation]]: reducing clinician documentation time while preserving diagnostic quality. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]] and [[concepts/luria-overview]].

A newer startup-application framing extends this idea beyond implementation detail: local inference can also function as a strategic positioning asset. In YC application materials, Apple MLX-based local inference is presented as evidence of technical viability, privacy alignment, regulatory fit, and defensibility for a solo-founder clinical AI product. This ties local inference not just to engineering choices, but to [[concepts/application-strategy]], [[concepts/startup-application-materials]], [[concepts/startup-differentiation]], [[concepts/market-sizing]], and [[concepts/yc-partner-preferences]]. The newer YC S26 planning note broadens this framing further by explicitly linking local inference to question-by-question application tactics, coding-session preparedness, solo-founder positioning, and a neuropsychological AI go-to-market story spanning competitive landscape, TAM expansion, and healthcare compliance considerations. See [[summaries/README_20260413235016]] and [[summaries/README_20260413235353]].

## Why Run Models Locally?

- **Privacy**: Prompts and responses never leave your hardware — critical for use cases involving sensitive code, documents, patient records, or personal data. See also [[concepts/privacy-first-software]] and [[concepts/clinical-data-privacy]].
- **No rate limits**: Workloads that would exhaust API quotas run unconstrained.
- **No vendor lock-in**: Independence from pricing changes, model deprecations, or service outages.
- **Cost ceiling**: A one-time hardware investment versus ongoing per-token costs.
- **Offline capability**: No network dependency — useful in clinical or air-gapped environments.
- **Operational control**: You control backend choice, provider routing, quantization, concurrency, and cancellation behavior.
- **Reliability tuning**: You can optimize for latency consistency and avoid system states that degrade model quality under contention. See [[concepts/concurrent-model-serving]].
- **Workflow compression**: In domain-specific systems, local inference can collapse multi-hour manual workflows into minutes while keeping humans in review loops. In Luria's Q4 2026 memo, this is presented as an 89% reduction in per-case time for neuropsychological reporting, enabled by local-first and hybrid local/cloud architecture choices constrained by privacy requirements. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]].
- **Strategic signaling**: For early-stage fundraising and accelerator applications, local inference can signal technical depth, privacy awareness, and realistic deployment strategy, especially in healthcare contexts. The YC S26 application research notes explicitly treat Apple MLX-based local inference as part of the company's application narrative. See [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], and [[concepts/founder-narrative]].
- **Regulatory fit**: Local-first inference can strengthen positioning under healthcare privacy and compliance expectations by reducing data egress and clarifying architectural boundaries. This is especially relevant when discussing HIPAA, FDA-adjacent software risk, APA-aligned professional context, and broader [[concepts/healthcare-ai-regulation]] concerns. See [[summaries/README_20260413235016]] and [[summaries/README_20260413235353]].
- **Solo-founder leverage**: In founder-facing materials, local inference can support the claim that a single technical founder can build and operate a serious privacy-sensitive workflow without a large infra team. This makes local deployment part of execution credibility, not just product architecture. See [[concepts/founder-track-record]] and [[summaries/README_20260413235353]].

## Key Use Cases

Local inference is especially compelling when data sensitivity is paramount. Concrete examples from deployed systems illustrate this:

1. **Clinical neuropsychological reporting** ([[summaries/002-mcp-llm-integration]], [[summaries/mcp-integration]]): The system uses Ollama as the local LLM backend (endpoint: `http://localhost:11434/v1`, default model: `ollama/llama3.1`) behind a [[concepts/model-context-protocol]] interface to extract structured data from psychological test PDFs (e.g., WISC-V, WAIS-IV) and generate clinical interpretations. Patient data never leaves the local machine — no external API calls are made.

2. **Offline-first application backends** ([[summaries/0001‑choose‑local‑llm]], [[summaries/index]]): Systems that must operate completely offline and avoid external API costs adopt a dual-backend strategy — a primary [[concepts/openai-compatible-api]]-compatible server (such as OMLX) with a fallback to Ollama — exposing a unified `embed_with_fallback()` / `generate_with_fallback()` interface so application code remains backend-agnostic.

3. **Neuropsychological report generation with style profiles** ([[summaries/0009-soul-local-llm-inference-with-omlx]], [[summaries/0008-soul-single-file-style-agent-architecture]], [[summaries/index]]): The `neuro_report_soul_agent.py` system uses local LLM inference for three distinct tasks — embedding document chunks, generating a style/soul profile from historical reports, and writing new report sections grounded in retrieved context. ADR 0009 consolidates the canonical decision: OMLX serves both the embedding model (`nomicai-modernbert-embed-base-bf16`) and generation model (`Qwopus3.5-9B-v3-PolarQuant-MLX-4bit`) at `http://127.0.0.1:8000/v1`.

4. **PAI RAG system for clinical neuropsychology** ([[summaries/conversation-export]], [[summaries/TECHNICAL_DOCS]]): A production-ready retrieval-augmented generation system built for the Personality Assessment Inventory (PAI) uses `nomic-embed-text` via Ollama to produce 768-dimensional embeddings across 2,546 clinical text chunks. See [[concepts/pai-knowledge-base]] and [[concepts/pai-assessment]].

5. **Neuropsych PDF RAG pipeline (Luria Streamlit App)** ([[summaries/README_luria]]): A HIPAA-conscious desktop application for solo clinicians ingesting neuropsychological PDF reports. The pipeline uses a two-tier local storage approach — SQLite (`data/neuropsych.db`) for structured test scores and LanceDB (`data/vectors/`) for semantic retrieval — with OMLX as the primary local inference server and optional fallback to Ollama. See [[concepts/luria-neuropsych-pipeline]] and [[concepts/lancedb-vector-store]].

6. **cingulate R package for neuropsychological assessment reporting** ([[summaries/README]]): The **cingulate** R package integrates local LLM inference via Ollama with specialized model routing for clinical report generation. It uses MedGemma 4B for domain-level summaries and Luria/Qwen 27B for cross-domain synthesis, configured via YAML with task-specific model selection. See [[concepts/neuropsychological-assessment-automation]] and [[concepts/r6-class-architecture]].

7. **Luria.app redesign — PHI-safe provider fallback chain** ([[summaries/redesign_20260623110817]], [[summaries/redesign_20260623110910]]): The redesigned Luria.app formalizes a four-tier LLM provider fallback chain in its agent pipeline (`src/services/llmClient.ts`). All patient-facing inference is gated behind a `restrictToPreferredProviders: true` flag that restricts requests to local providers. The chain in priority order:
   - **oMLX (#1)**: local OpenAI-compatible API — PHI-safe
   - **vMLX (#2)**: local Responses API — PHI-safe
   - **Ollama (#3)**: local native API — PHI-safe
   - **Cloud (#4)**: remote fallback — blocked for PHI

   PHI protection is enforced at the orchestration layer by `redactPhi()`, which runs before any text reaches the LLM client. An `llmAbortContext` using `AsyncLocalStorage` threads an abort signal through `generate()` so jobs can be cancelled mid-fallback. The `LocalFallbackLLMClient.generate()` method iterates the provider order at a single call-site boundary.

   The redesign document (Page 9) reveals the full three-tab architecture behind this pipeline:
   - **Tab 1 (Connectivity Map)**: UI layer (`IntakeDossier.tsx`, `ReportJobStatus`, `ConsoleChat`) → orchestration (`reportJobs.ts` with `createReportJob()` / `runReportJob()`) → agent layer (`agentRunner.ts` → section agents: `nseCodSummary`, `ROCFT`, report-section agents) → `LocalFallbackLLMClient` with `pickProvider()`.
   - **Tab 2 (Fallback Chain)**: `DEFAULT_PROVIDER_ORDER` iterates oMLX → vMLX → Ollama → Cloud (PHI-blocked). The `restrictToPreferredProviders: true` local-only gate is enforced in `LLMGenerateRequest` (`src/services/llmClient.ts#39-58`).
   - **Tab 3 (Data Flow)**: Three distinct flows — Intake → encrypted SQLite → `redactPhi()` → local LLM → clinical summary; UI → report job POST → `runReportJob()` → `agentRunner.ts` → fallback chain; Agent context (raw scores) → section agent → `generate()` → structured report section.

   The active case-file metadata (`oMLX Primary · M3 Max · v0.4.4`) confirms Apple Silicon as the expected deployment platform. See [[concepts/luria-overview]] and [[concepts/phi-data-handling]].

8. **Production-ready neuropsychology workflow automation** ([[summaries/Luria_AI_Q4_Investor_Memo_2026]]): Luria's Q4 2026 investor memo describes a local-first, privacy-preserving clinical platform that moved from prototype toward production readiness. The platform uses local and cloud models in a quality/cost-optimized multi-model architecture, but emphasizes local processing for PHI-sensitive workflows. The memo attributes reported gains — 47 active cases processed, 94% clinician-validated accuracy, and 89% time savings per case — to integrating local inference into the full chain of intake, scoring, interpretation, quality checks, and report generation. This use case highlights local inference as an enabling layer for [[concepts/clinical-ai-copilot]], [[concepts/clinical-ai-reasoning]], and [[concepts/neuropsychological-reporting]].

9. **Application and fundraising positioning for healthcare AI** ([[summaries/README_20260413235016]], [[summaries/README_20260413235353]], [[summaries/Apply-to-Y-Combinator]]): Local inference is used not only as runtime infrastructure but as a claim about product viability. The YC S26 application notes frame Apple MLX-based local LLM deployment as support for solo-founder execution, technical feasibility, privacy-sensitive healthcare workflows, coding-agent readiness, and non-overlap with competitors. In this use case, local inference becomes part of [[concepts/application-preparation]], [[concepts/application-strategy]], [[concepts/startup-fundraising]], and [[concepts/founder-evaluation]].

10. **IDE-integrated AI assistants**: Tools like the Positron Databot (Positron 2025.08.0+) run AI assistance within the IDE itself, connecting to local Ollama models or cloud APIs. See [[concepts/ide-ai-assistant-configuration]] and [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]] for setup details.

11. **High-capacity personal AI workstations**: Enthusiasts and researchers running large open-weight models locally for agentic workflows, coding assistance, and complex reasoning tasks.

12. **Reliability-sensitive explanatory writing** ([[summaries/LLM Benchmark Comparison]]): Local inference is increasingly used for tasks where the quality bar includes semantic cohesion, readable abstraction depth, and clinically useful explanatory hierarchy — for example psychoeducation, referral summaries, and executive-function explanations. In these settings, output quality depends both on the model and on whether the local system is free of competing heavyweight workloads.

The MCP integration pattern deserves particular attention: applications connect to a local MCP server, which in turn communicates with the inference backend via HTTP. This layered architecture lets the application invoke discrete AI tools through a standardized interface without coupling application code to a specific LLM backend.

## Architecture Decision Records for Local Inference

Several ADRs in the soul agent project govern local inference decisions. See [[concepts/architecture-decision-records]] for the broader pattern.

- **ADR 0001** ([[summaries/0001‑choose‑local‑llm]]): Establishes the dual-backend strategy with OMLX as primary and Ollama as fallback, and defines `embed_with_fallback()` / `generate_with_fallback()` as the single call-site boundary.
- **ADR 0009** ([[summaries/0009-soul-local-llm-inference-with-omlx]]): Consolidates two prior ADRs into a single canonical reference. Confirms OMLX (`http://127.0.0.1:8000/v1`) as the inference server, with default models `nomicai-modernbert-embed-base-bf16` (embedding) and `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` (generation).

## HTTP Client Implementation

The embedding client component (documented in [[summaries/embedding-client]], implemented in `soul/neuro_report_style_agent.py` lines 87–144) provides the HTTP transport layer for all local OMLX calls. A key design decision is using **Python stdlib `urllib` exclusively** — no third-party HTTP libraries — keeping the dependency surface minimal:

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

The **3000-second (50-minute) timeout** is a deliberate design choice reflecting large model inference characteristics. The two public endpoints this client wraps are:
- `POST {base_url}/embeddings` — extracts `data[0]["embedding"]` as `List[float]`
- `POST {base_url}/chat/completions` — extracts `choices[0]["message"]["content"]` as `str`

## Locally Stored Model Inventory

The following models are currently loaded in the local Model Manager (see [[summaries/local_models]]), totalling approximately 57 GB:

| Model | Size | Category |
|---|---|---|
| MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit | 18.19 GB | LLM (reasoning-distilled) |
| Qwen3.6-35B-A3B-oQ4 | 19.68 GB | LLM |
| Qwopus3.5-9B-v3-PolarQuant-MLX-4bit | 4.71 GB | LLM |
| medgemma-1.5-4b-it-bf16 | 9.30 GB | Medical LLM |
| OpenMed-PII-SuperClinical-Large-434M-v1 | 1.63 GB | Clinical NLP / PII |
| OpenMed-PII-BiomedELECTRA-Large-335M-v1-mlx | 1.25 GB | Biomedical NLP / PII |
| parakeet-tdt-0.6b-v3-mlx-8bit | 867.3 MB | Speech recognition |
| granite-docling-258M-mlx | 610.8 MB | Document processing |
| nomicai-modernbert-embed-base-bf16 | 288.2 MB | Embeddings |
| nomic-embed-text (via Ollama) | ~274 MB | Embeddings (PAI RAG) |

Several observations emerge from this collection:

- **[[concepts/mlx-framework]] dominance**: The majority of models are in Apple MLX format, confirming deployment on Apple Silicon hardware.
- **Quantization diversity**: 4-bit, 8-bit, bfloat16, and Q4 quantization formats coexist. See [[concepts/model-quantization]].
- **Domain specialization**: The collection includes dedicated medical/clinical NLP models, embedding models, a document parser, and a speech-to-text model.
- **Scale range**: Parameters span from 258M to ~35B active, illustrating the multi-tier model strategy.
- **Hybrid capability profile**: The Claude-distilled Qwen variant is notable because it appears to combine dense reasoning structure with smoother explanatory language, making it especially useful for interpretation-heavy writing. See [[summaries/LLM Benchmark Comparison]].
- **Clinical workflow alignment**: The mix of small utility models, embedding models, medical models, and larger synthesis models mirrors the workflow decomposition described in [[summaries/Luria_AI_Q4_Investor_Memo_2026]]: extraction, validity checking, interpretation support, quality assurance, and final narrative generation are better served by a model portfolio than by a single monolithic model.
- **Investor-story alignment**: The same Apple MLX model stack that powers local workflows also underpins external claims of technical feasibility in startup application materials, making the model inventory relevant not only operationally but narratively. See [[summaries/README_20260413235016]] and [[summaries/README_20260413235353]].

### Notable Models

**General LLMs**
- The two Qwen 35B variants — one reasoning-distilled from Claude Opus 4.6 — are the largest models and suited for complex reasoning, agentic tasks, and report generation.
- `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` at 4.71 GB fills a medium tier for faster utility tasks and is the default generation model for the soul agent.

**Medical / Clinical LLMs**
- `medgemma-1.5-4b-it-bf16` — Google's MedGemma instruction-tuned model for medical text comprehension; the default local chat model for the Luria Streamlit App and domain-level summarization in the cingulate R package.

**Clinical NLP / PII** (see [[concepts/pii-redaction-pipelines]] and [[concepts/clinical-nlp-pipelines]])
- `OpenMed-PII-BiomedELECTRA-Large-335M-v1-mlx` and `OpenMed-PII-SuperClinical-Large-434M-v1` — specialized models for detecting and redacting PII in clinical text, directly supporting [[concepts/phi-data-handling]] requirements.

**Embeddings**
- `nomicai-modernbert-embed-base-bf16` (288.2 MB) is the embedding backbone for the neuro report soul agent and the Luria Streamlit App's oMLX embedding server. See [[concepts/sqlite-as-vector-store]] and [[concepts/lancedb-vector-store]].
- `nomic-embed-text` (via Ollama) produces 768-dimensional vectors for the PAI RAG system. See [[concepts/duckdb-as-vector-store]] and [[concepts/parquet-as-knowledge-store]].

**Document Processing**
- `granite-docling-258M-mlx` — IBM's Docling model for document layout understanding and structured extraction, underpinning the [[concepts/docling-pdf-parsing]] step.

**Speech Recognition**
- `parakeet-tdt-0.6b-v3-mlx-8bit` — NVIDIA's Parakeet TDT model for local speech-to-text transcription.

## Hardware Requirements

The central constraint in local inference is memory. Model weights must fit entirely in fast memory to run at practical speeds. Apple Silicon Macs with large unified memory configurations have emerged as a compelling option. The Mac Studio M3 with 512GB unified memory can house models exceeding 600 billion parameters while remaining relatively affordable. See [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]] for a detailed account.

The Luria.app redesign ([[summaries/redesign_20260623110910]]) exemplifies this hardware dependency: it targets the M3 Max as part of its active case-file metadata (`oMLX Primary · M3 Max · v0.4.4`), confirming Apple Silicon as the expected deployment platform.

The Q4 2026 investor memo adds another practical hardware implication: if a platform promises near-production clinical throughput, hardware planning cannot optimize only for raw token speed. It must support predictable turnaround, local privacy constraints, and enough headroom for multiple stages of the workflow — extraction, scoring, reasoning, feedback loops, and reporting — without destabilizing output quality. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]].

The YC S26 application notes implicitly add a founder-facing hardware lesson: Apple Silicon and MLX are not just implementation conveniences, but part of a credible story for building privacy-sensitive AI products without a large infrastructure team. The newer note makes this more explicit by tying local MLX viability directly to solo-founder strategy, regulatory framing, and coding-session readiness. See [[summaries/README_20260413235016]] and [[summaries/README_20260413235353]].

### Key Hardware Metrics
- **Memory capacity**: Determines the largest model that can be loaded
- **Memory bandwidth**: The primary driver of token generation speed
- **Compute throughput**: Affects prompt processing speed
- **Thermal headroom**: Sustained local inference can be limited by thermal or resource saturation during long or concurrent jobs

### Apple Silicon Concurrency Constraints

Recent observations on M3 Max-class hardware show that local model quality and latency can degrade when multiple heavyweight models run simultaneously. This is not merely a throughput issue; it can alter the perceived intelligence of outputs.

Common failure modes include:
- memory pressure
- unified memory contention
- scheduling instability
- concurrent prefill contention
- KV/cache pressure
- speculative decoding interference
- cache fragmentation
- thermal saturation

In practice, on a 48GB M3 Max, the most reliable pattern is usually:
- **one large reasoning model at a time**
- **multiple small utility models concurrently when needed**
- **avoid multiple 30B+ reasoning models running simultaneously**

This matters especially during:
- PDF ingestion
- long context windows
- embeddings running at the same time as generation
- speculative decoding
- multi-agent or multi-report workloads

See [[concepts/concurrent-model-serving]], [[concepts/mlx-framework]], [[summaries/LLM Benchmark Comparison]], and [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]].

## Performance Dimensions

Local inference performance breaks down into two distinct phases:

1. **Prompt processing (prefill)**: Reading and encoding the input context. On Apple Silicon, processing 7,000 tokens takes approximately 30 seconds; 16,000 tokens takes ~90 seconds.
2. **Token generation (decode)**: Producing output tokens one at a time. Practical setups can stream output at more than 3× a human's reading speed.

For the PAI RAG system specifically, end-to-end query latency runs 3–8 seconds total. The embedding client's 3000-second timeout reflects the upper bound: for large model inference on very long prompts, generation can take tens of minutes.

A critical addition is that **performance variance matters as much as raw speed**. A model that is slightly slower but remains semantically coherent and stable under load may be more valuable than a faster model whose quality deteriorates unpredictably. This is especially important for [[concepts/clinical-ai-reasoning]], [[concepts/clinical-narrative-generation]], and [[concepts/neuropsychological-reporting]].

In clinical productivity systems, performance should also be measured at the workflow level, not just the token level. Luria's investor memo reports 8.5 hours reduced to 55 minutes per case, which highlights that the practical benchmark for local inference is often end-to-end clinician time saved rather than isolated tokens/second. This includes orchestration, structured data handling, QA, and revision loops in addition to pure generation speed. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]] and [[concepts/neuropsychological-assessment-pipeline]].

A parallel business-facing performance dimension is credibility: in startup materials, local inference helps answer whether a product can actually run in a privacy-constrained real-world setting. In that sense, performance includes deployability, not just throughput. The newer YC application planning note expands this by treating technical viability as part of the evaluation story for founder positioning and accelerator readiness, not merely an internal benchmark. See [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], and [[concepts/founder-track-record]].

## Software Stack

A typical local inference stack involves several layers:

- **Inference backend**: Libraries like **mlx-lm** (optimized for Apple Silicon via [[concepts/mlx-framework]]) or llama.cpp handle the actual computation.
- **Model server / runtime**: **Ollama** exposes an [[concepts/openai-compatible-api]] `/v1/chat/completions` HTTP endpoint. **OMLX** is an alternative that also exposes an OpenAI-compatible interface at `http://127.0.0.1:8000/v1`. The Luria.app redesign adds **vMLX** as a second local provider exposing the Responses API, creating a three-tier local stack (oMLX → vMLX → Ollama) before any cloud fallback. See [[summaries/redesign_20260623110910]] and [[concepts/omlx-server]].
- **HTTP client layer**: The embedding client ([[summaries/embedding-client]]) implements direct HTTP communication using only stdlib `urllib`, with no external dependencies.
- **Fallback strategy**: When a primary backend is unavailable, helper functions automatically retry against the next provider. The Luria.app formalizes this as `LocalFallbackLLMClient` iterating a `DEFAULT_PROVIDER_ORDER`. See [[concepts/fallback-strategy]].
- **PHI guard layer**: The Luria.app's `redactPhi()` function intercepts all PHI before it reaches the LLM client, and the `restrictToPreferredProviders` boolean gates cloud access. This is enforced at the orchestration layer in `reportJobs.ts` before any call to `agentRunner.ts`.
- **Protocol layer**: [[concepts/model-context-protocol]] servers sit above the model runtime, exposing AI capabilities as standardized, discoverable tools.
- **Orchestration**: LangGraph StateGraph pipelines (as in the Luria app) wire local inference steps as discrete nodes. See [[concepts/langgraph-agent-workflows]].
- **Workflow layer**: In production-oriented clinical systems, local inference is embedded inside a larger workflow engine that coordinates intake, scoring, interpretation, feedback loops, and report assembly. The Q4 2026 memo makes this explicit: the value comes from end-to-end orchestration, not just model invocation. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]], [[concepts/neuropsychological-assessment-workflow]], and [[concepts/clinical-ai-copilot]].
- **Application-strategy layer**: In founder materials, the stack is also translated into investor-readable language: local MLX inference, privacy-safe architecture, regulatory awareness, and coding-agent feasibility become evidence of execution capability. See [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], [[concepts/coding-agent-session]], and [[concepts/founder-narrative]].
- **IDE integration**: Tools like Positron's Databot extension connect directly to local Ollama instances. See [[concepts/ide-ai-assistant-configuration]].
- **Load discipline**: In serious local deployments, scheduler behavior and workload isolation are part of the effective software stack, because concurrent heavyweight jobs can materially degrade output quality.

## Backend Selection and Fallback Patterns

A robust local inference setup should not depend on a single backend being available. ADR 0001 and ADR 0009 codify a two-tier approach for the soul agent; the Luria.app redesign extends this to four tiers:

| Priority | Backend | Type | PHI-safe |
|---|---|---|---|
| #1 | oMLX | Local OpenAI-compatible | ✅ |
| #2 | vMLX | Local Responses API | ✅ |
| #3 | Ollama | Local native API | ✅ |
| #4 | Cloud | Remote OpenAI-compatible | ❌ (blocked for PHI) |

The key insight is that the first three backends expose an [[concepts/openai-compatible-api]], so the only application-level requirement is a thin wrapper that attempts each endpoint in order. The `restrictToPreferredProviders: true` flag in `LLMGenerateRequest` is the single boolean that enforces local-only operation. See [[concepts/fallback-strategy]] and [[concepts/llm-provider-abstraction]].

The `llmAbortContext` (`AsyncLocalStorage`) in the Luria.app threads an abort signal through the entire `generate()` call chain so a report job can be cancelled mid-fallback without leaving dangling inference calls. This is a significant operational improvement over simpler fallback patterns that lack cancellation support.

The investor memo adds a business reason for these patterns: when a clinical platform is pursuing healthcare-system pilots and aiming for 500+ cases/month, fallback design is no longer just a developer convenience. It becomes part of operational reliability, deployment readiness, and enterprise trust. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]] and [[concepts/healthcare-ai-regulation]].

The YC application notes add an adjacent founder-facing reason: fallback logic and local provider discipline are useful not only operationally but rhetorically, because they demonstrate that the product is designed around real privacy and deployment constraints rather than generic API usage. The newer note further frames this as part of a broader solo-founder execution narrative and competitive differentiation story. See [[summaries/README_20260413235016]] and [[summaries/README_20260413235353]].

Backend selection should also account for contention state, not just endpoint availability. A backend may be technically online but practically degraded because another large model is competing for memory and decode bandwidth. In mature local stacks, routing policy should eventually consider workload class, current concurrency, and model size rather than treating all available providers as equally healthy. See [[concepts/llm-provider-abstraction]] and [[concepts/concurrent-model-serving]].

## Embeddings as Part of Local Inference

Local inference is not limited to text generation. Embedding models can also run locally, enabling fully offline retrieval-augmented generation pipelines. Three systems demonstrate this pattern:

**Neuro report soul agent** ([[summaries/0009-soul-local-llm-inference-with-omlx]]):
- **Embedding model**: `nomicai-modernbert-embed-base-bf16` served via OMLX
- **Storage**: Float32 embeddings stored as JSON strings in [[concepts/sqlite-as-vector-store]]
- **Retrieval**: Cosine similarity computed in-memory

**PAI RAG system** ([[summaries/conversation-export]]):
- **Embedding model**: `nomic-embed-text` (768-D) served via Ollama
- **Storage**: [[concepts/duckdb-as-vector-store]] and [[concepts/parquet-as-knowledge-store]]
- **Retrieval**: [[concepts/hybrid-search-retrieval]] combining cosine similarity with SQL keyword search

**Luria Streamlit App** ([[summaries/README_luria]]):
- **Embedding model**: `nomicai-modernbert-embed-base-bf16` served via OMLX
- **Storage**: LanceDB (`data/vectors/`) — see [[concepts/lancedb-vector-store]]
- **Retrieval**: SQL filtering over structured scores combined with semantic search over narrative chunks

All three approaches keep the entire RAG stack — from embedding to retrieval to generation — on local hardware.

## Model Selection

Not all models are equal for local inference. Key trade-offs:

- **Parameter count vs. active parameters**: [[concepts/mixture-of-experts]] architectures keep large total weights in memory while activating only a fraction per token.
- **Quantization**: Compressing weights reduces memory footprint at some quality cost. See [[concepts/model-quantization]]; 4-bit dynamic quantization is the practical sweet spot on Apple Silicon.
- **Model tiers**: Agentic workflows benefit from running multiple model sizes in parallel. The cingulate package formalizes this through task-specific model selection across three performance modes.
- **Domain fit**: For specialized domains, local models can be selected for domain relevance. The inclusion of `medgemma` and the OpenMed PII models reflects this strategy.
- **Task specialization**: The model collection covers speech-to-text (parakeet), document layout parsing (granite-docling), PII detection (OpenMed PII), and embedding (nomicai-modernbert, nomic-embed-text).
- **Explanatory profile**: Some models are better at clinically useful explanation because they maintain semantic cohesion, preserve conceptual hierarchy, and operate at the right abstraction depth. These traits may matter more than raw benchmark scores for reporting tasks. See [[concepts/semantic-cohesion]], [[concepts/clinical-communication-register]], and [[summaries/LLM Benchmark Comparison]].
- **Concurrency tolerance**: A model that performs well in isolation may become a poor choice if it is highly sensitive to shared-memory contention in your actual deployment environment.
- **Workflow role fit**: The Q4 2026 memo underscores that different stages of a clinical workflow place different demands on the model stack — validity checks, structured reasoning across cognitive domains, differential interpretation, and final report drafting should not be treated as the same task. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]] and [[concepts/cognitive-domains]].
- **Narrative fit for external stakeholders**: In startup and accelerator contexts, model choice can also support claims about technical realism. Apple-optimized local stacks are easier to explain as deployable, privacy-preserving, and capital-efficient than abstract future infrastructure plans. The newer YC note sharpens this by connecting model choice to solo-founder strategy, partner-preference fit, and question-by-question application tactics. See [[summaries/README_20260413235016]] and [[summaries/README_20260413235353]].

A practical local stack often ends up looking like this:
- one large reasoning model for deep interpretation
- one medium model for fast drafting
- one specialized medical model for phrasing or domain summaries
- separate small models for embeddings, extraction, and PHI detection

That division of labor is often more reliable than trying to force a single heavyweight model to do everything at once.

## Optimization Techniques

### KV Cache and Prompt Caching
Multi-turn conversations benefit enormously from caching previously processed context. Any non-determinism in prompt construction can invalidate the cache and force full reprocessing.

### Query Embedding Cache
For retrieval systems like the PAI RAG, caching query embeddings avoids re-running the embedding model on repeated queries.

### Caching Extraction Results
For document processing pipelines, caching extracted JSON output avoids re-running expensive LLM inference on unchanged inputs.

### System Prompt Reduction
Agentic clients can generate system prompts exceeding 20,000 tokens. Trimming these to under 8,000 tokens cuts prefill time significantly.

### Batch and Parallel Processing
Parallel execution via `ThreadPoolExecutor` (Python) or `furrr` (R) allows multiple documents to be processed simultaneously.

### Concurrency Control
Parallelism should be applied selectively. Recent observations suggest that running many small utility jobs concurrently can be beneficial, but multiple large reasoning models often degrade overall stability and latency consistency on Apple Silicon systems. For many local stacks, serialization of heavyweight inference is an optimization, not a limitation.

### Workload Separation
Separate embeddings, document parsing, and generation workloads when possible. Even if the hardware can technically host them together, staggered execution often improves perceived quality and reduces crash risk.

### Human Feedback Loop Placement
When local inference is used in clinician-facing systems, performance can improve by placing human review at high-value checkpoints rather than after every low-level step. The Q4 2026 memo's mention of real-time clinician feedback loops suggests that workflow design, not just model speed, is an optimization surface. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]].

### Narrative Compression for Application Materials
When local inference is presented in applications or investor materials, the technical stack often needs its own optimization: translate detailed infrastructure into concise claims about privacy, feasibility, differentiation, and regulatory readiness. The YC notes suggest this as a practical communication layer for founders, especially when preparing personal prompts and the coding-agent session alongside the core company application. See [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], and [[concepts/dual-audience-design]].

## Context Engineering for Multi-Turn Conversations

Stateless local inference setups can simulate memory across turns through context engineering. The neuro report soul agent plans a context summary approach: generate a JSON summary after each turn, persist it to `context.json`, and prepend it to the next prompt. See [[concepts/persistent-memory]] for broader discussion.

## Security and Privacy in Practice

For clinical use cases, local inference enables a full privacy-preserving stack:

- **No data exfiltration**: All LLM operations remain on localhost
- **PHI handling**: The Luria.app enforces PHI redaction before any network call, and `restrictToPreferredProviders: true` prevents cloud fallback for sensitive data. See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]].
- **PII redaction**: Dedicated local models (OpenMed PII series) detect and redact sensitive information. See [[concepts/pii-redaction-pipelines]].
- **Zero external HTTP dependencies in client code**: The embedding client uses only Python stdlib `urllib`.
- **Audio privacy**: Audio transcription routes through MacWhisper CLI and oMLX summarization locally.
- **Cancellation support**: The Luria.app's `llmAbortContext` ensures no orphaned inference calls persist after job cancellation.
- **PHI guard at orchestration**: `redactPhi()` is invoked at the `/stt/summarize` stage, before any text reaches `llmClient.generate()`, providing defense-in-depth independent of the `restrictToPreferredProviders` flag.
- **Encrypted local storage**: The Luria.app stores patient data in encrypted SQLite (`patients-sqlite.ts`), keeping scores and history on-device. See [[concepts/clinical-data-management]].
- **Compliance pathway support**: Local inference can simplify the practical path toward healthcare deployment because privacy-preserving architecture, local-first processing, and auditable workflow boundaries align better with enterprise security review and regulatory preparation. The Q4 2026 memo explicitly links this architecture to HIPAA alignment, SOC 2 goals, and eventual FDA 510(k) planning. The newer YC S26 planning note shows the same architectural choices being reused as external-facing regulatory positioning in accelerator materials. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]], [[summaries/README_20260413235353]], and [[concepts/healthcare-ai-regulation]].
- **Positioning advantage**: The YC application notes show that privacy-preserving local inference is also legible to external evaluators as a product-strength claim, not just an internal engineering decision. See [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], and [[concepts/founder-evaluation]].

## IDE Integration: Positron Databot

Positron Databot (available in Positron 2025.08.0+) represents an IDE-native form of local LLM integration. When connected to a local Ollama instance, it provides AI assistance for R and Python code without data leaving the workstation. See [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]] and [[concepts/ide-ai-assistant-configuration]] for detailed setup and remediation steps.

## Capability Expectations

As of 2026, the best open-weight models running locally are roughly comparable to cloud frontier models from 6–12 months prior. This gap is meaningful but narrowing rapidly. Local inference rewards those willing to invest time in configuration and optimization. See [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]] for a concrete blueprint.

A more nuanced expectation is that capability should be judged in context:
- a local model may be excellent for structured extraction but mediocre for nuanced explanation
- a distilled hybrid may be especially strong for readable, clinically grounded synthesis
- a benchmark win in isolation may disappear under concurrent load

For real deployments, the most valuable local model is often the one that stays coherent, available, and predictable in the actual pipeline.

The investor memo adds a further expectation shift: in applied clinical systems, success is not defined by matching cloud frontier generality. It is defined by whether a local-first stack can deliver reliable clinician-validated outputs inside a complete workflow with meaningful labor savings. That is a narrower but often more commercially relevant standard. See [[summaries/Luria_AI_Q4_Investor_Memo_2026]], [[concepts/clinical-ai-copilot]], and [[concepts/founder-evaluation]].

The YC application notes add a final expectation shift for founders: local inference does not need to prove universal superiority to be strategically valuable. It only needs to convincingly solve a constrained, high-trust workflow with credible technical and regulatory positioning. The newer note makes clear that this credibility extends into competitive framing, market sizing, founder story, and the coding-agent session. See [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], [[concepts/application-strategy]], and [[concepts/yc-partner-preferences]].

## Trade-offs vs. Cloud APIs

| Dimension | Local Inference | Cloud API |
|---|---|---|
| Privacy | ✅ Data stays local | ❌ Data leaves environment |
| Cost model | One-time hardware | Per-token ongoing |
| Setup complexity | Higher | Lower |
| Model quality | Hardware-dependent | Consistently frontier |
| Offline support | ✅ | ❌ |
| Scalability | Limited by hardware | Elastic |
| Server management | Required | Managed by provider |
| PHI gate | `restrictToPreferredProviders` flag | Provider terms of service |
| Cancellation support | `AsyncLocalStorage` abort context | Varies by SDK |
| Latency consistency | Operator-controlled but fragile under contention | Usually more predictable |
| Concurrency tuning | Must be managed explicitly | Abstracted by provider |
| Enterprise compliance fit | Often easier to constrain for PHI-sensitive workflows | Depends heavily on vendor controls |
| Fundraising narrative | Can signal privacy-first technical defensibility | Easier to prototype but weaker autonomy story |

## Related Concepts

- [[concepts/mixture-of-experts]] — Architecture that makes trillion-parameter local inference feasible
- [[concepts/model-quantization]] — Compression techniques essential for fitting large models in memory
- [[concepts/mlx-framework]] — Apple Silicon-optimized inference framework used by most models in the local inventory
- [[concepts/privacy-first-software]] — Privacy as a primary motivator for local deployment
- [[concepts/clinical-data-privacy]] — Patient data protection requirements that make local inference essential in clinical settings
- [[concepts/phi-data-handling]] — Handling of protected health information in local pipelines
- [[concepts/pii-redaction-pipelines]] — Pipelines for detecting and redacting PII, enabled by specialized local models
- [[concepts/clinical-nlp-pipelines]] — Clinical NLP workflows that run fully on local hardware
- [[concepts/model-context-protocol]] — Standardized protocol for exposing local LLM capabilities as tools
- [[concepts/openai-compatible-api]] — Shared API surface enabling OMLX, vMLX, and Ollama to be used interchangeably
- [[concepts/fallback-strategy]] — Pattern for graceful degradation when a primary backend is unavailable
- [[concepts/llm-provider-abstraction]] — Abstracting provider selection so application code is backend-agnostic
- [[concepts/retrieval-augmented-generation]] — RAG pipelines that can run fully locally using local embedding and generation
- [[concepts/hybrid-search-retrieval]] — Combined semantic + keyword retrieval used in both the PAI RAG and Luria systems
- [[concepts/duckdb-as-vector-store]] — Storing embeddings in DuckDB as used by the PAI RAG system
- [[concepts/parquet-as-knowledge-store]] — Parquet-based storage for knowledge base chunks and embeddings
- [[concepts/sqlite-as-vector-store]] — Storing embeddings in SQLite as an alternative to dedicated vector databases
- [[concepts/lancedb-vector-store]] — LanceDB for local vector storage, used by the Luria Streamlit App
- [[concepts/docling-pdf-parsing]] — Local PDF parsing underpinning the Luria ingest pipeline
- [[concepts/style-profile-extraction]] — Learning voice and tone patterns from historical documents using local LLMs
- [[concepts/persistent-memory]] — Strategies for maintaining context across stateless local inference calls
- [[concepts/scaling-laws]] — The relationship between model size and capability that drives the push toward larger local models
- [[concepts/neuropsychological-assessment-automation]] — Domain where local inference is applied for clinical report generation
- [[concepts/neuropsychological-assessment-pipeline]] — Broader pipeline context for local inference in neuropsychology
- [[concepts/luria-neuropsych-pipeline]] — The specific LangGraph-based pipeline powering the Luria Streamlit App
- [[concepts/luria-overview]] — Overview of the Luria.app platform and its local-inference architecture
- [[concepts/langgraph-agent-workflows]] — LangGraph StateGraph orchestration for multi-step local inference pipelines
- [[concepts/ide-ai-assistant-configuration]] — Configuration of AI assistants within IDEs, including local Ollama backends
- [[concepts/pai-assessment]] — The Personality Assessment Inventory, for which a dedicated local RAG system has been built
- [[concepts/pai-knowledge-base]] — The 81-document, 2,546-chunk knowledge base powering the PAI RAG system
- [[concepts/architecture-decision-records]] — ADRs 0001 and 0009 document the canonical local inference decisions for the soul agent
- [[concepts/single-file-agent-pattern]] — The soul agent's single-file structure containing all inference call-sites
- [[concepts/r6-class-architecture]] — R6 classes used in the cingulate package to abstract Ollama model routing
- [[concepts/yaml-configuration]] — YAML-based model routing configuration in the cingulate package
- [[concepts/domain-processor-pattern]] — Modular domain processors in cingulate that consume local LLM outputs
- [[concepts/duckdb-data-staging]] — DuckDB used in cingulate for high-performance data staging alongside LLM inference
- [[concepts/omlx-server]] — The oMLX local inference server used as the primary provider in multiple Luria systems
- [[concepts/clinical-ai-reasoning]] — AI-assisted clinical reasoning grounded in local, PHI-safe inference
- [[concepts/clinical-ai-copilot]] — Local inference as a clinician-facing assistant layer across structured workflows
- [[concepts/clinical-data-management]] — Encrypted local storage of patient scores and structured data
- [[concepts/neuropsychological-assessment-workflow]] — End-to-end workflow from intake through report generation using local inference
- [[concepts/concurrent-model-serving]] — Operational constraints and scheduling trade-offs when serving multiple local models
- [[concepts/local-inference-reliability]] — Reliability and consistency considerations for local model deployment
- [[concepts/llm-evaluation]] — Evaluation of local models should include stability under real workload conditions
- [[concepts/semantic-cohesion]] — A key quality marker for explanatory outputs produced by local models
- [[concepts/clinical-communication-register]] — The target explanatory style often used to judge model usefulness in practice
- [[concepts/clinical-narrative-generation]] — Local generation of clinically useful prose, summaries, and interpretations
- [[concepts/healthcare-workforce-automation]] — The broader economic rationale for local clinical AI deployment
- [[concepts/healthcare-ai-regulation]] — Compliance and regulatory context shaping local deployment choices
- [[concepts/application-strategy]] — Local inference as part of startup and accelerator positioning
- [[concepts/startup-application-materials]] — Founder-facing materials that translate technical architecture into investor-readable claims
- [[concepts/market-sizing]] — The business context in which local inference is positioned as enabling a viable niche healthcare market
- [[concepts/yc-partner-preferences]] — Accelerator framing where privacy, execution clarity, and technical realism matter
- [[concepts/founder-narrative]] — Local inference as evidence of founder judgment and product strategy
- [[concepts/founder-track-record]] — Technical implementation choices as signals of execution ability
- [[concepts/application-preparation]] — Preparation process for expressing technical viability in applications
- [[concepts/startup-fundraising]] — Fundraising context where local-first architecture supports defensibility claims
- [[concepts/coding-agent-session]] — YC application component where technical architecture may be demonstrated in practice
- [[concepts/dual-audience-design]] — Need to explain local inference both to engineers and to evaluators
- [[concepts/startup-differentiation]] — Local deployment as part of product differentiation in healthcare AI

See also: [[summaries/embedding-client]], [[summaries/local_models]], [[summaries/0001‑choose‑local‑llm]], [[summaries/0009-soul-local-llm-inference-with-omlx]], [[summaries/0008-soul-single-file-style-agent-architecture]], [[summaries/index]], [[summaries/mcp-integration]], [[summaries/002-mcp-llm-integration]], [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]], [[summaries/TECHNICAL_DOCS]], [[summaries/conversation-export]], [[summaries/SESSION_SUMMARY_2025-04-28]], [[summaries/deepagents_merged_mem_notes]], [[summaries/README_luria]], [[summaries/overview]], [[summaries/report-generation]], [[summaries/mlx_embeddings]], [[summaries/AS_PROCESSING_COMPLETE]], [[summaries/COMPLETE_STATUS]], [[summaries/EMBEDDINGS_COMPLETE]], [[summaries/KNOWLEDGE_BASE_EXPLAINED]], [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]], [[summaries/QUICK_REFERENCE]], [[summaries/README]], [[summaries/README_AS_PROCESSING]], [[summaries/README_PIPELINE]], [[summaries/README_WORKFLOW]], [[summaries/REBUILD_COMPLETE]], [[summaries/REBUILD_FINAL_STATUS]], [[summaries/WORKFLOW_INSTRUCTIONS]], [[summaries/report-template]], [[summaries/soul-style-agent]], [[summaries/style-training-to-report-drafting]], [[summaries/customization]], [[summaries/full-pipeline]], [[summaries/installation]], [[summaries/agent-team]], [[summaries/CLAUDE]], [[summaries/LLM_AGENT_MAP]], [[summaries/LLM_INTEGRATION]], [[summaries/copilot-instructions]], [[summaries/neurocog.prompt]], [[summaries/redesign_20260623110817]], [[summaries/redesign_20260623110910]], [[summaries/LLM Benchmark Comparison]], [[summaries/Luria_AI_Q4_Investor_Memo_2026]], [[summaries/README_20260413235016]], [[summaries/README_20260413235353]], [[summaries/Apply-to-Y-Combinator]]

See also: [[summaries/entry_points]]

See also: [[summaries/top_level]]

See also: [[summaries/README_20260413235148]]

See also: [[summaries/README_20260413235533]]

See also: [[summaries/README_20260414001057]]