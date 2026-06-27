---
sources: [summaries/LLM Benchmark Comparison.md, summaries/top_level.md, summaries/entry_points.md, summaries/mlx_embeddings.md, summaries/local_models.md, summaries/deepagents_merged_mem_notes.md, summaries/A-Mac-Studio-for-Local-AI-6-Months-Later.md]
brief: Model quantization reduces neural network weight precision to cut memory and compute costs for LLM deployment.
---

# Model Quantization

Model quantization is a compression technique that reduces the numerical precision of neural network weights — from full 32-bit or 16-bit floats down to smaller representations such as 8-bit integers or 4-bit formats. Like other forms of lossy compression, the goal is to reduce size and computational cost without meaningfully degrading output quality.

Quantization is essential for running large language models on consumer hardware, where memory capacity and bandwidth are the primary bottlenecks. See [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]] for a detailed real-world account of quantization choices when running 600B+ parameter models on a Mac Studio.

## Why Quantization Matters

Modern frontier-class models have hundreds of billions of parameters. At full (BF16) precision, even a 70B model requires ~140GB of memory. Quantization makes it possible to fit much larger models into available hardware:

- A 1-trillion-parameter model like Kimi K2.5, compressed to ~2.5-bit mixed precision, fits within 512GB of unified memory
- Without quantization, such models would require multiple high-end datacenter GPUs

Beyond memory, quantized formats can be optimized for specific hardware instruction sets, improving inference throughput as well.

## Key Precision Levels

| Format | Notes |
|--------|-------|
| BF16 / FP16 | Near-lossless; baseline quality but large footprint |
| 8-bit (Q8) | Approximately equivalent quality to full precision at half the size; a recognized "optimized peak" |
| 4-bit (Q4) | The practical sweet spot on Apple Silicon; GPU-optimized, good balance of size and quality |
| Sub-4-bit | Aggressive compression; viable for very large models or with Quantization-Aware Training (QAT) |

These formats appear directly in real-world local model collections. For example, the [[summaries/local_models]] inventory includes models spanning the full range: BF16 (medgemma-1.5-4b-it-bf16, nomicai-modernbert-embed-base-bf16), 4-bit (MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit, Qwopus3.5-9B-v3-PolarQuant-MLX-4bit), and 8-bit (parakeet-tdt-0.6b-v3-mlx-8bit) quantizations — demonstrating that different tasks call for different precision tradeoffs.

## Library Support: bitsandbytes

The [[concepts/bitsandbytes-library]] is a lightweight Python library that provides first-class support for quantized training and inference of [[concepts/large-language-models]]. It enables 8-bit and 4-bit quantization directly within the PyTorch ecosystem, making it one of the most widely used tools for reducing GPU memory requirements during both fine-tuning and inference. bitsandbytes integrates with the Hugging Face ecosystem and works alongside libraries like Accelerate (see [[concepts/accelerate-library]]) to enable quantized model loading with minimal code changes. Its availability on standard Python package indexes makes it a practical entry point for users new to quantization.

## Quantization Modes in MLX

The [[concepts/mlx-framework]] introduces a structured vocabulary of quantization modes, formalized in tools like [[summaries/mlx_embeddings]]'s conversion utility. When converting Hugging Face models to MLX format, users can choose from:

| Mode | Group Size | Bits | Use Case |
|------|------------|------|----------|
| `affine` (default) | 64 | 4 | General-purpose quantization |
| `mxfp4` | 32 | 4 | MLX floating-point 4-bit |
| `nvfp4` | 16 | 4 | NVIDIA floating-point 4-bit |
| `mxfp8` | 32 | 8 | Higher precision, larger footprint |

This shows that even within a single bit-width (4-bit), meaningfully different quantization schemes exist, varying in group size and floating-point representation. Group size controls how many weights share a single scaling factor — smaller groups preserve more local structure at the cost of slightly more metadata overhead.

The MLX conversion tool (`mlx_embeddings.convert`) also supports dtype conversion (float16, bfloat16, float32) and dequantization of previously quantized models, enabling round-trip workflows.

## Dynamic and Mixed-Precision Quantization

Not all weights in a model are equally sensitive to precision loss. **Dynamic** or **mixed-precision** quantization exploits this by:

- Keeping the most sensitive layers (e.g., attention projections, early layers) at higher precision
- Aggressively compressing less sensitive layers (e.g., feed-forward middle layers)

This approach yields better quality-per-bit than uniform quantization, and is considered essential for optimal results. The Qwen 3.5 397B model, for example, runs at a 2.6-bit mixed average in [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]]. The "PolarQuant" designation on the Qwopus3.5-9B-v3-PolarQuant-MLX-4bit model in [[summaries/local_models]] is an example of a specialized mixed or structured quantization scheme applied at the community level.

## Hardware Dependence

Quantization formats are not universally portable:

