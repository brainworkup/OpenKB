---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md]
brief: Route LLM tasks by semantic role instead of hardcoded model names.
---

# Role-Based LLM Routing

Role-based LLM routing is an architectural pattern in which prompts are dispatched to language models not by explicit model name at the call site, but by a **semantic role** — a named intent that maps to a model configuration defined externally. This decouples prompt logic from model selection, making it easy to swap, upgrade, or A/B-test models without changing application code.

A key refinement from later operational experience is that routing should optimize not only for best-case model quality, but for **consistency under real system load**. In local inference environments, especially on Apple Silicon, routing decisions can materially affect latency stability, output quality, and crash risk when large models compete for memory and cache resources. In practice, role-based routing is therefore both a semantic abstraction layer and a runtime control surface for [[concepts/local-inference-reliability]].

## Core Idea

Instead of calling a specific model directly:

```r
# Tightly coupled — fragile
ollama_call(model = "qwen3:14b", prompt = "Summarize memory scores...")
```

The caller declares a **role**:

```r
# Role-based — decoupled
router$run(prompt = "Summarize memory scores...", role = "summarize_domain")
```

The router resolves `summarize_domain` → a specific model ID, endpoint, fallback strategy, and potentially an execution tier using a YAML configuration file. The caller never needs to know which model is running.

This separation matters because the best model for a task is often not simply the most capable model in the abstract, but the one that delivers the most stable and readable output within system constraints. That makes role-based routing a practical implementation of [[concepts/llm-evaluation]] inside production code.

## Implementation in `cingulate`

The [[summaries/LLM_AGENT_MAP]] and [[summaries/LLM_INTEGRATION]] documents describe this pattern as implemented by `OllamaModelRouterR6` in the `cingulate` R package. The `copilot-instructions` guide (see [[summaries/copilot-instructions]]) further clarifies that the enhanced LLM system supports **3 performance modes** — development, balanced, and production — with task-specific model selection via `get_model_for_task()` and `query_ollama_enhanced()`. The November 2025 integration update formalized this architecture by replacing the deprecated `make_mega()` and `make_mega_for_sirf()` functions with three coordinated R6 classes.

### Configuration

All role-to-model mappings live in `inst/config/ollama_models.yml`. Each entry specifies:
- The **role name** (semantic intent)
- The **primary model ID**
- A **fallback model** for when the primary is unavailable
- Generation parameters such as `temperature` and `max_tokens`
- Endpoint routing (Ollama vs. MLX)
- Implicit operational policy about whether a role should prefer speed, depth, or runtime stability

Example mappings from the current configuration:
- `summarize_domain` → `hf.co/Raymond-dev-546730/ClinicalThought-AI-8B:F16` with a Q8 fallback and `temperature: 0.7`
- `integrate_evaluation` → `alibayram/medgemma:27b` with a Meditron3-Phi4-14B fallback and `max_tokens: 128000`

This follows the [[concepts/yaml-configuration]] pattern for externalizing runtime behavior. Per-call overrides are also supported via `bridge$run_stage(..., options = list(temperature = 0.1))`.

The newer operational insight is that configuration should also reflect hardware realities. For example, a role that performs well in isolation may become a poor default if it destabilizes the system when another large model is running. Routing tables therefore encode not just preferences about style or reasoning depth, but also lessons about [[concepts/concurrent-model-serving]], resource contention, and when to avoid heavyweight models during ingestion or long-context tasks.

### Performance Modes

The `copilot-instructions` document introduces a three-tier performance mode system that complements role-based routing:

| Mode | Use Case |
|---|---|
| `development` | Fast iteration during authoring and testing |
| `balanced` | Day-to-day report generation |
| `production` | Final, publication-quality reports |

Mode selection can be combined with role specification:

```r
model <- get_model_for_task("domain_summary", "production")
response <- query_ollama_enhanced(prompt, task = "rating_scales", mode = "balanced")
```

Available task names in this system include: `domain_summary`, `rating_scales`, `overall_summary`, `recommendations`, `differential_dx`, and `quick_interpret`.

