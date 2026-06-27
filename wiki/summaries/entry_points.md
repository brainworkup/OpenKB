---
doc_type: short
full_text: sources/entry_points.md
---

# Entry Points: httpx

## Overview

This document defines the console script entry point for the `httpx` package. It maps the `httpx` command-line tool to the `main` function within the `httpx` module.

## Entry Point Definition

```
[console_scripts]
httpx = httpx:main
```

## Key Details

- **Script name**: `httpx` — the command available in the terminal after installation
- **Target**: `httpx:main` — the `main()` function inside the `httpx` Python package
- **Type**: `console_scripts` — a standard [[concepts/python-entry-points]] group used by setuptools/pip to install executable scripts

## Significance

This entry point enables users to invoke `httpx` directly from the command line (e.g., `httpx https://example.com`) after installing the package. It is a standard pattern in [[concepts/python-packaging]] for exposing CLI interfaces. The [[concepts/cli-entry-points]] mechanism is how Python packages register executable commands that become available system-wide after installation.

## Related Concepts
- [[concepts/python-project-structure]]

- [[concepts/python-entry-points]] — the entry point system this document configures
- [[concepts/python-packaging]] — entry points are a core feature of Python package distribution
- [[concepts/cli-entry-points]] — command-line interface tooling pattern