- **Apple Silicon (M3/M4)**: 4-bit is GPU-optimized; BF16 is fully supported; sub-4-bit requires mixed strategies
- **Older Apple Silicon (M1/M2)**: BF16 weights can cause performance degradation
- Future hardware generations may natively support additional formats

This means a quant that performs well on one machine may behave differently on another. Always verify against the target hardware. The prevalence of [[concepts/mlx-framework]] format models in the [[summaries/local_models]] inventory strongly implies deployment on Apple Silicon, where MLX-specific quantizations are first-class citizens.

## Quantization of Embedding Models

Quantization is not limited to generative LLMs — it is equally applicable to embedding models. The [[summaries/mlx_embeddings]] package demonstrates this concretely:

- The `mlx-community/all-MiniLM-L6-v2-4bit` model is a 4-bit quantized text embedding model used for sentence similarity
- `mlx-community/answerdotai-ModernBERT-base-4bit` applies 4-bit quantization to a bidirectional encoder for masked language modeling and classification
- Vision-language embedding models (SigLIP, ColQwen) are similarly distributed in MLX-quantized format

Embedding models are often small enough that BF16 is practical (e.g., nomicai-modernbert-embed-base-bf16 at only 288MB), but quantization still provides meaningful speedups and memory savings in batch inference scenarios. The [[concepts/multimodal-embeddings]] space inherits the same tradeoffs: multimodal models tend to be larger due to vision encoders, making quantization more impactful.

## Model Size and Quantization Stability

A key empirical finding: **larger models tolerate lower bit-widths more gracefully**. A 400B model compressed to 3-bit may lose less quality than a 7B model at the same compression ratio. This is because larger models have more redundancy and capacity to absorb precision loss.

Quantization-Aware Training (QAT) — where a model is fine-tuned with quantization applied during training — can further stabilize sub-4-bit representations.

This principle is visible in the [[summaries/local_models]] collection: the two largest models (Qwen3.6-35B-A3B-oQ4 at 19.68 GB and the reasoning-distilled Qwen3.5-35B at 18.19 GB) use Q4 quantization, while smaller specialty models like the 0.6B Parakeet speech model use 8-bit — consistent with the expectation that smaller models benefit from higher-precision quants.

## Quantization Across Model Domains

Quantization is not limited to general-purpose LLMs. The [[summaries/local_models]] inventory illustrates that specialized models across multiple domains are routinely quantized:

- **Medical NLP**: OpenMed PII models and MedGemma run in BF16, prioritizing precision for sensitive clinical inference (see [[concepts/pii-redaction-pipelines]] and [[concepts/clinical-nlp-pipelines]])
- **Embedding models**: nomicai-modernbert-embed-base-bf16 runs BF16 at only 288MB — small enough that aggressive quantization adds little benefit; community 4-bit variants (e.g., all-MiniLM-L6-v2-4bit) target resource-constrained deployments
- **Document processing**: granite-docling-258M-mlx at 610MB in MLX format
- **Speech recognition**: parakeet-tdt-0.6b-v3-mlx-8bit uses 8-bit, balancing quality and footprint for audio tasks
- **Vision-language models**: SigLIP and ColQwen multimodal models distributed in MLX format for local image-text retrieval

## Practical Guidance

- **Prefer published benchmarks** over subjective "vibes" when evaluating quant quality; perplexity scores and standardized evals (e.g., MMLU, HumanEval) are more reliable
- **4-bit dynamic quantization** is the recommended starting point on Apple Silicon
- **8-bit** is a safe choice when memory allows and maximum quality is needed
- Use [[concepts/bitsandbytes-library]] for GPU-based quantization within the PyTorch/Hugging Face ecosystem
- Avoid sub-4-bit for smaller models unless QAT was applied
- Use community-validated quants (e.g., from Hugging Face or mlx-community) with documented evaluation results
- For domain-sensitive tasks (medical, legal, clinical), prefer BF16 or Q8 where memory permits
- When converting models via MLX tools, choose quantization mode based on target hardware: `affine` for general use, `mxfp8` when higher precision is needed

## Relationship to Model Architecture

Quantization interacts closely with model architecture. [[concepts/mixture-of-experts]] models are particularly interesting in this context: their large total parameter count (which benefits from aggressive quantization to fit in memory) coexists with a smaller set of *active* parameters per token (which limits inference cost regardless of total size). This makes MoE models especially well-suited to the quantization-heavy workflows required for local inference.

For broader context on running quantized models locally, see [[concepts/local-llm-inference]] and [[concepts/mixture-of-experts]].

The relationship between model size, quantization level, and emergent capability also touches on [[concepts/scaling-laws]] — larger models not only perform better, but degrade more gracefully under compression.

See also: [[summaries/deepagents_merged_mem_notes]], [[summaries/local_models]], [[summaries/mlx_embeddings]], [[summaries/top_level]]

See also: [[summaries/LLM Benchmark Comparison]]