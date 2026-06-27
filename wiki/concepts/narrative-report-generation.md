---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/sirf_synthesis.md, summaries/nt_interpretation.md, summaries/nse_narrative.md, summaries/neurocog.prompt.md, summaries/neurobehav.prompt.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/agent-team.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/report-rendering-pipeline.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/report-template.md, summaries/report-generator.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0004-soul-style-profile-json.md, summaries/multi_patient_transcript.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/README.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/clinical-validity-reviewer.md, summaries/responses_to_claude.md, summaries/SKILL.md]
brief: Automated generation of clinician-ready neuropsychological narrative reports.
---

# Narrative Report Generation

Narrative report generation is the automated or semi-automated process of transforming raw neuropsychological assessment data, clinical observations, and test scores into a structured, clinician-ready written report. In the context of forensic and clinical neuropsychology, this process spans multiple output formats, pipeline stages, customization strategies, and pre-delivery review checks. Within the Luria ecosystem, it is also framed as the core mechanism for scaling a traditionally labor-intensive clinical workflow while preserving clinical voice, cross-domain integration, and strict local handling of sensitive data.

## Core Concepts

A narrative report is the primary deliverable of a [[concepts/neuropsychological-assessment-pipeline]]. It synthesizes quantitative test performance with clinical history to produce coherent, domain-organized prose that serves both legal and clinical audiences — a challenge addressed by [[concepts/dual-audience-design]].

The narrative is typically organized around [[concepts/cognitive-domains]] (e.g., memory, attention, executive functioning, language, visuospatial ability) and concludes with diagnostic impressions, forensic opinion, and recommendations aligned to forensic neuropsychological evaluation standards.

In the Luria context, narrative generation is not treated as a cosmetic final step but as the central product surface of the workflow: the report is the clinical deliverable given to patients, families, attorneys, schools, and other providers. The YC application for Luria emphasizes that much of the time burden in neuropsychological practice lies not only in testing but in integrating test batteries, interviews, observations, and behavioral data into a coherent written synthesis. This makes narrative generation the bottleneck that determines whether neuropsychological assessment can scale without sacrificing rigor. See [[summaries/Apply-to-Y-Combinator-JWT]] and [[concepts/luria-overview]].

## Domain-Level Prompt Templates

A foundational design pattern in narrative report generation is the **domain-level prompt template** — a structured instruction that configures an LLM to produce a 2–4 paragraph clinical interpretation for a single cognitive domain. The `nt_interpretation` template (see [[summaries/nt_interpretation]]) exemplifies this pattern:

- The LLM is cast as a **licensed neuropsychologist** writing an interpretive narrative for a specific `{domain}`.
- A `{score_lines}` variable injects the structured score summary (subtest names, scores, percentiles, ranges) for that domain.
- The output must open with a **domain-level summary sentence** stating overall functioning level (e.g., "Cognitive abilities in the domain of X fell in the Y range.").
- Subsequent paragraphs describe the **pattern of scores** at the subtest/scale level, noting intra-domain strengths and weaknesses.
- **Clinically significant scores** — those at or below the 5th percentile or at or above the 95th percentile — must be explicitly highlighted.
- Style constraints: third-person past tense, clinical but accessible language, flowing paragraphs (no bullet points).

This template is a concrete instance of [[concepts/neuropsychological-prompt-engineering]] and feeds directly into the broader [[concepts/clinical-narrative-generation]] workflow. It maps onto the `summarize_domain` and `detailed_interpretation` roles in the cingulate LLM stack's `OllamaModelRouterR6`, and corresponds to the per-domain prompt files in `inst/prompts/` (e.g., `proiq`, `promem`, `proexe`). See also [[summaries/neurocog.prompt]] for a related domain-level prompt configuration.

The percentile thresholds specified in the template (5th and 95th percentile) align with standard [[concepts/neuropsychological-score-interpretation]] conventions and are enforced consistently across all domain narrative tracks.

## Pipeline Tracks

Narrative report generation occurs within multi-stage pipelines. Four primary tracks exist in the broader ecosystem:

### RAG-Based Draft Generation (Soul/Style Agent)

The `write-report` command in `soul/neuro_report_style_agent.py` (see [[summaries/report-generator]]) implements a retrieval-augmented generation approach for producing new report drafts. It is the final stage of a three-step pipeline:

1. **build-index** — Historical neuropsychological PDF reports are indexed into a SQLite vector store ([[concepts/sqlite-as-vector-store]]).
2. **train-style** — A style profile JSON is extracted from the index, capturing the clinician's writing patterns. See [[concepts/style-profile-extraction]].
3. **write-report** — A user prompt is embedded, top-k relevant historical chunks are retrieved, and an LLM generates a polished draft section constrained by the style profile.

The generation prompt structure enforces three layers: a **Style Profile** (JSON rules), a **User Task** (the drafting instruction), and **Retrieval Context** (historical chunks with source attribution). The LLM is instructed never to fabricate patient data, to mark missing information as `[NEEDS DATA]`, and to frame all output as a draft for clinician review. Default temperature is 0.2 for consistency.

Key CLI parameters include `--db-path`, `--profile-path`, `--prompt`, `--top-k` (default 6), `--temperature` (default 0.2), and `--output`. Output goes to a file or stdout.

