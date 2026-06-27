---
doc_type: short
full_text: sources/deepagents_merged_mem_notes.md
---

# Corrected Luria Orchestration Plan — Biggie

**Source:** mem.ai notes (deepagents_merged_mem_notes) | May 10–14, 2026 | Joey Trampush

## Overview

This document is a corrected orchestration plan for the **Luria neuropsychological report pipeline**, written in response to an earlier "Deep Agents" sketch that contained hallucinated components and architectural errors. It grounds the plan in what actually exists on disk and replaces the fictional `deepagents` SDK with [[concepts/langgraph-agent-workflows]] + existing tooling.

---

## Core Corrections Made

| Issue | Original Sketch | Corrected Plan |
|---|---|---|
| Runtime | `pip install deepagents` (nonexistent) | LangGraph (`neuropsych_agent/graph.py`) |
| Domain count | 10 (included hallucinated `social`) | 9 confirmed: 7 neurocog + 2 neurobehav |
| Neurocog/neurobehav split | Collapsed into one swarm | Restored: B3 (7 domains), B4 (2 domains) |
| `sirf_impression` worker | Missing | Assigned: `Qwen3.6-35B-A3B-oQ4` |
| NSE PII redaction | "Docling" with no mechanism | Presidio (`pii_presidio.py`) + Docling |
| Concurrency claim | "40 concurrent tasks" | Max ~21 during B3 text/table/figure phase |
| Voice subagents | Claimed as solved | Marked TBD |
| Failure model | None | Explicit: partial output + validity flag |

---

## Pipeline Phase Structure

The five-phase structure is confirmed correct:

### Phase A — Intake (Visit 1 / NSE)

Sequential with one parallel window after A1:
- **A1** `nse_admin` — case intake (`granite-4.1-8b-nvfp4`)
- **A2** `nse_clinical_past` — prior medical records (`medgemma-1.5-4b-it-oQ8`)
- **A3** `nse_clinical_current` — current clinical forms (`gemma-4-E4B-it-MLX-8bit`)
- **A4** `nse_stt_transcript` — audio transcription (`parakeet-tdt-0.6b-v3-mlx-8bit`)
- **A5** `nse_cod_summary` — CoD summary + **PII redaction gate** via [[concepts/pii-redaction-pipelines]] (`Qwen3.6-35B-A3B-oQ4`). All downstream agents receive only redacted content.
- **A6** `nse_report` — full NSE narrative (`MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit`)

### Phase B — Testing (Visit 2 / NT)

B1 runs first, then B2 and B3 run in parallel:
- **B1** `nt_tests` — test list validation (`granite-4.1-8b-nvfp4`)
- **B2** `nt_behav_obs` — behavioral observations (`medgemma-1.5-4b-it-oQ8`)
- **B3** `nt_neurocog` — 7 neurocognitive domain subagents in parallel
- **B4** `nt_neurobehav` — 2 neurobehavioral domain subagents in parallel

**Confirmed 9 domains (no `social` domain):**

| # | Domain | Parquet | Tests |
|---|---|---|---|
| 1 | IQ | `iq.parquet` | WISC-V |
| 2 | Academics | `academics.parquet` | KTEA-3 |
| 3 | Verbal | `verbal.parquet` | WISC-V, NEPSY-2 |
| 4 | Spatial | `spatial.parquet` | WISC-V |
| 5 | Memory | `memory.parquet` | CVLT-C |
| 6 | Executive | `executive.parquet` | D-KEFS, WISC-V |
| 7 | Motor | `motor.parquet` | Pegboard, NEPSY-2 |
| 8 | ADHD | `adhd.parquet` | — |
| 9 | Emotion | `emotion.parquet` | — |

**Per-domain pipeline (identical for all):**
```
DataPrepAgent (granite-4.1-8b)
  └── filters/classifies → data.json
       ├── TextSubAgent (gpt-oss-20b) → _02-XX_domain_text.qmd
       ├── TableSubAgent (granite-4.1-8b) → TableGTR6.R → table PNG
       └── FigureSubAgent (granite-4.1-8b) → DotplotR6.R → figures
```

**Max concurrency:** ~21 (7 domains × 3 parallel Text/Table/Figure agents).

**Failure handling states:** `COMPLETED`, `DEGRADED`, `SKIPPED`, `FAILED` (retryable/non-retryable). Retryable failures get up to 2 retries; non-retryable failures skip domain with a report flag.

### Phase C — SIRF (Visit 3)

Sequential:
- **C1** `sirf_summary` — domain + whole-profile interpretation (`MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit`)
- **C2** `sirf_impression` — DSM-5/ICD-10 diagnosis (`Qwen3.6-35B-A3B-oQ4`) ← *model assignment was blank in original architecture doc*
- **C3** `sirf_recs` — recommendations with RAG (`Qwen3.6-35B-A3B-oQ4`)

### Phase D — Final Assembly

