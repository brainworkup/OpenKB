---
sources: [summaries/Introducing-FrontierCode.md, summaries/A-Mac-Studio-for-Local-AI-6-Months-Later.md, summaries/The-Complete-Guide-to-AI-Architectures-From-Neural-Networks-to-Foundation-Models.md]
brief: The three empirical laws governing how AI model performance improves with more resources.
---

# Scaling Laws in AI

Scaling laws are empirical relationships describing how AI model performance improves as a function of increased resources—whether parameters, data, compute, or inference time. Understanding scaling laws is essential for making principled decisions about where to invest resources when building and deploying AI systems.

See also: [[summaries/The-Complete-Guide-to-AI-Architectures-From-Neural-Networks-to-Foundation-Models]]

## The Three Scaling Laws (2025)

As of 2025, the field recognizes three distinct and complementary scaling laws, each representing a different axis along which AI performance can be improved.

### 1. Pre-Training Scaling

The original and most studied scaling law: performance improves predictably as models are trained on more data with more parameters using more compute. Foundational work by Kaplan et al. (OpenAI) and Hoffmann et al. (DeepMind, "Chinchilla") established that model size and training data should scale together for optimal efficiency.

Key implications:
- Larger models trained on proportionally more data yield consistent performance gains
- Loss decreases as a power law with respect to compute budget
- Drove the progression from GPT-2 (1.5B parameters) to GPT-4 (175B+ parameters)

Limitations now emerging: the **data wall** — high-quality training data is becoming scarce, threatening to cap progress along this axis.

### 2. Post-Training Scaling

Performance can be further improved after initial pre-training through fine-tuning, reinforcement learning from human feedback (RLHF), alignment optimization, and other post-training techniques. This scaling law recognizes that a pre-trained model is a starting point, not an endpoint.

Key techniques in post-training scaling:
- Supervised fine-tuning on curated instruction datasets
- RLHF and RLAIF (RL from AI feedback)
- Direct Preference Optimization (DPO)
- Targeted domain adaptation

This axis enabled models like Claude 4 to achieve 70.3% accuracy on software engineering benchmarks and drove the commercial viability of large language models.

### 3. Test-Time Compute Scaling

The most recent and arguably most disruptive scaling law: **performance improves when more compute is allocated at inference time**, allowing models to "think longer" on hard problems. This paradigm was crystallized by OpenAI's o1 and o3 reasoning models in 2024–2025.

The core finding: a model reasoning for 20 seconds at inference can match what would otherwise require **100,000× more parameters** under traditional pre-training scaling.

Mechanisms for test-time scaling:
- Chain-of-thought reasoning (generating intermediate reasoning steps)
- Extended internal "thinking" before producing an answer
- Self-consistency (sampling multiple solutions and selecting the best)
- Tree-of-thought search over reasoning paths
- Iterative refinement and self-correction

This law implies that computational resources can be **allocated dynamically based on problem complexity** — easy questions get fast answers, hard problems get extended reasoning — fundamentally changing the economics of AI deployment.

## Why Scaling Laws Matter

Scaling laws provide:
1. **Predictability**: Engineers can forecast performance improvements before committing to expensive training runs
2. **Resource allocation**: Organizations can decide whether to invest in bigger models, more data, better fine-tuning, or smarter inference
3. **Architectural guidance**: Understanding scaling behavior helps identify when an architecture is hitting a ceiling

## Scaling Laws and Architecture Choice

Different architectures interact with scaling laws differently:

- **[[concepts/transformer-architecture]]**: Exhibits strong pre-training and test-time scaling; the dominant architecture for reasoning models
- **Mixture of Experts (MoE)**: Enables pre-training scaling to trillions of parameters while keeping inference compute constant — a form of efficient scaling
- **Diffusion models**: Scale well with training compute for image quality; test-time scaling explored via iterative refinement steps
- **CNNs**: Show diminishing returns at very large scale compared to transformers for general tasks

## Current Challenges and Frontiers

### The Data Wall
High-quality human-generated text is finite. As pre-training scaling approaches this ceiling, the field is turning to:
- Synthetic data generation (models generating training data for other models)
- Multimodal data (images, video, audio) as additional training signal
- More efficient use of existing data through curriculum learning

### Energy and Infrastructure Limits
Large-scale pre-training now approaches the limits of available computational infrastructure. The industry is responding with:
- Specialized hardware (TPUs, neuromorphic chips)
- Alternative energy partnerships (including nuclear)
- Architectural innovations that achieve more per FLOP

### The Shift in Investment Logic
Test-time compute scaling changes where companies invest:
- Less pressure to always train larger base models
- More value in inference infrastructure that can support extended reasoning
- New product designs where users can trade latency for answer quality

## Summary

The three scaling laws — pre-training, post-training, and test-time — represent complementary levers for improving AI performance. The emergence of test-time compute scaling in 2025 is particularly significant: it decouples model capability from model size in a new way, opening a path forward even as traditional parameter scaling faces economic and physical constraints. For practitioners, understanding which scaling axis is most relevant to their application is now a core competency in AI system design.

See also: [[summaries/A-Mac-Studio-for-Local-AI-6-Months-Later]]

See also: [[summaries/Introducing-FrontierCode]]