Effective prompts specify section type (summary, test interpretation, recommendations), include patient demographics and presenting concerns, and state any constraints on length or tone. Example:
- *"Interpret WISC-V results for a teenager: VCI 115, PRI 108, WMI 92, PSI 85. Maintain objective tone and avoid definitive diagnostic statements."*

Post-generation review is always required: clinicians must verify no hallucinated scores, confirm patient identifiers, ensure recommendation appropriateness, and add their signature/disclaimer.

### Luria Streamlit App Track (LangGraph)

The [[summaries/README_luria]] documents a four-stage [[concepts/langgraph-agent-workflows]] ingest pipeline:

1. **Parse** — Docling ([[concepts/docling-pdf-parsing]]) extracts text and layout; PHI is redacted locally before any network call.
2. **Extract** — Claude Sonnet structures the narrative into JSON (test scores, clinical summaries).
3. **Index** — SQLite + [[concepts/lancedb-vector-store]] store structured and vector data locally.
4. **Report** — Generates a markdown narrative report; optional Typst or Quarto rendering for print-ready PDFs.

This track is designed for local-first, HIPAA-conscious deployment. The `build_ingest_graph()` function in `neuropsych_agent/graph.py` orchestrates the full pipeline. The `Narrative_Report_Generator` subagent handles the prose generation step, with output configurable as markdown, Typst, or Quarto-rendered PDF. A parallel `neuropsych_rag/` library provides a standalone RAG service reusable outside the Streamlit UI, supporting [[concepts/retrieval-augmented-generation]] Q&A over ingested reports.

The YC application adds important product framing for this track: Luria is explicitly described as a local-first, agent-based system intended to automate the neuropsychological workflow "from soup to nuts," with report generation as the culminating synthesis step. Its differentiator is not merely drafting text, but integrating across neurocognitive and neurobehavioral domains while keeping PHI local. That framing closely aligns this track with [[concepts/local-first-architecture]], [[concepts/privacy-first-software]], [[concepts/clinical-data-privacy]], and [[concepts/multi-agent-orchestration]]. See [[summaries/Apply-to-Y-Combinator-JWT]].

### Quarto + Typst Track (Cingulate Pipeline)

The `neuropsych-narrative-writer` agent (see [[summaries/neuropsych-narrative-writer]]) produces **per-domain prose narratives** as Quarto include files (`.qmd`) that drop into the cingulate template structure. Within the [[concepts/cingulate-engine]], the full pipeline runs:

```text
CSV (data-raw/csv/) → DuckDB/Parquet → R6 domain processors → per-domain QMD includes → Quarto/Typst → PDF
```

The **cingulate** R package (developed by Joey Trampush at the Brain Workup Lab) is the central orchestrator of this track. It combines DuckDB-backed data staging, an [[concepts/r6-class-architecture]]-based design with modular domain processors, local LLM integration via Ollama, and publication-quality output via GT tables, dot-plot visualizations, and Quarto/Typst report templates. The package supports adult, pediatric, and forensic neuropsychology report formats.

This track also reflects the founder's documented technical evolution: narrative generation grew out of years of report-writing automation work using R, RMarkdown, LaTeX, [[concepts/quarto]], and now [[concepts/typst-typesetting]], before being augmented with AI coding tools and LLM-driven prose synthesis. In that sense, the cingulate pipeline is not a standalone text generator but the mature expression of a longer effort in clinical workflow automation and reporting infrastructure. See [[summaries/Apply-to-Y-Combinator-JWT]] and [[concepts/r-neuropsych-packages]].

#### LLM Stack Architecture

The LLM Agent Map (`LLM_AGENT_MAP`) provides detailed orchestration guidance for this track. The LLM stack consists of four coordinated R6 components:

- **`OllamaModelRouterR6`** — Model gateway configured via `inst/config/ollama_models.yml`, mapping clinical *roles* to model IDs and fallbacks. Supports both Ollama and MLX endpoints. See [[concepts/role-based-llm-routing]] and [[concepts/llm-provider-abstraction]].
- **`DomainPrompterR6`** — Builds structured OpenAI-style message lists from test score data, supporting `clinical`, `parent`, and `school` output styles. Normalizes score fields and formats data as `Label | SS X | Yth percentile | Range` lines.
- **`ReportLLMBridgeR6`** — Orchestration layer that runs pipeline stages, writes artifacts (`.md`, `.meta.json`, `.jsonl`), and manages content-addressed caching ([[concepts/artifact-caching-pipeline]]).
- **`NeuropsychResultsR6`** — Connects processed domain data to Quarto (`.qmd`) report files, injecting LLM-generated summaries using `LLM_CONTEXT_START…END` markers.

#### Available LLM Roles

The router supports 13 clinical roles:

