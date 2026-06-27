---
doc_type: short
full_text: sources/A-Mac-Studio-for-Local-AI-6-Months-Later.md
---

# A Mac Studio for Local AI — 6 Months Later

## Overview
A detailed, experience-driven guide to running 600B+ parameter language models locally on a Mac Studio M3 with 512GB unified memory. The author chronicles hardware selection, software stack configuration, performance tuning, and practical trade-offs after six months of daily use.

## Hardware Rationale
- The 512GB Mac Studio is presented as the only sub-$10k machine capable of housing the best open-weight models (e.g., Kimi K2.5 at ~1 trillion parameters)
- Runs cool and quiet; does not require exotic power infrastructure
- Apple Silicon's unified memory architecture is central to the feasibility argument for [[concepts/local-llm-inference]]
- At the time of writing, open-weight models are roughly equivalent to cloud API models from 6–12 months prior

## Performance Characteristics
- Prompt processing (prefill): slowest bottleneck; ~30s for 7k tokens, ~90s for 16k tokens
- Token generation (decode): streams at >3× reading speed
- Slower than cloud APIs, but practical for real workloads

## Software Stack
- **mlx-lm** / **mlx-vlm**: Core inference backends; MLX is 10–25% faster than llama.cpp in 2026
- **llama-swap**: API gateway managing multiple parallel model instances with a single endpoint
- **claude-code-router**: Translates Anthropic `/v1/messages` format to OpenAI-compatible format
- **Claude Code**: Primary agentic coding interface powered by local models
- Package management via `brew`, `uv`, `pnpm`

## Key Configuration Steps
1. **Maximize GPU memory**: Override macOS default 75% GPU memory cap via `sysctl`; persist with a Launch Daemon (~506GB available to GPU on 512GB machine)
2. **MLX server setup**: `uvx`-based OpenAI-compatible server per model
3. **Model selection**: Run three tiers — small (Qwen 3.5 35B), medium (Qwen 3.5 397B), large (Kimi K2.5 1T)
4. **Quantization**: 4-bit dynamic/mixed quantization is the recommended sweet spot on Apple Silicon; see [[concepts/model-quantization]]
5. **API gateway**: llama-swap YAML config with model aliases, thinking mode toggles, and generation parameter overrides
6. **Agentic interface**: claude-code-router bridges Anthropic API format to local stack

## Model Architecture Insights
- [[concepts/mixture-of-experts]] (MoE) models are especially attractive: large total parameters (intelligence) but only a fraction activated per token (speed)
  - Kimi K2.5: 1T total, 32B active → good speed
  - GLM 5: 744B total, 40B active → 20% slower than Kimi
  - Qwen 3.5 397B: 17B active → ~2× faster than Kimi
- Speed scales with *active* parameters, not total; this is a key insight related to [[concepts/scaling-laws]]

## Quantization Guidance
- [[concepts/model-quantization]] — 4-bit is the GPU-optimized sweet spot on Apple Silicon
- 8-bit ≈ full precision quality at half size
- Dynamic/mixed precision essential: keep sensitive weights high, compress others
- Larger models tolerate lower quants (<4-bit); smaller models need ≥4-bit
- Prefer quants with published benchmarks

## Prompt Processing Optimizations

### Reducing System Prompt Size
- Claude Code's default system prompt exceeds 20,000 tokens (verbose, repetitive)
- Author distilled it to <8,000 tokens via a custom lean system prompt + shell script for environment context
- Limit tools to only those needed (e.g., `Bash, Glob, Grep, Read, Edit, Write`)
- Result: faster prefill, larger effective context window

### Fixing Prompt Cache Invalidation
- Claude Code non-deterministically reorders tool definitions and arguments between turns, breaking KV cache reuse
- Fix: patch the model's Jinja chat template to sort tools/arguments (`tojson(sort_keys=True)`, `dictsort`)
- Prompt caching is critical for multi-turn chat performance in [[concepts/local-llm-inference]]

### Speculative Prompt Caching
- Qwen 3.5's dynamic `<think>` tag insertion causes the previous assistant turn to be reprocessed every turn
- Author's workaround: *speculative prompt processing* — immediately after a response, pre-compute the expected next cache state so the server gets a head start
- Available as an unmerged PR / fork of mlx-lm (`--prompt-cache-warmup` flag)

## Privacy and Sovereignty Benefits
- Running models locally aligns with [[concepts/privacy-first-software]] principles: no data leaves the machine, no vendor lock-in, no rate limits
- Full control over model parameters, quantization, and system prompts

## Cost–Benefit Reflection
- Mac Studio 512GB ≈ cost of ~1 billion uncached Opus tokens, or 4 years of $200/month Claude Max
- Not a drop-in replacement for cloud APIs; requires significant time investment
- Capability level reached: roughly equivalent to Claude Sonnet 3.7 (state-of-the-art ~1 year prior)
- Benefits: full privacy, no rate limits, no vendor lock-in
- Author notes ongoing improvements in models, quantization, and hardware make the investment trajectory positive

## Related Pages
- [[concepts/local-llm-inference]]
- [[concepts/mixture-of-experts]]
- [[concepts/model-quantization]]
- [[concepts/privacy-first-software]]
- [[concepts/scaling-laws]]
- [[concepts/transformer-architecture]]