---
sources: [summaries/GitHub-Automattic-harper-Offline-privacy-first-grammar-checker.-Fast-open-source.md]
brief: WebAssembly (Wasm) is a binary instruction format enabling high-performance code execution in web browsers.
---

# WebAssembly

## Overview
WebAssembly (often abbreviated as Wasm) is a low-level binary instruction format designed to run code at near-native speed inside web browsers and other environments. It allows programs written in languages like Rust, C, and C++ to be compiled into a compact binary that any modern browser can execute efficiently — without plugins or server-side processing.

## How It Works
- Code is compiled from a source language (e.g., Rust) into a `.wasm` binary.
- The browser's WebAssembly runtime executes this binary directly, bypassing the slower interpretation typical of JavaScript.
- WebAssembly modules can interoperate with JavaScript, enabling hybrid web applications.

## Why It Matters for Local/Private Applications
WebAssembly is particularly significant for [[concepts/privacy-first-software]] because it enables full-featured applications to run entirely on the client side. There is no need to send data to a remote server — all computation happens in the user's own browser. This makes it an ideal deployment target for tools that prioritize user privacy.

## Harper as a Case Study
The grammar checker Harper (see [[summaries/GitHub-Automattic-harper-Offline-privacy-first-grammar-checker.-Fast-open-source]]) uses WebAssembly to deliver its functionality directly in the browser at [writewithharper.com](https://writewithharper.com). Harper's small binary size makes this feasible — many heavier tools would be impractical to load via Wasm due to download size and memory constraints.

This stands in contrast to tools like Grammarly, which rely on round-trips to remote servers, introducing both latency and privacy risks.

## Key Properties
| Property | Detail |
|---|---|
| **Execution speed** | Near-native performance |
| **Portability** | Runs in any modern browser |
| **Language support** | Rust, C, C++, and others |
| **Privacy benefit** | Enables fully client-side processing |
| **Binary size** | Compact; suitable for web delivery |

## Related Concepts
- [[concepts/privacy-first-software]] — WebAssembly is a key enabler of private, offline-capable web tools.