| Role | Purpose |
|---|---|
| `summarize_domain` | Summarise a single cognitive domain |
| `interpret_rating_scales` | Interpret behavioural rating scale data |
| `quick_interpret` | Short 1–2 sentence interpretation |
| `integrate_evaluation` | Cross-domain clinical impression |
| `generate_recommendations` | Evidence-based recommendations |
| `differential_diagnosis` | DSM-5 differential considerations |
| `overall_summary` | Executive summary |
| `detailed_interpretation` | Long-form narrative |
| `embeddings` | Generate text embeddings |
| `finalize_diagnostic_summary` | Final diagnostic paragraph |
| `quick_summary` | Brief parent/school summary |
| `generate_narrative` | Full prose narrative |
| `fallback_general` | Generic fallback for any prompt |

Specialized models include MedGemma 4B for domain-level summaries and Luria/Qwen 27B for cross-domain synthesis. `mode = "development"` is used for iteration vs. `production` for final reports. LLM model availability is verified with `check_available_models()` before any workflow that hits Ollama.

The `summarize_domain` and `detailed_interpretation` roles map directly to the `nt_interpretation` prompt template pattern — both expect a domain label and structured score lines as input, and produce 2–4 paragraphs of clinical prose following the same structural rules (domain summary sentence, pattern description, clinically significant score callouts).

#### Domain Processor and Factory Pattern

Domain processors (`DomainProcessorR6`), narrative generators (`NeuropsychResultsR6`), and rendering components (`TableGTR6`, `DotplotR6`) are all R6 classes wired together by the `WorkflowRunnerR6` orchestrator. The `DomainProcessorFactoryR6` centralizes domain processor creation with validation and supports multi-rater inputs for ADHD and emotion domains. See [[concepts/domain-processor-pattern]].

```r
# Batch all domains
all_processors <- factory$batch_create(
  domain_keys         = c("iq", "memory", "executive", "adhd", "emotion"),
  age_group           = "child",
  include_multi_rater = TRUE
)
```

#### Prompt-to-QMD Mapping

Each domain has a corresponding prompt keyword, a prompt template file in `inst/prompts/`, and a target QMD include file:

| Keyword | Target QMD |
|---|---|
| `proiq` | `_02-01_iq_text.qmd` |
| `proacad` | `_02-02_academics_text.qmd` |
| `proverb` | `_02-03_verbal_text.qmd` |
| `provis` | `_02-04_spatial_text.qmd` |
| `promem` | `_02-05_memory_text.qmd` |
| `proexe` | `_02-06_executive_text.qmd` |
| `promot` | `_02-07_motor_text.qmd` |
| `prosoc` | `_02-08_social_text.qmd` |
| `proadhd` | `_02-09_adhd_text.qmd` |
| `proemo` | `_02-10_emotion_text.qmd` |
| `proadapt` | `_02-11_adaptive_text.qmd` |
| `prodl` | daily living text |
| `provalid` | validity section |
| `pronse` | `_01-00_nse.qmd` (behavioral obs) |
| `prosirf` | `_03-00_sirf.qmd` (integrated summary) |
| `prorecs` | `_03-01_recs.qmd` (recommendations) |
| `progen` | general / summary |

The `proadhd` and `proemo` keywords correspond to the behavioral rating scale domains, which require special multi-rater handling (see below). Prefixes are **never renumbered** — `_quarto.yml` and `template.qmd` depend on this stable ordering.

#### Parallel Execution

Batch LLM calls can be parallelized across CPU cores:

```r
# Full LLM + render in one call
cingulate_run_llm_then_render(
  render_paths = "template.qmd",
  parallel     = TRUE,
  n_cores      = 6
)

# LLM generation only (no render)
run_llm_for_all_domains_parallel(
  prompts_dir = "inst/prompts",
  domains     = c("proiq", "promem", "proexe", "proadhd"),
  backend     = "ollama",
  temperature = 0.2,
  n_cores     = 4
)
```

#### High-Level API

The high-level API exposes three primary entry points:

```r
create_patient_workspace("ExamplePatient", age = 12)
results <- process_all_domains("data", age_group = "child")
generate_assessment_report(
  results,
  patient_info = list(name = "ExamplePatient", age = 12)
)
```

The complete orchestration entry points also include `cingulate_workflow()` for full workflow execution, `cingulate_quick_start()` for new-user onboarding, and `upload_files()` for CSV or PDF ingestion.

#### Domain Narrative File Naming

Each domain narrative file is named with a stable prefix:

| Prefix | Domain |
|---|---|
| `_02-01_iq` | General Cognitive Ability / IQ |
| `_02-02_academics` | Academic / Achievement |
| `_02-03_verbal` | Verbal / Language |
| `_02-04_spatial` | Visuospatial / Visual-Construction |
| `_02-05_memory` | Memory & Learning |
| `_02-06_executive` | Executive Function |
| `_02-07_motor` | Sensorimotor |
| `_02-08_social` | Social Cognition |
| `_02-09_adhd` | ADHD (multi-rater variants) |
| `_02-10_emotion` | Emotion/Behavior (multi-rater variants) |
| `_02-11_adaptive` | Adaptive Functioning |
| `_03-00_sirf` | Summary, Impressions, Recommendations & Formulation |
| `_03-01_recs` | Recommendations |

The `_quarto.yml` renders `template.qmd` using the `neurotyp-pediatric-typst` format from `inst/quarto/_extensions/brainworkup/`; adult and forensic variants live alongside under `inst/quarto/`. The R6 rendering layer handles score tables, plots, and final Typst PDF assembly. The narrative writer's job is **prose only**.

