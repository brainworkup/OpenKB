---
sources: [summaries/top_level.md]
brief: Filterbanks decompose audio signals into frequency sub-bands for encoding in deep learning pipelines.
---

# Filterbanks in Audio Signal Processing

Filterbanks are signal processing constructs that decompose an audio signal into multiple frequency sub-bands. In modern deep learning pipelines for audio, filterbanks serve as the front-end encoder — transforming raw waveforms into representations suitable for downstream neural network processing.

## Core Concepts

### What Is a Filterbank?

A filterbank is a collection of bandpass filters that partition the frequency spectrum of an input signal. Each filter passes a specific range of frequencies while attenuating others. The outputs of all filters together form a multi-channel time-frequency representation of the original waveform.

### Fixed vs. Learned Filterbanks

- **Fixed filterbanks** (e.g., mel-scale filterbanks, short-time Fourier transform) are hand-designed based on signal processing theory or auditory perception models. They are not updated during training.
- **Learned filterbanks** are parameterized as neural network layers whose weights are optimized end-to-end during training. This allows the model to discover task-optimal frequency decompositions automatically.

### Role in Neural Audio Processing

In neural network architectures for audio:

1. A filterbank **encoder** converts raw waveform samples into a latent representation.
2. A processing network (e.g., a separator or classifier) operates on this representation.
3. A filterbank **decoder** (synthesis filterbank) reconstructs the output waveform.

This encoder–decoder structure is foundational in audio source separation, speech enhancement, and audio codec design.

## Filterbanks in the Asteroid Toolkit

The `asteroid_filterbanks` module (referenced in [[summaries/top_level]]) provides implementations of both fixed and learnable filterbanks for use in the [[concepts/asteroid-audio-toolkit]] framework. Key capabilities include:

- Modular filterbank definitions usable as PyTorch `nn.Module` layers
- Support for short-time Fourier transform (STFT), free filterbanks, and analytic filterbanks
- Integration with encoder–decoder wrappers for end-to-end training
- Compatibility with multi-channel and single-channel audio pipelines

## Signal Encoding Architecture

The general pipeline structure using filterbanks:

```
Raw Waveform
    ↓
[Encoder Filterbank]  ← learned or fixed
    ↓
Latent Representation
    ↓
[Neural Network Processor]
    ↓
Separated Latent Streams
    ↓
[Decoder Filterbank]  ← synthesis / reconstruction
    ↓
Output Waveform(s)
```

## Related Concepts

- **Audio source separation**: The primary application of filterbank-based neural architectures, where the goal is to isolate individual sound sources from a mixture.
- **Mel-scale filterbanks**: A perceptually motivated fixed filterbank based on the mel frequency scale, widely used in speech recognition and audio classification.
- **Encoder–decoder architectures**: The broader architectural pattern within which filterbanks function as the waveform-domain interface.
- [[concepts/asio-audio]]: Low-latency audio I/O standards relevant to real-time filterbank processing.
- [[concepts/portaudio]]: Cross-platform audio I/O library that may interface with filterbank-based processing pipelines.

## References

- [[summaries/top_level]] — Top-level index entry for `asteroid_filterbanks`
