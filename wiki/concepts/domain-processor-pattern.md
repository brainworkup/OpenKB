---
sources: [summaries/Apply-to-Y-Combinator-JWT.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/README.md, summaries/CLAUDE.md]
brief: R6-based pattern where each cognitive domain has a dedicated processor class and QMD fragment for modular report generation.
---

# Domain Processor Pattern for Modular Clinical Report Generation

The domain processor pattern is a software architecture approach used in [[concepts/neuropsychological-assessment-pipeline]] systems where each cognitive domain is encapsulated in its own processor class and corresponding Quarto document fragment. This enables modular, independently-generated report sections that are assembled into a unified PDF output. It is the central architectural pattern of the `cingulate` R package (see [[summaries/README]] for the project overview and [[summaries/CLAUDE]] for the authoritative developer reference, and [[summaries/copilot-instructions]] for the AI agent quick-start guide).

## Core Concept

Rather than processing all neuropsychological data in a monolithic pipeline, the domain processor pattern assigns a dedicated processor class to each cognitive domain. A factory class auto-detects which domains are present in the input data and instantiates only the relevant processors — avoiding unnecessary computation and keeping reports focused on tested domains.

This pattern is central to the `cingulate` R package, which orchestrates a full pipeline from raw cognitive and behavioral test scores to polished, publication-quality PDF reports. The pipeline flows: **CSV → DuckDB/Parquet → R6 Processing → QMD sections → Quarto/Typst → PDF**. The `LLM_AGENT_MAP` reference document (see [[summaries/LLM_AGENT_MAP]]) provides a comprehensive orchestration guide showing precisely where and how each class is called across the full pipeline.

## Key Components

### Processor Classes

- **`DomainProcessorR6`** — base class defining the interface all domain processors implement
- **`DomainProcessorFactoryR6`** — inspects input data, detects available domains, and dynamically instantiates the appropriate processors
- **`WorkflowRunnerR6`** — orchestrates the entire pipeline, sitting above the domain processors
- **`NeuropsychResultsR6`** — generates clinical narrative text from test scores
- **`TableGTR6`** / **`DotplotR6`** — data visualization components consumed by domain QMDs

Domains handled include IQ / General Ability, Academics, Memory, Attention / Executive Function, Language / Verbal, Sensorimotor, and Social-Emotional / Behavioral. This maps directly to the [[concepts/cognitive-domains]] taxonomy used across neuropsychological practice.

### Full Domain Key Reference

The factory supports thirteen domain keys across two data sources:

| Key | Data source | Multi-rater? |
|---|---|:---:|
| `iq` | `neurocog.parquet` | |
| `academics` | `neurocog.parquet` | |
| `verbal` | `neurocog.parquet` | |
| `spatial` | `neurocog.parquet` | |
| `memory` | `neurocog.parquet` | |
| `executive` | `neurocog.parquet` | |
| `motor` | `neurocog.parquet` | |
| `social` | `neurocog.parquet` | |
| `adhd` | `neurobehav.parquet` | ✅ |
| `emotion` | `neurobehav.parquet` | ✅ |
| `adaptive` | `neurobehav.parquet` | |
| `daily_living` | `neurobehav.parquet` | |
| `validity` | `validity.parquet` | |

### Per-Domain QMD Generation

Each domain processor produces two output artifacts:

1. **Text QMD** (`_02-XX_domain_text.qmd`) — the clinical narrative for that domain, generated via `generate_domain_text_qmd()`. Narrative prose is produced by the [[concepts/narrative-report-generation]] subsystem.
2. **Shell QMD** (`_02-XX_domain.qmd`) — includes the text file and adds tables and visualizations (gt tables, dot plots, Highcharts drilldown).

This two-file structure is an instance of the [[concepts/modular-report-architecture]] principle: prose and presentation are separated, allowing each to be regenerated or hand-edited independently. Critically, `generate_domain_text_qmd()` must always be called **before** generating domain QMD files — file generation order is load-bearing.

### Prompt-to-QMD Mapping

Each domain is also associated with a prompt keyword and a corresponding prompt template file in `inst/prompts/`. This mapping drives `NeuropsychResultsR6$process()`, which injects LLM-generated text directly into the target QMD file using `LLM_CONTEXT_START…END` markers:

| Keyword | Prompt file | Target QMD |
|---|---|---|
| `proiq` | `inst/prompts/proiq.qmd` | `_02-01_iq_text.qmd` |
| `promem` | `inst/prompts/promem.qmd` | `_02-05_memory_text.qmd` |
| `proexe` | `inst/prompts/proexe.qmd` | `_02-06_executive_text.qmd` |
| `proadhd` | `inst/prompts/proadhd.qmd` | `_02-09_adhd_text.qmd` |
| `proemo` | `inst/prompts/proemo.qmd` | `_02-10_emotion_text.qmd` |
| `prosirf` | `inst/prompts/prosirf.qmd` | `_03-00_sirf.qmd` (integrated summary) |
| `prorecs` | `inst/prompts/prorecs.qmd` | `_03-01_recs.qmd` (recommendations) |

