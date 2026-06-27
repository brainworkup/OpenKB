---
sources: [summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/DEPENDENCIES.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/SETUP_SUMMARY.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/README.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/responses_to_claude.md]
brief: R package that transforms neuropsychological CSV test data into publication-quality PDF reports via R6/LLM pipeline.
---

# Cingulate Engine

The **cingulate engine** is an **R package** (with a Python sidecar) that turns neuropsychological test data into publication-quality PDF reports. Named after the cingulate cortex — a key brain hub for attention, cognition, and executive control — it sits at the center of the [[concepts/neuropsychological-assessment-pipeline]], consolidating all data workflows into a single, package-based system. Originally referenced as `R/cingulate` within the `~/luria` project tree, it replaced an earlier architecture that relied on Google Sheets for data storage and output.

Developed by Joey Trampush at the Brain Workup Lab, the package is available at `brainworkup/cingulate` and licensed under MIT.

## Pipeline Overview

The end-to-end pipeline is:

```
CSV (data-raw/csv/) → DuckDB/Parquet → R6 domain processors → per-domain QMD includes → Quarto/Typst → PDF
```

The engine is a proper R package — not a script collection. It must be loaded via `devtools::load_all('.')` rather than `source()`, because R6 generators register on package load. Two high-level entry points are exported: `cingulate_quick_start()` and `cingulate_workflow()`. Additional exported helpers include `create_patient_workspace()`, `process_all_domains()`, and `generate_assessment_report()`.

## Installation

```r
# From GitHub
pak::pak("brainworkup/cingulate")

# Development load from clone
devtools::load_all()
```

Key dependencies: **R6**, **dplyr**, **ggplot2**, **gt**, **quarto**, **duckdb**, **yaml**, **here**.

## Bootstrap Options

Three bootstrap paths are supported:

```r
# Option A: R package dev mode (recommended)
devtools::load_all("/path/to/cingulate")

# Option B: In-script (Quarto or Rscript)
source("R/setup_cingulate.R")
setup_cingulate()

# Option C: Use the workspace initializer
init_cingulate_workspace(patient_name = "John Doe")
```

Key exported setup functions: `setup_cingulate()`, `init_cingulate_workspace()`, `check_cingulate_setup()`.

## Architecture: R6-First Design

The system is built on [[concepts/r6-class-architecture]]. Each major concern is a class; workflow code wires them together.

### Core Pipeline Classes

- **`WorkflowRunnerR6`** — top-level orchestrator driving the full CSV→PDF pipeline
- **`DomainProcessorR6` + `DomainProcessorFactoryR6`** — one processor per cognitive domain (IQ, academics, memory, attention/executive, language, sensorimotor, social-emotional); the factory auto-detects which domains exist in the input data. See [[concepts/domain-processor-pattern]].
- **`NeuropsychResultsR6`** — converts processed scores into clinical narrative text; writes `LLM_CONTEXT_START…END` blocks into QMD files and injects `<summary>…</summary>` tags back into the same file

### LLM Stack Classes

Three R6 classes form the LLM integration layer, replacing the deprecated `make_mega()` and `make_mega_for_sirf()` functions:

- **`OllamaModelRouterR6`** (`R/OllamaModelRouterR6.R`, 170 lines) — model gateway that routes prompts to specific LLM models by clinical role, configured via `inst/config/ollama_models.yml`; supports Ollama and MLX backends. See [[concepts/role-based-llm-routing]].
- **`DomainPrompterR6`** (`R/DomainPrompterR6.R`, 220 lines) — builds structured OpenAI-style message lists from neuropsych test score data; supports `clinical`, `parent`, and `school` output styles. Uses domain-specific prompt templates in `inst/prompts/{domain}_prompt.txt` with `{{DATA}}` placeholder injection. See [[concepts/llm-provider-abstraction]].
- **`ReportLLMBridgeR6`** (`R/ReportLLMBridgeR6.R`, 280 lines) — orchestration layer that runs individual pipeline stages, writes artifacts (`.md`, `.meta.json`, `.jsonl`), and manages content-addressed caching. Per-call parameter overrides are possible via `bridge$run_stage(..., options = list(temperature = 0.1))`. See [[concepts/artifact-caching-pipeline]].