The practical meaning of these modes is not only speed versus quality, but also **consistency versus peak intelligence**. For a demo, clinical workflow, or batch report run, a slightly smaller or more readable model may be preferable to a larger model that becomes unstable under memory pressure. In that sense, performance modes help express deployment policy as much as model preference.

### Router Initialization

```r
router <- OllamaModelRouterR6$new(
  cfg_path  = "inst/config/ollama_models.yml",
  endpoints = list(
    ollama = Sys.getenv("OLLAMA_URL", "http://127.0.0.1:11434"),
    mlx    = Sys.getenv("MLX_URL",    "http://127.0.0.1:8081")
  ),
  allow_pull = FALSE
)
```

The router can target either Ollama or MLX backends, making it an instance of [[concepts/llm-provider-abstraction]]. When MLX is used on Apple Silicon, backend choice also becomes part of operational tuning, since endpoint selection can interact with throughput, memory behavior, and system responsiveness.

## The Three-Class Architecture

The router operates as one of three coordinated R6 classes (see [[concepts/r6-class-architecture]]):

| Class | File | Responsibility |
|---|---|---|
| `OllamaModelRouterR6` | `R/OllamaModelRouterR6.R` (170 lines) | Model routing and API calls |
| `DomainPrompterR6` | `R/DomainPrompterR6.R` (220 lines) | Prompt template management |
| `ReportLLMBridgeR6` | `R/ReportLLMBridgeR6.R` (280 lines) | Workflow orchestration and artifact management |

Each class has a clear responsibility boundary:
- **`OllamaModelRouterR6`**: Use for direct API calls, custom model configs, or non-standard tasks
- **`DomainPrompterR6`**: Use for template-based prompts and data formatting
- **`ReportLLMBridgeR6`**: Use for all standard report workflows with artifact management

This separation helps isolate concerns that are easy to entangle in local LLM systems: prompt semantics, model choice, and workflow execution. The router decides *which* model should handle a role; prompt classes shape *how* the task is framed; bridge classes decide *when* and *under what execution conditions* calls should run.

## Defined Roles

The `cingulate` system defines 13 clinical roles:

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

This taxonomy reflects the stages of [[concepts/neuropsychological-reporting]] — from single-domain summaries through integrated impressions and recommendations.

The new benchmark evidence also suggests that role definitions should reflect differences in abstraction depth and communication style. Some roles require concise, operational explanation; others require richer synthesis. A model that is excellent at preserving conceptual hierarchy and readable middle-depth explanation may be a better fit for `generate_recommendations`, `overall_summary`, or `detailed_interpretation` than a model that is merely more technical or verbose.

## Fallback Strategy

The router implements a [[concepts/fallback-strategy]]: if the primary model for a role is unavailable (not pulled, endpoint down), it automatically tries the configured fallback model. The `fallback_general` role itself serves as a last-resort handler for any prompt that does not match a specific role. Model availability can be checked at runtime via `check_available_models(models, "ollama")`.

Operationally, fallback should also be understood more broadly than endpoint failure. In local inference settings, a primary model may be *available* but still be a poor runtime choice because of memory pressure, context pressure, or contention with another large model. A mature router can therefore treat fallback as both a failure-recovery mechanism and a stability-preservation mechanism.

This is especially important where concurrent heavyweight reasoning models can degrade one another through unified memory contention, cache pressure, prefill overlap, or speculative decoding interference. In such cases, routing a role to a smaller fallback may improve total workflow quality even if the fallback is nominally weaker.

## Integration with the Broader Pipeline

Role-based routing does not stand alone — it is the first layer of a larger [[concepts/multi-agent-orchestration]] stack:

1. **Router** resolves role → model and executes the HTTP call
2. **`ReportLLMBridgeR6`** wraps the router to add artifact persistence and caching (see [[concepts/artifact-caching-pipeline]])
3. **`DomainPrompterR6`** builds the structured messages that are passed through the router (see [[concepts/domain-processor-pattern]])
4. Quarto document injection connects router output to rendered reports (see [[concepts/quarto]])

### Output Artifacts

Each `bridge$run_stage()` call produces a consistent set of artifacts per patient:
- `artifacts/patient_XXX/{stage_id}.md` — raw Markdown narrative
- `artifacts/patient_XXX/{stage_id}.meta.json` — model, role, timing, QC info
- `artifacts/patient_XXX/run.log.jsonl` — append-only audit log

