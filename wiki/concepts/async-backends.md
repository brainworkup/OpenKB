---
sources: [summaries/top_level.md]
brief: Async backends are runtime engines (asyncio, trio) that execute asynchronous Python code.
---

# Async Backends

Async backends are the underlying runtime engines that execute asynchronous Python code. They manage the event loop, task scheduling, I/O multiplexing, and cancellation semantics for async programs.

## Overview

In Python's async ecosystem, a **backend** is the engine that actually runs `async`/`await` code. Different backends make different design tradeoffs around performance, correctness guarantees, and developer ergonomics. Libraries that want broad adoption must either pick one backend or abstract over several.

## Primary Backends

### asyncio
Python's built-in async backend, included in the standard library since Python 3.4. It is the most widely used backend and the default choice for most async frameworks. See [[concepts/asyncio]] for more detail.

### trio
A third-party async library designed from the ground up around **structured concurrency** principles. Trio enforces strict task-tree discipline, making cancellation and error propagation more predictable than asyncio's looser model. It directly inspired [[concepts/structured-concurrency]] patterns now available in asyncio (via `asyncio.TaskGroup`) and in AnyIO.

## Backend Abstraction with AnyIO

[[summaries/top_level]] introduces **AnyIO**, a compatibility library that abstracts over async backends. Key ideas:

- **Write once, run anywhere**: Library authors write async code against the AnyIO API; end users choose which backend to run it on.
- **Structured concurrency by default**: AnyIO surfaces trio-style task groups and cancellation scopes regardless of which backend is active.
- **Portable primitives**: Locks, events, semaphores, and condition variables work identically across backends.
- **Streams and networking**: High-level byte-stream and TCP/UDP abstractions decouple application code from backend-specific socket APIs.
- **Testing**: `pytest-anyio` allows the same async test suite to be exercised against multiple backends.

## Why Backend Choice Matters

| Concern | asyncio | trio |
|---|---|---|  
| Standard library | ✅ | ❌ (third-party) |
| Structured concurrency | Partial (3.11+) | ✅ Core design |
| Ecosystem size | Large | Smaller |
| Cancellation safety | Opt-in | Enforced |

## Related Concepts

- [[concepts/asyncio]] — Python's built-in async backend
- [[concepts/structured-concurrency]] — Task-tree discipline pioneered by trio
- [[summaries/top_level]] — AnyIO top-level overview
