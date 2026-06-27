---
sources: [summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md]
brief: Content-addressed caching that stores and reuses LLM stage outputs to avoid redundant generation runs.
---

# Artifact Caching in LLM Pipelines

Artifact caching is a pattern for persisting the outputs of individual LLM generation stages so that identical inputs produce stored results rather than triggering new model calls. In [[concepts/neuropsychological-assessment-automation]] and other high-throughput clinical pipelines, regenerating the same narrative on every run is wasteful and introduces variability — caching addresses both problems.

## Core Idea

Each pipeline stage is assigned a **content-addressed cache key** derived from a hash of its inputs (prompt, model configuration, parameters). If the hash matches a stored result, the cached output is returned immediately. If not, the LLM is called and the result is stored under the new hash. This makes re-runs **idempotent**: as long as inputs are unchanged, outputs are stable and instant.

## Implementation in the `cingulate` Package

As documented in [[summaries/LLM_AGENT_MAP]], caching is managed by `ReportLLMBridgeR6` through its `enable_cache` flag:

```r
bridge <- ReportLLMBridgeR6$new(
  router       = router,
  out_dir      = "artifacts/patient_john_doe",
  seed         = 42L,
  enable_cache = TRUE
)
```

Each call to `bridge$run_stage()` writes a full artifact bundle:

| File | Content |
|---|---|
| `<stage_id>.md` | Generated narrative text |
| `<stage_id>.meta.json` | Model name, timing, QC scores |
| `run.log.jsonl` | Append-only audit trail |
| `.cache/<hash>.rds` | Content-addressed cached result |

The `.cache/` directory holds `.rds` files keyed by input hash. The audit log (`run.log.jsonl`) persists across runs, giving a chronological record of what was generated, when, and by which model.

## Why Content-Addressing Matters

- **Reproducibility** — The same inputs always yield the same output, essential for clinical documentation integrity.
- **Cost and latency reduction** — Avoids re-calling local or remote LLMs for unchanged domain data.
- **Selective invalidation** — Changing a prompt, score, or model config produces a new hash and forces regeneration only for that stage, leaving unaffected stages cached.
- **QMD injection safety** — `NeuropsychResultsR6$process()` uses the same content-addressing approach: the `<summary>…</summary>` block injected into `.qmd` files survives re-runs unchanged if the underlying data has not changed.

## Relationship to the Broader Pipeline

Artifact caching is one layer in a multi-level [[concepts/agent-pipeline-state-management]] strategy:

1. **Data layer** — [[concepts/duckdb-data-staging]] produces Parquet files that themselves act as a stable, versioned input cache.
2. **Stage layer** — `ReportLLMBridgeR6` caches per-stage LLM outputs as `.rds` files.
3. **Document layer** — `NeuropsychResultsR6` protects injected narrative blocks in QMD files from being overwritten.

This tiered approach means a full pipeline re-run only performs new work where inputs have actually changed.

## Relationship to Role-Based Routing

Caching interacts closely with [[concepts/role-based-llm-routing]]: the cache key should incorporate the resolved model ID (not just the role name), since the same role may map to different models over time as `ollama_models.yml` is updated. Including the model ID in the hash prevents stale cached results from being served after a model upgrade.

## Artifact Audit Trail

The `run.log.jsonl` file is an append-only structured log, consistent with [[concepts/plain-text-documentation]] principles. It records every stage execution — whether a cache hit or a live generation — allowing downstream QA processes to reconstruct exactly what was generated and when. This supports [[concepts/report-review-qa]] workflows and clinical compliance requirements.

## Related Concepts

- [[concepts/agent-pipeline-state-management]]
- [[concepts/role-based-llm-routing]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/narrative-report-generation]]
- [[concepts/duckdb-data-staging]]
- [[concepts/local-llm-inference]]
- [[concepts/report-review-qa]]
- [[concepts/yaml-configuration]]
- [[concepts/r6-class-architecture]]


See also: [[summaries/LLM_INTEGRATION]]