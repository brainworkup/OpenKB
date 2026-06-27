---
sources: [summaries/autonomous-execution.md, summaries/agentic-workflows.md, summaries/README_20260414001057.md, summaries/README_20260413235353.md, summaries/README_20260413235148.md, summaries/README_20260413235016.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/File Folder Structure Rebuild.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/top_level.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/requirements.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md, summaries/README.md, summaries/DEPENDENCIES.md, summaries/installation.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/vector-store.md, summaries/text-extraction.md, summaries/soul-style-agent.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md]
brief: Architecture that keeps sensitive data and core workflows on the local machine.
---

# Local-First Architecture

Local-first architecture is a design philosophy that keeps data storage, computation, and core application logic on the user's own machine rather than depending on remote cloud services. The approach prioritizes user control, privacy, portability, inspectability, and operational simplicity over the coordination and scaling advantages of centralized infrastructure. In the Luria ecosystem, local-first design is not just a product preference but a foundational constraint shaped by neuropsychological practice: patient evaluations contain highly sensitive information, report writing depends on nuanced clinical synthesis, and clinicians need systems they can run, inspect, and trust within strict privacy boundaries.

Across Luria materials, local-first architecture is presented as both a technical and strategic differentiator. The founder's YC framing describes Luria as a local-first, agent-based neuropsych workflow system, while the Q4 2026 investor memo extends that position by treating local-first processing as a key enabler of a production-ready clinical AI platform. In that memo, privacy-preserving architecture and local-plus-cloud model orchestration are part of the company's argument for why neuropsychological AI can now be clinically usable at scale.

## Core Principles

- **Data sovereignty**: Persistent state lives in files and databases the user controls directly.
- **Offline capability**: Core workflows should continue to function without network access.
- **Portability**: Moving systems between machines should primarily involve copying files and reinstalling local dependencies, not recreating cloud infrastructure.
- **Minimal external dependence**: Remote services are optional or sharply constrained by data type.
- **Auditability**: Local artifacts can be inspected, backed up, versioned, and reasoned about by the operator.
- **Privacy by architecture**: Sensitive data stays local because the system is designed that way, not merely because policy says it should.

## The Cloud/Local Boundary: Enforced at the Data Layer

The Luria ecosystem makes a principled distinction between two classes of data:

| Data Class | Where It Runs | Reason |
|---|---|---|
| **Patient data** — case histories, intake notes, score data, draft reports, profile data, anything identifiable | **Local only** | PHI must remain on device; clinical workflows require strict privacy controls. |
| **Reference material** — curated knowledge bases, documentation, public diagnostic material, scoring references | Cloud APIs allowed when desired | Public or non-patient knowledge can use higher-quality remote inference without exposing patient information. |

This boundary is enforced at the **data layer**, not merely by user intent. The same feature type may run locally or remotely depending on whether the underlying inputs contain PHI. That is one of the most important local-first ideas in Luria: the architecture is determined by what the data is, not by what the interface is doing.

Both the YC framing and the Q4 2026 investor memo make this explicit in business terms. The founder argues that generic cloud-based AI tools are a poor fit for neuropsychological evaluation because the field requires strict PHI boundaries together with clinically credible interpretation. The investor memo adds that local LLMs and privacy-preserving architecture are part of the reason the timing is now favorable for deployment in healthcare systems. Local-first architecture therefore serves multiple roles at once: privacy protection, workflow fit, compliance alignment, and product differentiation.

This directly connects to [[concepts/phi-data-handling]], [[concepts/clinical-data-privacy]], and [[concepts/privacy-first-software]].

## The Luria Streamlit App: Local-First in Practice

The Luria Streamlit app, documented in [[summaries/README]], is the clearest operational expression of local-first architecture in the current system. It is described as a local-first, HIPAA-conscious desktop workflow for solo clinicians who need real patient information to remain on their own machine.

Its architecture instantiates the core principles in several ways:

- **PHI-sensitive processing is local**: parsing, staging, storage, and retrieval happen on the user's machine.
- **No cloud vector store by default**: local vector and relational backends are used instead of hosted storage.
- **No cloud spreadsheet or SaaS database**: structured clinical data stays in local databases.
- **Audio processing can be local**: transcription and summarization can run on-device.
- **Secrets isolation**: local environment configuration keeps credentials off version control.

