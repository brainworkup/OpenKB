---
sources: [summaries/top_level.md]
brief: Mathematical techniques for manipulating, analyzing, and transforming discrete digital audio and signal data.
---

# Digital Signal Processing

Digital Signal Processing (DSP) refers to the mathematical manipulation of discrete-time signals—typically represented as sequences of numerical samples—to analyze, filter, encode, or transform them for various applications.

## Core Principles

### Discrete Sampling
Analog signals (such as sound waves) are converted to digital form by sampling them at regular intervals. Each sample is stored as an integer or floating-point value. Common sample widths include:
- **8-bit**: Low fidelity, used in telephony
- **16-bit**: CD-quality audio
- **32-bit**: High-fidelity or professional audio

### Time-Domain Operations
Many DSP operations work directly on sample sequences:
- **Addition/mixing**: Summing two signals sample-by-sample
- **Scaling/multiplication**: Adjusting amplitude by multiplying each sample by a factor
- **Bias**: Shifting the DC offset of a signal
- **Reversal**: Reversing a sample sequence in time

### Frequency and Rate Conversion
Changing the sampling rate (resampling) is a fundamental DSP operation used when converting audio between formats or devices. This involves interpolation and filtering to avoid aliasing artifacts.

### Statistical Analysis
- **RMS (Root Mean Square)**: A measure of the power or loudness of an audio signal
- **Peak detection**: Finding maximum absolute sample values
- **Averaging**: Computing mean sample amplitude

## Audio Encoding Formats

DSP underpins numerous [[concepts/audio-encoding]] compression and transmission schemes:

- **PCM (Pulse Code Modulation)**: Raw linear [[concepts/pcm-audio]] samples, the baseline digital audio format
- **u-law / A-law**: Logarithmic companding algorithms used in telephony (G.711 standard) that reduce dynamic range for transmission
- **ADPCM (Adaptive Differential PCM)**: A lossy compression scheme that stores differences between samples rather than absolute values, reducing bandwidth

## Filterbanks and Spectral Processing

[[concepts/filterbanks]] are banks of bandpass filters applied to a signal to analyze its frequency content. They are foundational to:
- Speech recognition front-ends
- Audio compression (e.g., MP3, AAC)
- Feature extraction for machine learning

## DSP in Python

Python's standard library `audioop` module (see [[summaries/top_level]]) provides basic DSP operations on audio fragments:

| DSP Concept | `audioop` Function |
|---|---|
| Sample-wise mixing | `add()` |
| Amplitude scaling | `mul()` |
| RMS power | `rms()` |
| Rate conversion | `ratecv()` |
| u-law encoding | `lin2ulaw()` / `ulaw2lin()` |
| ADPCM encoding | `lin2adpcm()` / `adpcm2lin()` |

For more advanced DSP workflows, libraries such as NumPy and SciPy are preferred over `audioop`, which was deprecated in Python 3.11.

## Related Concepts

- [[concepts/audio-encoding]] — Specific codec formats built on DSP principles
- [[concepts/pcm-audio]] — The raw linear sample format central to DSP pipelines
- [[concepts/filterbanks]] — Frequency-domain decomposition tools
- [[concepts/asio-audio]] — Low-latency audio I/O relevant to real-time DSP
- [[concepts/portaudio]] — Cross-platform audio I/O layer used with DSP software
- [[concepts/audio-transcription-pipeline]] — Applied DSP in speech-to-text systems

## Applications
- Telephony and VoIP codec processing
- Audio format conversion and archiving
- Speech and music analysis
- Real-time audio effects and mixing
- Machine learning feature extraction from audio
