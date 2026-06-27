---
doc_type: short
full_text: sources/mlx_embeddings.md
---

# MLX-Embeddings

**Source:** [GitHub – Blaizzy/mlx-embeddings](https://github.com/Blaizzy/mlx-embeddings)  
**Author:** Blaizzy  
**License:** GNU GPL v3

## Overview

MLX-Embeddings is a Python package for running embedding models (text and vision) locally on Apple Silicon Macs using Apple's [[concepts/mlx-framework]] machine learning framework. It supports single-item and batch processing, similarity comparison utilities, model conversion, and quantization.

## Supported Model Architectures

| Architecture | Notes |
|---|---|
| XLM-RoBERTa | Cross-lingual, multilingual text embedding |
| BERT | Classic bidirectional encoder |
| ModernBERT | Modernized bidirectional encoder-only Transformer |
| Qwen3 | Text embedding model |
| Qwen3-VL | Multimodal embedding + reranking |
| Llama Bidirectional | e.g. NVIDIA NV-Embed |
| Llama Nemotron VL | SigLIP vision + bidirectional Llama |
| SigLIP | Vision-language similarity scoring |
| ColPali / ColQwen | [[concepts/late-interaction-retrieval]] multimodal models |

## Installation

```shell
pip install mlx-embeddings
```

## Key Features

### Text Embedding
- Load models via `load(model_name)` returning `(model, tokenizer)`
- Access raw CLS token embeddings or mean-pooled/normalized `text_embeds`
- Pooling strategy: mean pooling for BERT/XLM-RoBERTa; config-defined for ModernBERT

### Multimodal Embedding (Qwen3-VL)
- High-level `model.process(inputs, processor=processor)` API
- Inputs can be text-only, image-only, or text+image dicts
- Outputs dense embeddings (e.g., shape `(4, 2048)`); similarity via dot product
- Relates to [[concepts/multimodal-embeddings]] — joint text+image embedding spaces

### Multimodal Reranking (Qwen3-VL-Reranker)
- Scores a query against a list of documents (text or image)
- Returns relevance scores per document

### Vision Models (SigLIP)
- Image-text matching via `logits_per_image` → `mx.sigmoid` → match probabilities
- Batch processing: multiple images vs. multiple text descriptions

### Late Interaction Retrieval (ColPali/ColQwen)
- Token-level embeddings for queries and images; see [[concepts/late-interaction-retrieval]]
- MaxSim scoring: `sum(max(query_embeds @ image_embeds.T, axis=1))`
- Manual pipeline: `get_input_embeddings_batch`, RoPE position IDs, `embedding_proj_layer`, `normalize_embeddings`

### Sequence Classification
- Supported via `pooler_output` logits and `id2label` from model config

### Masked Language Modeling
- Predict `[MASK]` tokens using `pooler_output` logits

## Model Conversion & Quantization

Convert Hugging Face models to MLX format:

```shell
python -m mlx_embeddings.convert --hf-path <model> --mlx-path <output>
```

### Quantization Modes

| Mode | Group Size | Bits | Use Case |
|---|---|---|---|
| `affine` (default) | 64 | 4 | General purpose |
| `mxfp4` | 32 | 4 | MLX float 4-bit |
| `nvfp4` | 16 | 4 | NVIDIA float 4-bit |
| `mxfp8` | 32 | 8 | Higher precision |

Additional options: `--dtype` (float16/bfloat16/float32), `--dequantize`, `--upload-repo` (Hugging Face Hub). See also [[concepts/model-quantization]] for broader context on quantization strategies.

## Related Concepts
- [[concepts/transformer-architecture]]

- [[concepts/mlx-framework]] — Apple's ML framework for Apple Silicon
- [[concepts/multimodal-embeddings]] — joint text+image embedding spaces
- [[concepts/late-interaction-retrieval]] — token-level multi-vector retrieval (ColPali/ColQwen)
- [[concepts/model-quantization]] — reducing model size/latency via lower-bit representations
- [[concepts/local-llm-inference]] — running models locally on consumer hardware
- [[concepts/retrieval-augmented-generation]] — downstream use case for embedding models

{"brief": "MLX-Embeddings enables local text and vision embedding model inference on Apple Silicon Macs via MLX.", "content": "# MLX-Embeddings\n\n**Source:** [GitHub – Blaizzy/mlx-embeddings](https://github.com/Blaizzy/mlx-embeddings)  \n**Author:** Blaizzy  \n**License:** GNU GPL v3\n\n## Overview\n\nMLX-Embeddings is a Python package for running embedding models (text and vision) locally on Apple Silicon Macs using Apple's [[concepts/mlx-framework]] machine learning framework. It supports single-item and batch processing, similarity comparison utilities, model conversion, and quantization.\n\n## Supported Model Architectures\n\n| Architecture | Notes |\n|---|---|\n| XLM-RoBERTa | Cross-lingual, multilingual text embedding |\n| BERT | Classic bidirectional encoder |\n| ModernBERT | Modernized bidirectional encoder-only Transformer |\n| Qwen3 | Text embedding model |\n| Qwen3-VL | Multimodal embedding + reranking |\n| Llama Bidirectional | e.g. NVIDIA NV-Embed |\n| Llama Nemotron VL | SigLIP vision + bidirectional Llama |\n| SigLIP | Vision-language similarity scoring |\n| ColPali / ColQwen | [[concepts/late-interaction-retrieval]] multimodal models |\n\n## Installation\n\n```shell\npip install mlx-embeddings\n```\n\n## Key Features\n\n### Text Embedding\n- Load models via `load(model_name)` returning `(model, tokenizer)`\n- Access raw CLS token embeddings or mean-pooled/normalized `text_embeds`\n- Pooling strategy: mean pooling for BERT/XLM-RoBERTa; config-defined for ModernBERT\n\n### Multimodal Embedding (Qwen3-VL)\n- High-level `model.process(inputs, processor=processor)` API\n- Inputs can be text-only, image-only, or text+image dicts\n- Outputs dense embeddings (e.g., shape `(4, 2048)`); similarity via dot product\n- Relates to [[concepts/multimodal-embeddings]] — joint text+image embedding spaces\n\n### Multimodal Reranking (Qwen3-VL-Reranker)\n- Scores a query against a list of documents (text or image)\n- Returns relevance scores per document\n\n### Vision Models (SigLIP)\n- Image-text matching via `logits_per_image` → `mx.sigmoid` → match probabilities\n- Batch processing: multiple images vs. multiple text descriptions\n\n### Late Interaction Retrieval (ColPali/ColQwen)\n- Token-level embeddings for queries and images; see [[concepts/late-interaction-retrieval]]\n- MaxSim scoring: `sum(max(query_embeds @ image_embeds.T, axis=1))`\n- Manual pipeline: `get_input_embeddings_batch`, RoPE position IDs, `embedding_proj_layer`, `normalize_embeddings`\n\n### Sequence Classification\n- Supported via `pooler_output` logits and `id2label` from model config\n\n### Masked Language Modeling\n- Predict `[MASK]` tokens using `pooler_output` logits\n\n## Model Conversion & Quantization\n\nConvert Hugging Face models to MLX format:\n\n```shell\npython -m mlx_embeddings.convert --hf-path <model> --mlx-path <output>\n```\n\n### Quantization Modes\n\n| Mode | Group Size | Bits | Use Case |\n|---|---|---|---|\n| `affine` (default) | 64 | 4 | General purpose |\n| `mxfp4` | 32 | 4 | MLX float 4-bit |\n| `nvfp4` | 16 | 4 | NVIDIA float 4-bit |\n| `mxfp8` | 32 | 8 | Higher precision |\n\nAdditional options: `--dtype` (float16/bfloat16/float32), `--dequantize`, `--upload-repo` (Hugging Face Hub). See also [[concepts/model-quantization]] for broader context on quantization strategies.\n\n## Related Concepts\n\n- [[concepts/mlx-framework]] — Apple's ML framework for Apple Silicon\n- [[concepts/multimodal-embeddings]] — joint text+image embedding spaces\n- [[concepts/late-interaction-retrieval]] — token-level multi-vector retrieval (ColPali/ColQwen)\n- [[concepts/model-quantization]] — reducing model size/latency via lower-bit representations\n- [[concepts/local-llm-inference]] — running models locally on consumer hardware\n- [[concepts/retrieval-augmented-generation]] — downstream use case for embedding models"}