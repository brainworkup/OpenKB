---
sources: [summaries/agentic-workflows.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/pai_01.md, summaries/multi_patient_transcript.md, summaries/NP-20240415-001_report.md, summaries/sirf_synthesis.md, summaries/neurocog.prompt.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/README.md]
brief: End-to-end AI-assisted pipelines for neuropsych assessment and report generation.
---

# Neuropsychological Assessment Automation

Neuropsychological assessment automation refers to the use of software pipelines, structured data staging, privacy-conscious AI systems, agent-based orchestration, and AI-assisted narrative generation to transform raw cognitive and behavioral test data into complete clinical reports while reducing manual effort and preserving clinical accuracy, privacy, and workflow fidelity. In the Luria context, this automation is not limited to score interpretation alone; it is framed as support for nearly the entire neuropsychological evaluation workflow, from data collection and organization through synthesis, quality review, and report drafting. The newer agentic framing extends this further by treating the workflow as a coordinated system of orchestrators, tools, and specialized subagents that can automate ingestion, processing, redaction, table and figure creation, and reporting with selective human review. The Q4 2026 investor framing likewise presents Luria as an end-to-end clinical platform for intake, scoring, interpretation, quality assurance, and reporting, with clinician time savings positioned as a direct response to workforce shortage and burnout. See also [[concepts/luria-overview]] and [[concepts/luria-neuropsych-pipeline]].

## Why Automate?

Traditional neuropsychological reporting is labor-intensive: clinicians must integrate scores from many standardized tests across multiple cognitive domains, reconcile behavioral and interview data, write individualized narrative summaries, and format outputs for different audiences such as families, referral sources, schools, and forensic settings. The founder narrative behind Luria adds an important practical motivation: rising clinical demand and clinician scarcity make manual reporting increasingly difficult to sustain at high quality and acceptable turnaround times.

Automation addresses:

- **Scale**: Enabling higher-throughput reporting across large caseloads and healthcare systems
- **Consistency**: Applying uniform scoring logic, validity checks, and narrative structures
- **Quality**: Embedding clinical review checkpoints, structured reasoning, and validity language
- **Efficiency**: Reducing turnaround time from assessment to delivered report
- **Integration**: Combining test scores, interviews, observations, and multi-source records into a unified interpretation
- **Privacy**: Supporting local-first processing for highly sensitive PHI-bound workflows
- **Workforce leverage**: Allowing existing neuropsychologists to manage more cases without proportional increases in documentation burden
- **Selective autonomy**: Letting low-risk steps run silently while escalating high-risk clinical decisions for review

This is especially important in pediatric and forensic neuropsychology, where evaluations can consume many hours and the final report is the central clinical deliverable. In the Luria investor memo, the value proposition is quantified as substantial per-case time reduction while maintaining clinician-validated quality, making automation not just a convenience but an operational solution to healthcare capacity constraints. The agentic-workflows framing reinforces that the objective is not merely faster text generation but dependable orchestration of the full assessment pipeline, including upstream data work and downstream deliverable assembly. Related pages include [[concepts/neuropsychological-assessment-workflow]], [[concepts/neuropsychological-synthesis]], [[concepts/clinical-data-privacy]], [[concepts/forensic-neuropsychological-evaluation]], and [[concepts/healthcare-workforce-automation]].

## Core Pipeline Stages

### 1. Data Ingestion & Staging
Raw test scores and source materials are ingested and staged using [[concepts/duckdb-data-staging]], converting data through CSV → DuckDB → Parquet for high-performance querying. The `load_data_duckdb()` function reads CSVs, computes z-scores, and splits data into `neurocog`, `neurobehav`, and `validity` tables written in multiple formats such as Parquet, Feather, and CSV. This provides a structured, queryable backbone for downstream processing.

