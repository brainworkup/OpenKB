---
sources: [summaries/top_level.md]
brief: Immutability is the design principle of making data structures unmodifiable after creation to ensure correctness and safety.
---

# Immutability in Software Design

Immutability is the principle that once a data structure is created (or explicitly locked), its contents cannot be changed. This constraint eliminates entire classes of bugs related to unexpected mutation and makes programs easier to reason about, test, and share across concurrent contexts.

## Core Idea

In a mutable world, any reference to an object is a potential site of change. Immutability converts objects into stable, trustworthy values: if you hold a reference, you know it will not change beneath you. This is especially valuable in:

- **Concurrent and asynchronous code** — no locks needed when data cannot change.
- **Configuration management** — settings frozen after startup cannot be accidentally overwritten.
- **Functional programming** — pure functions operating on immutable values are composable and predictable.

## Python's Immutability Landscape

Python provides several built-in immutable types:

| Type | Notes |
|------|-------|
| `tuple` | Immutable sequence, always |
| `frozenset` | Immutable set |
| `str`, `bytes` | Immutable by design |

However, none of these allow a **mutable-then-freeze** lifecycle. That gap is filled by libraries like `frozenlist` (see [[summaries/top_level]]), which provides a list that starts mutable and can be permanently locked via `.freeze()`.

## Mutable-to-Immutable Transition

Some designs need a **builder phase** followed by a **read-only phase**:

1. Populate the structure freely during initialization.
2. Call a lock/freeze method.
3. Any subsequent mutation raises an error (e.g., `RuntimeError`).

This pattern is safer than always-mutable (accidental writes) and more flexible than always-immutable (cannot build incrementally).

## Use Cases

- **Shared configuration** — frozen after application startup.
- **Async pipelines** — data passed between coroutines without defensive copying (relevant to [[concepts/asyncio]] and [[concepts/async-backends]]).
- **[[concepts/python-data-structures]]** — designing APIs that prevent misuse.
- **Agent state** — immutable snapshots of [[concepts/agent-memory]] or [[concepts/agent-pipeline-state-management]] checkpoints.
- **Structured concurrency** — [[concepts/structured-concurrency]] benefits from data that cannot be mutated by concurrent tasks.

## Immutability and Correctness

Immutable objects are inherently thread-safe and can be freely shared. They also serve as natural candidates for caching and memoization, since their identity equals their value.

## Related Concepts

- [[concepts/python-data-structures]] — the broader landscape of Python containers.
- [[concepts/asyncio]] — async programming where shared mutable state is dangerous.
- [[concepts/async-backends]] — execution environments that benefit from immutable shared data.
- [[concepts/structured-concurrency]] — concurrency model that pairs well with immutable state.
- [[concepts/agent-pipeline-state-management]] — pipeline states that may be frozen between stages.
- [[concepts/iterators]] — often paired with immutable sequences for safe traversal.

## See Also

- [[summaries/top_level]] — `frozenlist`, a practical implementation of mutable-to-immutable transition in Python.
