---
sources: [summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/README.md, summaries/CLAUDE.md]
brief: R6-first OOP architecture powering the cingulate neuropsych report pipeline from CSV to PDF.
---

# R6 Class Architecture in R Packages

R6 is a lightweight, performant class system for R that provides mutable, reference-based objects with encapsulated state and methods. Unlike S3 or S4, R6 classes behave more like objects in Python or Java — instances are modified in place rather than copied, making them well-suited for stateful pipelines, resource management, and complex orchestration logic.

## Why R6 for Package Architecture

R6 is chosen in complex R packages when:

- **State must persist** across multiple method calls (e.g., database connections, cached results, configuration)
- **Encapsulation** is needed to hide internals from callers
- **Inheritance** is useful for specializing processors (e.g., one base processor, many domain-specific subclasses)
- **Lifecycle management** matters — `initialize()` and `finalize()` hooks manage setup/teardown
- **Interoperability** with Python objects (via `reticulate`) benefits from a shared mental model

## R6 in the Cingulate Package

The `cingulate` package is explicitly **R6-first** — every major concern is an R6 class, and workflow code wires them together. This architecture separates concerns cleanly across a large codebase with 40+ heavy imports, spanning the full pipeline from raw test scores through DuckDB-backed staging, domain processing, LLM-assisted narrative generation, and publication-quality report output.

The pipeline flows: **CSV → DuckDB/Parquet → R6 Processing → QMD sections → Quarto/Typst → PDF**.

The package targets adult, pediatric, and forensic neuropsychology formats, with R6 classes providing the modularity needed to support all three report types from a shared core.

## Entry Points & Commands

```fish
# Interactive shell workflow (recommended)
./unified_neuropsych_workflow.sh "Patient Name"

# Programmatic R workflow
Rscript inst/scripts/00_complete_neuropsych_workflow.R

# Quick convenience functions
Rscript -e "library(cingulate); run_workflow()"
Rscript -e "library(cingulate); quick_rerender()"
```

```r
# Development iteration cycle
devtools::load_all('.')
devtools::test()
devtools::document()
devtools::check()
```

### Core Orchestration

- **`WorkflowRunnerR6`** — top-level orchestrator driving the full CSV→PDF pipeline; also exposed as `cingulate_workflow()` for end-to-end runs
- **`DomainProcessorR6`** — base class for per-cognitive-domain processing, covering IQ, academics, memory, attention/executive function, language, sensorimotor, and social-emotional domains
- **`DomainProcessorFactoryR6`** — dynamically detects which domains exist in input data and instantiates only the needed processors; a classic factory pattern in R6. Also exposes `validate_domain_data(domain_key, detailed=TRUE)` for pre-processing checks and `batch_create()` for multi-domain instantiation including multi-rater domains (ADHD, emotion)
- **`NeuropsychResultsR6`** — generates clinical narrative text from test scores, injecting LLM output into QMD files via `LLM_CONTEXT_START…END` markers

This maps directly to the [[concepts/domain-processor-pattern]] used throughout the system.

### Domain Processing Pattern

The domain processing workflow follows a strict two-stage authoring pattern:

1. **Domain Text Files** (`_02-XX_domain_text.qmd`) are created by `generate_domain_text_qmd()` — always first
2. **Domain QMD Files** (`_02-XX_domain.qmd`) include text files and generate tables/plots
3. **Multi-rater domains** (emotion/ADHD) create separate files per rater (self/parent/teacher)

Always call `generate_domain_text_qmd()` before generating QMD files, and check `check_rater_data_exists()` before creating rater-specific files.

### Data and Rendering Classes

| Class | Responsibility |
|---|---|
| `DuckDBProcessorR6` | CSV → Parquet → in-memory query staging via [[concepts/duckdb-data-staging]] |
| `NeuropsychResultsR6` | Score → clinical narrative conversion; injects LLM output into QMD files via `LLM_CONTEXT_START…END` markers |
| `TableGTR6` | gt table rendering |
| `DotplotR6` | Dot plot visualization |
| `DrilldownR6` | Highcharts drilldown rendering |