In the broader Luria framing, ingestion extends beyond spreadsheets to PDFs and other file types, reflecting the need to consolidate heterogeneous clinical inputs into one patient workspace. The agentic-workflows summary makes this stage more explicit as one step in a larger autonomous pipeline, where an orchestrating agent can load data from files, databases, or APIs and hand off normalized outputs to downstream processors. The investor memo also implies a broader intake layer spanning end-to-end case intake before scoring and interpretation begin, reinforcing automation as a full workflow rather than a report-writing utility. See also [[concepts/long-format-clinical-data]], [[concepts/neuropsychological-test-scores]], [[concepts/pdf-data-extraction]], [[concepts/pdf-score-extraction]], [[concepts/clinical-pdf-assessment]], and [[concepts/report-ingestion-pipeline]].

### 2. Domain Processing
Modular processors handle distinct [[concepts/cognitive-domains]] including:
- IQ / General Intelligence
- Academic Achievement
- Memory
- Attention and Executive Function (see [[concepts/executive-function-deficits]])
- Language / Verbal
- Spatial / Visuospatial
- Motor
- Social-Emotional
- ADHD and Emotion (multi-rater: self, parent, teacher)
- Adaptive and Daily Living
- Validity

This is implemented via the [[concepts/domain-processor-pattern]] using an [[concepts/r6-class-architecture]]. The `DomainProcessorFactoryR6` class applies the factory pattern to create domain-specific processors, with `batch_create()` enabling all domains to be processed in a single call. Multi-rater domains such as ADHD and emotion return separate processor instances per rater via `create_multi_processor()`.

The central clinical goal is not just isolated score description but integrated interpretation across neurocognitive and neurobehavioral domains, which is one of the key differentiators emphasized in both the Luria materials and the investor memo. The newer agentic framing also places this step within a broader planner-executor workflow: structured processors act like specialized workers that can be invoked, monitored, and retried by an orchestrating layer. Q4 2026 framing adds emphasis on domain-specific reasoning chains, clinical validity checks, and differential support across multiple neuropsychological domains rather than generic summarization. Related pages include [[concepts/neuropsychological-score-interpretation]], [[concepts/neurobehavioral-status-exam]], and [[concepts/neuropsychological-synthesis]].

### 3. LLM Routing & Model Gateway
The `OllamaModelRouterR6` class acts as a model gateway, routing clinical prompts to specific LLM models based on a role system configured in `inst/config/ollama_models.yml`. This YAML file maps every role to a model ID and fallback, enabling [[concepts/role-based-llm-routing]] without changing application code. Available roles include:

- `summarize_domain` — domain-level narrative
- `interpret_rating_scales` — behavioral scale interpretation
- `integrate_evaluation` — cross-domain clinical impression
- `generate_recommendations` — evidence-based recommendations
- `differential_diagnosis` — DSM-5 differential considerations
- `finalize_diagnostic_summary` — final diagnostic paragraph
- `generate_narrative` — full prose narrative
- `fallback_general` — generic fallback

This role abstraction is an instance of [[concepts/llm-provider-abstraction]], supporting both Ollama and MLX backends via configurable endpoint URLs. Within Luria, local routing is not merely a deployment preference but a privacy requirement for PHI-sensitive workflows. The agentic-workflows summary adds a useful architectural interpretation here: role routing is part of a tool-using agent system in which a central orchestrator dispatches subtasks to the right model, runtime, or external tool and then aggregates outputs. The investor memo adds that the production strategy is multi-model rather than single-model, combining local and cloud LLMs for quality/cost optimization while maintaining a privacy-preserving architecture. That makes this concept closely tied to [[concepts/local-first-architecture]], [[concepts/local-llm-inference]], [[concepts/privacy-first-software]], [[concepts/phi-data-handling]], and [[concepts/concurrent-model-serving]].

### 4. Prompt Building
The `DomainPrompterR6` class builds structured OpenAI-style message lists from neuropsychological test score data. It normalizes score fields, generates style-specific system instructions such as `clinical`, `parent`, or `school`, and formats scores as `Label | SS X | Yth percentile | Range` lines. This structured approach to [[concepts/narrative-report-generation]] ensures prompt consistency across domains.