The system uses a **two-stage rendering with edit protection** pattern: regenerating QMDs may overwrite manual edits, so the pipeline checks whether a target file has been hand-edited before regenerating.

### Legacy / Alternative Track (Google Docs)

In some workflow configurations, a `Narrative_Report_Generator` produces a structured prose document in Google Docs format as the primary clinical artifact, written in a [[concepts/clinical-communication-register]] appropriate for medicolegal review. A parallel `typst-report-formatter` skill then produces a `.typ` source file using the `forensic_report.typ` canonical template.

This dual-output approach reflects a [[concepts/modular-report-architecture]] in which format concerns are separated from content generation.

## Behavioral Rating Scale Interpretation

A specialized prompt template, `neurobehav`, is dedicated to the interpretation of behavioral rating scale data — one of the most nuanced domains in neuropsychological report generation. This prompt configures the LLM as a board-certified clinical neuropsychologist with specific expertise in psychodiagnostic, psychiatric, and personality assessment, including neurodevelopment, ADHD, executive functioning, and autism spectrum disorder evaluation. See [[summaries/neurobehav.prompt]] and [[concepts/behavioral-rating-scales]].

The `neurobehav` prompt covers four assessment domains sourced from multi-informant rating scales (child self-report, parent ratings, teacher ratings, and clinician ratings):

1. **ADHD / Executive Functioning** — Attention regulation, inhibition, working memory, cognitive flexibility; maps to the `proadhd` and `proexe` pipeline keywords.
2. **Social Cognition** — Theory of mind, social awareness, social communication; maps to `prosoc`.
3. **Emotional, Behavioral, and Personality Functioning** — Mood, affect, behavioral patterns, personality traits; maps to `proemo`.
4. **Adaptive Functioning** — Daily living skills, independence, functional competence; maps to `proadapt`.

The generated output must adhere to specific stylistic requirements that align with the broader narrative generation standards:

- **Tense**: Past tense throughout
- **Voice**: Third-person perspective
- **Register**: Mixed technical and accessible language, leaning professional — interpretable by both clinicians and non-specialists; this reflects the broader principle of [[concepts/dual-audience-design]]
- **Format**: Continuous paragraph-form prose (not bullet lists)
- **Completeness**: Sophisticated, integrated, and clinically complete
- **Pronouns**: Patient-specific pronouns used when available

These same style constraints are shared with the `nt_interpretation` domain-level template, confirming a consistent house style across all domain narrative prompt templates in the ecosystem.

The model parameters for this prompt template reflect a preference for consistent, clinically precise language: temperature 0.4, presence penalty 0.5, frequency penalty 0.5, and Mirostat 0.8 — somewhat more expressive than the default 0.2 used elsewhere in the pipeline, balancing precision with natural prose variation. The stop sequence `--END--` enforces clean output boundaries.

In the cingulate LLM stack, the `interpret_rating_scales` role in the `OllamaModelRouterR6` corresponds directly to this prompt's function. The `DomainPrompterR6` component builds structured message lists for this role, supporting `clinical`, `parent`, and `school` output styles to match the multi-informant nature of behavioral rating data. [[concepts/executive-function-deficits]] and [[concepts/working-memory]] are among the constructs most commonly interpreted through this prompt.

The YC application's emphasis on integration across neurocognitive and neurobehavioral domains is especially relevant here. The founder explicitly contrasts Luria with incumbent test-maker reports that remain siloed by instrument, arguing that the defining skill in neuropsychology is cross-domain synthesis. Behavioral rating scale interpretation therefore matters not just as a separate section, but as a key bridge between formal testing, observed behavior, and final integrated impressions. See [[summaries/Apply-to-Y-Combinator-JWT]].

## Customization

The Soul/Style Agent RAG track supports extensive customization for different clinicians, populations, and output requirements. See [[summaries/customization]] for the full reference.

### Per-Clinician and Per-Population Style Profiles

Separate [[concepts/style-profiles]] can be trained for individual clinicians or patient populations (pediatric, adult, forensic) by building an index from a targeted reports directory and running `train-style` to produce a dedicated `.json` profile. This allows the generation step to mirror each clinician's distinctive voice and each population's register requirements.

Profiles can also be **manually edited** for fine-grained control. Key configurable fields include:
- `voice` — narrative persona.
- `tone` — affective register (e.g., "Objective yet empathetic, strength-based").
- `structure_patterns` — ordered structural conventions.
- `do_rules` — enforced inclusions (e.g., person-first language, percentile ranks, test version citations).
- `avoid_rules` — enforced exclusions (e.g., definitive diagnostic statements, casual contractions, omission of confidence intervals).

This customization layer is central to preserving what the YC application calls the non-negotiable "clinical voice." In practice, narrative generation is useful only if it sounds like a credible neuropsychologist rather than a generic assistant. The style-profile mechanism operationalizes that requirement and helps distinguish clinician-authored, specialty-specific narrative generation from commodity cloud AI drafting. See [[summaries/Apply-to-Y-Combinator-JWT]].

### Chunking Strategy

