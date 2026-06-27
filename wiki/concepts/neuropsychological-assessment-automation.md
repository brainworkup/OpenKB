---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/pai_01.md, summaries/multi_patient_transcript.md, summaries/NP-20240415-001_report.md, summaries/sirf_synthesis.md, summaries/neurocog.prompt.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/README.md]
brief: AI-driven automation of end-to-end neuropsych assessment and reporting.
---

# Neuropsychological Assessment Automation

Neuropsychological assessment automation refers to the use of software pipelines, structured data staging, privacy-conscious AI systems, and AI-assisted narrative generation to transform raw cognitive and behavioral test data into complete clinical reports while reducing manual effort and preserving clinical accuracy, privacy, and workflow fidelity. In the Luria context, this automation is not limited to score interpretation alone; it is framed as support for nearly the entire neuropsychological evaluation workflow, from data collection and organization through synthesis and report drafting. The Q4 2026 investor framing extends this further: Luria presents automation as an end-to-end clinical platform for intake, scoring, interpretation, quality assurance, and reporting, with clinician time savings positioned as a direct response to workforce shortage and burnout. See also [[concepts/luria-overview]] and [[concepts/luria-neuropsych-pipeline]].

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

This is especially important in pediatric and forensic neuropsychology, where evaluations can consume many hours and the final report is the central clinical deliverable. In the Luria investor memo, the value proposition is quantified as substantial per-case time reduction while maintaining clinician-validated quality, making automation not just a convenience but an operational solution to healthcare capacity constraints. Related pages include [[concepts/neuropsychological-assessment-workflow]], [[concepts/neuropsychological-synthesis]], [[concepts/clinical-data-privacy]], [[concepts/forensic-neuropsychological-evaluation]], and [[concepts/healthcare-workforce-automation]].

## Core Pipeline Stages

### 1. Data Ingestion & Staging
Raw test scores and source materials are ingested and staged using [[concepts/duckdb-data-staging]], converting data through CSV → DuckDB → Parquet for high-performance querying. The `load_data_duckdb()` function reads CSVs, computes z-scores, and splits data into `neurocog`, `neurobehav`, and `validity` tables written in multiple formats such as Parquet, Feather, and CSV. This provides a structured, queryable backbone for downstream processing.

In the broader Luria framing, ingestion extends beyond spreadsheets to PDFs and other file types, reflecting the need to consolidate heterogeneous clinical inputs into one patient workspace. The investor memo also implies a broader intake layer spanning end-to-end case intake before scoring and interpretation begin, reinforcing automation as a full workflow rather than a report-writing utility. See also [[concepts/long-format-clinical-data]], [[concepts/neuropsychological-test-scores]], [[concepts/pdf-data-extraction]], [[concepts/pdf-score-extraction]], [[concepts/clinical-pdf-assessment]], and [[concepts/report-ingestion-pipeline]].

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

The central clinical goal is not just isolated score description but integrated interpretation across neurocognitive and neurobehavioral domains, which is one of the key differentiators emphasized in both the Luria materials and the investor memo. Q4 2026 framing adds emphasis on domain-specific reasoning chains, clinical validity checks, and differential support across multiple neuropsychological domains rather than generic summarization. Related pages include [[concepts/neuropsychological-score-interpretation]], [[concepts/neurobehavioral-status-exam]], and [[concepts/neuropsychological-synthesis]].

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

This role abstraction is an instance of [[concepts/llm-provider-abstraction]], supporting both Ollama and MLX backends via configurable endpoint URLs. Within Luria, local routing is not merely a deployment preference but a privacy requirement for PHI-sensitive workflows. The investor memo adds that the production strategy is multi-model rather than single-model, combining local and cloud LLMs for quality/cost optimization while maintaining a privacy-preserving architecture. That makes this concept closely tied to [[concepts/local-first-architecture]], [[concepts/local-llm-inference]], [[concepts/privacy-first-software]], [[concepts/phi-data-handling]], [[concepts/concurrent-model-serving]], and [[concepts/human-in-the-loop-clinical-ai]].

### 4. Prompt Building
The `DomainPrompterR6` class builds structured OpenAI-style message lists from neuropsychological test score data. It normalizes score fields, generates style-specific system instructions such as `clinical`, `parent`, or `school`, and formats scores as `Label | SS X | Yth percentile | Range` lines. This structured approach to [[concepts/narrative-report-generation]] ensures prompt consistency across domains.

Prompt construction in this setting also has to preserve clinical voice, audience-appropriate tone, and discipline-specific reasoning norms. The investor memo implicitly reinforces this by stressing structured clinical reasoning, diagnostic accuracy, and preservation of clinical nuance as core product claims rather than cosmetic output goals. That makes this stage closely related to [[concepts/clinical-communication-register]], [[concepts/clinical-ai-reasoning]], [[concepts/neuropsychological-prompt-engineering]], and [[concepts/neuropsychological-prompt-configuration]].