The metadata layer is especially useful for routing analysis because it makes it possible to compare output quality, latency, and model selection by role over time. That supports empirical refinement of role mappings instead of relying only on static benchmark impressions.

### Report Render Integration

The full pipeline from routing to rendered report involves:
1. `load_data_duckdb()` — stage patient data (see [[concepts/duckdb-data-staging]])
2. `cingulate_workflow()` — generate QMD scaffolding (see [[concepts/quarto]])
3. `cingulate_run_llm_then_render()` — execute all role-based LLM calls in parallel and render to Typst

The `copilot-instructions` document notes that the system uses **two-stage rendering with edit protection** — always check for manual edits before regenerating QMD files. This is an instance of the [[concepts/edit-protection-pattern]].

The newer systems insight is that parallelism should be applied selectively. Running multiple small utility roles concurrently is often reasonable, but running multiple large reasoning roles at once may reduce overall throughput and produce worse latency consistency. Routing policy therefore needs to coordinate with execution policy, especially in [[concepts/local-llm-inference]] stacks on Apple Silicon.

## Migration from Legacy Patterns

| Old Pattern | New Pattern |
|---|---|
| `make_mega(data, ...)` | `bridge$run_stage("summarize_domain", "01_domain", messages = msgs)` |
| `make_mega_for_sirf(...)` | `bridge$run_stage("integrate_evaluation", "04_sirf", prompt = "...")` |
| Hardcoded prompts | `DomainPrompterR6$build_messages()` with templates |
| Manual model selection | `OllamaModelRouterR6` auto-routes by role |
| Direct `query_ollama()` | `query_ollama_enhanced()` with task + mode |

This migration is not just an API cleanup. It represents a shift from manually choosing models ad hoc to encoding reproducible policy about model purpose, fallback behavior, and quality tier. That makes the system more maintainable and more robust when local hardware constraints affect observed model performance.

## Why This Pattern Matters

- **Model agnosticism** — The application logic is insulated from model churn. Switching models requires only a YAML edit.
- **Clinical semantics** — Role names encode clinical intent, making the system self-documenting and auditable.
- **Multi-backend support** — The same role can be served by Ollama locally or MLX for Apple Silicon acceleration (see [[concepts/local-llm-inference]] and [[concepts/mlx-framework]]).
- **Testability** — Individual roles can be tested in isolation without affecting others.
- **Parallel execution** — Some workflows can execute multiple roles concurrently across patient domains.
- **Performance tiering** — The development/balanced/production mode system allows the same role to use different models depending on speed vs. quality requirements.
- **Stability-aware deployment** — Routing can encode the practical lesson that one large reasoning model plus several small utility models is often more reliable than multiple simultaneous heavyweight models.
- **Quality control through fit** — Different roles can be assigned to models whose explanatory style, semantic cohesion, and abstraction depth best match the task.

The broader significance is that role-based routing turns model choice into an explicit architectural concern rather than an incidental coding detail. In systems like Luria, where outputs support [[concepts/clinical-narrative-generation]], [[concepts/neuropsychological-assessment-automation]], and patient-facing explanation, the right routing policy improves not only technical performance but communicative quality.

## Related Concepts

- [[concepts/llm-provider-abstraction]]
- [[concepts/local-llm-inference]]
- [[concepts/local-inference-reliability]]
- [[concepts/fallback-strategy]]
- [[concepts/multi-agent-orchestration]]
- [[concepts/concurrent-model-serving]]
- [[concepts/artifact-caching-pipeline]]
- [[concepts/yaml-configuration]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/narrative-report-generation]]
- [[concepts/domain-processor-pattern]]
- [[concepts/mlx-framework]]
- [[concepts/r6-class-architecture]]
- [[concepts/duckdb-data-staging]]
- [[concepts/quarto]]
- [[concepts/edit-protection-pattern]]
- [[concepts/llm-evaluation]]
- [[concepts/semantic-cohesion]]

## Related Documents
- [[summaries/LLM Benchmark Comparison]]


See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]