Prompt construction in this setting also has to preserve clinical voice, audience-appropriate tone, and discipline-specific reasoning norms. In an agentic pipeline, prompt building functions as a context-packaging layer that prepares each worker step with the right structured inputs before generation or interpretation occurs. The investor memo implicitly reinforces this by stressing structured clinical reasoning, diagnostic accuracy, and preservation of clinical nuance as core product claims rather than cosmetic output goals. That makes this stage closely related to [[concepts/clinical-communication-register]], [[concepts/clinical-ai-reasoning]], [[concepts/neuropsychological-prompt-engineering]], and [[concepts/neuropsychological-prompt-configuration]].

### 5. Orchestrated Stage Execution with Artifacts
The `ReportLLMBridgeR6` class orchestrates pipeline stages with full artifact management and content-addressed caching. Each stage produces:
- A narrative `.md` file
- A metadata `.meta.json` file containing model, timing, and QC information
- An append-only audit `.jsonl` log
- A content-addressed cache entry (`.cache/<hash>.rds`)

This caching strategy means re-runs are idempotent: existing outputs survive unchanged unless inputs change. In the broader platform framing, orchestration also includes real-time clinician feedback loops and quality assurance automation, showing that artifact management is part of a larger reviewable clinical production system rather than a pure generation pipeline. The agentic-workflows document is especially relevant here because it frames the whole system as an orchestrator-worker architecture: a top-level agent plans, dispatches, monitors progress, handles retries, and assembles outputs across ingestion, processing, redaction, and reporting tasks. This makes stage execution not just sequential automation but managed delegation across specialized components. See [[concepts/artifact-caching-pipeline]], [[concepts/agent-pipeline-state-management]], [[concepts/decision-logging]], [[concepts/report-review-qa]], and [[concepts/orchestrator-worker-pattern]].

### 6. QMD Injection & Report Assembly
The `NeuropsychResultsR6` class connects processed domain data directly to [[concepts/quarto]] report files. It writes a `LLM_CONTEXT_START…END` block into the target `.qmd` file, runs narrative generation, and injects a `<summary>…</summary>` tag back into the same file. Each domain has a corresponding prompt keyword such as `promem` → `_02-05_memory_text.qmd` and prompt template file (`inst/prompts/pro*.qmd`), supporting a [[concepts/modular-report-architecture]] via Quarto includes.

The surrounding report stack also reflects a gradual evolution from R Markdown and LaTeX toward Quarto and Typst, making report generation both programmable and style-aware. In the larger automation concept, this stage is where structured findings become clinician-facing deliverables with consistent documentation quality across cases. The agentic-workflows framing broadens this stage to include not only narrative assembly but also creation of tables and figures as separately managed outputs that can be composed into the final report package. See also [[concepts/quarto-extensions]], [[concepts/typst-typesetting]], [[concepts/typst-modules]], [[concepts/clinical-report-structure]], and [[concepts/clinical-narrative-generation]].

### 7. Parallel Orchestration
Batch LLM calls can be parallelized across CPU cores using `run_llm_for_all_domains_parallel()`, and the full LLM-plus-render pipeline can be triggered with `cingulate_run_llm_then_render()`. This enables efficient processing of all major cognitive domains concurrently.

At a higher architectural level, the Luria materials describe the system as a coordinated team of agents and subagents, which connects this pipeline approach to [[concepts/multi-agent-orchestration]], [[concepts/subagent-architecture]], [[concepts/langgraph-agent-workflows]], and [[concepts/persistent-memory]]. The agentic-workflows summary reinforces this interpretation by describing long-horizon pipelines in which a central agent maintains state, delegates subtasks, tracks progress, and aggregates the resulting outputs. The investor memo adds a more operational scaling claim: automation benefits should scale roughly with case volume, making parallel orchestration relevant not only for engineering efficiency but also for healthcare deployment readiness.

### 8. PII Redaction and Risk-Gated Autonomy
A key stage highlighted by the agentic-workflows document is PII redaction before or during downstream generation. In neuropsychological assessment automation, this matters both for protecting patient privacy and for enabling flexible combinations of local and external tools when appropriate. Automated redaction pipelines can remove or mask identifying data before storage, retrieval, sharing, or model calls, supporting privacy-preserving clinical workflows.