### LLM Integration Classes

The package integrates local LLMs via Ollama (and optionally MLX), with R6 classes managing the full routing, prompting, and orchestration lifecycle. These classes are documented in detail in [[summaries/LLM_AGENT_MAP]] and [[summaries/LLM_INTEGRATION]].

Three R6 classes introduced in November 2025 replace the deprecated `make_mega()` and `make_mega_for_sirf()` functions:

- **`OllamaModelRouterR6`** (`R/OllamaModelRouterR6.R`, 170 lines) — Model gateway that routes prompts to specific local models based on 13 clinical *roles* (e.g., `summarize_domain`, `integrate_evaluation`, `differential_diagnosis`, `generate_recommendations`). Configured via `inst/config/ollama_models.yml`, which maps every role to a primary model plus fallback. Supports both Ollama and MLX endpoints via environment variables.

- **`ReportLLMBridgeR6`** (`R/ReportLLMBridgeR6.R`, 280 lines) — Orchestration layer that runs individual pipeline stages with full artifact tracking. Per stage it writes:
  - `artifacts/patient_XXX/{stage_id}.md` — raw Markdown narrative
  - `artifacts/patient_XXX/{stage_id}.meta.json` — model, role, timing, QC info
  - `artifacts/patient_XXX/run.log.jsonl` — append-only audit log
  - `.cache/<hash>.rds` — content-addressed cache entries
  
  Exposes `run_stage(role, stage_id, messages, options)` and `run_pipeline(stages)`.

- **`DomainPrompterR6`** (`R/DomainPrompterR6.R`, 220 lines) — Constructs structured OpenAI-style message lists (`role/content` pairs) from neuropsych test score data. Supports `clinical`, `parent`, and `school` output styles. Domain-specific prompt templates live in `inst/prompts/{domain}_prompt.txt` and use `{{DATA}}` as the injection placeholder.

Model configuration is stored in `inst/config/ollama_models.yml` and consumed by these classes. See [[concepts/local-llm-inference]], [[concepts/role-based-llm-routing]], and [[concepts/narrative-report-generation]] for the broader context.

The LLM system also supports 3 performance modes (development/balanced/production), selectable via `get_model_for_task("domain_summary", "production")` and `query_ollama_enhanced(prompt, task = "rating_scales", mode = "balanced")`.

#### LLM Role Table

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

#### Migration from Deprecated Functions

| Old Pattern | New Pattern |
|---|---|
| `make_mega(data, ...)` | `bridge$run_stage("summarize_domain", "01_domain", messages = msgs)` |
| `make_mega_for_sirf(...)` | `bridge$run_stage("integrate_evaluation", "04_sirf", prompt = "...")` |
| Hardcoded prompts | `DomainPrompterR6$build_messages()` with templates |
| Manual model selection | `OllamaModelRouterR6` auto-routes by role |

### Infrastructure Classes

- `ConfigManagerR6` — centralizes configuration loading from [[concepts/yaml-configuration]] files
- `ScoreTypeCacheR6` — caches score-type detection results
- `ErrorHandlerR6` — standardizes error capture via `safe_execute(expr, context)`
- `ExecutionTrackerR6` — monitors pipeline execution state (`R/ExecutionTrackerR6.R`)
- `TemplateContentManagerR6` — manages QMD template content for Quarto/Typst report templates
- `PackageManagerR6` — handles package-level lifecycle

## Key R6 Patterns Used

### Factory Pattern
`DomainProcessorFactoryR6` exemplifies the factory pattern: it inspects available input data and returns only the R6 processor instances needed. It supports single-domain creation, multi-rater creation (`create_multi_processor()`), and full batch creation (`batch_create()`). This avoids hard-coding domain lists and supports extensibility as new cognitive domains are added.

