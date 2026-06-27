---
sources: [summaries/top_level.md, summaries/entry_points.md]
brief: CFFI is a Python library for calling C functions from Python via compiled C extension modules.
---

# CFFI: C Foreign Function Interface for Python

## Overview

CFFI (C Foreign Function Interface) is a Python library that allows Python code to call C functions and use C data types directly. It provides a mechanism for building compiled C extension modules that integrate seamlessly with the Python runtime, making it a foundational tool for performance-critical Python packages that need to interface with native C libraries.

## Package Structure

The CFFI distribution exposes two top-level modules:

- **`_cffi_backend`** — The compiled C extension backend that powers CFFI's low-level operations. This is the native code layer responsible for performance-critical interfacing between Python and C.
- **`cffi`** — The main Python package providing the high-level API that package authors and users interact with directly.

This two-layer design is common in Python packages that use native code: a pure-Python API layer (`cffi`) sits atop a compiled C extension (`_cffi_backend`) that does the heavy lifting.

## How CFFI Works

CFFI operates by:
1. Defining C function signatures and data structures in Python
2. Compiling a C extension module (`.so` / `.pyd`) at install or build time
3. Exposing the compiled interface to Python code at runtime

This approach gives Python access to native C performance and existing C library ecosystems without requiring manual wrapping in hand-written C extension code.

## Setuptools Integration

CFFI integrates with Python's build toolchain through a distutils/setuptools plugin mechanism. The entry point registration (see [[summaries/entry_points]]) makes this possible:

```ini
[distutils.setup_keywords]
cffi_modules = cffi.setuptools_ext:cffi_modules
```

This entry point registers the `cffi_modules` keyword for use in `setup()` calls. Package authors can then declare their C extension modules like:

```python
setup(
    name="mypackage",
    cffi_modules=["src/mymodule_build.py:ffi"],
)
```

When a user installs the package, setuptools delegates build processing to `cffi.setuptools_ext:cffi_modules`, which compiles the C extension automatically.

## Packaging Metadata

The `top_level.txt` metadata file in CFFI's distribution lists both `_cffi_backend` and `cffi` as top-level modules. This file is used by Python packaging tools (e.g., setuptools, pip) to track which top-level namespaces belong to the installed distribution — useful for uninstallation, conflict detection, and introspection.

## Role in the Python Ecosystem

- **Alternative to ctypes**: CFFI is generally preferred over ctypes for complex C interfaces due to better performance and safety.
- **Alternative to Cython**: Unlike Cython, CFFI does not require a separate language; C definitions are written as strings in Python.
- **Used by major packages**: Libraries such as `cryptography`, `bcrypt`, and `PyNaCl` use CFFI for their C bindings.

## Related Concepts

- [[concepts/python-entry-points]] — The plugin registration mechanism CFFI uses to hook into setuptools
- [[concepts/python-project-structure]] — How CFFI modules fit into Python package layouts
- [[concepts/python-environment-management]] — Managing environments where CFFI extensions are compiled and installed
- [[concepts/cli-entry-points]] — Broader entry point patterns in the Python packaging ecosystem

## Key Takeaway

CFFI's two-module structure (`cffi` + `_cffi_backend`) reflects a best-practice pattern for Python packages with native C extensions: a clean Python API layer backed by a compiled extension for performance. Its setuptools integration via the `distutils.setup_keywords` entry point allows package authors to declare C extension modules declaratively in `setup.py`, enabling automatic compilation during package installation.

## Related Documents
- [[summaries/top_level]]
