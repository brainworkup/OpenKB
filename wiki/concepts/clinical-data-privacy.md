---
sources: [summaries/clinical-assessment.md, summaries/agentic-workflows.md, summaries/README_20260414001057.md, summaries/README_20260413235353.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/File Folder Structure Rebuild.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md, summaries/installation.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/full-pipeline.md, summaries/style-training-to-report-drafting.md, summaries/text-extraction.md, summaries/soul-style-agent.md, summaries/report-generator.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/0004-soul-style-profile-json.md, summaries/0002-soul-sqlite-vector-storage.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/RECOVERY_NOTES.md, summaries/README.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/QUICK_REFERENCE.md, summaries/COMPLETE_STATUS.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/overview.md, summaries/002-mcp-llm-integration.md]
brief: Privacy architecture for AI systems handling sensitive clinical data.
---

# Clinical Data Privacy in AI Systems

Clinical data privacy refers to the architectural, workflow, and governance decisions that keep sensitive patient information — including psychological assessments, medical records, test scores, and personally identifiable information — protected throughout AI-enabled clinical workflows. In clinical settings, privacy is both a regulatory requirement and an ethical constraint. Systems must minimize exposure of Protected Health Information, preserve clinician control over data flow, and ensure that AI processing does not widen the risk surface for patient information.

In the Luria ecosystem, clinical privacy is treated not as a secondary compliance layer but as a product-defining capability. The Q4 2026 investor memo ([[summaries/Luria_AI_Q4_Investor_Memo_2026]]) makes this explicit by describing privacy-preserving architecture, local-first processing, and multi-model deployment as core parts of a production-ready neuropsychology platform rather than optional technical preferences.

## Core Principles

### 1. Data Locality
The strongest privacy guarantee is that sensitive data remains within the local environment. Running models on-premises or on clinician-controlled hardware reduces the chance that PHI is exposed to third-party processors or transmitted across external networks. This principle is foundational in the Luria Streamlit App ([[summaries/README]]), the SOUL style agent, the PAI RAG system, and the Voice project, and it remains central to the Luria.app redesign ([[summaries/redesign_20260623110817]]).

The Q4 2026 investor memo reinforces this same position at the company level: local-first processing is presented as a key part of Luria AI's technical infrastructure and as one of the enabling conditions for privacy-preserving clinical AI at scale.

### 2. No Third-Party Data Exposure
Using external AI APIs can create risk that clinical text is logged, stored, or processed outside the organization’s direct control. Local inference removes most of this exposure. This is a concrete design constraint in the PAI RAG system (documented in [[summaries/TECHNICAL_DOCS]]) and the broader SOUL project infrastructure.

The Luria Streamlit App makes this especially explicit: PHI redaction happens locally during the Docling parse stage before any text is sent to Anthropic's cloud API for extraction. Even where a cloud call exists, the architecture attempts to ensure that raw PHI does not cross the network boundary.

The investor memo adds a broader platform framing: Luria AI's multi-model architecture uses local and cloud LLMs together for quality/cost optimization, but privacy-preserving design remains a core requirement. This means provider selection is not only a performance decision; it is part of the system's privacy model.

### 3. Auditability and Control
Clinical AI systems must support reviewability, reproducibility, and operational oversight. Local deployments improve control over model versions, inference paths, and data retention. This matters for clinical accountability, internal audits, and future regulatory review.

The investor memo's emphasis on production readiness, quality assurance automation, and structured clinical reasoning implies that privacy and auditability are linked: if a system participates in clinical documentation and decision support, organizations must be able to inspect how it handled sensitive inputs and generated outputs.

### 4. Minimization of PHI Surface Area
Only the minimum required information should be exposed to AI components. This can be enforced through selective extraction, de-identification, scoped prompts, and [[concepts/pii-redaction-pipelines]]. In the Luria pipeline, this is enforced architecturally: the parse node runs redaction locally before the extract node makes any network call.

This principle also aligns with the investor memo's description of workflow orchestration and quality assurance automation. End-to-end automation increases throughput, but it can also increase privacy risk unless each stage deliberately limits what data is passed onward.

---

## Application in the Luria.app Redesign