### Registry / Cache Pattern
`ScoreTypeCacheR6` illustrates using R6 for mutable caching — state that must be shared across calls without passing arguments through every function in a chain. `ReportLLMBridgeR6` extends this idea to content-addressed disk caching of LLM outputs, so re-runs skip unchanged stages. This implements the [[concepts/artifact-caching-pipeline]] pattern.

### Bridge Pattern
`ReportLLMBridgeR6` acts as a bridge between the report generation subsystem and the LLM routing subsystem, decoupling the two concerns and making it possible to swap model backends without touching report logic. This is an instance of the [[concepts/llm-provider-abstraction]] pattern.

### Multi-Rater Pattern
Domains like ADHD and emotion require scores from multiple informants (self, parent, teacher, observer). `DomainProcessorFactoryR6$create_multi_processor()` returns a named list of processors (`$self`, `$parent`, `$teacher`), each an independent R6 instance, enabling parallel processing of multi-rater data.

### Edit Protection
The system has **two-stage rendering with edit protection** — always check if files are manually edited before regenerating. This is a critical operational constraint managed by the R6 orchestration layer and relates to the [[concepts/edit-protection-pattern]] pattern.

## End-to-End Agent Flow

The five R6 classes most critical to the LLM orchestration layer compose in a fixed sequence:

```r
# 1. Data layer
load_data_duckdb("data-raw/csv", output_dir = "data", output_format = "all")

# 2. LLM stack initialization
router   <- OllamaModelRouterR6$new("inst/config/ollama_models.yml")
bridge   <- ReportLLMBridgeR6$new(router, out_dir = "artifacts/patient_001", seed = 42)
prompter <- DomainPrompterR6$new(style = "clinical")

# 3. Domain processing loop
factory <- DomainProcessorFactoryR6$new()
for (dk in c("iq", "memory", "executive", "adhd", "emotion")) {
  proc <- factory$create_processor(dk, age_group = "child")
  proc$load_data(); proc$filter_by_domain(); proc$select_columns()
  msgs <- prompter$build_messages(domain = dk, data = as.list(proc$data), task = "summarize_domain")
  bridge$run_stage("summarize_domain", sprintf("01_%s", dk), messages = msgs)
}

# 4. Integration and render
cingulate_run_llm_then_render(render_paths = "template.qmd", parallel = FALSE)
```

## Connecting LLM Output to Report Rendering

The Markdown content produced by `ReportLLMBridgeR6$run_stage()` is automatically injected into the corresponding `_text.qmd` include files. The full render pipeline is:

1. `load_data_duckdb()` — stage patient CSV exports as parquet/DuckDB files
2. `cingulate_workflow(..., generate_report = FALSE)` — generate QMD scaffolding
3. `cingulate_run_llm_then_render(render_paths = "template.qmd", parallel = TRUE, n_cores = 6)` — runs `run_llm_for_all_domains_parallel()` per domain then renders to the Typst profile configured in `_quarto.yml`

## High-Level API

While the R6 classes power the internals, the package also exposes high-level functional helpers that wrap the R6 orchestration:

```r
cingulate_quick_start(patient = "John Doe", age = 12)
create_patient_workspace("ExamplePatient", age = 12)
results <- process_all_domains("data", age_group = "child")
generate_assessment_report(results, patient_info = list(name = "ExamplePatient", age = 12))
```

Batch LLM generation supports parallelism:

```r
run_llm_for_all_domains_parallel(
  prompts_dir = "inst/prompts",
  domains     = c("proiq", "promem", "proexe", "proadhd"),
  backend     = "ollama",
  n_cores     = 4
)
```

## Package Development Context

This is an **R package** (not just scripts), requiring specific workflows:

- R >= 4.5 required
- Use `devtools::load_all('.')` for rapid iteration, not `source()`
- Documentation via roxygen2: `devtools::document()` regenerates man pages
- Tests in `tests/testthat/` run via `devtools::test()`
- Package namespace managed in `NAMESPACE` (auto-generated, never edit directly)
- Internal data in `R/sysdata.rda` loaded via `load_scales_internal()`
- `man/*.Rd` help files are auto-generated — never edit directly

