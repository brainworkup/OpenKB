---
sources: [summaries/top_level.md]
brief: Core data structures in Python, including mutable, immutable, and lifecycle-controlled containers.
---

# Python Data Structures

Python provides a rich set of built-in and library-level data structures that cover a wide range of use cases, from simple ordered sequences to complex mappings and sets. A recurring design axis in Python data structures is the **mutability spectrum** — whether a structure can be changed after creation.

## Built-in Structures

- **`list`**: Ordered, mutable sequence. Supports append, insert, delete, and in-place modification.
- **`tuple`**: Ordered, immutable sequence. Fixed at creation; supports indexing and iteration.
- **`dict`**: Mutable key-value mapping.
- **`set` / `frozenset`**: Unordered collections. `frozenset` is the immutable counterpart to `set`.

## Lifecycle-Controlled Structures

Beyond the built-ins, some libraries extend Python's data model with **lifecycle-aware** structures — containers that begin mutable and can be transitioned to immutable at runtime.

### `FrozenList` (frozenlist library)

`FrozenList` exemplifies this pattern: it behaves like a standard `list` until `.freeze()` is called, after which all mutation attempts raise a `RuntimeError`. This enables:

- **Safe sharing** of configuration or state data.
- **Runtime enforcement** of [[concepts/immutability]] without requiring a different type from the start.
- **Integration** with async ecosystems (e.g., `aiohttp`, `yarl`) where shared state must be locked after initialization.

See [[summaries/top_level]] for the full source context.

## Mutability vs. Immutability

| Structure | Mutable | Hashable | Notes |
|---|---|---|---|
| `list` | Yes | No | General-purpose sequence |
| `tuple` | No | Yes (if elements are) | Lightweight immutable sequence |
| `FrozenList` | Initially yes | After freeze | Lifecycle-controlled |
| `frozenset` | No | Yes | Immutable set |

See [[concepts/immutability]] for broader discussion of immutability patterns.

## Iterators and Sequences

All standard Python data structures implement the iterator protocol, enabling use in `for` loops, comprehensions, and generator pipelines. See [[concepts/iterators]] for more.

## Async Contexts

In [[concepts/asyncio]]-based applications, immutable-after-init data structures like `FrozenList` are especially valuable because they prevent accidental mutation across coroutines running concurrently. The `frozenlist` library was designed with this use case in mind.

## Related Concepts

- [[concepts/immutability]]
- [[concepts/iterators]]
- [[concepts/asyncio]]
- [[concepts/structured-concurrency]]
