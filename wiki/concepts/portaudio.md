---
sources: [summaries/top_level.md, summaries/README.md]
brief: PortAudio is an open-source, cross-platform audio I/O library with pre-compiled binaries for Windows and macOS.
---

# PortAudio: Cross-Platform Audio I/O Library

PortAudio is a free, open-source audio I/O library designed to provide a uniform interface for low-latency audio input and output across multiple operating systems and hardware backends. It is widely used in audio software, research tools, and digital signal processing applications.

## Overview

PortAudio abstracts platform-specific audio APIs behind a single, consistent programming interface. Applications written against PortAudio can run on Windows, macOS, and Linux without requiring changes to the core audio logic.

- **Authors**: Ross Bencina and Phil Burk
- **License**: MIT
- **Website**: [http://www.portaudio.com/](http://www.portaudio.com/)

## Pre-Compiled Binaries

Pre-compiled dynamic libraries are distributed for ease of integration. See [[summaries/README]] for the full distribution details.

### Windows DLLs (32-bit and 64-bit)

Two variants are provided:

1. **Standard DLL** — Includes the default Windows host APIs:
   - **MME** (Multimedia Extensions): legacy Windows audio API
   - **DirectSound**: Microsoft DirectX audio layer
   - **WDM/KS** (Windows Driver Model / Kernel Streaming): low-latency driver-level access
   - **WASAPI** (Windows Audio Session API): modern Windows audio stack

2. **ASIO DLL** (`*-asio.dll`) — Adds support for [[concepts/asio-audio]] (Steinberg Audio Stream I/O API), enabling professional, ultra-low-latency audio on compatible hardware.

### macOS dylib (64-bit Universal)

The `libportaudio.dylib` file is a [[concepts/universal-binary]], supporting:
- Intel (`x86_64`) architecture
- Apple Silicon (`arm64`) architecture

This ensures compatibility across the full range of modern Mac hardware.

## Build and Distribution

All binaries are automatically generated via **GitHub Actions**, ensuring reproducible and auditable builds. The build configuration lives in `.github/workflows/build-libs.yml`.

## Related Concepts

- [[concepts/asio-audio]] — Low-latency audio API by Steinberg, optionally bundled with PortAudio
- [[concepts/universal-binary]] — macOS fat binaries supporting multiple CPU architectures
- [[concepts/audio-transcription-pipeline]] — Pipelines that may consume audio I/O libraries like PortAudio

## Use Cases

- Real-time audio processing applications
- Cross-platform audio tools and DAW plugins
- Research and speech/audio ML pipelines requiring hardware-level audio capture
- Integration with transcription and speech recognition systems


See also: [[summaries/top_level]]