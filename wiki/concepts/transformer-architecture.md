---
sources: [summaries/mlx_embeddings.md, summaries/A-Mac-Studio-for-Local-AI-6-Months-Later.md, summaries/The-Complete-Guide-to-AI-Architectures-From-Neural-Networks-to-Foundation-Models.md]
brief: The transformer architecture uses parallel self-attention to model sequences, dominating modern AI.
---

# Transformer Architecture

The transformer architecture, introduced in the landmark 2017 paper *"Attention is All You Need"* by Vaswani et al. at Google, fundamentally changed how machines process sequential data. By replacing recurrent processing with parallel self-attention mechanisms, transformers enabled training of far larger models while capturing long-range dependencies more effectively than any previous approach.

See also: [[summaries/The-Complete-Guide-to-AI-Architectures-From-Neural-Networks-to-Foundation-Models]]

## Core Mechanism: Self-Attention

The central innovation is the self-attention mechanism, which allows every position in a sequence to attend to every other position simultaneously. Three learned linear projections — Query (Q), Key (K), and Value (V) — define the operation:

```
Attention(Q, K, V) = softmax(QK^T / √d_k) V
```

The scaling factor `√d_k` prevents the dot products from growing too large in high-dimensional spaces, stabilizing gradients through the softmax.

### Multi-Head Attention

In practice, multiple attention functions are run in parallel as **attention heads**. Each head can specialize in different types of relationships — syntactic structure, semantic similarity, or long-range co-reference — and their outputs are concatenated and linearly projected. This allows the model to simultaneously represent multiple types of relationships within the same layer.

### Positional Encoding

Because transformers process all positions simultaneously rather than sequentially, they have no inherent sense of order. Positional encodings — either fixed sinusoidal functions (original paper) or learned embeddings — are added to token embeddings to supply position information. Modern implementations often use relative position encodings that generalize better to sequence lengths unseen during training.

## Encoder-Decoder Structure and Major Variants

The original transformer used a full encoder-decoder design: the encoder reads and represents the input; the decoder generates output tokens autoregressively, attending to both its own previous outputs and the encoder's representations. Three major variants have since emerged:

| Variant | Components | Training Objective | Strengths |
|---|---|---|---|
| **BERT** | Encoder only | Masked language modeling (predict masked words) | Deep bidirectional understanding; classification, QA |
| **GPT** | Decoder only | Next-token prediction (autoregressive) | Scalable generation; few-shot learning |
| **T5** | Encoder + Decoder | Text-to-text (all tasks as generation) | Unified multi-task framework |

### BERT
BERT processes entire sequences bidirectionally, seeing context from both left and right at every layer. Its pre-training task — predicting randomly masked tokens — builds rich linguistic representations useful for downstream understanding tasks such as sentiment analysis, question answering, and named entity recognition.

### GPT
GPT-style models predict the next token given all previous tokens. This autoregressive objective scales remarkably well: GPT-4 (175B+ parameters) demonstrates emergent capabilities including few-shot learning and chain-of-thought reasoning that were not explicitly trained.

### T5
T5 frames every NLP task — translation, summarization, classification — as a text-to-text problem. This unification allows a single model to handle diverse tasks through different input prompt formats.

## Scaling and Emergent Capabilities

Transformers are the primary subject of modern [[concepts/scaling-laws]] research. Scaling model size, dataset size, and compute together produces predictable improvements — and at sufficient scale, qualitatively new capabilities emerge that smaller models lack entirely.

Key scaling milestones:
- **BERT (2018)**: Demonstrated pre-training + fine-tuning paradigm
- **GPT-3 (2020)**: 175B parameters; strong few-shot performance
- **GPT-4 (2023)**: Multimodal understanding; professional-level reasoning
- **Claude 4 (2025)**: 70.3% accuracy on software engineering benchmarks

## Reasoning Models and Test-Time Compute

A pivotal 2025 development is the shift from parameter scaling to **test-time compute scaling**. Models like OpenAI's o1 and o3 allocate additional computation at inference time — reasoning explicitly for up to 20 seconds on hard problems — achieving results that would require orders-of-magnitude more parameters under traditional scaling. This represents a third scaling axis alongside pre-training and post-training scaling.

The implication is that computational resources can be allocated dynamically: simple queries receive fast responses, while complex problems trigger extended internal reasoning chains before any answer is produced.

## Multimodal Transformers

Modern transformers extend beyond text. Models such as GPT-4 Vision and Gemini 2.5 process text, images, audio, and video within unified transformer frameworks. Image patches are embedded as tokens (see Vision Transformers), audio frames are similarly tokenized, and cross-modal attention allows the model to relate information across modalities. The trend points toward "any-to-any" architectures capable of ingesting and generating any combination of modalities.

## Practical Considerations

- **Memory**: Peak usage ≈ 16 × number of parameters + buffer; a 7B model requires ~28 GB during training
- **Frameworks**: PyTorch (research, dynamic graphs) and TensorFlow (production, static optimization) both have mature transformer support via Hugging Face Transformers
- **Efficiency techniques**: Quantization (75–80% size reduction, <2% accuracy loss), gradient checkpointing, mixed precision training, and knowledge distillation reduce deployment costs substantially
- **Attention complexity**: Naive self-attention is O(n²) in sequence length; sparse attention, linear attention, and sliding-window variants address this for long sequences

## Relationship to Other Architectures

Transformers have displaced recurrent neural networks (RNNs, LSTMs, GRUs) for most NLP tasks due to parallelism and superior long-range modeling. Vision Transformers (ViTs) apply the same mechanism to image patches, competing with and often surpassing convolutional neural networks on large-scale benchmarks. Mixture-of-experts layers are frequently integrated within transformer blocks to scale capacity without proportional increases in per-token compute.

## Key Takeaway

The transformer is the dominant architectural paradigm of modern AI. Its scalability, flexibility across modalities and tasks, and compatibility with emergent reasoning behaviors make it the foundation on which virtually every frontier AI system in 2025 is built.

See also: [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]]

See also: [[summaries/mlx_embeddings]]