### Rendering Components

- **`TableGTR6`** — gt tables
- **`DotplotR6`** — dot plot visualizations
- **`DrilldownR6`** — Highcharts drilldown visuals

### Data Layer

- **`DuckDBProcessorR6`** — DuckDB-backed staging for performance; CSV → Parquet → in-memory queries via `query_neuropsych()`. See [[concepts/duckdb-data-staging]] and [[concepts/parquet-as-knowledge-store]].

### Supporting Infrastructure

`ConfigManagerR6`, `ScoreTypeCacheR6`, `ErrorHandlerR6`, `ExecutionTrackerR6`, `TemplateContentManagerR6`, `PackageManagerR6`

### Workflow Glue

Lives in `R/workflow_*.R`: `workflow_setup`, `workflow_config`, `workflow_data_processor`, `workflow_domain_generator`, `workflow_report_generator`, `workflow_utils`. Two competing entry points exist (`Cingulate2MainR6.R` and `cingulateMainR6`) — verify which is active before adding new code.

## LLM Role System

The engine uses a role-based routing system where every prompt is dispatched by clinical role rather than by model name directly. Roles are mapped to model IDs and fallbacks in `inst/config/ollama_models.yml`. The system supports **3 performance modes**: `development` (fast iteration), `balanced`, and `production` (final reports). See [[concepts/role-based-llm-routing]].

### Available Roles

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

Examples from `inst/config/ollama_models.yml`:
- `summarize_domain` → `hf.co/Raymond-dev-546730/ClinicalThought-AI-8B:F16` with a Q8 fallback, `temperature: 0.7`
- `integrate_evaluation` → `alibayram/medgemma:27b` with `hf.co/mradermacher/Meditron3-Phi4-14B-GGUF:Q8_0` fallback, `max_tokens: 128000`
- **MedGemma 4B**: used for domain-level summaries
- **Luria/Qwen 27B**: used for cross-domain synthesis
- Models are selected per task and per performance mode from `inst/config/ollama_models.yml`
- Always verify model availability with `check_available_models()` before running workflows that hit Ollama

### LLM Quick Start

```r
# Initialize the LLM system
source("R/OllamaModelRouterR6.R")
source("R/DomainPrompterR6.R")
source("R/ReportLLMBridgeR6.R")

router <- OllamaModelRouterR6$new("inst/config/ollama_models.yml")
bridge <- ReportLLMBridgeR6$new(router, out_dir = "artifacts/patient_123", seed = 42)
prompter <- DomainPrompterR6$new(style = "clinical")

# Generate domain narrative
msgs <- prompter$build_messages(
  domain = "Academics",
  data = list(reading_fluency = list(ss = 75, pct = 5, range = "Below Average")),
  task = "summarize_domain"
)
result <- bridge$run_stage("summarize_domain", "01_academics", messages = msgs)
```

### Task-Specific Model Selection

```r
# Task-specific model selection across 3 performance modes
model <- get_model_for_task("domain_summary", "production")
response <- query_ollama_enhanced(prompt, task = "rating_scales", mode = "balanced")

# Available tasks: domain_summary, rating_scales, overall_summary,
# recommendations, differential_dx, quick_interpret
```

### Migration from Old System

| Old Pattern | New Pattern |
|---|---|
| `make_mega(data, ...)` | `bridge$run_stage("summarize_domain", "01_domain", messages = msgs)` |
| `make_mega_for_sirf(...)` | `bridge$run_stage("integrate_evaluation", "04_sirf", prompt = "...")` |
| Hardcoded prompts | `DomainPrompterR6$build_messages()` with templates |
| Manual model selection | `OllamaModelRouterR6` auto-routes by role |

## Data Loading: CSV → DuckDB → Parquet

```r
load_data_duckdb(
  file_path     = "data-raw/csv",    # dir with neurocog.csv, neurobehav.csv, validity.csv
  output_dir    = "data",
  output_format = "all",             # "csv" | "parquet" | "arrow" | "all"
  patient       = "John Doe"
)
```