Chunk size affects retrieval quality (see [[concepts/rag-chunking]] and [[concepts/text-chunking]]):

| Use Case | `--chunk-size` | `--overlap` |
|---|---|---|
| Long-form comprehensive reports | 2000 | 200 |
| Brief screening reports | 600 | 75 |
| Default | ~1000 | ~100 |

Section-aware chunking can be implemented by pre-extracting named sections before indexing and attaching section metadata for more targeted retrieval.

### Retrieval and Generation Parameters

Retrieval behavior is tuned via `--top-k` (number of retrieved chunks) and `--temperature`:

| Mode | `--top-k` | `--temperature` |
|---|---|---|
| High-context (complex cases) | 12 | 0.15 |
| Quick drafting (standard cases) | 4 | 0.25 |
| Balanced (default) | 6 | 0.2 |

### Model Configuration

Embedding and generation models are configurable via `--omlx-embed-model` and `--omlx-gen-model`. Alternative local inference backends (e.g., Ollama) can be substituted by setting the `OMLX_URL` environment variable. See [[concepts/omlx-server]] and [[concepts/local-llm-inference]]. In the cingulate pipeline, the YAML model configuration (`inst/config/ollama_models.yml`) governs all role-to-model assignments. See [[concepts/yaml-configuration]].

### Section-Level and Multi-Section Generation

Template-based prompts enable generation of targeted sections in isolation — executive summaries, background/history, test results tables, and domain-specific recommendations. Each section can be generated independently and concatenated into a full report, supporting modular review and editing workflows. The `neurobehav` prompt and the `nt_interpretation` template both exemplify this pattern: each is scoped narrowly to a specific domain or domain class, designed to be composed with other domain-specific prompts into a complete evaluation report.

### A/B Profile Testing and Quality Assurance

Two profiles can be compared by generating parallel drafts and running `diff`, supporting iterative profile refinement. Automated post-generation shell checks scan drafts for unfilled `[NEEDS DATA]` markers, word count, and prohibited casual contractions.

## Narrative Writer Workflow

The `neuropsych-narrative-writer` agent follows a structured workflow for each domain:

1. **Read** the extractor's CSV (via `Read` or `Glob` for multiple files).
2. For each subdomain present in the data, draft **2–4 short paragraphs** covering:
   - **Performance summary** — what was tested; qualitative range drawn verbatim from the `range` column (never invented).
   - **Pattern interpretation** — relative strengths/weaknesses, intra-test scatter, score-type discrepancies when `score_type` mixes scaled/T/standard scores.
   - **Functional implication** — one hedged sentence linking to everyday, academic, or occupational impact.
3. Write each domain file to the patient workspace at the orchestrator-specified path.
4. **Multi-rater domains** (ADHD, Emotion/Behavior): produce one file per rater present in the data (e.g., `_02-09_adhd_self_text.qmd`, `_02-09_adhd_parent_text.qmd`). Always call `check_rater_data_exists()` / `check_domain_raters()` before generating these — child vs adult reports diverge here. If a rater has no data, skip that file — never fabricate.
5. **Edit-protection check** ([[concepts/edit-protection-pattern]]): before overwriting any existing `_text.qmd`, read it first. If clinician hand-edits are detected, append the new draft as a `<!-- DRAFT: ... -->` comment block and report a warning rather than overwriting.

This paragraph-level structure (performance summary → pattern interpretation → functional implication) directly parallels the `nt_interpretation` template's three-part output requirement (domain-level summary → subtest pattern → clinically significant score callout), ensuring consistency whether the narrative is produced by the cingulate pipeline or a standalone prompt invocation.

For behavioral rating scale domains specifically, the `neurobehav` prompt template guides prose generation. The multi-informant nature of rating scales (self, parent, teacher) is reflected in the multi-rater file pattern: each informant's perspective may yield a separate narrative file, which the `DomainProcessorFactoryR6` coordinates via `include_multi_rater = TRUE`.

Score-type handling is also important: footnotes and z-score conversions branch on whether a row is `t_score`, `scaled_score`, or `standard_score`. The `ScoreTypeCacheR6` caches this detection; when adding a new test, its score type must be recognized by `R/score_type_utils.R` and `R/neuropsych_test_scoring.R`.

This workflow exists because neuropsychological reports often require hours of manual integration even in straightforward cases, and far longer in pediatric and forensic evaluations. The YC application situates narrative generation as the point where accumulated structured data becomes a clinically useful product, making reliable prose generation the practical endpoint of the broader automation pipeline. See [[concepts/forensic-neuropsychological-evaluation]] and [[summaries/Apply-to-Y-Combinator-JWT]].

## End-to-End Agent Script Pattern

The LLM Agent Map documents a canonical end-to-end script that illustrates how all components connect:

