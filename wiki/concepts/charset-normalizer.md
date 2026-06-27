---
sources: [summaries/top_level.md]
brief: A Python library for detecting and normalizing character encodings, with optional mypyc-compiled extensions.
---

# charset-normalizer

`charset-normalizer` is a Python library designed to detect and normalize character encodings in text data. It serves as a modern, pure-Python alternative to the older `chardet` library, offering reliable encoding detection with improved accuracy and performance.

## Overview

Character encoding detection is a fundamental challenge in text processing: given a sequence of bytes, determine which character encoding (UTF-8, Latin-1, Shift-JIS, etc.) was used to produce them. `charset-normalizer` approaches this problem using statistical analysis and Unicode normalization techniques.

## Key Features

- **Encoding detection**: Identifies the most likely character encoding for a byte stream.
- **Normalization**: Converts detected text to a canonical Unicode representation.
- **Pure Python fallback**: Ships as a standard Python package (`charset_normalizer`).
- **mypyc-compiled extension**: Optionally includes a [[concepts/mypyc]]-compiled binary extension (e.g., `81d243bd2c585b0f4821__mypyc`) for significantly improved runtime performance.
- **Drop-in compatibility**: Designed to be compatible with `chardet`'s API, making migration straightforward.

## Distribution Structure

As seen in [[summaries/top_level]], the package ships two top-level modules:

1. `charset_normalizer` — the main pure-Python package.
2. `81d243bd2c585b0f4821__mypyc` — a mypyc-compiled C extension for performance-critical paths.

This dual-distribution pattern allows the library to take advantage of native code speed when available, while remaining fully functional in pure-Python environments.

## Relation to mypyc

The inclusion of a [[concepts/mypyc]] compiled module reflects a broader trend of using Python's type annotation system not just for static analysis, but as a compilation target. mypyc translates typed Python source into C extensions, which can yield substantial speedups for CPU-bound operations like byte-stream analysis.

## Use Cases

- HTTP response decoding in networking libraries (e.g., `requests`, `httpx`).
- Processing text files with unknown encodings.
- Data pipeline ingestion where source encoding is inconsistent.
- Any application requiring robust [[concepts/python-networking]] or text normalization.

## Related Concepts

- [[concepts/mypyc]] — The compiler used to generate the native extension bundled with this library.
- [[concepts/python-project-structure]] — How the dual-module layout fits into Python packaging conventions.
- [[concepts/python-networking]] — A common domain where charset detection is essential.