Reads all CSVs via DuckDB, computes z-scores, splits by `test_type` into three tables:
- `data/neurocog.parquet` — cognitive domains (IQ, academics, verbal, spatial, memory, executive, motor, social)
- `data/neurobehav.parquet` — behavioral domains (ADHD, emotion, adaptive, daily living)
- `data/validity.parquet` — validity indicators

This aligns with the broader [[concepts/clinical-data-management]] strategy of keeping data portable, auditable, and privacy-safe without external service dependencies.

## Domain Coverage

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

ADHD and emotion domains support multi-rater inputs (self, parent, teacher) via `factory$create_multi_processor()`. Always call `check_rater_data_exists()` / `check_domain_raters()` — child vs adult reports diverge here.

## Domain QMD Generation Pattern

Reports are assembled from numbered per-domain [[concepts/quarto]] includes:

1. **Text first**: `generate_domain_text_qmd()` writes `_02-XX_domain_text.qmd` (the narrative)
2. **QMD shell second**: the `_02-XX_domain.qmd` file includes the text file and adds tables/plots
3. **Multi-rater domains** (emotion, ADHD): produce separate per-rater files (self/parent/teacher)
4. **Stable numbering convention**: `01_iq`, `02_academics`, `03_verbal`, etc. — do not renumber casually

> **Critical ordering rule**: Always call `generate_domain_text_qmd()` BEFORE generating domain QMD files. Check `check_rater_data_exists()` before creating rater-specific files.

### Prompt-to-QMD Mapping

Each domain maps a prompt keyword to a template file and a target QMD section:

| Keyword | Prompt file | Target QMD |
|---|---|---|
| `proiq` | `inst/prompts/proiq.qmd` | `_02-01_iq_text.qmd` |
| `proacad` | `inst/prompts/proacad.qmd` | `_02-02_academics_text.qmd` |
| `proverb` | `inst/prompts/proverb.qmd` | `_02-03_verbal_text.qmd` |
| `provis` | `inst/prompts/provis.qmd` | `_02-04_spatial_text.qmd` |
| `promem` | `inst/prompts/promem.qmd` | `_02-05_memory_text.qmd` |
| `proexe` | `inst/prompts/proexe.qmd` | `_02-06_executive_text.qmd` |
| `promot` | `inst/prompts/promot.qmd` | `_02-07_motor_text.qmd` |
| `prosoc` | `inst/prompts/prosoc.qmd` | `_02-08_social_text.qmd` |
| `proadhd` | `inst/prompts/proadhd.qmd` | `_02-09_adhd_text.qmd` |
| `proemo` | `inst/prompts/proemo.qmd` | `_02-10_emotion_text.qmd` |
| `proadapt` | `inst/prompts/proadapt.qmd` | `_02-11_adaptive_text.qmd` |
| `pronse` | `inst/prompts/pronse.qmd` | `_01-00_nse.qmd` (behavioral obs) |
| `prosirf` | `inst/prompts/prosirf.qmd` | `_03-00_sirf.qmd` (integrated summary) |
| `prorecs` | `inst/prompts/prorecs.qmd` | `_03-01_recs.qmd` (recommendations) |

The system has **two-stage rendering with edit protection**: regenerating QMDs may overwrite manual edits. See [[concepts/edit-protection-pattern]].

## Artifact Pipeline

Every `ReportLLMBridgeR6$run_stage()` call produces:

| File | Content |
|---|---|
| `artifacts/.../01_memory.md` | Generated narrative |
| `artifacts/.../01_memory.meta.json` | Model, timing, QC scores |
| `artifacts/.../run.log.jsonl` | Append-only audit trail |
| `artifacts/.../.cache/<hash>.rds` | Content-addressed cache |

Content-addressed caching means summaries survive re-runs unchanged, avoiding redundant LLM calls. See [[concepts/artifact-caching-pipeline]].

## Parallel and Batch Execution

```r
# Full LLM + render in one call
cingulate_run_llm_then_render(
  render_paths = "template.qmd",
  parallel     = TRUE,
  n_cores      = 6
)

# LLM generation only
run_llm_for_all_domains_parallel(
  prompts_dir = "inst/prompts",
  domains     = c("proiq", "promem", "proexe", "proadhd"),
  backend     = "ollama",
  n_cores     = 4
)
```

