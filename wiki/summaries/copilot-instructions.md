---
doc_type: short
full_text: sources/copilot-instructions.md
---

# Cingulate AI Agent Guide

## Overview

`cingulate` is an R package that transforms CSV neuropsychological test data into professional PDF reports. The pipeline follows: **CSV → DuckDB/Parquet → R6 Processing → QMD sections → Quarto/Typst → PDF**.

## Architecture

### Core Design
- **R6-first OOP** design with modular workflow components — see [[concepts/r6-class-architecture]]
- R6 classes handle orchestration, domain processing, and clinical text generation
- [[concepts/quarto]] and [[concepts/typst-typesetting]] used for publication-quality report rendering

### Key R6 Classes
| Class | Role |
|---|---|
| `WorkflowRunnerR6` | Orchestrates entire pipeline |
| `DomainProcessorR6` | Domain-specific data processing and QMD generation |
| `NeuropsychResultsR6` | Clinical narrative text from test scores |
| `TableGTR6` / `DotplotR6` | Data visualization components |

## Data Pipeline

1. **Input**: CSV files in `data-raw/csv/`
2. **Processing**: [[concepts/duckdb-data-staging]] for 4–5x performance boost via DuckDB/Parquet
3. **Domain mapping**: Standardized numbers (`01_iq`, `02_academics`, etc.)
4. **Score types**: Dynamic footnotes based on t_score/scaled_score/standard_score — see [[concepts/neuropsychological-test-scores]]
5. **Output**: PDF via [[concepts/quarto]] / [[concepts/typst-typesetting]]

## LLM Integration

An enhanced [[concepts/local-llm-inference]] system with **3 performance modes** (development / balanced / production):
- Task-specific model selection via `get_model_for_task()` — see [[concepts/role-based-llm-routing]]
- Configured in `inst/config/ollama_models.yml` using [[concepts/yaml-configuration]]
- Local inference via **Ollama**
- Available tasks: `domain_summary`, `rating_scales`, `overall_summary`, `recommendations`, `differential_dx`, `quick_interpret`

## Domain Processing Pattern

- **Text files** (`_02-XX_domain_text.qmd`) created by `generate_domain_text_qmd()` — see [[concepts/domain-processor-pattern]]
- **QMD files** (`_02-XX_domain.qmd`) include text files + generate tables/plots
- **Multi-rater domains** (emotion/ADHD) create separate files per rater (self/parent/teacher)
- Always generate domain text QMD **before** domain QMD files
- Two-stage rendering uses [[concepts/edit-protection-pattern]] — always check for manual edits before regenerating

## Entry Points & Commands

```fish
# Interactive shell
./unified_neuropsych_workflow.sh "Patient Name"

# Programmatic R
Rscript inst/scripts/00_complete_neuropsych_workflow.R

# Quick functions
Rscript -e "library(cingulate); run_workflow()"
Rscript -e "library(cingulate); quick_rerender()"
```

```r
# Development cycle
devtools::load_all('.')
devtools::test()
devtools::document()
devtools::check()
```

## Common Pitfalls

### R Session Startup (Critical)
- **"JEP 66 handshake failed"**: 40+ heavy dependencies cause timeout in [[concepts/positron-ide]]
- **Never** auto-load `library(cingulate)` in `.Rprofile`
- Always use `devtools::load_all('.')` in console *after* session starts
- Clear corrupted cache: `rm -rf ~/.cache/R/`

### File Generation Order
- Call `generate_domain_text_qmd()` **before** domain QMD files
- Check `check_rater_data_exists()` before rater-specific file creation
- Multi-rater domains need special child vs. adult pattern handling

### LLM Issues
- Check model availability with `check_available_models()`
- Use development mode for fast iteration, production for final reports

## Package Development

- R package (requires R >= 4.5) — part of the [[concepts/r-neuropsych-packages]] ecosystem
- Documentation via **roxygen2** (`devtools::document()`) — see [[concepts/documentation-as-code]]
- Tests in `tests/testthat/`
- Internal data in `R/sysdata.rda` via `load_scales_internal()`
- `NAMESPACE` and `man/*.Rd` are auto-generated — never edit directly
- Two-stage rendering with [[concepts/edit-protection-pattern]] — check for manual edits before regenerating

## Related Concepts
- [[concepts/cingulate-engine]]
- [[concepts/modular-report-architecture]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/narrative-report-generation]]
