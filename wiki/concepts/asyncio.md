---
sources: [summaries/entry_points.md, summaries/top_level.md]
brief: Python's standard async I/O framework powering coroutines, event loops, and the broader aio-libs ecosystem.
---

# Python asyncio: Asynchronous I/O Framework

## Overview

`asyncio` is Python's standard library framework for writing **concurrent, non-blocking code** using the `async`/`await` syntax. It provides an event loop that manages the execution of coroutines, enabling high-throughput I/O-bound applications without the complexity of threads or multiprocessing.

Built on top of asyncio, **AnyIO** extends this foundation into a backend-agnostic compatibility layer — allowing library authors and application developers to write async code that runs on asyncio, trio, or other supported backends without modification. See [[concepts/async-backends]] for a broader discussion of this abstraction.

## Core Concepts

### Event Loop
The central execution mechanism in asyncio. It schedules and runs coroutines, handles I/O events, and manages callbacks. Only one event loop runs at a time per thread.

### Coroutines
Functions defined with `async def` that can suspend execution at `await` points, yielding control back to the event loop while waiting for I/O or other async operations.

### Tasks and Futures
- **Task**: A wrapped coroutine scheduled to run on the event loop.
- **Future**: A low-level object representing a result that may not yet be available.

### Non-blocking I/O
asyncio enables a single thread to handle thousands of concurrent connections by suspending coroutines during I/O waits rather than blocking the entire thread.

## AnyIO: Backend-Agnostic Async

**AnyIO** is a compatibility layer built on top of asyncio (and trio) that provides:

- **Unified async primitives**: Portable locks, events, semaphores, and condition variables that work across backends.
- **Structured concurrency**: Task groups and cancellation scopes inspired by trio, available in asyncio-based projects. See [[concepts/structured-concurrency]].
- **Streams and networking**: High-level abstractions for byte streams, object streams, TCP, and UDP.
- **Async file I/O**: Backend-compatible async file operations.
- **Testing support**: A pytest plugin that enables backend-agnostic async test execution.

AnyIO is designed as a compatibility shim rather than a replacement — it surfaces trio's best practices (particularly structured concurrency) within the broader asyncio ecosystem.

## Signals and Event Hooks

A common pattern built on top of asyncio is the **signal** (or callback registry) — a list of coroutines or callables that are invoked when a particular event fires. The `aiosignal` library formalizes this pattern:

- A `Signal` object holds a list of async callbacks registered by external code.
- Signals are fired with `await signal.send()`, invoking all registered callbacks in order.
- Signals can be **frozen** after setup to prevent further modification, ensuring safe dispatch.
- This pattern is closely related to the [[concepts/observer-pattern]] and [[concepts/event-driven-architecture]].
- aiohttp uses `aiosignal` internally for request/response lifecycle hooks.

This signal mechanism enables decoupled, plugin-friendly architectures in async Python applications.

## The aio-libs Ecosystem and frozenlist

The `aio-libs` organization maintains a suite of focused libraries that underpin the asyncio ecosystem. One such library is **`frozenlist`**, which provides a list-like data structure that can be transitioned from mutable to immutable by calling `.freeze()`. Once frozen, any mutation attempt raises a `RuntimeError`.

`frozenlist` is a direct dependency of `aiohttp` and `yarl`, and is closely tied to the signal mechanism described above: the `aiosignal` `Signal` type is built on `FrozenList`, which is frozen before dispatch to prevent race conditions. Key characteristics:

- **Mutable-to-immutable lifecycle**: Start populating the list, then lock it — a pattern distinct from Python's built-in `tuple` (always immutable) or `list` (always mutable).
- **[[concepts/immutability]]**: Runtime enforcement of immutability through the freeze mechanism.
- **[[concepts/python-data-structures]]**: Implements the full Python sequence protocol (indexing, iteration, slicing) with added lifecycle semantics.
- **Concurrency safety**: Freezing before event emission ensures no callbacks are added mid-dispatch.

See [[summaries/top_level]] for more detail on the `frozenlist` package.

## Plugin Registration via Entry Points