Key functions in `R/cingulate_llm.R`: `run_llm_for_all_domains()`, `run_llm_for_all_domains_parallel()`, `cingulate_run_llm_then_render()`, `generate_domain_summary_from_master()`, `get_model_config()`, `check_available_models()`.

## Connecting to the Report Render

1. Run `load_data_duckdb()` to stage the patient CSV exports as parquet/DuckDB files.
2. Generate/refresh the `_02-XX_*` QMD scaffolding with `cingulate_workflow(..., generate_report = FALSE)`.
3. Call `cingulate_run_llm_then_render(render_paths = "template.qmd", parallel = TRUE, n_cores = 6)` to:
   - Execute `run_llm_for_all_domains_parallel()` (which calls `ReportLLMBridgeR6$run_stage()` per domain)
   - Render `template.qmd` to the Typst profile configured in `_quarto.yml`

This keeps CI/CD or automation scripts simple — one function handles both generation and rendering once the models are warmed up.

## High-Level Orchestration Entry Points

| Use case | Function | File |
|---|---|---|
| Full workflow (data + domains + LLM + report) | `cingulate_workflow()` | `R/cingulateMainR6.R` |
| Quick new-user setup | `cingulate_quick_start(patient, age)` | `R/cingulateMainR6.R` |
| Create scaffolded patient workspace | `create_patient_workspace(name, age)` | `R/cingulate-package.R` |
| Upload CSVs or extract from PDF | `upload_files()` | `R/file_upload_helpers.R` |
| Full shell-based setup | `bash inst/scripts/report_setup.sh` | `inst/scripts/` |
| Render existing QMDs to PDF | `quarto render template.qmd` | CLI |

### Shell Entry Points

```fish
# Interactive shell workflow (recommended)
./unified_neuropsych_workflow.sh "Patient Name"

# Programmatic R workflow
Rscript inst/scripts/00_complete_neuropsych_workflow.R

# Quick convenience functions
Rscript -e "library(cingulate); run_workflow()"
Rscript -e "library(cingulate); quick_rerender()"
```

## Agent Task Quick Reference

| Agent task | What to call |
|---|---|
| Route prompt to model by clinical role | `router$run(prompt, role)` |
| Build structured messages from test scores | `prompter$build_messages(domain, data, task)` |
| Orchestrate a stage with artifacts + cache | `bridge$run_stage(role, stage_id, messages)` |
| Run all stages for a patient sequentially | `bridge$run_pipeline(stages)` |
| Inject LLM text into a QMD file | `NeuropsychResultsR6$process(llm=TRUE, domain_keyword)` |
| Get available models from Ollama | `check_available_models(models, backend="ollama")` |
| Get recommended models by performance tier | `get_model_config(section="production", tier="primary")` |
| Extract scores from an existing PDF | `upload_files(method="pdf", test_type="wisc5")` |
| Validate domain data before processing | `factory$validate_domain_data(domain_key, detailed=TRUE)` |
| Get usage / token logs | `log_llm_usage()`, `llm_usage_log()` |
| Load config from YAML | `ConfigManagerR6$new("config.yml")` |
| Safe execution with logging | `ErrorHandlerR6$safe_execute(expr, context)` |

## Repository Layout

| Path | Role |
|---|---|
| `R/` | R6 classes, workflow runners, scoring helpers, Quarto utilities |
| `inst/quarto/` | Quarto/Typst report templates (neurotyp adult/pediatric/forensic) |
| `inst/rmarkdown/` | R Markdown skeleton templates |
| `inst/prompts/` | LLM prompt templates (`pro*.qmd`) |
| `inst/config/` | Model routing YAML (`ollama_models.yml`) |
| `inst/scripts/` | Maintenance and batch generation scripts |
| `inst/instructions/` | Architecture and integration documentation |
| `tests/` | Automated tests |
| `pyproject.toml` | Optional Python tooling (LiteLLM, PyMuPDF via `uv`) |

## Data Formats

The engine deliberately avoids spreadsheet formats (XLSX, Google Sheets) in favor of columnar and flat formats:

| Format | Use |
|---|---|
| CSV | Human-readable interchange; canonical input at `data-raw/csv/` (gitignored) |
| Arrow | In-memory columnar processing |
| Parquet | Compressed on-disk storage |
| DuckDB | Staged queries over Parquet for performance |

## Score-Type Handling

Footnotes and z-score conversions branch on whether a row is `t_score`, `scaled_score`, or `standard_score`. `ScoreTypeCacheR6` caches this detection. When adding a new test, its score type must be recognized — see `R/score_type_utils.R` and `R/neuropsych_test_scoring.R`. See also [[concepts/neuropsychological-test-scores]].

## Report Templates

- `_quarto.yml` renders `template.qmd` using the **`neurotyp-pediatric-typst`** format from the project-local extension at `inst/quarto/_extensions/brainworkup/`
- Adult, pediatric, and forensic variants live in `inst/quarto/` (see [[concepts/forensic-neuropsychological-evaluation]])
- `inst/rmarkdown/` skeletons are legacy/secondary
- [[concepts/typst-typesetting]] is the primary typesetting backend

## Package Development

This is an **R package** (requires R >= 4.5), not just a script collection:

- Use `devtools::load_all('.')` for rapid iteration, not `source()`
- Documentation via roxygen2: `devtools::document()` regenerates man pages
- Tests in `tests/testthat/` run via `devtools::test()`
- Package namespace managed in `NAMESPACE` (auto-generated, never edit directly)
- Internal data in `R/sysdata.rda` loaded via `load_scales_internal()`

```fish
# Fast edit-test cycle
Rscript -e "devtools::load_all('.')"
Rscript -e "devtools::test()"

# Generate docs and check package
Rscript -e "devtools::document()"
Rscript -e "devtools::check()"

# Render reports locally
quarto render
```

## Key Files an Agent Should Always Know

| File | Purpose |
|---|---|
| `inst/config/ollama_models.yml` | Role → model mapping (edit to change models) |
| `inst/prompts/pro*.qmd` | Domain prompt templates |
| `inst/prompts/{domain}_prompt.txt` | Domain-specific LLM prompt templates with `{{DATA}}` placeholder |
| `inst/quarto/templates/typst-report/config.yml` | Patient / report config defaults |
| `inst/quarto/templates/typst-report/_quarto.yml` | Quarto + Typst format config |
| `inst/instructions/LLM_INTEGRATION.md` | Quick-start for the LLM system |
| `inst/instructions/copilot-instructions.md` | Agent quick-start protocol |
| `R/OllamaModelRouterR6.R` | Model gateway (all HTTP calls go here) |
| `R/DomainPrompterR6.R` | Prompt builder |
| `R/ReportLLMBridgeR6.R` | Orchestration + artifacts |
| `R/cingulate_llm.R` | High-level helpers + batch functions |
| `R/NeuropsychResultsR6.R` | Domain data → QMD injection |

## Python Sidecar

A small Python sidecar (managed via `uv sync`) provides LiteLLM, PyMuPDF, and mlx-* support for tasks that are not required for core R-only workflows. This is only needed for certain scripts, not the core R pipeline. See [[concepts/r-python-integration]] and [[concepts/python-environment-management]].

## Common Pitfalls

| Pitfall | Detail |
|---|---|
| **Ollama must be running** | Start with `ollama serve` before any LLM calls |
| **Models must be pulled** | Run `ollama pull <model>` if model not found |
| **JEP 66 handshake timeout** | Never auto-load `library(cingulate)` in `.Rprofile`; [[concepts/positron-ide]] will time out on 40+ heavy imports |
| **`source()` won't work** | R6 generators register on package load; use `devtools::load_all('.')` |
| **R6 class initialization slow** | Positron has ~30–60s handshake timeout; use `.rs.restartR()` then `devtools::load_all('.')` after startup |
| **Corrupted package cache** | Clear with `rm -rf ~/.cache/R/` then reinstall via `devtools::install()` |
| **Input path ambiguity** | Some helpers accept paths with/without `data/` prefix; `data-raw/csv/` is canonical but gitignored |
| **Internal scales data** | In `R/sysdata.rda`; use `load_scales_internal()` / `safe_use_data_internal()` |
| **LLM model availability** | Verify with `check_available_models()`; use `mode = "development"` for fast iteration |
| **Legacy functions** | DO NOT use `make_mega()` or `make_mega_for_sirf()` — these are deprecated |
| **Output directory** | Bridge creates `artifacts/` directory automatically |
| **Structural drift** | Reorganizations can resurrect old code; two competing main entry points exist — verify which is active |
| **File generation order** | Always generate domain text QMD before domain QMD files |
| **Edit protection** | Two-stage rendering may overwrite manual edits — always check before regenerating |