The same document also emphasizes that autonomy should be risk-gated rather than absolute. Deterministic or low-risk actions such as file handling, type coercion, or routine formatting may run silently, while sensitive decisions such as ambiguous PHI removal, diagnostic synthesis, or final sign-off should surface for human review. This aligns neuropsychological assessment automation with practical clinical governance rather than unconstrained agent autonomy. Related pages include [[concepts/pii-redaction-pipelines]], [[concepts/phi-deidentification-pipeline]], [[concepts/clinical-data-privacy]], [[concepts/phi-data-handling]], and [[concepts/silent-operation]].

### 9. Hybrid Python–R Execution
The agentic-workflows document also clarifies an important implementation reality in the Luria ecosystem: the broader system may be Python-dominant while substantial workflow logic lives in the R-based `cingulate` engine. Neuropsychological assessment automation therefore includes cross-language orchestration, not just single-runtime scripting. In practice, this means a top-level coordinator may call R scripts or functions from Python, pass structured data between runtimes, and handle retries and errors across language boundaries.

This hybrid execution model matters because it separates the orchestration layer from the analytical engine layer. It allows a Python application or agent framework to coordinate user interaction, retrieval, and workflow state while relying on mature R neuropsychology code for scoring, processing, and report assembly. Related pages include [[concepts/r-python-integration]], [[concepts/r-neuropsych-packages]], [[concepts/cingulate-engine]], and [[concepts/luria-neuropsych-pipeline]].

## Key Software: `cingulate`

The [[summaries/LLM_AGENT_MAP]] and [[summaries/README]] describe the cingulate R package as the primary implementation of this automation approach. Key R6 components include:

| Class | Role |
|---|---|
| `OllamaModelRouterR6` | Model gateway + role routing |
| `DomainPrompterR6` | Prompt builder |
| `ReportLLMBridgeR6` | Stage orchestration + artifacts |
| `DomainProcessorFactoryR6` | Domain processor factory |
| `NeuropsychResultsR6` | Data → QMD injection |
| `ConfigManagerR6` | YAML config management |
| `ErrorHandlerR6` | Safe execution + logging |
| `ExecutionTrackerR6` | Step tracking |

High-level entry points include:
- `cingulate_workflow()` — full end-to-end pipeline
- `cingulate_quick_start(patient, age)` — new-user onboarding
- `create_patient_workspace(name, age)` — scaffold patient directory ([[concepts/per-patient-workspace]])
- `load_data_duckdb()` — CSV to Parquet data staging
- `upload_files()` — CSV upload or PDF score extraction

Configuration is driven by [[concepts/yaml-configuration]] files, particularly `inst/config/ollama_models.yml` for model routing and `inst/quarto/templates/typst-report/config.yml` for report defaults.

In the broader Luria vision, cingulate appears as one important implementation layer within a larger clinical automation stack rather than the whole system by itself. The agentic-workflows document strengthens this interpretation by treating cingulate-style components as specialized workers in a larger orchestrated pipeline. The investor memo reinforces this distinction by describing a production-ready platform that includes workflow orchestration, clinician interaction, QA automation, local-first privacy controls, and enterprise deployment ambitions. See also [[concepts/cingulate-engine]], [[concepts/luria-skills]], [[summaries/README_luria]], and [[concepts/knowledge-base-architecture]].

## Related Patterns

