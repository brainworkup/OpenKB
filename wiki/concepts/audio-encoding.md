---
sources: [summaries/top_level.md]
brief: Audio encoding formats define how raw audio samples are compressed or represented for storage and transmission.
---

# Audio Encoding Formats

Audio encoding formats define how raw audio samples are represented, compressed, or transformed for efficient storage, transmission, or processing. Different formats balance trade-offs between audio quality, compression ratio, computational complexity, and hardware compatibility.

## Core Concepts

### Linear PCM (Pulse-Code Modulation)
Linear PCM is the most fundamental audio representation: each audio sample is stored as a direct numerical value with no compression. It is the baseline format from which other encodings convert.

- **Sample widths**: 8-bit, 16-bit, or 32-bit integers per sample
- **Signed vs. unsigned**: Samples may be stored as signed or unsigned integers
- **Lossless**: No information is discarded

See [[concepts/pcm-audio]] for more detail.

### Logarithmic Companding Codecs
These codecs compress audio dynamic range using logarithmic curves, reducing bandwidth while maintaining perceptual quality. Widely used in telephony.

#### u-law (μ-law)
- Standard in North America and Japan
- Compresses 13–14 bits of linear PCM into 8 bits
- Conversion functions: `lin2ulaw` / `ulaw2lin` (as found in Python's `audioop` module — see [[summaries/top_level]])

#### A-law
- Standard in Europe and international telephony (ITU-T G.711)
- Similar compression to u-law with a different curve
- Conversion functions: `lin2alaw` / `alaw2lin`

### ADPCM (Adaptive Differential PCM)
- Encodes the *difference* between successive samples rather than absolute values
- The "adaptive" element adjusts the step size dynamically to match signal characteristics
- Intel/DVI ADPCM compresses 16-bit PCM to 4 bits per sample (4:1 ratio)
- Requires stateful processing: the codec state must be maintained across buffer boundaries
- Conversion functions: `lin2adpcm` / `adpcm2lin`
- Used in game audio, VoIP, and embedded systems

## Stateful vs. Stateless Encoding

Some audio codecs (like ADPCM) are **stateful**: encoding or decoding a chunk of audio depends on the output of the previous chunk. This requires passing state between calls when processing streaming audio, a pattern demonstrated in Python's `audioop` module.

Stateless formats (like raw u-law) can encode/decode each sample independently.

## Sample Rate Conversion

Beyond encoding format, audio may also need **sample rate conversion** — changing the number of samples per second (e.g., from 8 kHz telephony to 44.1 kHz CD quality). This is distinct from sample encoding but is often performed alongside format conversion.

See [[concepts/digital-signal-processing]] for broader signal transformation context.

## Practical Applications

| Encoding | Bit Rate | Common Use Case |
|---|---|---|
| Linear PCM 16-bit | 256 kbps (mono, 8 kHz) | Raw audio, editing |
| u-law 8-bit | 64 kbps | North American telephony (PSTN, VoIP) |
| A-law 8-bit | 64 kbps | European telephony |
| Intel ADPCM | 32 kbps | Game audio, embedded systems |

## Relationship to Audio Pipelines

Audio encoding formats are foundational to [[concepts/audio-transcription-pipeline]] systems, where raw audio must be decoded to PCM before speech recognition models can process it. Format conversion is also relevant to [[concepts/asio-audio]] and [[concepts/portaudio]] interfaces which operate on specific sample formats.

## See Also
- [[summaries/top_level]] — Python `audioop` module implementing core encoding operations
- [[concepts/pcm-audio]] — Linear PCM format in depth
- [[concepts/digital-signal-processing]] — Broader signal processing context
- [[concepts/filterbanks]] — Frequency-domain audio representations
- [[concepts/asio-audio]] — Low-latency audio interface formats
- [[concepts/portaudio]] — Cross-platform audio I/O library