The Luria.app redesign ([[summaries/redesign_20260623110817]]) extends and formalizes the privacy-first architecture across a full clinical neuropsychological platform. The most significant privacy mechanism is a tiered local inference fallback chain implemented in `src/services/llmClient.ts` via the `LocalFallbackLLMClient`:

| Priority | Provider | PHI Safety | Notes |
|---|---|---|---|
| 1 | **oMLX** | ✅ PHI-safe | Local OpenAI-compatible API |
| 2 | **vMLX** | ✅ PHI-safe | Local Responses API |
| 3 | **Ollama** | ✅ PHI-safe | Local native API |
| 4 | **Cloud** | ⚠️ Non-PHI only | Blocked by `restrictToPreferredProviders: true` |

The `restrictToPreferredProviders: true` flag acts as a local-only gate: cloud inference is unreachable for any request carrying PHI. This is enforced at the `pickProvider()` level, not as a user-configurable setting.

The PHI boundary is further protected by `redactPhi()` — a dedicated guard function in the orchestration layer (`src/services/ai.ts`) that intercepts all text passing through `/stt/summarize` before it reaches any LLM client. PHI redaction is architecturally prior to inference, not a post-hoc safeguard.

Cancellation safety is handled via `llmAbortContext`, an `AsyncLocalStorage` instance that threads an abort signal through the full `generate()` call, ensuring that mid-flight PHI-bearing requests can be cleanly terminated without partial data leakage.

Data flow paths in the redesign maintain the privacy boundary at each stage:
- **Path A (Intake → Inference):** `IntakeDossier.tsx` → encrypted SQLite → `redactPhi()` → local LLM → Clinical Summary
- **Path B (Report Job):** UI → `reportJobs.ts` → `agentRunner.ts` → `LocalFallbackLLMClient` → fallback chain
- **Path C (Agent → Result):** `AgentContext` (raw clinical data) → section agent → `generate()` → structured report text

In all three paths, patient scores and clinical data remain in encrypted local SQLite (`patients-sqlite.ts`) and never transit external networks unless `restrictToPreferredProviders` is explicitly disabled.

The investor memo provides strategic context for these implementation details. Its description of local-first processing, privacy-preserving architecture, real-time clinician feedback loops, and quality assurance automation suggests that the redesign is not merely a technical experiment but part of a broader enterprise deployment path for a neuropsychology platform.

---

## Application in the Luria Streamlit App

The Luria Streamlit App ([[summaries/README]]) is one of the clearest implementations of clinical data privacy in this knowledge base. It is explicitly described as HIPAA-conscious and built for solo clinicians who need real PHI to stay on their own machine. Its privacy architecture operates at every layer:

| Component | Privacy Approach |
|---|---|
| PDF parsing (Docling) | Runs entirely locally; PHI redacted before any network call |
| LLM extraction (Claude Sonnet) | Only receives post-redaction text; single cloud call |
| Vector store (LanceDB) | Local only; no cloud vector database |
| Structured store (SQLite) | Local `data/neuropsych.db`; no cloud spreadsheet |
| Audio transcription (MacWhisper) | Runs locally via CLI |
| Summarization (oMLX) | Local inference server; no external calls |

The app's four-stage LangGraph ingest pipeline — parse → extract → index → report — enforces the privacy boundary structurally. Only the extraction step calls Anthropic's API, and only after local PHI redaction. All other stages, including vector indexing via [[concepts/lancedb-vector-store]] and structured storage in SQLite, remain fully on-device.

This local-first pattern is consistent with the investor memo's broader claim that privacy-preserving architecture is a necessary condition for scaling AI into healthcare systems. In other words, the Streamlit app can be understood as an early concrete implementation of principles later presented as platform-level strategy.

> **Security note from the project**: Rotate any API keys that may have been committed before `.gitignore` was finalized. Never commit real API keys.

---

## Application in the SOUL Style Agent (OMLX)

The SOUL style agent ([[summaries/0009-soul-local-llm-inference-with-omlx]]) provides a concrete implementation of clinical data privacy. Because source text and outputs are PHI-adjacent, all LLM inference must run locally on the clinician's machine. Cloud APIs were explicitly ruled out for privacy and data handling reasons.

The canonical solution uses OMLX ([[concepts/omlx-server]]) running at `http://127.0.0.1:8000/v1`, with two default models:

| Task | Model |
|------|-------|
| Embedding | `nomicai-modernbert-embed-base-bf16` |
| Generation | `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` |

All LLM I/O is routed through `embed_with_fallback` and `generate_with_fallback` in `soul/neuro_report_style_agent.py` — a single call-site for inference that enforces the privacy boundary and supports a future secondary provider without widening exposure across the codebase. This is documented in [[summaries/0008-soul-single-file-style-agent-architecture]] and aligns with the [[concepts/single-file-agent-pattern]].

Privacy consequences of this design:
- **No PHI leaves the machine**
- **No per-token API costs**
- **No external network round-trips** during drafting, training, or indexing
- **Apple Silicon fit** via [[concepts/mlx-framework]]

Trade-offs include a hardware dependency, an operational requirement to keep the OMLX server running, and user responsibility for model downloads and updates. The shared use of local inference infrastructure across SOUL and Luria also foreshadows the investor memo's company-level emphasis on multi-model clinical infrastructure.

---

## Application in the PAI RAG System

The PAI RAG system ([[summaries/TECHNICAL_DOCS]]) directly encodes clinical data privacy into its design through a tiered provider model:

| Provider | Privacy Level | Recommendation |
|---|---|---|
| Ollama (local) | ✅ Full — no external calls | **Preferred for clinical data** |
| OpenAI (cloud) | ⚠️ Data sent to third-party servers | Check institutional policy |
| Anthropic Claude (cloud) | ⚠️ Data sent to third-party servers | Check institutional policy |

The system's explicit recommendation is to use Ollama for sensitive clinical data, and [[concepts/local-llm-inference]] is treated as the default safe path. With Ollama:
- All 2,546 PAI knowledge base chunks and query vectors remain on-device
- No usage tracking or billing data is generated
- The system can operate fully offline
- HIPAA compliance is more achievable, given proper system configuration

Cloud APIs are supported for use cases where sensitivity permits, but the architecture makes the privacy trade-off explicit and visible to practitioners.

---

## Application in the Voice Project

The Voice project explicitly addresses clinical data privacy through its choice of Model Context Protocol servers backed by [[concepts/local-llm-inference]] via Ollama. As documented in [[summaries/mcp-integration]] and [[summaries/002-mcp-llm-integration]]:

- The local Ollama backend (`http://localhost:11434/v1`) ensures patient data from psychological test reports is never transmitted externally
- No API costs or cloud dependencies mean no third-party data processing agreements are required
- The system can operate fully offline, further reducing exposure
- Models are customizable for clinical and psychological terminology without sharing proprietary data with model providers
- The MCP server is restricted to localhost by default, with no external network access and user-level permissions enforced

This stands in direct contrast to the rejected alternative of direct API calls to OpenAI or Anthropic, which were ruled out explicitly due to privacy concerns, ongoing costs, and network dependency.

---

## Privacy Controls in MCP Architecture

The [[summaries/mcp-integration]] document details several concrete privacy controls built into the MCP integration layer:

| Control | Implementation |
|---|---|
| Local-only inference | Ollama running on `localhost:11434`, no external calls |
| Access restriction | MCP server bound to localhost by default |
| PHI sanitization | Remove or anonymize patient identifiers before processing |
| Anonymized identifiers | Use surrogate IDs in place of real patient names |
| Secure storage | Sensitive output data stored with appropriate access controls |

The workflow processes raw psychological test PDFs containing patient demographics and clinical scores — precisely the kind of data that requires the strictest handling.

---

## Relevant Threat Model