- [[concepts/neuropsychological-assessment-pipeline]] — end-to-end workflow design
- [[concepts/neuropsychological-reporting]] — output standards and clinical formatting
- [[concepts/neuropsychological-report-variables]] — structured data fields used across the pipeline
- [[concepts/clinical-report-structure]] — how sections are organized in output documents
- [[concepts/agent-pipeline-state-management]] — managing state across pipeline stages
- [[concepts/multi-agent-orchestration]] — coordinating multiple LLM agents for synthesis
- [[concepts/phi-data-handling]] — privacy and compliance for patient health information
- [[concepts/r6-class-architecture]] — object-oriented design pattern used throughout
- [[concepts/local-llm-inference]] — running models locally for privacy and performance
- [[concepts/local-first-architecture]] — keeping sensitive clinical processing on-device or in controlled local environments
- [[concepts/clinical-ai-copilot]] — AI assistance embedded within clinician workflows
- [[concepts/clinical-narrative-generation]] — generating clinician-facing prose from structured findings
- [[concepts/healthcare-ai-regulation]] — compliance and regulatory context for deployment
- [[concepts/orchestrator-worker-pattern]] — top-level planning and delegation across workflow stages
- [[concepts/subagent-architecture]] — specialized workers for domain-specific or tool-specific tasks
- [[concepts/retrieval-augmented-generation]] — retrieval of relevant context before generation or synthesis
- [[concepts/r-python-integration]] — bridging Python orchestration with R-based neuropsych engines
- [[concepts/pii-redaction-pipelines]] — automated privacy protection during processing and reporting

## Challenges & Considerations

- **Clinical validity**: Automated narratives must accurately reflect score patterns without clinical hallucination; see [[concepts/validity-language]] and [[concepts/report-review-qa]]
- **Privacy**: Patient data requires careful handling under [[concepts/clinical-data-privacy]] and [[concepts/phi-data-handling]]
- **Customization**: Templates must adapt to referral context, patient age group, audience, and report purpose
- **Model selection**: Trade-offs between speed, accuracy, and local vs. cloud inference affect output quality; [[concepts/role-based-llm-routing]] and [[concepts/fallback-strategy]] address this
- **Cross-domain synthesis**: High-value neuropsychological reporting depends on integration across cognitive, behavioral, developmental, and contextual data rather than single-test summaries
- **Clinical voice**: Output must preserve the discipline-specific register expected in neuropsychological and forensic reporting
- **Caching**: Content-addressed caching via `ReportLLMBridgeR6` ensures idempotent re-runs
- **Quality assurance**: Production use requires explicit QA automation plus clinician review loops rather than one-pass text generation
- **Risk-gated autonomy**: Silent execution is appropriate for low-risk deterministic steps, but sensitive steps such as PII decisions and final report sign-off require review
- **Hybrid runtime complexity**: Python orchestration and R execution introduce data-passing, error-handling, and retry concerns across language boundaries
- **Regulatory readiness**: Scaling into healthcare systems may require security, auditability, and alignment with evolving [[concepts/healthcare-ai-regulation]] pathways
- **Enterprise deployment**: Moving from prototype to healthcare-system use introduces implementation, infrastructure, and support demands beyond the core report pipeline

## See Also

- [[summaries/agentic-workflows]] — agent-based framing for end-to-end data and report pipelines
- [[summaries/Apply-to-Y-Combinator-JWT]] — founder framing of Luria as a local-first, agent-based neuropsych workflow system
- [[summaries/Luria_AI_Q4_Investor_Memo_2026]] — Q4 2026 investor framing of platform traction, validation, and scale strategy
- [[summaries/LLM_AGENT_MAP]] — complete orchestration reference for the cingulate package
- [[summaries/README]] — cingulate package overview
- [[summaries/README_luria]] — Luria pipeline details
- [[summaries/neuropsych-narrative-writer]] — narrative generation component
- [[summaries/neuropsych-data-extractor]] — data extraction component
- [[summaries/report-generation]] — report generation workflows
- [[concepts/luria-neuropsych-pipeline]] — the Luria-specific implementation
- [[concepts/cingulate-engine]] — the cingulate core engine

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/copilot-instructions]]

See also: [[summaries/neurocog.prompt]]

See also: [[summaries/sirf_synthesis]]

See also: [[summaries/NP-20240415-001_report]]

See also: [[summaries/multi_patient_transcript]]

See also: [[summaries/pai_01]]

See also: [[summaries/redesign_20260623110910]]