The app's ingestion, querying, knowledge, and audio workflows all assume that the primary system of record is the local machine. Remote model providers may be used for selected tasks only when configuration allows it and only when data handling rules permit it.

The Q4 2026 investor memo reinforces this implementation stance by highlighting local-first processing, a privacy-preserving architecture, and Desktop Commander integration for seamless clinician workflow. It also describes a multi-model setup in which local and cloud LLMs are combined for quality and cost optimization. In local-first terms, that means remote models may participate in the workflow, but the architecture still centers local control over sensitive case handling.

## The KB Knowledge Layer: Local-First for Reference Material

The knowledge-base layer shows that local-first is not synonymous with "never use the cloud." Instead, it means that the system preserves a stable privacy boundary and keeps operator-controlled local copies of important knowledge artifacts.

The current architecture separates retrieval by data characteristics:

**Tier 1 — wiki notes**: A small curated markdown corpus used as an inspectable reference layer. This material is compact enough to support direct prompting strategies and transparent citation.

**Tier 2 — larger stores**: Heavier documentation and heterogeneous corpora use embeddings and local storage/indexing strategies where scale demands them.

This split supports a practical local-first rule: keep sensitive operational data local always, and treat public reference knowledge as a separate class with more flexibility. The resulting system remains auditable because the knowledge artifacts themselves are stored and organized locally even when optional cloud inference is used against non-sensitive content.

See also [[concepts/knowledge-base-architecture]] and [[concepts/retrieval-augmented-generation]].

## Installation and Environment Isolation

Luria's installation model reflects local-first principles from the ground up. As documented in [[summaries/installation]], the recommended setup uses local Python environments with no mandatory hosted control plane. Virtual environments provide the isolation that makes a local-first deployment reproducible and portable.

Key installation choices that reinforce this design:

- **[[concepts/python-environment-management]]**: Virtual environments keep dependencies self-contained on the user's machine.
- **Local `.env` configuration**: API keys, model endpoints, and database paths are stored locally and excluded from version control.
- **Optional local model endpoints**: local inference servers can be substituted for cloud APIs.
- **Source builds**: users can clone and run the system from source, preserving control over the software stack.

macOS is currently the primary target platform, largely because local tooling for Apple Silicon makes on-device inference and desktop workflows practical. See [[concepts/mlx-framework]] and [[concepts/omlx-server]].

R integration follows the same pattern: packages are installed locally and connected through local configuration. See [[concepts/r-python-integration]] and [[concepts/r-neuropsych-packages]].

## Storage Backends

The Luria ecosystem uses multiple local storage backends, each chosen for a specific role:

| Backend | Purpose |
|---|---|
| SQLite | Structured clinical records, extracted scores, summaries, and lightweight application state |
| LanceDB | Local semantic search over narrative content |
| SQLite (style index) | Style exemplar vectors and report-writing memory |
| DuckDB | Analytical and retrieval-oriented columnar storage |
| JSON files | Profiles, configuration, and portable structured state |

All of these default to local disk. This is a deliberate architectural choice rather than a temporary implementation shortcut. For a solo or small-practice workflow, local files and embedded databases often provide the right operational scale while preserving inspectability and control.

See [[concepts/lancedb-vector-store]], [[concepts/sqlite-as-vector-store]], and [[concepts/duckdb-as-vector-store]].

## In the Soul Style Agent

The `soul/` subsystem is a useful micro-example of local-first architecture. As described in [[summaries/0008-soul-single-file-style-agent-architecture]], it keeps both state and behavior easy to inspect:

- **SQLite for vector storage** rather than a hosted vector database
- **JSON for profile state** rather than a remote configuration service
- **Minimal runtime dependencies** for easier portability
- **Transparent on-disk artifacts** that a clinician or developer can move, back up, or review

This pattern matters because local-first architecture is not only about privacy. It also supports knowledge continuity and tool longevity: the system remains understandable even when disconnected from remote services or organizational infrastructure.

## LangGraph Pipeline and Local Operation

The LangGraph-based pipelines are designed to run locally by default:

- **Ingest pipeline**: local parsing, local extraction when configured, local writes to embedded stores, and local rendering of final artifacts
- **RAG/query pipeline**: local semantic search plus local structured queries, followed by synthesis through local or optionally remote models depending on data sensitivity

