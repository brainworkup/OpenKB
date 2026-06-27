---
sources: [summaries/top_level.md, summaries/README.md]
brief: Steinberg's proprietary low-latency audio driver API enabling direct hardware access for professional audio.
---

# ASIO: Steinberg Low-Latency Audio API

## Overview

ASIO (Audio Stream I/O) is a proprietary audio driver protocol developed by **Steinberg Media Technologies GmbH**. It is designed to provide ultra-low-latency audio input/output by allowing audio software to communicate more directly with audio hardware, bypassing the latency overhead introduced by standard operating system audio stacks.

ASIO is widely used in professional audio production, music recording, and real-time audio processing contexts where minimal latency is critical.

## Relationship to PortAudio

[[concepts/portaudio]] is a cross-platform audio I/O library that can optionally support ASIO as a host API on Windows. Because the Steinberg ASIO SDK has licensing restrictions that differ from PortAudio's MIT License, ASIO support is distributed as a **separate optional binary** rather than being included in the standard build.

As described in [[summaries/README]]:
- Standard Windows DLLs include: MME, DirectSound, WDM/KS, and WASAPI.
- A separate `*-asio.dll` variant is provided for users who require ASIO support.

## Windows Host APIs Compared

| API | Latency | Notes |
|---|---|---|
| MME | High | Legacy Windows multimedia API |
| DirectSound | Medium | Older DirectX audio layer |
| WDM/KS | Low | Kernel Streaming, near-hardware access |
| WASAPI | Low | Modern Windows Audio Session API |
| **ASIO** | **Very Low** | Direct hardware access, professional audio |

## Licensing

- **PortAudio** is released under the MIT License (open source).
- **Steinberg ASIO SDK** is proprietary and governed by Steinberg Media Technologies GmbH's developer license. This is why ASIO-enabled builds are kept separate from the standard open-source distribution.
- More information: [Steinberg Developer Portal](http://www.steinberg.net/en/company/developers.html)

## Key Characteristics

- **Platform**: Windows only (ASIO is a Windows-specific protocol)
- **Latency**: Typically single-digit milliseconds, far below OS-level APIs
- **Use cases**: Digital audio workstations (DAWs), real-time audio synthesis, professional recording
- **Hardware support**: Requires ASIO-compatible audio interface or generic ASIO wrapper (e.g., ASIO4ALL)

## Related Concepts

- [[concepts/portaudio]] — The cross-platform audio library that optionally integrates ASIO
- [[concepts/universal-binary]] — macOS binary format (ASIO is Windows-only; macOS uses CoreAudio instead)
- [[concepts/audio-transcription-pipeline]] — Audio processing pipelines that may depend on low-latency I/O


See also: [[summaries/top_level]]