Libraries built on asyncio often integrate with the broader Python ecosystem through the **`entry_points`** packaging mechanism. AnyIO registers a pytest plugin via the standard `[pytest11]` entry point group:

```ini
[pytest11]
anyio = anyio.pytest_plugin
```

This means:
- When AnyIO is installed, pytest automatically discovers and loads `anyio.pytest_plugin`.
- Developers get async test support (fixtures, markers, backend selection) without manual plugin registration.
- This uses [[concepts/python-entry-points]] — Python's mechanism for packages to advertise components to other packages at install time.
- The pytest plugin integrates with [[concepts/pytest-plugins]] to support running async tests on any AnyIO-compatible backend.

This pattern is common across the asyncio ecosystem: async libraries advertise their testing utilities, CLI tools, or framework hooks through entry points, enabling seamless integration.

## Why asyncio Matters for HTTP and Networking

Libraries like **aiohttp** are built directly on asyncio to provide:
- Asynchronous HTTP client and server capabilities
- WebSocket support via [[concepts/websockets]]
- Connection pooling through persistent `ClientSession` objects
- Streaming of large request/response bodies without blocking
- Lifecycle event hooks via the `aiosignal` pattern (backed by `frozenlist`)

Without asyncio, such high-concurrency networking would require complex thread management or external frameworks.

## Common Use Cases

- **Web frameworks and HTTP clients**: Python web frameworks, aiohttp
- **Real-time communication**: [[concepts/websockets]], chat servers, live dashboards
- **Concurrent API calls**: Fetching from multiple endpoints simultaneously
- **Audio/streaming pipelines**: Non-blocking data streams (see [[concepts/audio-transcription-pipeline]])
- **Agent orchestration**: Async agent loops in [[concepts/multi-agent-orchestration]] and [[concepts/langgraph-agent-workflows]]
- **Event hook systems**: Plugin and lifecycle callback registries via async signals backed by frozen lists
- **Async testing**: AnyIO's pytest plugin enables backend-agnostic async test execution
- **Backend-agnostic libraries**: AnyIO enables code portable across asyncio and trio

## Key asyncio Primitives

| Primitive | Purpose |
|---|---|
| `asyncio.run()` | Entry point to run the top-level coroutine |
| `asyncio.gather()` | Run multiple coroutines concurrently |
| `asyncio.create_task()` | Schedule a coroutine as a Task |
| `asyncio.sleep()` | Non-blocking sleep/yield |
| `asyncio.Queue` | Thread-safe async queue for producer/consumer patterns |

## Relationship to Other Frameworks

asyncio is the foundation beneath many Python async ecosystems:
- **aiohttp** — HTTP client/server with signal-based lifecycle hooks
- **aiosignal** — Async callback/signal registry built on asyncio, backed by `frozenlist`
- **frozenlist** — Mutable-to-immutable list used throughout the aio-libs ecosystem
- **AnyIO** — Async compatibility layer over asyncio and trio, with pytest integration via [[concepts/python-entry-points]] and structured concurrency via [[concepts/structured-concurrency]]
- **FastAPI / Starlette** — async web frameworks
- **LangGraph** — agent workflow orchestration ([[concepts/langgraph-agent-workflows]])
- **MLX and local inference servers** — async API layers ([[concepts/mlx-framework]], [[concepts/omlx-server]])

## Considerations

- asyncio is best suited for **I/O-bound** workloads, not CPU-bound tasks (use multiprocessing for those).
- Mixing synchronous blocking calls inside async code can stall the event loop; use `asyncio.to_thread()` or `run_in_executor()` for blocking operations.
- Python 3.7+ is required for `asyncio.run()`; Python 3.10+ improved error messages significantly.
- Signal objects (from `aiosignal`) should be frozen via `frozenlist` before dispatching to prevent race conditions during event emission.
- When building async libraries, registering pytest plugins via the `[pytest11]` entry point group ensures developers can test async code without extra configuration.
- AnyIO is a compatibility layer, not a replacement for backends — it surfaces trio patterns within the asyncio ecosystem.
- `frozenlist` enforces [[concepts/immutability]] at runtime, making it safe for shared state in async contexts.

## Related Documents
- [[summaries/entry_points]]
- [[summaries/top_level]]