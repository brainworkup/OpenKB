---
sources: [summaries/top_level.md, summaries/entry_points.md]
brief: Console script entry points map shell commands to Python functions, registered during package installation.
---

# CLI Entry Points and Console Scripts

CLI entry points are declarations in Python packaging configuration that register shell commands when a package is installed. They map a command name (e.g., `accelerate-launch`) to a specific Python function (e.g., `accelerate.commands.launch:main`), making that function directly callable from the terminal.

## How Console Scripts Work

In Python packaging (via `setup.py`, `setup.cfg`, or `pyproject.toml`), entry points are organized into named sections. The two standard sections are:

- **`[console_scripts]`**: Registers command-line executables that run in a terminal.
- **`[gui_scripts]`**: Registers graphical application executables that do not attach to a terminal.

Each entry takes the form:

```
command-name = package.module:function
```

When the package is installed (e.g., via `pip install`), the packaging tool creates a platform-appropriate executable in the environment's `bin/` or `Scripts/` directory. This executable calls the mapped Python function when invoked from the shell. An empty `[console_scripts]` or `[gui_scripts]` section is valid and serves as a placeholder or template for future script registration.

## Examples in This Knowledge Base

### The httpx CLI

The `httpx` HTTP client library registers a single console script:

| Shell Command | Python Entry Point |
|---|---|
| `httpx` | `httpx:main` |

After installation, users can run `httpx <url>` directly from the terminal. The command maps to the `main()` function at the top level of the `httpx` package, exposing the library's HTTP client capabilities as a command-line tool.

### The charset_normalizer CLI

The `charset_normalizer` package registers a single console script for character encoding detection:

| Shell Command | Python Entry Point |
|---|---|
| `normalizer` | `charset_normalizer.cli:cli_detect` |

After installation, users can run `normalizer <file_or_input>` from the terminal to detect character encoding. The command maps to the `cli_detect` function within the `cli` module, providing a convenient shell interface to the library's charset detection logic.

### The Alembic CLI

The `alembic` database migration tool registers a single console script:

| Shell Command | Python Entry Point |
|---|---|
| `alembic` | `alembic.config:main` |

After installation, the `alembic` command is available system-wide, invoking the `main` function from `alembic.config` — the standard entry point for running database migrations and related management tasks. See [[concepts/alembic]] for more on the migration tool itself.

### The Accelerate CLI Commands

The [[concepts/accelerate-library]] registers five console scripts:

| Shell Command | Python Entry Point |
|---|---|
| `accelerate` | `accelerate.commands.accelerate_cli:main` |
| `accelerate-config` | `accelerate.commands.config:main` |
| `accelerate-estimate-memory` | `accelerate.commands.estimate:main` |
| `accelerate-launch` | `accelerate.commands.launch:main` |
| `accelerate-merge-weights` | `accelerate.commands.merge:main` |

- **`accelerate`**: The main dispatcher command, likely delegating to subcommands.
- **`accelerate-config`**: Walks users through configuring their distributed training environment (hardware, precision, backend).
- **`accelerate-estimate-memory`**: Estimates GPU/CPU memory requirements for a given model, aiding resource planning before training runs.
- **`accelerate-launch`**: Launches training or inference scripts under the configured distributed environment.
- **`accelerate-merge-weights`**: Merges model weights, supporting workflows like LoRA adapter merging or checkpoint consolidation.

### The fontTools CLI Commands

The `fontTools` library registers four console scripts for font manipulation and conversion:

| Shell Command | Python Entry Point |
|---|---|
| `fonttools` | `fontTools.__main__:main` |
| `pyftmerge` | `fontTools.merge:main` |
| `pyftsubset` | `fontTools.subset:main` |
| `ttx` | `fontTools.ttx:main` |

- **`fonttools`**: The main CLI dispatcher for the fontTools suite, entry via `fontTools.__main__:main`.
- **`pyftmerge`**: A font merging utility for combining multiple font files.
- **`pyftsubset`**: A font subsetting tool that reduces font file size by removing unused glyphs — important for web performance optimization.
- **`ttx`**: Converts fonts to and from the TTX format, an XML representation of binary font data, enabling human-readable font inspection and editing. See [[concepts/fonttools]] and [[concepts/fonttools-table-converters]] for more detail on this toolchain.

## Design Principles

- **Separation of concerns**: Each command maps to its own module, keeping logic isolated and maintainable.
- **Discoverability**: Named commands are easier to remember and document than module invocation strings.
- **Minimal surface area**: A package may expose just one command (like `alembic`, `normalizer`, or `httpx`), several (like `accelerate` or `fonttools`), or none at all — an empty entry points file is a valid starting point.
- **Composability**: The pattern fits well with [[concepts/python-project-structure]] and [[concepts/uv-workspace-layout]], where multiple packages in a workspace can each register their own commands.
- **Tooling integration**: IDEs, shell completions, and CI scripts can reference stable command names rather than internal module paths.
- **GUI separation**: The `[gui_scripts]` section separates windowed applications from terminal tools, enabling platform-specific launcher behavior.

## Related Concepts

- [[concepts/python-project-structure]] — Packaging layout that hosts entry point declarations
- [[concepts/python-packaging]] — The broader packaging ecosystem in which entry points are defined
- [[concepts/deployment-automation]] — Automation pipelines that invoke CLI commands
- [[concepts/python-environment-management]] — Environment setup that activates registered console scripts
- [[concepts/accelerate-library]] — The library that registers the accelerate-specific entry points
- [[concepts/local-llm-inference]] — A downstream use case enabled by `accelerate-launch`
- [[concepts/model-quantization]] — Related to memory estimation and weight merging workflows
- [[concepts/alembic]] — Database migration tool that exposes a single `alembic` console script
- [[concepts/fonttools]] — Font manipulation library exposing `fonttools`, `pyftmerge`, `pyftsubset`, and `ttx` commands
- [[concepts/fonttools-table-converters]] — Related fontTools internals accessed via the `ttx` CLI

## Related Documents
- [[summaries/entry_points]]

See also: [[summaries/top_level]]