### 5. Orchestrated Stage Execution with Artifacts
The `ReportLLMBridgeR6` class orchestrates pipeline stages with full artifact management and content-addressed caching. Each stage produces:
- A narrative `.md` file
- A metadata `.meta.json` file containing model, timing, and QC information
- An append-only audit `.jsonl` log
- A content-addressed cache entry (`.cache/<hash>.rds`)

This caching strategy means re-runs are idempotent: existing outputs survive unchanged unless inputs change. In the broader platform framing, orchestration also includes real-time clinician feedback loops and quality assurance automation, showing that artifact management is part of a larger reviewable clinical production system rather than a pure generation pipeline. See [[concepts/artifact-caching-pipeline]], [[concepts/agent-pipeline-state-management]], [[concepts/decision-logging]], and [[concepts/report-review-qa]].

### 6. QMD Injection & Report Assembly
The `NeuropsychResultsR6` class connects processed domain data directly to [[concepts/quarto]] report files. It writes a `LLM_CONTEXT_START…END` block into the target `.qmd` file, runs narrative generation, and injects a `<summary>…</summary>` tag back into the same file. Each domain has a corresponding prompt keyword such as `promem` → `_02-05_memory_text.qmd` and prompt template file (`inst/prompts/pro*.qmd`), supporting a [[concepts/modular-report-architecture]] via Quarto includes.

The surrounding report stack also reflects a gradual evolution from R Markdown and LaTeX toward Quarto and Typst, making report generation both programmable and style-aware. In the larger automation concept, this stage is where structured findings become clinician-facing deliverables with consistent documentation quality across cases. See also [[concepts/quarto-extensions]], [[concepts/typst-typesetting]], [[concepts/typst-modules]], [[concepts/clinical-report-structure]], and [[concepts/clinical-narrative-generation]].

### 7. Parallel Orchestration
Batch LLM calls can be parallelized across CPU cores using `run_llm_for_all_domains_parallel()`, and the full LLM-plus-render pipeline can be triggered with `cingulate_run_llm_then_render()`. This enables efficient processing of all major cognitive domains concurrently.

At a higher architectural level, the Luria materials describe the system as a coordinated team of agents and subagents, which connects this pipeline approach to [[concepts/multi-agent-orchestration]], [[concepts/subagent-architecture]], [[concepts/langgraph-agent-workflows]], and [[concepts/persistent-memory]]. The investor memo adds a more operational scaling claim: automation benefits should scale roughly with case volume, making parallel orchestration relevant not only for engineering efficiency but also for healthcare deployment readiness.

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

In the broader Luria vision, cingulate appears as one important implementation layer within a larger clinical automation stack rather than the whole system by itself. The investor memo reinforces this distinction by describing a production-ready platform that includes workflow orchestration, clinician interaction, QA automation, local-first privacy controls, and enterprise deployment ambitions. See also [[concepts/cingulate-engine]], [[concepts/luria-skills]], [[summaries/README_luria]], and [[concepts/knowledge-base-architecture]].

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
- [[concepts/human-in-the-loop-clinical-ai]] — clinician oversight, revision, and feedback in the generation loop
- [[concepts/healthcare-ai-regulation]] — compliance and regulatory context for deployment

## Challenges & Considerations

- **Clinical validity**: Automated narratives must accurately reflect score patterns without clinical hallucination; see [[concepts/validity-language]] and [[concepts/report-review-qa]]
- **Privacy**: Patient data requires careful handling under [[concepts/clinical-data-privacy]] and [[concepts/phi-data-handling]]
- **Customization**: Templates must adapt to referral context, patient age group, audience, and report purpose
- **Model selection**: Trade-offs between speed, accuracy, and local vs. cloud inference affect output quality; [[concepts/role-based-llm-routing]] and [[concepts/fallback-strategy]] address this
- **Cross-domain synthesis**: High-value neuropsychological reporting depends on integration across cognitive, behavioral, developmental, and contextual data rather than single-test summaries
- **Clinical voice**: Output must preserve the discipline-specific register expected in neuropsychological and forensic reporting
- **Caching**: Content-addressed caching via `ReportLLMBridgeR6` ensures idempotent re-runs
- **Quality assurance**: Production use requires explicit QA automation plus clinician review loops rather than one-pass text generation
- **Regulatory readiness**: Scaling into healthcare systems may require security, auditability, and alignment with evolving [[concepts/healthcare-ai-regulation]] pathways
- **Enterprise deployment**: Moving from prototype to healthcare-system use introduces implementation, infrastructure, and support demands beyond the core report pipeline

## See Also

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