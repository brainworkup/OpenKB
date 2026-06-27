---
doc_type: short
full_text: sources/CLAUDE.md
---

# CLAUDE.md — Cingulate R Package Developer Guide

## Overview

`cingulate` is an **R package** (with a Python sidecar) that converts neuropsychological test data into publication-quality PDF reports. It is a proper package, not a script collection, and must be loaded via `devtools::load_all('.')` rather than `source()`.

### Pipeline
```
CSV (data-raw/csv/) → DuckDB/Parquet → R6 domain processors → per-domain QMD includes → Quarto/Typst → PDF
```

## Common Commands

- **Load package**: `Rscript -e "devtools::load_all('.')"` (never via `.Rprofile`)
- **Tests**: `Rscript -e "devtools::test()"`
- **Docs**: `Rscript -e "devtools::document()"` (regenerates `man/*.Rd` and `NAMESPACE` via roxygen2)
- **Render report**: `quarto render` (from project root)
- **Entry points**: `cingulate_quick_start()` / `cingulate_workflow()`
- **Python sidecar**: `uv sync` (LiteLLM, PyMuPDF, mlx-*)

Tests live as a flat file `tests/model_testing.R`, not under `tests/testthat/`.

## Architecture: R6-First Design

The system is [[concepts/r6-class-architecture]]-based. Key classes:

### Core Pipeline Classes
- **`WorkflowRunnerR6`** — top-level orchestrator for CSV→PDF pipeline
- **`DomainProcessorR6` + `DomainProcessorFactoryR6`** — one processor per cognitive domain (IQ, academics, memory, attention/executive, language, sensorimotor, social-emotional); factory auto-detects domains; see [[concepts/domain-processor-pattern]]
- **`NeuropsychResultsR6`** — converts processed scores into clinical narrative text

### Rendering Components
- **`TableGTR6`** — gt tables
- **`DotplotR6`** — dot plot visualizations
- **`DrilldownR6`** — Highcharts drilldown visuals

### Data Layer
- **`DuckDBProcessorR6`** — [[concepts/parquet-as-knowledge-store]] staging for performance; CSV → Parquet → in-memory queries via `query_neuropsych()`

### LLM Integration
- **`OllamaModelRouterR6` / `ReportLLMBridgeR6` / `DomainPrompterR6`** — [[concepts/narrative-report-generation]] routing
- Models selected per task and per performance mode (`development` / `balanced` / `production`) from `inst/config/ollama_models.yml`; see [[concepts/yaml-configuration]]
- Tasks: `domain_summary`, `rating_scales`, `overall_summary`, `recommendations`, `differential_dx`, `quick_interpret`

### Infrastructure
- `ConfigManagerR6`, `ScoreTypeCacheR6`, `ErrorHandlerR6`, `ExecutionTrackerR6`, `TemplateContentManagerR6`, `PackageManagerR6`

### Workflow Glue
Lives in `R/workflow_*.R`: `workflow_setup`, `workflow_config`, `workflow_data_processor`, `workflow_domain_generator`, `workflow_report_generator`, `workflow_utils`.

Two competing entry points exist: `Cingulate2MainR6.R` and `cingulateMainR6` — verify which is active before adding code.

## Domain QMD Generation Pattern

Reports assembled from numbered per-domain [[concepts/quarto]] includes:

1. **Text first**: `generate_domain_text_qmd()` → `_02-XX_domain_text.qmd`
2. **QMD shell second**: `_02-XX_domain.qmd` includes text file, adds tables/plots
3. **Multi-rater domains** (emotion, ADHD): separate per-rater files (self/parent/teacher); use `check_rater_data_exists()` / `check_domain_raters()`; child vs adult diverge
4. **Stable numbering convention**: `01_iq`, `02_academics`, `03_verbal`, etc. — do not renumber

**Two-stage rendering with edit protection**: check for hand-edits before regenerating QMDs; see [[concepts/edit-protection-pattern]].

## Score-Type Handling

Footnotes and z-score conversions branch on `t_score`, `scaled_score`, or `standard_score`. `ScoreTypeCacheR6` caches detection. New tests must have recognized score types — see `R/score_type_utils.R` and `R/neuropsych_test_scoring.R`. See also [[concepts/neuropsychological-test-scores]].

## Report Templates

- `_quarto.yml` renders `template.qmd` using **`neurotyp-pediatric-typst`** format from `inst/quarto/_extensions/brainworkup/`; see [[concepts/quarto-extensions]] and [[concepts/typst-typesetting]]
- Adult/forensic variants also in `inst/quarto/`; see [[concepts/forensic-neuropsychological-evaluation]]
- `inst/rmarkdown/` skeletons are legacy/secondary

## Key Pitfalls

| Pitfall | Detail |
|---|---|
| **JEP 66 handshake timeout** | Never auto-load `library(cingulate)` in `.Rprofile`; [[concepts/positron-ide]] will time out on 40+ heavy imports |
| **`source()` won't work** | R6 generators register on package load; use `devtools::load_all('.')` |
| **Input path ambiguity** | Some helpers accept paths with/without `data/` prefix; `data-raw/csv/` is canonical but gitignored |
| **Internal scales data** | In `R/sysdata.rda`; use `load_scales_internal()` / `safe_use_data_internal()` |
| **LLM model availability** | Verify with `check_available_models()`; use `mode = "development"` for iteration; see [[concepts/local-llm-inference]] |

## Reference Docs in Repo

- `inst/instructions/copilot-instructions.md` — AI-agent guide; see [[concepts/ide-ai-assistant-configuration]]
- `inst/instructions/LLM_INTEGRATION.md` — [[concepts/llm-provider-abstraction]] routing pattern
- `inst/instructions/LLM_AGENT_MAP.md` — [[concepts/multi-agent-orchestration]] agent topology
- `inst/instructions/cingulate_architecture_map.pdf` — architecture diagram
- `.claude/agents/` — [[concepts/subagent-architecture]] definitions

## Related Concepts
- [[concepts/cingulate-engine]]
- [[concepts/modular-report-architecture]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/cognitive-domains]]
- [[concepts/long-format-clinical-data]]
- [[concepts/r-python-integration]]
- [[concepts/neuropsychological-assessment-pipeline]]