```r
library(cingulate)

# 1. Load data
load_data_duckdb("data-raw/csv", output_dir = "data", output_format = "all")

# 2. Initialize LLM stack
router   <- OllamaModelRouterR6$new("inst/config/ollama_models.yml")
bridge   <- ReportLLMBridgeR6$new(router, out_dir = "artifacts/patient_001", seed = 42)
prompter <- DomainPrompterR6$new(style = "clinical")

# 3. Process and generate each domain
factory     <- DomainProcessorFactoryR6$new()
domain_keys <- c("iq", "memory", "executive", "adhd", "emotion")

for (dk in domain_keys) {
  proc <- factory$create_processor(dk, age_group = "child")
  proc$load_data()
  proc$filter_by_domain()
  proc$select_columns()

  msgs <- prompter$build_messages(
    domain = dk,
    data   = as.list(proc$data),
    task   = "summarize_domain"
  )

  bridge$run_stage("summarize_domain", sprintf("01_%s", dk), messages = msgs)
}

# 4. Integration + recommendations
bridge$run_stage("integrate_evaluation",    "04_integrated",
  prompt = "Integrate all domain findings into a unified clinical impression.")
bridge$run_stage("generate_recommendations", "05_recs",
  prompt = "Generate evidence-based recommendations based on the evaluation.")

# 5. Render report
cingulate_run_llm_then_render(render_paths = "template.qmd", parallel = FALSE)
```

Each stage produces a narrative `.md`, metadata `.meta.json`, and appends to an audit `.jsonl` log — constituting a complete artifact trail.

## Voice and Style

The narrative writer adapts its register to the assessment context:

- Professional, APA-style neuropsychological report prose.
- Hedged language throughout: *"performance is consistent with…"*, *"results suggest…"*, *"indicates relative weakness in…"*. No bare assertions of pathology.
- Register adapts to the `age_group` column: pediatric language for child reports, adult language for adult reports, forensic register for forensic evaluations.
- Can mirror a `STYLE_PROFILE` or `EXEMPLAR_SNIPPETS` block if provided, but **only the extractor's CSV is the evidence base** — no patient-specific facts are imported from exemplars.
- Markdown only; no raw HTML; fully Quarto-compatible.
- Patient pronouns are used when available, consistent with contemporary clinical communication standards.

In the RAG-based Soul/Style Agent track, style is enforced via a loaded JSON style profile applied directly in the generation prompt. In the Luria Streamlit App track, style and voice can be further shaped by optional Luria Voice integration (BRAND, SOUL, STYLE layers), injecting de-identified exemplar prose from the clinician's own prior reports. See [[concepts/style-profile-extraction]] for how style profiles are derived and applied.

The [[concepts/clinical-communication-register]] is particularly important for behavioral rating scale sections, which must be interpretable by parents and teachers as well as referring clinicians. The `neurobehav` prompt explicitly targets this mixed-register requirement, leaning professional while remaining accessible. The `nt_interpretation` template shares this goal through its instruction to use "clinical but accessible language."

The YC application reinforces that style is not a superficial preference but part of the product's defensibility. The founder argues that generic AI systems fail because they do not preserve the discipline-specific clinical voice expected in neuropsychological evaluations. Narrative report generation therefore includes voice control, hedging discipline, and domain-sensitive framing as first-class requirements rather than optional polish. See [[summaries/Apply-to-Y-Combinator-JWT]].

## Safety Guardrails

Across all tracks, shared safety rules govern generation:

- **No fabrication**: LLMs are explicitly instructed not to invent patient data, test scores, or clinical history.
- **Missing data handling**: The `[NEEDS DATA]` placeholder (RAG track) or skipped file (narrative writer track) makes data gaps explicit rather than masking them.
- **Style adherence**: Generation is constrained by a loaded profile rather than unconstrained open generation.
- **Clinician-in-the-loop**: All output is framed as a draft requiring clinical review before sign-out.
- **PHI protection**: Assumes upstream scrubbing; real names are replaced with `[PATIENT]` and a WARNING is emitted. See [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]].

The YC application highlights why these guardrails are architectural, not merely procedural: neuropsychological reports contain sensitive patient information, and the founder positions local-first deployment as necessary because generic cloud AI tools cannot meet the required PHI boundaries. This reinforces the tight coupling between narrative generation, [[concepts/clinical-data-privacy]], and [[concepts/privacy-first-software]]. See [[summaries/Apply-to-Y-Combinator-JWT]].

## Pre-Delivery Review (QA)

Before sign-out, a dedicated read-only parallel agent — the `clinical-validity-reviewer` (see [[summaries/clinical-validity-reviewer]]) — reviews the draft narrative files alongside the extracted score CSV. This is a core component of [[concepts/report-review-qa]] and runs in parallel with the narrative-writer so the main thread can keep working.

The reviewer evaluates drafts across six axes:

1. **Completeness** — Every CSV subdomain has a corresponding narrative file; multi-rater domains cover all raters; the narrative addresses the referral question.
2. **Validity language** — Effort/validity test findings require appropriate hedging; prohibits terms like "malingering" absent positive findings; flags overcertain language. See [[concepts/validity-language]].
3. **Premorbid context** — Narrative must integrate education, occupation, and baseline functioning; discrepancies between current performance and premorbid expectation must be noted; cultural and linguistic confounds flagged when relevant.
4. **Score–narrative consistency** — Every qualitative claim must be backed by a CSV row; score ranges must match the CSV `range` column exactly; no invented test names or diagnoses.
5. **PHI leaks** — Grep-based scan for names, DOB patterns, SSNs, MRNs, phone numbers, addresses, and facility names.
6. **Tone & style** — Checks for drift from a supplied style profile; flags bare-assertion clinical claims, unexplained jargon, emoji, first-person clinician voice, and self-praise.

