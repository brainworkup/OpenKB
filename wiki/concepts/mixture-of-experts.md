---
sources: [summaries/LLM Benchmark Comparison.md, summaries/top_level.md, summaries/local_models.md, summaries/deepagents_merged_mem_notes.md, summaries/A-Mac-Studio-for-Local-AI-6-Months-Later.md]
brief: MoE splits large models into sparse expert sub-networks, enabling frontier-scale capacity at practical inference cost.
---

# Mixture of Experts (MoE)

Mixture of Experts (MoE) is a neural network architecture where a large model is divided into many specialized sub-networks ("experts"), with a routing mechanism that selects only a small subset of those experts to process each individual token. This decouples two properties that are normally in tension: **total model capacity** (which correlates with intelligence) and **computational cost per token** (which determines inference speed).

## Core Principle

In a standard dense model, every parameter is used for every token. In an MoE model:

- **Total parameters** determine the model's knowledge and reasoning ceiling — more is generally smarter.
- **Active parameters** (those activated per token) determine how much compute is spent, and therefore how fast inference runs.

This means a 1-trillion-parameter MoE model can run at speeds closer to a 32-billion-parameter dense model, if only 32B parameters are active at a time.

## Why MoE Matters for Local Inference

For local hardware with large but finite memory — like the Mac Studio M3 with 512GB unified memory — MoE is particularly attractive. The full model weights must fit in memory (which favors large unified memory), while only the active parameters need to be computed per token (which keeps inference practical).

As described in [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]], MoE architectures make it possible to run frontier-class models on a single machine:

> "This makes Mixture-of-Experts (MoE) architectures especially attractive. These keep the full model weights in memory (which the Mac Studio has in abundance) while activating only a subset per token (which keeps inference speed usable."

## Real-World Examples (2025–2026)

| Model | Total Parameters | Active Parameters | Relative Speed |
|---|---|---|---|
| Kimi K2.5 | ~1 trillion | 32B | Baseline |
| GLM 5 | 744B | 40B | ~20% slower than Kimi |
| Qwen 3.5 397B | 397B | 17B | ~2× faster than Kimi |

The pattern is predictable: speed scales inversely with active parameters, not total parameters.

## MoE Models in Local Deployment

The locally stored model collection documented in [[summaries/local_models]] includes several models that embody MoE principles. The two largest — **MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit** (18.19 GB) and **Qwen3.6-35B-A3B-oQ4** (19.68 GB) — are both based on the Qwen 35B architecture with an A3B (approximately 3B active) designation. This means:

- **Total parameters**: ~35 billion
- **Active parameters per token**: ~3 billion

This ~10:1 ratio of total-to-active parameters is a hallmark of modern MoE design, and explains how a 35B-class model can be stored in under 20 GB at 4-bit quantization while still delivering competitive reasoning performance. The "A3B" suffix in model names is a direct shorthand for the active parameter count, making it easy to distinguish MoE models from dense ones at a glance.

These models are distributed in [[concepts/mlx-framework]] format, optimized for Apple Silicon's unified memory architecture — precisely the hardware environment where MoE's memory-vs-compute split is most beneficial.

## Trade-offs and Considerations

- **Memory requirement**: The entire model must reside in memory even though only a fraction is used per token. This is why large unified memory (as on Apple Silicon) is a prerequisite for running the largest MoE models locally.
- **Quantization interaction**: MoE models, due to their size, can often tolerate more aggressive quantization (below 4-bit) without severe quality degradation. See [[concepts/model-quantization]].
- **Routing overhead**: The gating/routing mechanism adds a small amount of overhead, but this is generally negligible compared to the compute savings.
- **Load balancing**: During training, ensuring experts are used evenly is a known challenge; poorly balanced models may have underutilized experts.
- **Naming conventions**: Active parameter counts (e.g., A3B) are increasingly surfaced in model names, allowing practitioners to quickly estimate inference cost without reading full model cards.

## Relationship to Scaling Laws

MoE can be seen as a way to climb [[concepts/scaling-laws]] more efficiently — achieving the benefits of a larger parameter count without a proportional increase in inference cost. It separates the *training-time* and *inference-time* scaling curves.

## Relationship to Local LLM Inference

MoE is one of the key architectural reasons that running very large models locally has become feasible. Combined with advances in [[concepts/model-quantization]] and hardware like Apple Silicon, MoE models have shifted local inference from a hobbyist curiosity toward practical utility. See [[concepts/local-llm-inference]] for the broader context of running models on personal hardware.


See also: [[summaries/deepagents_merged_mem_notes]], [[summaries/local_models]]

See also: [[summaries/top_level]]

See also: [[summaries/LLM Benchmark Comparison]]