Sequential:
- **D1** `luria-neuropsych-orchestrator` — full report assembly. Voice subagents (Style/Brand/Soul) exist in code (`soul-agent/`) but are **not yet wired** — marked TBD, not a blocker.
- **D2** `luria-quality-review` — completeness, PHI leaks, consistency, validity language, test security (`Qwen3.6-35B-A3B-oQ4`). Max 3 correction cycles before human escalation.

### Phase E — Render

Deterministic, no LLM:
```
quarto render Biggie.qmd --to neurotyp-pediatric-typst → Biggie.pdf
```

---

## Model Routing Summary

| Role | Model | Notes |
|---|---|---|
| Orchestrator / patient-facing output | `MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit` | Best quality |
| Summary / diagnosis / review | `Qwen3.6-35B-A3B-oQ4` | Strong reasoning |
| Domain narrative (Text subagent) | `gpt-oss-20b-MXFP4-Q8` | Clinical prose |
| Fast / deterministic / DataPrep | `granite-4.1-8b-nvfp4` | Temp=0 |
| Clinical light | `medgemma-1.5-4b-it-oQ8` | Medical fine-tune |
| Clinical heavy forms | `gemma-4-E4B-it-MLX-8bit` | Moderate |
| STT | `parakeet-tdt-0.6b-v3-mlx-8bit` | MacWhisper |

Models are served via the local oMLX endpoint (port 8000); see [[concepts/local-llm-inference]] for the broader context of running these quantized models locally. Several models (Qwen3.6-35B, gpt-oss-20b) are mixture-of-experts architectures; see [[concepts/mixture-of-experts]]. Model weights are loaded in 4-bit and 8-bit quantized formats; see [[concepts/model-quantization]].

---

## Runtime Implementation

Extend existing `neuropsych_agent/graph.py` ([[concepts/langgraph-agent-workflows]] StateGraph) with:
- Fan-out nodes for B3/B4 domain parallelism
- **Recommended approach:** LangGraph `Send` API (v0.2+) for per-domain tracing in production; `asyncio.gather` inside a single node for the first working version

The overall design is an instance of [[concepts/multi-agent-orchestration]], with a top-level orchestrator delegating to specialized domain subagents. PHI handling throughout the pipeline is governed by [[concepts/phi-data-handling]] principles, with the A5 redaction gate as the primary enforcement point enforced via [[concepts/pii-redaction-pipelines]]. C3 recommendations leverage [[concepts/retrieval-augmented-generation]] via PageIndex + local `rag_db/`. The R-to-Python model-routing bridge is an example of [[concepts/r-python-integration]].

---

## Verified Pipeline Results (Session 2026-05-09)

All 7 neurocognitive domains generated clinical narratives via `gpt-oss-20b-MXFP4-Q8`:

| Domain | Avg Latency | Key Finding |
|---|---|---|
| General Cognitive Ability | 1.9s | Average FSIQ, variable indices |
| Academic Skills | 3.0s | Mixed — reading 61st%, math near Below Average |
| Verbal/Language | 2.5s | Solid Average |
| Visual Perception | 2.5s | Solid Average |
| Memory | 3.8s | Low Average ⚠️ (16th–50th %ile) |
| Attention/Executive | 3.6s | Variable across 6 exec measures |
| Motor | 2.5s | WNL, limited data |

**Total sequential time: ~19.8s. Avg: 2.8s/domain.**

This validates the domain subagent pattern for [[concepts/neuropsychological-reporting]] — all 7 domains produced clinically plausible narratives with correct percentile ranges, domain classifications, and test references.

---

## Priority Build Order (for Biggie report)

1. Extend `graph.py` with B3 fan-out node (7 neurocog domains) — biggest unlock
2. Wire `pii_presidio.py` into A5 (exists but not called anywhere)
3. Assign `sirf_impression` model in config
4. Generate first full report **without** voice subagents
5. Layer in voice after soul agent is validated

---

## Key Concepts

- [[concepts/langgraph-agent-workflows]] — orchestration runtime replacing the fictional `deepagents` SDK
- [[concepts/pii-redaction-pipelines]] — Presidio-based redaction gate at A5; all downstream agents receive only redacted content
- [[concepts/phi-data-handling]] — HIPAA-aware permission model denying `data-raw/` to subagents
- [[concepts/multi-agent-orchestration]] — the full A→E pipeline coordinating specialized subagents across clinical visits
- [[concepts/neuropsychological-reporting]] — the clinical domain this pipeline serves, covering NSE, NT testing, SIRF, and report assembly
- [[concepts/local-llm-inference]] — oMLX endpoint (port 8000) with role-based model assignment
- [[concepts/retrieval-augmented-generation]] — PageIndex + local `rag_db/` used in C3 recommendations
- [[concepts/mixture-of-experts]] — architecture of the primary reasoning models (Qwen3.6-35B, gpt-oss-20b)
- [[concepts/model-quantization]] — 4-bit and 8-bit model weights throughout the pipeline
- [[concepts/r-python-integration]] — R6 model router bridging R domain scripts to Python/LLM layer

## Related Concepts
- [[concepts/persistent-memory]]
