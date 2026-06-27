---
doc_type: short
full_text: sources/LLM_INTEGRATION.md
---

# LLM Integration Guide for cingulate

**Source:** LLM_INTEGRATION
**Added:** November 2025

## Overview

This guide describes the new LLM integration architecture for the `cingulate` R package. Three [[concepts/r6-class-architecture]] replace the deprecated `make_mega()` and `make_mega_for_sirf()` functions, providing a modular system for generating clinical narrative reports via locally-hosted [[concepts/local-llm-inference]] models served through [[concepts/openai-compatible-api]].

## Core Components

### Three R6 Classes

| Class | File | Responsibility |
|---|---|---|
| `OllamaModelRouterR6` | `R/OllamaModelRouterR6.R` (170 lines) | Model routing and API calls |
| `DomainPrompterR6` | `R/DomainPrompterR6.R` (220 lines) | Prompt template management |
| `ReportLLMBridgeR6` | `R/ReportLLMBridgeR6.R` (280 lines) | Workflow orchestration and artifact management |

## Model Configuration

Roles are defined in `inst/config/ollama_models.yml` with primary models and fallbacks, following a [[concepts/role-based-llm-routing]] pattern:

- `summarize_domain` → `hf.co/Raymond-dev-546730/ClinicalThought-AI-8B:F16` (Q8 fallback, temperature: 0.7)
- `integrate_evaluation` → `alibayram/medgemma:27b` (Meditron3-Phi4-14B fallback, max_tokens: 128000)
- `generate_recommendations`, `quick_interpret`, and other roles follow the same layout

Per-call overrides are possible via `bridge$run_stage(..., options = list(temperature = 0.1))`. The fallback strategy aligns with the [[concepts/fallback-strategy]] pattern used elsewhere in the system.

## Prompt Templates

- Domain-specific prompts stored in `inst/prompts/{domain}_prompt.txt`
- Use `{{DATA}}` placeholder for data injection
- Two styles: `"clinical"` (narrative) and `"technical"` (professional)
- `DomainPrompterR6$build_messages()` handles template loading and data formatting
- Configuration managed through [[concepts/yaml-configuration]]

## Output Artifacts

Each stage run by `bridge$run_stage()` produces:

- `artifacts/patient_XXX/{stage_id}.md` — raw Markdown narrative
- `artifacts/patient_XXX/{stage_id}.meta.json` — model, role, timing, QC info
- `artifacts/patient_XXX/run.log.jsonl` — append-only audit log

Markdown content is injected into corresponding `_text.qmd` includes for [[concepts/quarto]] report rendering, following the [[concepts/modular-report-architecture]] pattern.

## Critical Workflow Pattern

```r
# Initialize once per patient
router <- OllamaModelRouterR6$new("inst/config/ollama_models.yml")
bridge <- ReportLLMBridgeR6$new(router, out_dir = "artifacts/patient_456", seed = 123)
prompter <- DomainPrompterR6$new(style = "clinical")

# Run domain summaries
domains <- c("Cognitive", "Academics", "Executive", "Memory", "Language")
for (domain in domains) {
  msgs <- prompter$build_messages(domain = domain, data = load_domain_data(domain), task = "summarize_domain")
  bridge$run_stage("summarize_domain", sprintf("01_%s", tolower(domain)), messages = msgs)
}

# Integration stage
bridge$run_stage("integrate_evaluation", "04_integrated", prompt = "Integrate findings...")
```

The domain loop reflects the [[concepts/domain-processor-pattern]], iterating over [[concepts/cognitive-domains]] such as Cognitive, Academics, Executive, Memory, and Language.

## Report Render Pipeline

1. `load_data_duckdb()` — stage patient CSV exports as parquet/DuckDB files (see [[concepts/duckdb-data-staging]])
2. `cingulate_workflow(..., generate_report = FALSE)` — generate QMD scaffolding
3. `cingulate_run_llm_then_render(render_paths = "template.qmd", parallel = TRUE, n_cores = 6)` — runs LLM generation and Typst rendering

This single-function call handles both LLM generation and [[concepts/typst-typesetting]] rendering for CI/CD pipelines, producing output via the [[concepts/narrative-report-generation]] workflow.

## Migration from Old System

| Old Pattern | New Pattern |
|---|---|
| `make_mega(data, ...)` | `bridge$run_stage("summarize_domain", "01_domain", messages = msgs)` |
| `make_mega_for_sirf(...)` | `bridge$run_stage("integrate_evaluation", "04_sirf", prompt = "...")` |
| Hardcoded prompts | `DomainPrompterR6$build_messages()` with templates |
| Manual model selection | `OllamaModelRouterR6` auto-routes by role |

## Common Pitfalls

1. Ollama must be running (`ollama serve`) before any LLM calls
2. Models must be pulled first (`ollama pull <model>`)
3. `artifacts/` directory is created automatically by the bridge
4. `make_mega()` and `make_mega_for_sirf()` are **deprecated** — do not use

## When to Use Each Component

- **OllamaModelRouterR6**: Direct API calls, custom model configs, non-standard tasks
- **DomainPrompterR6**: Template-based prompts, data formatting
- **ReportLLMBridgeR6**: Standard report workflows, artifact management — use for all report generation

## Related Concepts
- [[concepts/cingulate-engine]]
- [[concepts/per-patient-workspace]]

- [[concepts/local-llm-inference]] — Local LLM serving infrastructure
- [[concepts/r6-class-architecture]] — R6 object-oriented design pattern in R
- [[concepts/quarto]] — Quarto/Typst report generation pipeline
- [[concepts/cognitive-domains]] — Cognitive, Academics, Executive, Memory, Language domains
- [[concepts/role-based-llm-routing]] — Role-based model selection and dispatch
- [[concepts/narrative-report-generation]] — Automated clinical narrative generation
- [[concepts/neuropsychological-assessment-automation]] — Broader automation context
- [[concepts/modular-report-architecture]] — Modular QMD include-based report structure
- [[concepts/artifact-caching-pipeline]] — Artifact management and audit logging
- [[concepts/llm-provider-abstraction]] — Abstracting LLM provider details behind a routing layer