## AI Agent Quick Start Protocol

1. **Understand the flow**: Read `R/cingulateMainR6.R` and `inst/scripts/00_complete_neuropsych_workflow.R`
2. **Test the pipeline**: Run `devtools::load_all('.')` then `run_workflow()` to see the system in action
3. **Check dependencies**: Ensure Ollama models are available (`bash setup_ollama.sh`)
4. **Modify carefully**: This system has **two-stage rendering** with edit protection — always check if files are manually edited before regenerating

## Quick Start

```r
create_patient_workspace("ExamplePatient", age = 12)
results <- process_all_domains("data", age_group = "child")
generate_assessment_report(
  results,
  patient_info = list(name = "ExamplePatient", age = 12)
)
```

## End-to-End Agent Script Template

```r
library(cingulate)  # or devtools::load_all(".")

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

## Configuration

1. Install R package and dependencies (DuckDB, ggplot2, quarto, rmarkdown, R6, etc.)
2. Configure DuckDB connection pointing to your data store
3. Customize Quarto/R Markdown templates from `inst/quarto/` and `inst/rmarkdown/` as needed
4. Set environment variables (e.g., `REPORT_OUTPUT_DIR`, `OLLAMA_URL`, `MLX_URL`)
5. Verify Ollama model availability before LLM-assisted workflows

See [[concepts/yaml-configuration]] for model routing configuration via `ollama_models.yml`.

## Relationship to Subagents

The cingulate engine is the downstream consumer of outputs from the subagent architecture — specifically agents like the Neuropsych Data Extractor and PDF Ingestion Parser. Extracted scores and structured records flow into the engine before being shaped into final reports via [[concepts/narrative-report-generation]].

## Related Concepts

- [[concepts/r6-class-architecture]] — Foundational design pattern for all engine classes
- [[concepts/domain-processor-pattern]] — Per-domain processing abstraction
- [[concepts/role-based-llm-routing]] — Role-based model dispatch via YAML config
- [[concepts/artifact-caching-pipeline]] — Content-addressed stage caching
- [[concepts/llm-provider-abstraction]] — Prompt builder and backend abstraction
- [[concepts/quarto]] — Primary report rendering layer
- [[concepts/typst-typesetting]] — Typesetting backend used with Quarto
- [[concepts/local-llm-inference]] — LLM narrative generation within the engine
- [[concepts/clinical-data-management]] — Broader data governance context
- [[concepts/neuropsychological-assessment-pipeline]] — End-to-end workflow the engine anchors
- [[concepts/pdf-score-extraction]] — Key input source for engine data
- [[concepts/edit-protection-pattern]] — Two-stage QMD rendering safety
- [[concepts/parquet-as-knowledge-store]] — On-disk storage format
- [[concepts/duckdb-data-staging]] — High-performance data staging layer
- [[concepts/r-python-integration]] — Python sidecar interoperability
- [[concepts/yaml-configuration]] — Model routing configured via `ollama_models.yml`
- [[concepts/neuropsychological-assessment-automation]] — Broader automation context
- [[concepts/narrative-report-generation]] — LLM-assisted text generation for reports
- [[concepts/forensic-neuropsychological-evaluation]] — Forensic report variant support
- [[concepts/neuropsychological-test-scores]] — Score type detection and caching
- [[concepts/positron-ide]] — IDE with known package-load timing constraints
- [[concepts/python-environment-management]] — Python sidecar management via uv

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/neuropsych-data-extractor]]

See also: [[summaries/neuropsych-narrative-writer]]

See also: [[summaries/PROJECT_SETUP_COMPLETE]]

See also: [[summaries/README]]

See also: [[summaries/SESSION_SUMMARY_2025-04-28]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/copilot-instructions]]