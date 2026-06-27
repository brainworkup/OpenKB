---
sources: [summaries/top_level.md, summaries/entry_points.md]
brief: Hugging Face Accelerate is a library enabling distributed and accelerated ML training via a unified CLI.
---

# Accelerate Library

Hugging Face **Accelerate** is a Python library that simplifies running machine learning training scripts across various hardware configurations — including multi-GPU, multi-node, and mixed-precision environments — without requiring major code changes.

## CLI Entry Points

Accelerate installs several shell commands via Python `console_scripts` entry points. These are defined in the package metadata (e.g., `pyproject.toml`) and become available after installation. See [[summaries/entry_points]] for the full specification.

| Command | Purpose |
|---|---|
| `accelerate` | Main CLI dispatcher |
| `accelerate-config` | Interactive configuration wizard for hardware setup |
| `accelerate-launch` | Launch distributed or accelerated training scripts |
| `accelerate-estimate-memory` | Estimate memory requirements before running |
| `accelerate-merge-weights` | Merge model weights (e.g., for fine-tuned adapters) |

## Key Capabilities

### Distributed Training Launch
The `accelerate-launch` command wraps training scripts to automatically handle distributed data parallelism, mixed precision, and hardware backend selection (CUDA, MPS, CPU). This is the core workflow entry point for most users.

### Configuration Management
The `accelerate-config` command generates a YAML configuration file (typically `~/.cache/huggingface/accelerate/default_config.yaml`) that records hardware topology and preferences. This relates to [[concepts/yaml-configuration]] patterns used throughout ML tooling.

### Memory Estimation
Before launching expensive training runs, `accelerate-estimate-memory` allows practitioners to predict VRAM and RAM requirements — relevant to [[concepts/model-quantization]] decisions and hardware planning.

### Weight Merging
The `accelerate-merge-weights` command supports merging of model weights, applicable to parameter-efficient fine-tuning workflows and adapter-based model composition.

## Relation to Local LLM Workflows

Accelerate integrates naturally with local inference and fine-tuning pipelines. It complements tools described in [[concepts/local-llm-inference]] and [[concepts/mlx-framework]] for running models on consumer and professional hardware.

## Python Project Integration

As a installable Python package with registered entry points, Accelerate exemplifies best practices in [[concepts/python-project-structure]] and [[concepts/cli-entry-points]], making its commands immediately accessible after `pip install accelerate`.

## Related Concepts

- [[concepts/cli-entry-points]] — How Python packages expose shell commands
- [[concepts/local-llm-inference]] — Running models locally, complementary use case
- [[concepts/mlx-framework]] — Alternative local inference framework for Apple Silicon
- [[concepts/model-quantization]] — Technique often used alongside Accelerate for memory savings
- [[concepts/python-project-structure]] — Package structure and entry point conventions
- [[concepts/yaml-configuration]] — Configuration files used by Accelerate's config system


See also: [[summaries/top_level]]