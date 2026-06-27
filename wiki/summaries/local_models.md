---
doc_type: short
full_text: sources/local_models.md
---

# Local Models — Model Manager Inventory

## Overview
This document lists all AI models currently stored locally in the Model Manager, including their sizes on disk. The collection spans large language models (LLMs), medical/clinical NLP models, embedding models, and speech recognition models.

## Model Inventory

| Model | Size |
|---|---|
| MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit | 18.19 GB |
| OpenMed-PII-BiomedELECTRA-Large-335M-v1-mlx | 1.25 GB |
| OpenMed-PII-SuperClinical-Large-434M-v1 | 1.63 GB |
| Qwen3.6-35B-A3B-oQ4 | 19.68 GB |
| Qwopus3.5-9B-v3-PolarQuant-MLX-4bit | 4.71 GB |
| granite-docling-258M-mlx | 610.8 MB |
| medgemma-1.5-4b-it-bf16 | 9.30 GB |
| nomicai-modernbert-embed-base-bf16 | 288.2 MB |
| parakeet-tdt-0.6b-v3-mlx-8bit | 867.3 MB |

## Model Categories

### Large Language Models (LLMs)
- **MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit** (18.19 GB) — A reasoning-distilled model combining Qwen 3.5 35B architecture with Claude Opus 4.6 distillation, quantized to 4-bit for the [[concepts/mlx-framework]].
- **Qwen3.6-35B-A3B-oQ4** (19.68 GB) — Qwen 3.6 35B parameter model with Q4 quantization.
- **Qwopus3.5-9B-v3-PolarQuant-MLX-4bit** (4.71 GB) — A 9B parameter model with PolarQuant 4-bit quantization for MLX.

### Medical / Clinical NLP Models
See [[concepts/clinical-nlp-pipelines]] and [[concepts/pii-redaction-pipelines]] for cross-document synthesis.
- **OpenMed-PII-BiomedELECTRA-Large-335M-v1-mlx** (1.25 GB) — Biomedical ELECTRA model for PII detection, 335M parameters, MLX format.
- **OpenMed-PII-SuperClinical-Large-434M-v1** (1.63 GB) — Clinical NLP model for PII detection, 434M parameters.
- **medgemma-1.5-4b-it-bf16** (9.30 GB) — Google's MedGemma 4B instruction-tuned model in bfloat16, for medical text tasks.

### Embedding Models
- **nomicai-modernbert-embed-base-bf16** (288.2 MB) — Nomic AI's ModernBERT-based embedding model in bfloat16; smallest model in the collection. Relevant to [[concepts/retrieval-augmented-generation]].

### Document Processing
- **granite-docling-258M-mlx** (610.8 MB) — IBM Granite model for document parsing/layout understanding via Docling, in [[concepts/mlx-framework]] format.

### Speech Recognition
- **parakeet-tdt-0.6b-v3-mlx-8bit** (867.3 MB) — NVIDIA Parakeet TDT 0.6B speech-to-text model, 8-bit quantized for [[concepts/mlx-framework]].

## Key Observations
- **Total approximate storage**: ~57 GB across 9 models.
- **Quantization formats**: 4-bit, 8-bit, bfloat16 (bf16), and Q4 are all represented, reflecting tradeoffs between size and quality. See [[concepts/model-quantization]].
- **MLX prevalence**: Many models are in Apple's [[concepts/mlx-framework]] format, suggesting deployment on Apple Silicon hardware. This aligns with [[concepts/local-llm-inference]] patterns documented elsewhere in the wiki.
- **Domain diversity**: Collection covers general reasoning, medical/clinical NLP (see [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]]), embeddings, and speech-to-text.
- **Parameter range**: From 258M (granite-docling) to ~35B (Qwen variants), with several models using [[concepts/mixture-of-experts]] architecture (the A3B suffix indicates active-parameter MoE variants).

## Related Concepts
- [[concepts/privacy-first-software]]