The reviewer returns a structured verdict (`ready_to_sign_out`, `revise_before_signout`, or `block_signout`) with categorized findings (CRITICAL, MAJOR, MINOR) and per-axis pass/fail status. It never modifies files.

## Hard Rules for Narrative Generation

- **No raw scores or percentiles in prose** — the R6 / cingulate layer renders score tables; prose refers to performance qualitatively using the `range` column only.
- **No diagnoses, etiology, or prognosis** unless explicitly present in the source extraction.
- **No output outside the patient workspace** — never write to cingulate's package internals, R source files, or any non-workspace destination.
- **PHI**: assumes upstream scrubbing by prior pipeline stages. If real names appear, replace with `[PATIENT]` and emit a WARNING.

## Report Structure

Every generated narrative report follows a canonical section sequence consistent with [[concepts/clinical-report-structure]]:

- **Introduction and Purpose** — referral reason and clinical questions
- **Records Reviewed** — source documents informing the evaluation
- **Background Information** — demographics, history, presenting concerns
- **Tests Administered** — battery listing
- **Neuropsychological Findings** — domain-by-domain narrative with embedded [[concepts/neuropsychological-test-scores]]
- **Cognitive Profile Summary** — integrated interpretation
- **Clinical Impressions and Diagnostic Formulation** — DSM-5/ICD-11 aligned
- **Recommendations** — numbered action items
- **Limitations and Caveats** — validity, effort, cultural factors
- **Forensic Opinion** — highlighted opinion statement within a reasonable degree of neuropsychological certainty

This structure reflects the founder's broader claim that the report is the true product of the evaluation, not just a byproduct of testing. Narrative generation is therefore tasked with preserving integrated clinical reasoning across sections rather than merely producing isolated paragraphs. See [[summaries/Apply-to-Y-Combinator-JWT]].

## Score Presentation

Within domain sections, structured score tables present [[concepts/neuropsychological-test-scores]] in a standardized columnar format: Test/Subtest, Score, Score Type, Percentile, and Classification. This bridges raw data from [[concepts/pdf-score-extraction]] and [[concepts/neuropsychological-report-variables]] with readable clinical prose. Score-narrative consistency is enforced by the QA reviewer, which cross-checks every qualitative descriptor against the source CSV.

In the Luria Streamlit App track, the SQLite `TestScores` table (with columns `test_name`, `subtest_name`, `scaled_score`, `standard_score`, `t_score`, `percentile_rank`, `classification`, `cognitive_domains_affected`) serves as the authoritative structured source, complemented by [[concepts/lancedb-vector-store]] semantic search over narrative chunks for RAG Q&A.

## Agent Output Summary Format

At the end of each run, the `neuropsych-narrative-writer` emits a structured completion block:

```text
WORKSPACE_DIR: <absolute path where files were written>
FILES_WRITTEN:
- _02-05_memory_text.qmd (memory domain, 3 paragraphs)
- _02-06_executive_text.qmd (executive function, 4 paragraphs)
- ...
DOMAINS_SKIPPED: <subdomains with too few measures to interpret>
EDIT_PROTECTION_HITS: <files where existing content prevented overwrite>
NEXT_STEP: Run `quarto render` from the workspace, or invoke cingulate_workflow() to assemble the Typst PDF.
```

## Integration Points

