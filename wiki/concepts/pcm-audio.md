---
sources: [summaries/top_level.md]
brief: PCM audio stores sound as sequences of discrete integer samples at fixed bit widths.
---

# PCM Audio and Sample Formats

Pulse Code Modulation (PCM) is the standard method for digitally representing sampled analog audio. Audio data is encoded as a sequence of discrete numeric samples, each capturing the amplitude of the sound wave at a specific point in time.

## Core Concepts

### Sample Width
The **bit depth** (or sample width) determines the resolution of each amplitude measurement:
- **8-bit**: 256 possible amplitude values; low fidelity, small file size
- **16-bit**: 65,536 values; CD-quality audio standard
- **32-bit**: ~4.3 billion values; high-precision professional audio

In Python's `audioop` module (see [[summaries/top_level]]), functions accept a `width` parameter specifying the byte width of each sample (1, 2, or 4 bytes for 8, 16, or 32-bit respectively).

### Sample Rate
The number of samples captured per second (Hz). Common rates:
- 8,000 Hz — telephony (narrowband)
- 22,050 Hz — radio quality
- 44,100 Hz — CD audio
- 48,000 Hz — professional/broadcast

### Signed vs. Unsigned Samples
- **Unsigned**: values range from 0 to 2ⁿ−1 (e.g., 0–255 for 8-bit)
- **Signed**: values range from −2ⁿ⁻¹ to 2ⁿ⁻¹−1 (e.g., −128 to 127 for 8-bit); silence is at 0

## Linear PCM vs. Compressed Encodings

Linear PCM stores raw, uncompressed sample values. Several compressed or companded encodings reduce data size at the cost of some fidelity:

| Encoding | Description |
|---|---|
| **u-law (μ-law)** | Logarithmic companding used in North American telephony |
| **A-law** | Similar logarithmic companding used in European telephony |
| **ADPCM** | Adaptive Differential PCM; encodes differences between samples |

The `audioop` module provides conversion functions between linear PCM and these formats (e.g., `lin2ulaw`, `alaw2lin`, `lin2adpcm`), supporting [[concepts/audio-encoding]] workflows.

## Fragment-Based Processing

In Python's `audioop`, audio is passed as **fragments** — bytes-like objects containing a flat sequence of samples. Operations are performed on entire fragments:
- `getsample(fragment, width, index)` — extract individual sample
- `max(fragment, width)` — peak amplitude
- `rms(fragment, width)` — root-mean-square energy
- `add` / `mul` — arithmetic on sample arrays

This fragment model is well suited to streaming pipelines where audio arrives in chunks.

## Stereo and Mono Channels

PCM audio can be:
- **Mono**: single channel, one sample per time step
- **Stereo**: two interleaved channels (left, right), two samples per time step

Conversion functions like `tomono` and `tostereo` handle channel manipulation at the sample level.

## Rate Conversion

Changing the sample rate of a PCM fragment (upsampling or downsampling) requires resampling — `audioop.ratecv` performs this with a stateful algorithm that can be applied to streaming audio buffers.

## Relationship to Digital Signal Processing

PCM is the foundation for nearly all [[concepts/digital-signal-processing]] tasks: filtering, FFT analysis, feature extraction (e.g., [[concepts/filterbanks]]), and codec pipelines. Understanding sample width, rate, and encoding is prerequisite to any audio signal work.

## Related Pages
- [[summaries/top_level]] — `audioop` module overview
- [[concepts/audio-encoding]] — compressed audio encoding formats
- [[concepts/digital-signal-processing]] — broader DSP concepts
- [[concepts/filterbanks]] — frequency-domain analysis of audio
- [[concepts/asio-audio]] — low-latency audio I/O
- [[concepts/portaudio]] — cross-platform audio I/O library
