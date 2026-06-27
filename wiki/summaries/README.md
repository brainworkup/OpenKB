---
doc_type: short
full_text: sources/README.md
---

# README — PortAudio Binaries

## Overview
This repository distributes pre-compiled dynamic libraries for [PortAudio](http://www.portaudio.com/), a cross-platform audio I/O library. Libraries are provided for Windows (32-bit and 64-bit) and macOS (64-bit universal binary), built automatically via GitHub Actions.

## Provided Binaries

### Windows DLLs
- **Standard DLL**: Includes default host APIs:
  - MME (Multimedia Extensions)
  - DirectSound
  - WDM/KS (Windows Driver Model / Kernel Streaming)
  - WASAPI (Windows Audio Session API)
- **ASIO DLL** (`*-asio.dll`): Adds support for the [[concepts/asio-audio]] (Steinberg ASIO SDK) on top of the standard host APIs.

### macOS dylib
- `libportaudio.dylib`: [[concepts/universal-binary]] compatible with both:
  - Intel (`x86_64`)
  - Apple Silicon (`arm64`)

## Build Process
- All binaries are auto-generated using **GitHub Actions**.
- Build configuration is defined in `.github/workflows/build-libs.yml`.

## Key Concepts
- [[concepts/portaudio]] — cross-platform audio I/O library
- [[concepts/asio-audio]] — low-latency audio API by Steinberg
- [[concepts/universal-binary]] — macOS binaries supporting multiple CPU architectures

## Copyright
- **PortAudio**: Ross Bencina and Phil Burk — MIT License.
- **Steinberg Audio Stream I/O API (ASIO)**: Steinberg Media Technologies GmbH.

## Related Concepts
- [[concepts/deployment-automation]]
