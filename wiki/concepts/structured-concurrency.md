---
sources: [summaries/top_level.md]
brief: A concurrency paradigm ensuring tasks have well-defined lifetimes within scoped, nested execution contexts.
---

# Structured Concurrency

Structured concurrency is a programming paradigm that constrains the lifetime of concurrent tasks to well-defined lexical scopes, making async code easier to reason about, debug, and maintain. Rather than spawning fire-and-forget tasks that outlive their creator, structured concurrency ensures that all child tasks complete (or are cancelled) before their parent scope exits.

## Core Principles

- **Scoped task lifetimes**: Every spawned task is bound to a specific scope (e.g., a task group or nursery). The scope does not exit until all tasks within it have finished.
- **Automatic cancellation propagation**: If one task in a group fails, sibling tasks are cancelled and the error is propagated to the parent.
- **No orphaned tasks**: It is impossible for a task to outlive the scope that created it, eliminating a major source of resource leaks and race conditions.
- **Deterministic cleanup**: Because task lifetimes are scoped, resources can be reliably released at scope exit.

## Task Groups and Cancellation Scopes

The two primary building blocks of structured concurrency are:

1. **Task Groups** (also called *nurseries* in `trio`): A context manager that spawns and tracks child tasks. The group waits for all tasks before proceeding.
2. **Cancellation Scopes**: A mechanism to apply a deadline or manually cancel a group of tasks, with the cancellation propagating down through all nested scopes.

## Relationship to AnyIO

[[summaries/top_level]] describes AnyIO as a Python library that brings structured concurrency patterns — originally pioneered by the `trio` backend — to both `asyncio` and `trio` in a backend-agnostic way. AnyIO exposes:

- `anyio.create_task_group()` — a portable task group API
- Cancellation scopes compatible with both backends
- Consistent error propagation semantics regardless of the underlying runtime

This makes structured concurrency accessible to `asyncio` developers who previously had to work around its fire-and-forget `asyncio.create_task()` model.

## Comparison with Traditional Async Models

| Traditional Async | Structured Concurrency |
|---|---|
| Tasks can outlive their creator | Tasks are scoped to parent lifetime |
| Errors may be silently dropped | Errors propagate reliably |
| Manual cleanup required | Automatic cleanup at scope exit |
| Hard to reason about task topology | Topology mirrors call stack |

## Related Concepts

- [[concepts/async-backends]] — The runtimes (asyncio, trio) that implement or support structured concurrency
- [[concepts/asyncio]] — Python's built-in async runtime, which AnyIO extends with structured concurrency
- [[concepts/event-driven-architecture]] — A related paradigm for coordinating concurrent operations
- [[concepts/structured-concurrency]] — This concept page

## References

- [[summaries/top_level]] — AnyIO top-level overview introducing structured concurrency support
