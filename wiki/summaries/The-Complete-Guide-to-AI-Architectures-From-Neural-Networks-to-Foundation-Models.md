---
doc_type: short
full_text: sources/The-Complete-Guide-to-AI-Architectures-From-Neural-Networks-to-Foundation-Models.md
---

# The Complete Guide to AI Architectures: From Neural Networks to Foundation Models

## Overview
A comprehensive survey of AI architectures published July 2025, covering mathematical foundations, practical implementations, historical evolution, and real-world applications. The guide frames 2025 as a pivotal year where test-time compute scaling joins pre-training and post-training scaling as a third major scaling paradigm, all explored through the lens of [[concepts/scaling-laws]].

## Three Scaling Laws
- **Pre-training scaling**: More data and parameters
- **Post-training scaling**: Fine-tuning and alignment optimization
- **Test-time scaling**: Inference-time reasoning (e.g., o1/o3 reasoning for 20 seconds to match 100,000× more parameters)

## Core Architectures

### [[concepts/transformer-architecture]]
- Introduced in "Attention is All You Need" (2017); replaces recurrence with parallel self-attention
- Core equation: `Attention(Q,K,V) = softmax(QK^T/√d_k)V`
- Multi-head attention allows specialization across syntax, semantics, and long-range dependencies
- Three major variants: **BERT** (encoder, bidirectional understanding), **GPT** (decoder, autoregressive generation), **T5** (unified text-to-text)
- Modern: GPT-4.5, Claude 4 (70.3% on SWE benchmarks), Gemini 2.5 — all multimodal
- Reasoning models (o1, o3) represent test-time compute scaling paradigm

### Convolutional Neural Networks
- Exploit translation invariance and local connectivity for spatial pattern recognition
- Key formula: `(f * g)[n] = Σ f[m] * g[n-m]`
- **ResNet**: Skip connections solve vanishing gradients; learns residual F(x) = H(x) − x
- **DenseNet**: Every layer connects to all subsequent layers; efficient feature reuse
- **EfficientNet**: Compound scaling of depth, width, resolution via principled formula
- Applications: 96% accuracy on radiology, 95%+ object detection for autonomous vehicles

### Vision Transformers
- Divide images into 16×16 or 32×32 patches treated as token sequences
- Require large datasets (ImageNet-22K, JFT-300M) due to lacking CNN inductive biases
- **Swin Transformer**: Shifted windows reduce attention complexity from quadratic to linear
- ViT-Huge (632M parameters) achieves state-of-the-art on multiple benchmarks
- Hybrid CNN+ViT architectures combine spatial inductive biases with transformer capacity

### Generative Adversarial Networks
- Minimax game: Generator vs. Discriminator
- `min_G max_D V(D,G) = E[log D(x)] + E[log(1 − D(G(z)))]`
- **StyleGAN**: Mapping network + adaptive instance normalization for controllable generation
- Challenges: mode collapse, training instability; mitigated by Wasserstein loss, spectral normalization
- Applications: art generation, fashion design, data augmentation

### Diffusion Models
- Learn to reverse a noise-corruption process: forward (add noise) → reverse (denoise)
- Forward: `q(x_t|x_{t-1}) = N(x_t; √(1-β_t)x_{t-1}, β_t I)`
- **Stable Diffusion**: Latent diffusion in compressed space; uses CLIP text encoder
- Techniques: classifier-free guidance, DDIM for faster sampling
- Powers DALL-E 2, Midjourney, video generation (Sora, Veo 3)

### Recurrent Neural Networks
- Sequential hidden state: `h_t = tanh(W_hh h_{t-1} + W_ih x_t + b_h)`
- Vanishing gradient problem solved by **LSTM** (forget/input/output gates, cell state)
- **GRU**: Simplified two-gate version; comparable performance, more efficient
- Still relevant for real-time streaming tasks despite transformer dominance