- [[concepts/neuropsychological-reporting]] — overarching reporting standards and conventions
- [[concepts/neuropsychological-assessment-pipeline]] — the upstream data extraction and scoring pipeline
- [[concepts/luria-neuropsych-pipeline]] — the multi-stage pipeline this agent belongs to
- [[concepts/cingulate-engine]] — the R6 rendering layer that assembles the final Typst PDF
- [[concepts/r6-class-architecture]] — the class-based design underpinning the cingulate pipeline
- [[concepts/domain-processor-pattern]] — per-domain processor pattern used in cingulate
- [[concepts/role-based-llm-routing]] — role-based LLM model selection via YAML config
- [[concepts/llm-provider-abstraction]] — abstraction layer supporting Ollama, MLX, and other backends
- [[concepts/artifact-caching-pipeline]] — content-addressed caching for bridge stage outputs
- [[concepts/quarto]] — the document format used for narrative include files
- [[concepts/quarto-extensions]] — cingulate-specific Quarto extensions (neurotyp-adult, pediatric, forensic)
- [[concepts/typst-typesetting]] — the final typesetting system for PDF output
- [[concepts/modular-report-architecture]] — architectural pattern separating prose, tables, and rendering
- [[concepts/clinical-nlp-pipelines]] — NLP tools that may assist in drafting or structuring prose
- [[concepts/multi-agent-orchestration]] — the agent framework coordinating pipeline stages
- [[concepts/subagent-architecture]] — parallel agent design enabling the reviewer to run concurrently
- [[concepts/report-review-qa]] — the broader QA framework the validity reviewer implements
- [[concepts/edit-protection-pattern]] — the check-before-overwrite pattern used to preserve clinician edits
- [[concepts/phi-data-handling]] — PHI scrubbing and anonymization requirements
- [[concepts/long-format-clinical-data]] — the CSV data format fed into the narrative writer
- [[concepts/retrieval-augmented-generation]] — RAG used for both draft generation and Q&A over ingested reports
- [[concepts/rag-chunking]] — chunking strategies for indexing historical reports
- [[concepts/text-chunking]] — lower-level text segmentation patterns
- [[concepts/style-profiles]] — per-clinician and per-population style profile management
- [[concepts/style-profile-extraction]] — how style profiles are derived and applied across tracks
- [[concepts/langgraph-agent-workflows]] — orchestration framework for the Streamlit pipeline
- [[concepts/lancedb-vector-store]] — local vector store for semantic retrieval in the Streamlit track
- [[concepts/sqlite-as-vector-store]] — SQLite index used by the Soul/Style Agent RAG track
- [[concepts/docling-pdf-parsing]] — local PDF parsing stage upstream of narrative generation
- [[concepts/local-llm-inference]] — optional local LLM for summarization and chat
- [[concepts/omlx-server]] — local inference server used by the Soul/Style Agent track
- [[concepts/privacy-first-software]] — design philosophy underlying local-first deployment
- [[concepts/local-first-architecture]] — local deployment model for PHI-sensitive report generation
- [[concepts/clinical-data-privacy]] — privacy constraints shaping architecture and deployment
- [[concepts/fallback-strategy]] — LLM fallback handling used in generation and embedding calls
- [[concepts/yaml-configuration]] — model and mode configuration in `ollama_models.yml`
- [[concepts/duckdb-data-staging]] — high-performance data staging layer in the cingulate pipeline
- [[concepts/neuropsychological-assessment-automation]] — broader automation context the cingulate package addresses
- [[concepts/per-patient-workspace]] — workspace isolation pattern used by the cingulate pipeline
- [[concepts/behavioral-rating-scales]] — multi-informant rating scales interpreted by the neurobehav prompt
- [[concepts/executive-function-deficits]] — a primary construct assessed in behavioral rating scale domains
- [[concepts/working-memory]] — assessed within ADHD/executive functioning rating scale domains
- [[concepts/autism-spectrum-disorder-clinical-features]] — relevant to social cognition rating scale interpretation
- [[concepts/dual-audience-design]] — principle governing mixed professional/lay register in report prose
- [[concepts/clinical-communication-register]] — register standards applied in behavioral and adaptive sections
- [[concepts/neuropsychological-prompt-engineering]] — prompt design principles underlying domain-specific templates like `nt_interpretation` and `neurobehav`
- [[concepts/neuropsychological-score-interpretation]] — score classification conventions (5th/95th percentile thresholds) referenced in domain prompt templates
- [[concepts/luria-overview]] — product-level framing for local-first agentic neuropsychological workflow automation
- [[concepts/clinical-ai-copilot]] — adjacent framing for clinician-in-the-loop drafting and review
- [[summaries/nt_interpretation]] — domain-level prompt template for single-domain interpretive narratives
- [[summaries/LLM_AGENT_MAP]] — detailed orchestration map for the cingulate LLM stack
- [[summaries/customization]] — customization workflows for style profiles, chunking, retrieval, and QA
- [[summaries/report-generator]] — RAG-based draft generation component specification
- [[summaries/neuropsych-narrative-writer]] — full specification of the narrative writer agent
- [[summaries/neuropsych-data-extractor]] — upstream data extraction stage
- [[summaries/clinical-validity-reviewer]] — full specification of the pre-delivery review agent
- [[summaries/report-generation]] — project-level notes on report generation implementation
- [[summaries/WORKFLOW_INSTRUCTIONS]] — workflow context for report generation triggers
- [[summaries/responses_to_claude]] — related implementation notes
- [[summaries/README]] — cingulate R package README documenting the full pipeline
- [[summaries/README_luria]] — Luria Streamlit App overview documenting the LangGraph report generation track
- [[summaries/CLAUDE]] — cingulate R package developer guide with LLM routing and pipeline details
- [[summaries/neurobehav.prompt]] — behavioral rating scale interpretation prompt template
- [[summaries/neurocog.prompt]] — cognitive domain prompt configuration
- [[summaries/nse_narrative]] — neurobehavioral status exam narrative template
- [[summaries/Apply-to-Y-Combinator-JWT]] — YC application framing narrative generation as the core scalable product surface

See also: [[summaries/NP-20240415-001_report]]

See also: [[summaries/report_body]]

See also: [[summaries/multi_patient_transcript]]

See also: [[summaries/0004-soul-style-profile-json]]

See also: [[summaries/0007-style-modular-report-sections-via-quarto-includes]]

See also: [[summaries/0010-voice-quarto-typst-reporting]]

See also: [[summaries/report-template]]

See also: [[summaries/soul-style-agent]]

See also: [[summaries/style-trainer]]

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/style-training-to-report-drafting]]

See also: [[summaries/full-pipeline]]

See also: [[summaries/0007-voice-modular-report-sections-via-quarto-includes]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/agent-team]]

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/copilot-instructions]]

See also: [[summaries/sirf_synthesis]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]