| Threat | Mitigation |
|---|---|
| Data exfiltration via API calls | Local LLM inference only ([[concepts/local-llm-inference]]); PHI redaction before any cloud call ([[summaries/README]]) |
| PHI reaching cloud via report jobs | `restrictToPreferredProviders: true` gate; `redactPhi()` guard ([[summaries/redesign_20260623110817]]) |
| Model provider data logging | No external API usage for PHI-containing text |
| Network interception | Offline-capable deployment |
| Unauthorized access to reports | Structured tool interfaces with defined input/output boundaries |
| Re-identification from AI outputs | Output review and [[concepts/pii-redaction-pipelines]] |
| Unauthorized MCP server access | Localhost-only binding, user-level permissions |
| Knowledge base exposure | Local DuckDB + Parquet storage ([[concepts/duckdb-as-vector-store]], [[concepts/parquet-as-knowledge-store]]) |
| PHI-adjacent style agent outputs | On-device OMLX inference, single call-site enforcement ([[summaries/0009-soul-local-llm-inference-with-omlx]]) |
| Raw PDF PHI reaching cloud APIs | Pre-call redaction in Docling parse stage ([[concepts/docling-pdf-parsing]]) |
| API key exposure | `.env` gitignored; key rotation required if previously committed |
| Mid-flight PHI request leakage | `llmAbortContext` AsyncLocalStorage for clean cancellation |
| Privacy drift during scale-up | Enterprise gates, provider restrictions, and workflow-level QA as described in [[summaries/Luria_AI_Q4_Investor_Memo_2026]] |
| Increased exposure from end-to-end automation | Stage-bounded data handling in [[concepts/neuropsychological-assessment-pipeline]] and local-first orchestration |

---

## Knowledge Base Storage and Privacy

The PAI RAG system stores its knowledge base — 81 clinical PDFs chunked into 2,546 segments with 768-dimensional embeddings — entirely in local files:

- **DuckDB** (in-process, no server): No network port, no remote access surface
- **Parquet files**: Static compressed files on disk, not exposed to network
- **Total footprint**: ~6.9 MB, easily kept on encrypted local storage

Similarly, the Luria Streamlit App stores all structured data in a local SQLite database (`data/neuropsych.db`) and all vector data in a local LanceDB instance (`data/vectors/`). The Luria.app redesign uses encrypted SQLite (`patients-sqlite.ts`) for the same purpose. Neither storage layer exposes a network port or transmits data externally. See [[concepts/duckdb-as-vector-store]], [[concepts/parquet-as-knowledge-store]], and [[concepts/lancedb-vector-store]] for implementation detail.

The investor memo broadens the significance of these storage choices by positioning the platform for healthcare system pilots. Local and encrypted storage are not only implementation details; they become prerequisites for institutional adoption, especially when negotiating pilots and preparing enterprise deployment infrastructure.

---

## Relationship to PHI Handling

Protected Health Information handling — governed by HIPAA in the US — requires that any system processing patient data implement technical safeguards. [[concepts/phi-data-handling]] addresses the broader data lifecycle, while clinical data privacy in AI systems focuses specifically on the inference and AI processing layer.

The SOUL ADR series ([[summaries/0009-soul-local-llm-inference-with-omlx]], [[summaries/0002-soul-sqlite-vector-storage]]) operationalizes these requirements as explicit architectural constraints. The Luria Streamlit App's design — described as HIPAA-conscious and local-first — reflects the same philosophy at the application level. The Luria.app redesign ([[summaries/redesign_20260623110817]]) extends this to a full multi-workspace clinical platform with a formal `redactPhi()` orchestration guard and a `restrictToPreferredProviders` flag.

The Q4 2026 investor memo adds a strategic layer: privacy is presented alongside structured reasoning, workflow orchestration, and quality assurance as part of a production-ready [[concepts/clinical-ai-copilot]] for neuropsychology. This links privacy not just to safe handling of data, but to the viability of AI-enabled clinical operations.

---

## Trade-offs

Local, privacy-preserving AI comes with real costs:
- Requires dedicated GPU/CPU hardware (see [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]])
- Model quality may be lower than frontier cloud models
- Initial setup and maintenance complexity is higher
- Local embedding models may have narrower coverage than cloud equivalents
- Users must manage model downloads, updates, and inference server health
- Hardware requirements can be significant
- The one cloud call in the Luria pipeline introduces network dependency and trust in the post-redaction boundary
- The `restrictToPreferredProviders` gate requires operational discipline: if all local providers fail, the system degrades rather than falling through to cloud for PHI requests
- Enterprise deployment adds new burdens such as certification, security review, and implementation support, themes that appear in [[summaries/Luria_AI_Q4_Investor_Memo_2026]] through planned SOC 2 work and healthcare-system pilots
- Multi-model architectures improve resilience and quality/cost control but can increase privacy complexity if routing policies are not rigidly enforced

These trade-offs are generally acceptable in clinical contexts where privacy is a non-negotiable constraint rather than a mere optimization target.