The complete mapping covers all domains plus special sections for behavioral observations (`pronse`), validity (`provalid`), and general summaries (`progen`).

### Stable Domain Numbering

Domains follow a stable numbering convention (`01_iq`, `02_academics`, `03_verbal`, …). This ordering is load-bearing — `_quarto.yml` and the rendering pipeline depend on it. Renumbering requires coordinated updates across multiple files.

## LLM Integration and Model Routing

A key extension of the domain processor pattern in `cingulate` is its tight integration with local LLM inference. The enhanced LLM system supports **3 performance modes** (development / balanced / production), and each domain processor can invoke LLM-assisted narrative generation through specialized model routing via [[concepts/role-based-llm-routing]]:

- **`DomainPrompterR6`** — constructs domain-specific clinical prompt templates stored in `inst/prompts/`, supporting `clinical`, `parent`, and `school` output styles. Formats test scores as `Label | SS X | Yth percentile | Range` lines and assembles OpenAI-style message lists.
- **`OllamaModelRouterR6`** — routes domain-level summary tasks to specific models based on 13 defined clinical roles (e.g., `summarize_domain`, `integrate_evaluation`, `generate_recommendations`, `differential_diagnosis`). Configured via `inst/config/ollama_models.yml`.
- **`ReportLLMBridgeR6`** — bridges domain processors to the LLM layer, managing per-stage artifact output and content-addressed caching.

Model routing is configured via [[concepts/yaml-configuration]] and supports development mode for fast iteration and production mode for final reports. Task-specific model selection is exposed via `get_model_for_task()` and `query_ollama_enhanced()`. This makes the domain processor pattern LLM-aware without coupling each processor directly to a specific model. See [[concepts/local-llm-inference]] for broader context on local inference architecture.

### LLM Role Catalogue

The router exposes the following roles, each mapped to a model ID and fallback in the YAML config:

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
| `finalize_diagnostic_summary` | Final diagnostic paragraph |
| `generate_narrative` | Full prose narrative |
| `embeddings` | Generate text embeddings |
| `fallback_general` | Generic fallback for any prompt |

## Artifact and Caching Architecture

The `ReportLLMBridgeR6` implements a robust artifact trail for every pipeline stage, connecting to the [[concepts/artifact-caching-pipeline]] pattern:

| File | Content |
|---|---|
| `artifacts/.../01_memory.md` | Generated narrative |
| `artifacts/.../01_memory.meta.json` | Model, timing, QC scores |
| `artifacts/.../run.log.jsonl` | Append-only audit trail |
| `artifacts/.../.cache/<hash>.rds` | Content-addressed cache |

Because summaries are content-addressed, re-running the pipeline on unchanged data produces identical outputs without re-invoking the LLM.

## Data Staging and Score-Type Awareness

Upstream of the domain processors, raw test data flows through a [[concepts/duckdb-data-staging]] pipeline (CSV → Parquet → DuckDB) via `load_data_duckdb()`. This function reads CSV files from `data-raw/csv/`, computes z-scores, and splits data by `test_type` into `neurocog`, `neurobehav`, and `validity` tables in multiple output formats — delivering a 4–5x performance boost over direct CSV loading.

Each domain processor must correctly handle score types (`t_score`, `scaled_score`, `standard_score`) for footnotes and z-score conversions. The `ScoreTypeCacheR6` class caches score-type detection results. New tests added to a domain must have their score types registered in `R/score_type_utils.R` and `R/neuropsych_test_scoring.R`. See [[concepts/neuropsychological-test-scores]] for broader context.

## Multi-Rater Domain Handling

Some domains (emotion, ADHD) involve multiple raters — self-report, parent, and teacher. The factory extends to these via `create_multi_processor()`:

```r
processors <- factory$create_multi_processor("adhd", age_group = "child")
# Returns: processors$self, processors$parent, processors$teacher
```

Before generating per-rater QMD files, processors must call:

- `check_rater_data_exists()` — confirms data is available for a rater
- `check_domain_raters()` — identifies which raters contributed data

Child vs. adult report variants diverge here, as pediatric reports may include parent and teacher ratings while adult reports focus on self-report. This links to the [[concepts/dual-audience-design]] concern across the system.

## Edit Protection