## Common Operational Pitfalls

### R Session Startup Issues (CRITICAL)

**Symptom**: "JEP 66 handshake failed" or "R failed to start up (exit code 0)"

The package has 40+ dependencies (DuckDB, Arrow, GT, Quarto integration). Positron's R session has a handshake timeout (~30–60 seconds), and R6 class initialization can be slow on first load.

**Solutions**:
- Never auto-load `library(cingulate)` in `.Rprofile`
- Use `devtools::load_all('.')` in console *after* session starts
- Clear corrupted package cache: `rm -rf ~/.cache/R/`
- Don't call `setup_cingulate()` in startup hooks

This is a fundamental constraint related to the [[concepts/positron-ide]] startup behavior where auto-loading a heavy R6-based package in `.Rprofile` causes IDE handshake timeouts.

### File Generation Order
- Always call `generate_domain_text_qmd()` **before** generating QMD files
- Check `check_rater_data_exists()` before creating rater-specific files
- Multi-rater domains require special handling for child vs. adult patterns

### LLM Configuration Issues
- Ollama must be running: `ollama serve`
- Models must be pulled: `ollama pull <model>`
- Check model availability: `check_available_models(models, "ollama")`
- Use development mode for fast iteration, production for final reports
- Do NOT use deprecated `make_mega()` or `make_mega_for_sirf()`

### Data Loading Issues
- Input files may have "data/" prefix — handle both cases in file paths
- Load scales from `R/sysdata.rda` with proper error handling
- Use `safe_use_data_internal()` for internal data updates

## Critical Loading Requirement

Because R6 generators **register on package load**, files cannot be `source()`d individually. The package must always be loaded via:

```r
devtools::load_all('.')
```

Alternatively, once the package has a valid `DESCRIPTION` and `NAMESPACE`, it can be installed directly:

```r
pak::pak("brainworkup/cingulate")
library(cingulate)
```

## R6 vs. Alternatives

| System | Mutability | Reference Semantics | Use Case |
|---|---|---|---|
| S3 | Copy-on-modify | No | Simple dispatch |
| S4 | Copy-on-modify | No | Formal type checking |
| R5/RefClass | Mutable | Yes | Legacy reference classes |
| **R6** | **Mutable** | **Yes** | **Modern OOP pipelines** |
| R7 | Mutable | Yes | Emerging standard |

## Related Concepts

- [[concepts/domain-processor-pattern]] — the specific per-domain processor design built on R6
- [[concepts/neuropsychological-assessment-pipeline]] — the pipeline R6 classes orchestrate
- [[concepts/local-llm-inference]] — LLM classes integrated via R6 router
- [[concepts/narrative-report-generation]] — output produced by R6 result classes
- [[concepts/yaml-configuration]] — config consumed by `ConfigManagerR6`
- [[concepts/r-python-integration]] — R6 mental model aligns with Python OOP for `reticulate` interop
- [[concepts/quarto]] — downstream rendering target of the R6 pipeline
- [[concepts/duckdb-data-staging]] — data layer managed by `DuckDBProcessorR6`
- [[concepts/neuropsychological-assessment-automation]] — the broader automation goal the R6 architecture serves
- [[concepts/role-based-llm-routing]] — role-to-model mapping consumed by `OllamaModelRouterR6`
- [[concepts/artifact-caching-pipeline]] — content-addressed caching implemented by `ReportLLMBridgeR6`
- [[concepts/llm-provider-abstraction]] — backend-agnostic design of the router/bridge pair
- [[concepts/positron-ide]] — IDE where the load-order constraint matters most
- [[concepts/edit-protection-pattern]] — two-stage rendering protection managed by the orchestration layer
- [[concepts/modular-report-architecture]] — QMD include pattern used by domain text files

## Related Documents

- [[summaries/copilot-instructions]] — AI agent guide for the cingulate repository
- [[summaries/README]]
- [[summaries/LLM_AGENT_MAP]]
- [[summaries/LLM_INTEGRATION]]