---

## Related Concepts

- [[concepts/local-llm-inference]] — Technical foundation for keeping data on-premises
- [[concepts/phi-data-handling]] — Broader PHI lifecycle management
- [[concepts/pii-redaction-pipelines]] — Pre-processing to minimize sensitive data exposure
- [[concepts/neuropsychological-assessment-pipeline]] — Primary workflow context requiring clinical privacy
- [[concepts/privacy-first-software]] — Broader design philosophy
- [[concepts/duckdb-as-vector-store]] — Local vector storage without network exposure
- [[concepts/parquet-as-knowledge-store]] — Offline-safe compressed knowledge storage
- [[concepts/retrieval-augmented-generation]] — Retrieval pattern often applied under privacy constraints
- [[concepts/pai-knowledge-base]] — Clinical knowledge base managed under these constraints
- [[concepts/local-first-architecture]] — Broader design principle for privacy-preserving systems
- [[concepts/llm-provider-abstraction]] — Provider switching without widening PHI exposure
- [[concepts/openai-compatible-api]] — API standard used by both Ollama and OMLX for local inference
- [[concepts/single-file-agent-pattern]] — Pattern used in SOUL to enforce a single inference call-site boundary
- [[concepts/fallback-strategy]] — Fallback logic that preserves the privacy boundary across providers
- [[concepts/model-quantization]] — Technique enabling local deployment of larger models
- [[concepts/mlx-framework]] — Apple Silicon framework underlying OMLX local inference
- [[concepts/lancedb-vector-store]] — Local vector store used in Luria
- [[concepts/docling-pdf-parsing]] — Local PDF parsing stage where PHI redaction occurs
- [[concepts/omlx-server]] — Local OpenAI-compatible inference server used across projects
- [[concepts/luria-neuropsych-pipeline]] — Full Luria pipeline context within which privacy controls operate
- [[concepts/clinical-data-management]] — Broader data management practices in clinical AI systems
- [[concepts/agent-pipeline-state-management]] — State handling relevant to PHI flow between agents
- [[concepts/human-in-the-loop-clinical-ai]] — Clinician feedback loops that constrain risk in sensitive workflows
- [[concepts/clinical-ai-reasoning]] — Clinical reasoning systems whose utility depends on trustworthy privacy boundaries
- [[concepts/healthcare-ai-regulation]] — Regulatory context shaping acceptable privacy architectures
- [[concepts/healthcare-workforce-automation]] — Strategic driver for scale that increases the importance of privacy-safe automation

See also: [[summaries/TECHNICAL_DOCS]], [[summaries/overview]], [[summaries/mcp-integration]], [[summaries/report-generation]], [[summaries/AS_PROCESSING_COMPLETE]], [[summaries/COMPLETE_STATUS]], [[summaries/QUICK_REFERENCE]], [[summaries/README_PIPELINE]], [[summaries/README_WORKFLOW]], [[summaries/REBUILD_COMPLETE]], [[summaries/WORKFLOW_INSTRUCTIONS]], [[summaries/README]], [[summaries/RECOVERY_NOTES]], [[summaries/SESSION_SUMMARY_2025-04-28]], [[summaries/0002-soul-sqlite-vector-storage]], [[summaries/0004-soul-style-profile-json]], [[summaries/0008-soul-single-file-style-agent-architecture]], [[summaries/0009-soul-local-llm-inference-with-omlx]], [[summaries/embedding-client]], [[summaries/report-generator]], [[summaries/soul-style-agent]], [[summaries/text-extraction]], [[summaries/style-training-to-report-drafting]], [[summaries/full-pipeline]], [[summaries/2026-04-26-cingulate-agent-team-design]], [[summaries/installation]], [[summaries/redesign_20260623110817]], [[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]], [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]], [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]], [[summaries/Luria_AI_Q4_Investor_Memo_2026]], [[summaries/redesign_20260623110910]], [[summaries/Apply-to-Y-Combinator-JWT]], [[summaries/File Folder Structure Rebuild]]

See also: [[summaries/README_20260413235353]]

See also: [[summaries/README_20260414001057]]

See also: [[summaries/agentic-workflows]]

See also: [[summaries/clinical-assessment]]