Because QMD files may be hand-edited by clinicians after initial generation, the system implements a two-stage rendering workflow with edit detection. The system has **two-stage rendering with edit protection** — before regenerating a domain's QMD, the pipeline checks whether the target file has been manually modified. This is an application of the [[concepts/edit-protection-pattern]] concept. Always check if files are manually edited before regenerating.

## Parallel Execution

Batch LLM calls across all domains can be parallelized using `run_llm_for_all_domains_parallel()` (specifying `n_cores`) or combined with rendering via `cingulate_run_llm_then_render(parallel = TRUE)`. Sequential execution is also supported for debugging via `run_llm_for_all_domains()`. All three functions live in `R/cingulate_llm.R`.

## High-Level API and Entry Points

The domain processor pattern is exposed through high-level workflow helpers:

```r
# Factory-driven batch processing
factory <- DomainProcessorFactoryR6$new()
all_processors <- factory$batch_create(
  domain_keys         = c("iq", "memory", "executive", "adhd", "emotion"),
  age_group           = "child",
  include_multi_rater = TRUE
)

# Full workflow entry point
cingulate_workflow()

# Quick new-user setup
cingulate_quick_start(patient = "John Doe", age = 12)
```

Shell and script entry points:

```fish
# Interactive shell workflow
./unified_neuropsych_workflow.sh "Patient Name"

# Programmatic R workflow
Rscript inst/scripts/00_complete_neuropsych_workflow.R

# Quick convenience functions
Rscript -e "library(cingulate); run_workflow()"
Rscript -e "library(cingulate); quick_rerender()"
```

`process_all_domains()` triggers the factory pattern — detecting available domains in the input data and invoking the appropriate processors — while `generate_assessment_report()` assembles domain QMDs into the final rendered output.

## Package Development Context

The domain processor pattern lives inside a proper R package (requiring R >= 4.5), which imposes specific development conventions:

- Use `devtools::load_all('.')` for rapid iteration, not `source()`
- Documentation via roxygen2: `devtools::document()` regenerates man pages
- Tests in `tests/testthat/` run via `devtools::test()`
- `NAMESPACE` and `man/*.Rd` are auto-generated — never edit directly
- Internal data accessed via `load_scales_internal()` from `R/sysdata.rda`
- R6 class initialization can be slow on first load; avoid auto-loading in `.Rprofile`

## Integration Points

| Concern | Class / File |
|---|---|
| Data ingestion | `DuckDBProcessorR6` → [[concepts/duckdb-data-staging]] |
| Score processing | `DomainProcessorR6`, `ScoreTypeCacheR6` |
| Narrative text | `NeuropsychResultsR6`, `DomainPrompterR6` |
| LLM routing | `OllamaModelRouterR6` → [[concepts/role-based-llm-routing]] |
| Artifact caching | `ReportLLMBridgeR6` → [[concepts/artifact-caching-pipeline]] |
| Rendering | Quarto + Typst → [[concepts/typst-typesetting]] |
| Configuration | `inst/config/ollama_models.yml` → [[concepts/yaml-configuration]] |
| Per-patient workspace | `create_patient_workspace()` → [[concepts/per-patient-workspace]] |
| Local inference | `OllamaModelRouterR6` → [[concepts/local-llm-inference]] |

## Relationship to R6 Architecture

The domain processor pattern is an application of [[concepts/r6-class-architecture]] to clinical reporting: each domain is an object with encapsulated state and behavior, instantiated by a factory, and composed into a larger workflow by `WorkflowRunnerR6`. This keeps domain logic isolated and testable independent of the overall pipeline. The full set of R6 classes (`OllamaModelRouterR6`, `ReportLLMBridgeR6`, `DomainPrompterR6`, `DuckDBProcessorR6`, `ScoreTypeCacheR6`) together form the `cingulate` engine architecture described in [[concepts/cingulate-engine]].

## Related Concepts

- [[concepts/neuropsychological-reporting]] — clinical output goals
- [[concepts/clinical-report-structure]] — how domain sections assemble into a full report
- [[concepts/neuropsychological-assessment-automation]] — the broader automation goal
- [[concepts/quarto]] — the rendering engine consuming domain QMDs
- [[concepts/quarto-extensions]] — format extensions used for PDF output
- [[concepts/luria-neuropsych-pipeline]] — alternative neuropsych pipeline architecture
- [[concepts/r-neuropsych-packages]] — R ecosystem context
- [[concepts/r-python-integration]] — Python sidecar (LiteLLM, PyMuPDF) used alongside the R core
- [[concepts/multi-agent-orchestration]] — broader orchestration patterns the pipeline participates in
- [[concepts/narrative-report-generation]] — LLM-driven prose generation subsystem

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]