This fits the founder's YC description of Luria as a system of agents and subagents that can execute the neuropsychological workflow nearly end-to-end. The Q4 2026 investor memo extends this picture by describing a production-oriented orchestration layer covering intake, scoring, interpretation, reporting, clinician feedback, and quality assurance. Local-first architecture is what makes that agentic design clinically usable: autonomous workflows are only acceptable when the most sensitive records remain under local control.

See [[concepts/langgraph-agent-workflows]] and [[concepts/subagent-architecture]], along with [[concepts/multi-agent-orchestration]].

## Customization Without Leaving Local Bounds

The local-first model does not prevent customization; in Luria it enables it.

As shown in [[summaries/customization]], users can tailor the system while keeping everything self-contained:

- clinician-specific and population-specific [[concepts/style-profiles]]
- direct editing of local JSON configuration and profile files
- configurable [[concepts/rag-chunking]] and retrieval parameters
- model swapping through environment variables or CLI settings
- local batch indexing of larger corpora
- output to Markdown, Typst, PDF, or structured formats on disk

This is especially important in neuropsychological reporting, where report tone, structure, and interpretive emphasis are part of professional practice rather than mere presentation settings.

## Relationship to Privacy and Clinical Context

Local-first architecture is especially important in clinical settings because privacy is inseparable from system design. In neuropsychological evaluation, reports integrate interviews, observations, test scores, developmental history, and often forensic or school-related context. That combination produces records that are both sensitive and difficult to de-identify perfectly.

Luria's product materials repeatedly underscore this point. The YC application argues that the field's value lies in integrated interpretation across cognitive and neurobehavioral domains. The Q4 2026 investor memo makes the same point from an operational angle: the platform's goal is to automate scoring, interpretation, and report generation while maintaining clinician-validated quality and diagnostic specificity. Because that synthesis depends on rich case context, the system cannot assume that cloud tools are safe by default. A local-first posture is therefore aligned with both privacy requirements and the substantive nature of the clinical task.

Key protective measures include:

1. **PHI-sensitive processing stays local by default**
2. **Local storage backends are the system of record**
3. **Fully local inference is available for end-to-end private workflows**
4. **Secrets and configuration are isolated in local environment files**

This connects directly to [[concepts/phi-data-handling]], [[concepts/clinical-data-privacy]], [[concepts/pii-redaction-pipelines]], and [[concepts/privacy-first-software]].

## Relationship to Local LLM Inference

Local-first storage pairs naturally with [[concepts/local-llm-inference]]. In Luria, local model serving through [[concepts/mlx-framework]] and [[concepts/omlx-server]] makes it possible to keep both data and inference on-device. The use of an [[concepts/openai-compatible-api]] interface allows local and remote backends to share the same application surface, which helps preserve portability without weakening the privacy boundary.

The Q4 2026 investor memo makes this relationship more central to the company's platform strategy. It identifies local LLMs as one of the enabling technologies behind the product's timing and describes a multi-model architecture that combines local and cloud inference for quality and cost optimization. In conceptual terms, local-first architecture does not require ideological rejection of remote models; it requires that remote participation be subordinate to the privacy boundary and the data classification rules.

This relationship is important conceptually: local-first architecture is strongest when not only the files but also the reasoning pipeline can remain inside the user's environment.

## Optional Integrations That Preserve Local Posture

Even optional integrations are designed to preserve the local-first boundary:

- **Peer-observation or workflow integrations** remain optional rather than foundational. See [[concepts/honcho-ai-peer-observation]].
- **Tracing and developer telemetry** are opt-in, not required for operation.
- **Rendering tools** such as Quarto and Typst run locally as CLI tools.
- **Audio tooling** can remain on-device. See [[concepts/audio-transcription-pipeline]].

The larger principle is that external services may enrich the workflow, but they should not become mandatory for the safe handling of sensitive clinical data.

## Retrieval Strategy and the Right-Sizing Principle

The broader system illustrates a useful corollary of local-first design: retrieval and storage should be right-sized to the actual corpus and workflow.

- Small curated corpora may not need embeddings-heavy infrastructure.
- Larger corpora can use local vector indexes or analytical stores.
- Public knowledge and patient data should not be collapsed into one undifferentiated retrieval tier.

