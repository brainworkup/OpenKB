---
sources: [summaries/top_level.md]
brief: Python iterators and cycling enable repeated iteration over sequences, foundational to property cycling in visualization.
---

# Python Iterators and Cycling

Iterators are a core Python programming concept that allow sequential traversal of data collections. **Cycling** extends this idea by enabling infinite or repeated iteration over a finite sequence — looping back to the start once the end is reached.

## Core Concepts

### Iterators
An iterator is any Python object implementing `__iter__()` and `__next__()`. When a sequence is exhausted, `StopIteration` is raised. Standard library tools like `itertools.cycle` wrap any iterable to repeat it indefinitely.

### Cyclers
A **cycler** is a composable cycling object that yields keyword-argument dictionaries on each iteration. Rather than cycling a single value, a cycler bundles multiple properties together — for example, cycling through `{'color': 'red'}`, `{'color': 'blue'}`, etc.

See [[summaries/top_level]] for the source reference to the `cycler` library.

## The `cycler` Library

The Python `cycler` package provides a high-level API for creating and composing property cycles. Key features include:

- **Composition via operators**: Cycles can be combined with `+` (zip/chain) and `*` (product) operators.
- **Keyword argument output**: Each step yields a dict of named properties, directly usable as `**kwargs`.
- **Finite and infinite modes**: Cycles can be consumed once or looped indefinitely.

### Example Use
```python
from cycler import cycler

color_cycle = cycler(color=['red', 'green', 'blue'])
style_cycle = cycler(linestyle=['-', '--'])

combined = color_cycle * style_cycle
for props in combined:
    print(props)  # e.g., {'color': 'red', 'linestyle': '-'}
```

## Relationship to Visualization

The `cycler` library was created primarily to support [[concepts/matplotlib]], where it drives the automatic cycling of visual properties (colors, markers, line styles) across plotted elements. This eliminates manual repetition and enables consistent, configurable plot aesthetics.

More broadly, cycling iterators are a pattern used throughout data visualization tooling to manage style variation programmatically.

## Related Concepts

- [[concepts/matplotlib]] — Primary consumer of the cycler library for plot property cycling
- [[concepts/iterators]] — Underlying Python iterator protocol that cycling builds upon
- [[summaries/top_level]] — Source document referencing the `cycler` library
