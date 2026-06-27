---
sources: [summaries/top_level.md]
brief: Mypyc compiles typed Python source into C extensions for faster runtime execution.
---

# Mypyc: Python-to-C Extension Compiler

Mypyc is a compiler that translates statically typed Python source code — annotated with [mypy](https://mypy.readthedocs.io/)-compatible type hints — into native C extensions (`.so` / `.pyd` files). The resulting compiled modules execute significantly faster than interpreted Python, while remaining importable as normal Python packages.

## How It Works

1. **Type annotation analysis**: Mypyc leverages mypy's type-checking infrastructure to understand the types of all values in the code.
2. **C extension generation**: The typed Python AST is compiled into C source, then compiled with the platform's C compiler into a native shared library.
3. **Transparent import**: The compiled `.so` or `.pyd` artifact is placed alongside the pure Python source. Python's import system picks up the compiled version automatically when available.

## Naming Convention

Mypyc-compiled modules follow a specific naming pattern:

```
<hash>__mypyc.<ext>
```

For example, `81d243bd2c585b0f4821__mypyc` is the compiled extension seen in the [[summaries/top_level]] document. The hash prefix ensures the compiled artifact matches the exact source version it was built from, preventing stale cache issues.

## Performance Characteristics

- Typical speedups range from **2× to 10×** over CPython for compute-intensive, well-typed code.
- Code that relies heavily on dynamic Python features (e.g., `getattr`, metaclasses, `**kwargs`-heavy dispatch) benefits less.
- Pure I/O-bound code sees minimal improvement.

## Use in charset-normalizer

The [[concepts/charset-normalizer]] library ships a mypyc-compiled extension (`81d243bd2c585b0f4821__mypyc`) alongside its pure Python implementation. This lets the library provide near-native performance for character encoding detection without requiring users to install a separate C extension manually — the compiled module is bundled in the wheel.

## Relationship to Python Tooling

- **mypy**: Mypyc is part of the mypy project. The same type annotations used for static analysis power compilation.
- **cffi / C extensions**: Mypyc produces standard C extensions, comparable in mechanism to [[concepts/cffi]]-based bindings but generated automatically from Python source.
- **Python project structure**: The compiled artifact integrates cleanly into [[concepts/python-project-structure]] alongside the source package.

## Limitations

- Not all Python constructs are supported (some dynamic features must be avoided or worked around).
- Compiled extensions are platform-specific; packages must ship separate wheels per OS/architecture.
- Debugging compiled code is more complex than debugging pure Python.

## Related Concepts

- [[concepts/charset-normalizer]] — library that ships a mypyc-compiled binary
- [[concepts/cffi]] — alternative approach to Python/C interop
- [[concepts/python-project-structure]] — how compiled extensions fit into package layout
- [[summaries/top_level]] — source document showing mypyc module in the wild
