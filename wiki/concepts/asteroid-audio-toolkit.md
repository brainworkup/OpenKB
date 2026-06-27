---
sources: [summaries/top_level.md]
brief: Asteroid is a PyTorch-based open-source toolkit for audio source separation with modular filterbank components.
---

# Asteroid Audio Source Separation Toolkit

Asteroid is an open-source, PyTorch-based framework designed for audio source separation research and deployment. It provides modular, reusable components — most notably **filterbanks** — that serve as signal encoding front-ends in deep learning audio pipelines.

## Core Components

### Filterbanks

Filterbanks are the primary signal-processing building blocks in Asteroid. They decompose an audio mixture signal into frequency sub-bands, enabling neural networks to operate on a learned or fixed spectral representation.

- **Learned filterbanks**: Trained end-to-end as part of the model, allowing the network to discover optimal signal decompositions for separation tasks.
- **Fixed filterbanks**: Based on classical signal processing (e.g., STFT, mel-scale), applied as frozen front-ends.
- The `asteroid_filterbanks` module exposes these encoders and decoders as composable PyTorch modules.

See [[concepts/filterbanks]] for a broader treatment of filterbank theory and usage across audio pipelines.

## Relationship to Audio Source Separation

Audio source separation is the task of isolating individual sound sources (e.g., speakers, instruments) from a mixed audio signal. Asteroid provides:

- Encoder–decoder architectures (analysis filterbank → separator network → synthesis filterbank)
- Pretrained models and training recipes
- Integration with standard loss functions for time-domain and frequency-domain separation

## Toolkit Architecture

| Layer | Role |
|---|---|
| Filterbanks | Signal encoding/decoding front-end |
| Separator | Neural network core (e.g., Conv-TasNet, DPRNN) |
| Loss functions | SI-SNR, PIT (Permutation Invariant Training) |
| Data utilities | Dataset loaders, mixing simulators |

## Relation to This Wiki

The document [[summaries/top_level]] serves as a top-level index entry pointing to the `asteroid_filterbanks` submodule, suggesting it is part of a larger documentation or package structure built around the Asteroid ecosystem.

## Related Concepts

- [[concepts/filterbanks]] — The core signal processing concept underlying Asteroid's encoder/decoder design
- [[concepts/asio-audio]] — Low-latency audio I/O relevant to real-time source separation deployment
- [[concepts/portaudio]] — Cross-platform audio I/O library that may integrate with Asteroid-based pipelines
- [[concepts/audio-transcription-pipeline]] — A downstream application of source separation: isolating speech before transcription
- [[concepts/mlx-framework]] — Apple Silicon ML framework that could accelerate local Asteroid inference
