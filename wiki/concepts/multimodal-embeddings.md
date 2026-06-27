---
sources: [summaries/mlx_embeddings.md]
brief: Multimodal embeddings map text and images into a shared vector space for unified retrieval and comparison.
---

# Multimodal Embeddings

Multimodal embeddings are dense vector representations that encode multiple data modalities — most commonly text and images — into a shared geometric space. Because items from different modalities occupy the same space, similarity between a text query and an image (or vice versa) can be measured directly with standard metrics such as dot product or cosine similarity.

## Core Idea

A unimodal embedding model maps one type of input (e.g., a sentence) to a vector. A multimodal embedding model maps *several* types of input through separate encoders that are trained jointly, so that semantically related items cluster together regardless of modality. The classic example is CLIP-style training: image–text pairs are pushed together while unrelated pairs are pushed apart.

## Modalities and Encoder Strategies

| Modality combination | Typical encoder | Example models |
|---|---|---|
| Text + Image | Vision Transformer (ViT) + language encoder | SigLIP, CLIP |
| Text + Image (generative backbone) | Vision encoder + causal/bidirectional LLM | Qwen3-VL, Llama Nemotron VL |
| Token-level multi-vector | Patch embeddings + query token embeddings | ColPali, ColQwen |

## Use Cases

- **Cross-modal retrieval** — find images with a text query, or find captions matching an image
- **Multimodal reranking** — score a set of candidate documents (text or image) against a query
- **Image-text matching / zero-shot classification** — estimate match probability between an image and a set of text descriptions
- **Multimodal RAG** — retrieve image or text chunks jointly in a [[concepts/retrieval-augmented-generation]] pipeline

## Implementation on Apple Silicon (MLX)

The [[summaries/mlx_embeddings]] package provides first-class multimodal embedding support via the [[concepts/mlx-framework]]. Key patterns:

### High-level API (Qwen3-VL)

```python
from mlx_embeddings import load
model, processor = load("Qwen/Qwen3-VL-Embedding-2B")

inputs = [
    {"text": "A dog on a beach", "instruction": "Retrieve relevant images."},
    {"image": "https://example.com/photo.jpg"},
    {"text": "Caption text", "image": "https://example.com/photo.jpg"},
]
embeddings = model.process(inputs, processor=processor)
similarity = embeddings @ embeddings.T  # shape: (N, N)
```

Inputs can be text-only, image-only, or combined text+image dicts. The model returns a single embedding per input regardless of modality mix.

### Vision-Language Scoring (SigLIP)

SigLIP-style models produce `logits_per_image` that can be passed through a sigmoid to yield per-pair match probabilities:

```python
outputs = model(pixel_values=pixel_values, input_ids=input_ids)
probs = mx.sigmoid(outputs.logits_per_image)
```

### Late Interaction (ColPali / ColQwen)

[[concepts/late-interaction-retrieval]] models such as ColQwen do *not* collapse inputs to a single vector. Instead, they produce a matrix of token-level embeddings per input, and relevance is scored with MaxSim:

```
score = sum(max(query_embeds @ image_embeds.T, axis=1))
```

This token-level approach preserves fine-grained local information and is especially powerful for document image retrieval.

## Reranking with Multimodal Embeddings

Multimodal reranking models (e.g., Qwen3-VL-Reranker) accept a structured `{instruction, query, documents}` object and return a scalar relevance score per document — enabling a two-stage retrieve-then-rerank pipeline.

## Model Quantization

Multimodal models are large. The MLX-Embeddings conversion tool supports [[concepts/model-quantization]] to 4-bit or 8-bit precision (affine, mxfp4, nvfp4, mxfp8), making it practical to run these models locally on a Mac.

## Relationship to Local Inference

Running multimodal embedding models locally (via [[concepts/local-llm-inference]] patterns and the [[concepts/mlx-framework]]) avoids sending sensitive image or text data to remote APIs, which is important for [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]] scenarios.

## Related Concepts

- [[concepts/mlx-framework]] — the Apple Silicon ML framework powering local execution
- [[concepts/late-interaction-retrieval]] — token-level multi-vector retrieval (ColPali/ColQwen)
- [[concepts/retrieval-augmented-generation]] — downstream use of embeddings for document retrieval
- [[concepts/model-quantization]] — compressing multimodal models for local deployment
- [[concepts/local-llm-inference]] — running embedding and generative models on-device
- [[concepts/transformer-architecture]] — the backbone of most multimodal encoders