This right-sizing principle is especially relevant to Luria because the founder's account frames the product as emerging from a single clinician's real workflow over several years. The architecture reflects that origin: it favors practical embedded tools that can scale meaningfully for one practitioner or a small group before introducing distributed infrastructure. The Q4 2026 memo suggests the next step in this logic: a local-first system can still scale into healthcare pilots if the privacy boundary remains stable while orchestration, deployment, and QA mature around it.

See [[concepts/knowledge-base-architecture]], [[concepts/vector-search]], and [[concepts/retrieval-augmented-generation]].

## Tradeoffs and Scalability Ceiling

Local-first choices come with real tradeoffs:

- **Storage scale**: embedded databases are excellent at small-to-medium scale but may need re-architecture for larger multi-tenant workloads.
- **Compute scale**: local inference quality and latency depend on available hardware.
- **Collaboration**: multi-user workflows require explicit sync, export, or coordination mechanisms.
- **Operational burden**: the user or organization must manage local environments, backups, and upgrades.
- **Device dependence**: the clinician's machine becomes a critical infrastructure component.

The investor memo is useful here because it clarifies why these tradeoffs are acceptable in Luria's context. The company's stated near-term path is not generic consumer SaaS scale but healthcare-system pilots, clinical validation, enterprise deployment infrastructure, and compliance work. In neuropsychological evaluation, the primary challenge is preserving privacy and quality in a workflow where integrated interpretation is the product. That makes local-first architecture a rational default even if some future layers eventually become more networked.

## Relationship to Luria's Product Identity

Within Luria, local-first architecture is not just an implementation pattern; it is part of the product's identity. The founder's YC narrative and the Q4 2026 investor memo together position Luria as:

- a system built from firsthand clinical pain points
- an agent-based workflow engine for neuropsychological evaluation
- a privacy-conscious alternative to generic cloud AI tools
- a reporting system capable of domain-specific synthesis rather than generic test summaries
- a platform aiming to address clinician shortages through workflow automation without sacrificing quality

That framing ties local-first architecture directly to [[concepts/luria-overview]], [[concepts/luria-neuropsych-pipeline]], [[concepts/neuropsychological-assessment-automation]], [[concepts/narrative-report-generation]], and [[concepts/healthcare-workforce-automation]].

## Related Concepts

- [[concepts/local-llm-inference]]
- [[concepts/privacy-first-software]]
- [[concepts/phi-data-handling]]
- [[concepts/clinical-data-privacy]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/clinical-ai-copilot]]
- [[concepts/clinical-ai-reasoning]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/neuropsychological-assessment-workflow]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/narrative-report-generation]]
- [[concepts/luria-overview]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/multi-agent-orchestration]]
- [[concepts/subagent-architecture]]
- [[concepts/langgraph-agent-workflows]]
- [[concepts/sqlite-as-vector-store]]
- [[concepts/lancedb-vector-store]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/python-environment-management]]
- [[concepts/r-python-integration]]
- [[concepts/r-neuropsych-packages]]
- [[concepts/knowledge-base-architecture]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/vector-search]]
- [[concepts/style-profiles]]
- [[concepts/rag-chunking]]
- [[concepts/mlx-framework]]
- [[concepts/omlx-server]]
- [[concepts/openai-compatible-api]]
- [[concepts/audio-transcription-pipeline]]
- [[concepts/honcho-ai-peer-observation]]
- [[concepts/healthcare-ai-regulation]]
- [[concepts/human-in-the-loop-clinical-ai]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/README]]

See also: [[summaries/installation]]

See also: [[summaries/0009-soul-local-llm-inference-with-omlx]]

See also: [[summaries/0008-soul-single-file-style-agent-architecture]]

See also: [[summaries/customization]]

See also: [[summaries/full-pipeline]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/soul-style-agent]]

See also: [[summaries/vector-store]]

See also: [[summaries/File Folder Structure Rebuild]]

See also: [[summaries/LLM Benchmark Comparison]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/README_20260413235016]]

See also: [[summaries/README_20260413235148]]

See also: [[summaries/README_20260413235353]]

See also: [[summaries/README_20260414001057]]

See also: [[summaries/agentic-workflows]]

See also: [[summaries/autonomous-execution]]