---
sources: [summaries/top_level.md]
brief: A Python library enabling efficient LLM training and inference via quantization and memory optimization.
---

# bitsandbytes Library

bitsandbytes is a lightweight Python library that provides efficient quantization and memory-saving optimization techniques for training and running [[concepts/large-language-models]]. It is widely used to reduce GPU memory requirements when working with large neural networks.

## Overview

bitsandbytes exposes CUDA custom functions, particularly for 8-bit and 4-bit quantization, enabling users to run and fine-tune models that would otherwise exceed available GPU memory. It integrates tightly with the Hugging Face ecosystem, including the [[concepts/accelerate-library]], and supports [[concepts/model-quantization]] workflows.

## Key Features

- **8-bit and 4-bit quantization**: Reduces model weight precision from 32-bit or 16-bit floats to 8-bit integers (LLM.int8()) or 4-bit formats (NF4, FP4), dramatically reducing memory footprint.
- **8-bit optimizers**: Provides memory-efficient Adam, AdamW, and other optimizers using 8-bit statistics, reducing optimizer state memory by up to 75%.
- **Paged optimizers**: Manages optimizer memory via CPU offloading to handle memory spikes during training.
- **QLoRA support**: Enables quantized low-rank adaptation for fine-tuning [[concepts/large-language-models]] on consumer hardware.

## Relationship to Local LLM Inference

bitsandbytes is a core dependency in [[concepts/local-llm-inference]] stacks. It enables practitioners to run models locally on modest hardware by reducing precision without catastrophic accuracy loss. This aligns with [[concepts/local-first-architecture]] approaches used in privacy-sensitive clinical and research contexts.

## Integration Points

- Works alongside [[concepts/mlx-framework]] and other inference frameworks as an alternative quantization backend.
- Referenced in [[summaries/top_level]] as the top-level package entry for the bitsandbytes project.
- Relevant to [[concepts/mixture-of-experts]] and [[concepts/transformer-architecture]] model serving, where memory constraints are a primary bottleneck.

## Use Cases

1. Fine-tuning large models on single consumer GPUs using QLoRA.
2. Running inference on quantized models with reduced VRAM.
3. Research into scaling and efficiency via [[concepts/scaling-laws]].

## See Also

- [[concepts/model-quantization]]
- [[concepts/local-llm-inference]]
- [[concepts/large-language-models]]
- [[concepts/mlx-framework]]
- [[concepts/accelerate-library]]
- [[summaries/top_level]]