### Variational Autoencoders
- Learn distributions over latent space; optimize ELBO: `E_q[log p(x|z)] − KL(q(z|x)||p(z))`
- Reparameterization trick enables gradient flow: `z = μ + σ ⊙ ε`
- **β-VAE**: Higher KL weight encourages disentangled latent dimensions
- **VQ-VAE**: Discrete codebook latents for high-fidelity image/audio generation
- Applications: drug discovery, anomaly detection, dimensionality reduction

### Graph Neural Networks
- Message passing framework: `h_v^(l+1) = UPDATE(h_v^(l), AGGREGATE({h_u^(l) : u ∈ N(v)}))`
- **GCN**: Spectral convolution via normalized adjacency matrix
- **GAT**: Learned attention weights over neighbors; multi-head variant
- Applications: molecular property prediction, fraud detection, knowledge graph completion, traffic forecasting

### Reinforcement Learning Architectures
- **DQN**: Neural Q-function with experience replay and target networks
- **Actor-Critic**: Separate policy (actor) and value (critic) networks
- **PPO**: Clipped objective prevents destructive policy updates
- Applications: AlphaGo, robotics, autonomous vehicles, algorithmic trading

## Emerging Architectures
- **NAS (Neural Architecture Search)**: DARTS makes search differentiable
- **Capsule Networks**: Vector activations encoding feature properties and pose
- **Neural ODEs**: Continuous dynamics `dh/dt = f(h(t), t, θ)`; adaptive compute depth
- **Mixture of Experts**: Sparse activation; scales to trillions of parameters at constant inference cost; likely used in GPT-4, PaLM

## Historical Evolution
| Era | Key Development |
|---|---|
| 1950s–1970s | Symbolic AI (Logic Theorist, LISP); brittle, knowledge bottleneck |
| 1980s–1990s | Backpropagation (1986), Bayesian networks, SVMs |
| 2012 | AlexNet on ImageNet — 15.3% vs. 26.1% error; deep learning revolution |
| 2017 | Transformers ("Attention is All You Need") |
| 2018–2023 | BERT → GPT-3 → GPT-4; exponential capability growth |
| 2025 | Reasoning models, multimodal unification, test-time scaling |

Key figures: Geoffrey Hinton (backpropagation/deep learning), Yann LeCun (CNNs), Yoshua Bengio (RNNs/attention), Vaswani et al. (transformers).

## Practical Implementation
- **PyTorch**: Best for research; dynamic graphs, Hugging Face ecosystem
- **TensorFlow**: Best for production; TFX pipelines, TF Lite for mobile
- **JAX**: XLA compilation; functional approach for custom research architectures
- Memory rule: peak ≈ 16 × params + 4 × buffer bytes (7B model ≈ 28 GB)
- Optimization: quantization (75–80% size reduction, <2% accuracy loss), pruning (30–50% parameter removal), knowledge distillation (90–95% teacher performance)

## Industry Impact (2025)
- Healthcare AI: $32.3B → $208.2B by 2030; 96% radiology accuracy
- Finance: 300% fraud detection improvement; 1.5B+ chatbot interactions
- Manufacturing: 25% defect reduction via computer vision
- Creative: 100,000–150,000 AI-assisted songs/day; video indistinguishable from live action
- Productivity: 20–30% gains reported across AI-integrated organizations

## Future Directions
- **Neuromorphic computing**: Spiking neural networks; dramatically lower power
- **Neuro-symbolic AI**: Combining learned representations with logical reasoning
- **Quantum ML**: Potential acceleration of certain algorithms
- **Embodied AI**: Integration with robotics and physical systems
- Data wall and energy constraints are key scaling challenges per [[concepts/scaling-laws]]; synthetic data and mixture of experts are mitigation paths

## Key Takeaway
Architecture selection should match problem requirements: CNNs for spatial patterns, transformers (see [[concepts/transformer-architecture]]) for sequential/reasoning tasks, graph neural networks for relational data, diffusion models for high-quality generation. The emergence of test-time compute scaling — a new dimension described in [[concepts/scaling-laws]] — fundamentally changes the performance/parameter tradeoff.