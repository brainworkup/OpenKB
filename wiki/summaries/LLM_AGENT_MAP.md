---
doc_type: short
full_text: sources/LLM_AGENT_MAP.md
---

# LLM Agent Orchestration Map — Summary

## Overview

This document is a comprehensive reference guide for building and orchestrating LLM agents using the `cingulate` R package. It covers the full pipeline from environment setup and data loading through LLM-driven narrative generation and report rendering for neuropsychological evaluations. It connects directly to the [[concepts/neuropsychological-assessment-pipeline]] and [[concepts/cingulate-engine]].

## Core Architecture

The system is composed of several interconnected R6 classes and helper functions (see [[concepts/r6-class-architecture]]):

- **`OllamaModelRouterR6`** — Model gateway that routes prompts to specific LLM models based on clinical *roles*, configured via `inst/config/ollama_models.yml`. Supports Ollama and MLX backends. This implements [[concepts/role-based-llm-routing]] and [[concepts/llm-provider-abstraction]].
- **`DomainPrompterR6`** — Builds structured OpenAI-style message lists from neuropsych test score data, supporting `clinical`, `parent`, and `school` output styles. Relates to [[concepts/clinical-communication-register]] and [[concepts/dual-audience-design]].
- **`ReportLLMBridgeR6`** — Orchestration layer that runs individual pipeline stages, writes artifacts (`.md`, `.meta.json`, `.jsonl`), and manages content-addressed caching. See [[concepts/artifact-caching-pipeline]] and [[concepts/agent-pipeline-state-management]].
- **`DomainProcessorFactoryR6`** / **`DomainProcessorR6`** — Factory pattern for creating domain-specific data processors, including multi-rater support for ADHD and emotion domains. See [[concepts/domain-processor-pattern]].
- **`NeuropsychResultsR6`** — Connects processed domain data to Quarto (`.qmd`) report files, injecting LLM-generated summaries directly into the document. See [[concepts/quarto]] and [[concepts/narrative-report-generation]].

## Pipeline Stages

1. **Bootstrap** — Initialize workspace via `setup_cingulate()` or `init_cingulate_workspace()`. See [[concepts/per-patient-workspace]].
2. **Data Loading** — `load_data_duckdb()` reads CSVs, computes z-scores, and outputs Parquet/Feather/CSV files split into `neurocog`, `neurobehav`, and `validity` tables. See [[concepts/duckdb-data-staging]] and [[concepts/parquet-as-knowledge-store]].
3. **LLM Router Setup** — Instantiate `OllamaModelRouterR6` with a [[concepts/yaml-configuration]] file mapping roles to model IDs and fallbacks.
4. **Prompt Building** — `DomainPrompterR6$build_messages()` formats test scores into structured prompts.
5. **Stage Execution** — `ReportLLMBridgeR6$run_stage()` runs a prompt through the router and saves all artifacts.
6. **Domain Batch Processing** — `DomainProcessorFactoryR6` handles all [[concepts/cognitive-domains]] in batch, with optional parallel execution.
7. **QMD Injection** — `NeuropsychResultsR6$process()` injects LLM output into Quarto report files using `LLM_CONTEXT_START…END` markers. See [[concepts/edit-protection-pattern]].
8. **Parallel Orchestration** — `run_llm_for_all_domains_parallel()` and `cingulate_run_llm_then_render()` run all domains concurrently. See [[concepts/multi-agent-orchestration]].

## LLM Roles

The router supports 13 clinical roles including:
- `summarize_domain`, `interpret_rating_scales`, `quick_interpret`
- `integrate_evaluation`, `generate_recommendations`, `differential_diagnosis`
- `overall_summary`, `detailed_interpretation`, `finalize_diagnostic_summary`
- `generate_narrative`, `embeddings`, `fallback_general`

The `fallback_general` role implements [[concepts/fallback-strategy]] for unmatched prompts.

## Domain Coverage

Thirteen neuropsychological domains are supported, sourced from `neurocog.parquet` (IQ, academics, verbal, spatial, memory, executive, motor, social) and `neurobehav.parquet` (ADHD, emotion, adaptive, daily living), plus validity data. ADHD and emotion domains support multi-rater inputs (self, parent, teacher). Score data conforms to [[concepts/neuropsychological-test-scores]] and [[concepts/long-format-clinical-data]].

## Prompt-to-QMD Mapping

Each domain has a corresponding prompt keyword (e.g., `promem` → `_02-05_memory_text.qmd`) and a prompt template file (`inst/prompts/pro*.qmd`). Special sections include behavioral observations (`pronse`), integrated summary (`prosirf`), and recommendations (`prorecs`). This mapping underpins the [[concepts/modular-report-architecture]] of the reporting system.

## Key Design Patterns

- **Content-addressed caching** — Bridge stages cache results by hash, surviving re-runs unchanged. See [[concepts/artifact-caching-pipeline]].
- **Artifact trail** — Every stage produces a narrative `.md`, metadata `.meta.json`, and appends to an audit `.jsonl` log.
- **Role-based routing** — Model selection is decoupled from prompt logic via YAML config. See [[concepts/role-based-llm-routing]].
- **Factory pattern** — `DomainProcessorFactoryR6` centralizes domain processor creation with validation. See [[concepts/domain-processor-pattern]].
- **Parallel execution** — Batch LLM calls can be parallelized across CPU cores.
- **Local-first inference** — The Ollama and MLX backends support [[concepts/local-llm-inference]] and [[concepts/local-first-architecture]].

## High-Level Entry Points

| Function | Purpose |
|---|---|
| `cingulate_workflow()` | Full end-to-end pipeline |
| `cingulate_quick_start()` | New-user onboarding |
| `create_patient_workspace()` | Scaffold patient directory |
| `cingulate_run_llm_then_render()` | LLM generation + Quarto render |

## Related Concepts
- [[concepts/neuropsychological-assessment-automation]]

- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/r6-class-architecture]]
- [[concepts/role-based-llm-routing]]
- [[concepts/narrative-report-generation]]
- [[concepts/quarto]]
- [[concepts/domain-processor-pattern]]
- [[concepts/artifact-caching-pipeline]]
- [[concepts/multi-agent-orchestration]]
- [[concepts/cingulate-engine]]
- [[concepts/cognitive-domains]]
- [[concepts/local-llm-inference]]