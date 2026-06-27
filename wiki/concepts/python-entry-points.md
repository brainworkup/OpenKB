---
sources: [summaries/cli-commands.md, summaries/cli-command-usage.md, summaries/bash-prompts.md, summaries/arc-clients.md, summaries/top_level.md, summaries/entry_points.md]
brief: Standard Python packaging mechanism for advertising CLI commands, plugins, and hooks across packages.
---

# Python Entry Points

Python entry points are a standard packaging mechanism that allows installed packages to advertise named components — such as plugins, commands, or hooks — to other packages or frameworks, without requiring explicit imports or manual registration.

## How Entry Points Work

Entry points are declared in a package's metadata (historically `setup.py` or `setup.cfg`, now commonly `pyproject.toml`). When a package is installed, the Python packaging system registers these declarations so other tools can discover them at runtime using the `importlib.metadata` API (or the older `pkg_resources` API).

Each entry point declaration follows the pattern:
```
[group]
name = module.path:callable
```

For example:
```ini
[pytest11]
anyio = anyio.pytest_plugin
```
- **`[pytest11]`** is the entry point **group** — a conventional namespace that pytest uses to auto-discover plugins.
- **`anyio`** is the entry point **name** — how the plugin is identified within the group.
- **`anyio.pytest_plugin`** is the **target** — the Python module or object being advertised.

## Common Entry Point Groups

| Group | Purpose |
|---|---|
| `pytest11` | pytest plugin auto-discovery |
| `console_scripts` | CLI commands installed as executables |
| `gui_scripts` | GUI application launchers |
| `distutils.setup_keywords` | Custom keywords for `setup()` calls |
| Custom groups | Application-specific plugin systems |

## console_scripts: CLI Entry Points

The `console_scripts` group is the standard way to register command-line tools that become available as shell commands after package installation.

### httpx CLI Command

A minimal, focused example is the `httpx` HTTP client package, which registers a single CLI entry point:

```ini
[console_scripts]
httpx = httpx:main
```

This maps the `httpx` shell command directly to the `main()` function in the `httpx` Python module, enabling invocations like `httpx https://example.com` after installation. See [[summaries/entry_points]] for the source.

### fontTools CLI Commands

The fontTools package provides a richer example of `console_scripts` registration:

```ini
[console_scripts]
fonttools = fontTools.__main__:main
pyftmerge = fontTools.merge:main
pyftsubset = fontTools.subset:main
ttx = fontTools.ttx:main
```

This registers four shell commands:
- **`fonttools`** — Main CLI dispatcher via `fontTools.__main__:main`.
- **`pyftmerge`** — Font merging utility via `fontTools.merge:main`.
- **`pyftsubset`** — Font subsetting tool that removes unused glyphs to reduce file size, via `fontTools.subset:main`.
- **`ttx`** — Converts fonts to/from a human-readable XML format, via `fontTools.ttx:main`.

This pattern is documented in [[concepts/cli-entry-points]], which covers the broader use of `console_scripts` for command-line tool distribution.

### Other console_scripts Examples

Another example comes from the `charset_normalizer` package:

```ini
[console_scripts]
normalizer = charset_normalizer.cli:cli_detect
```

This exposes `normalizer` as a terminal command for charset/encoding detection after `pip install charset-normalizer`. See [[concepts/charset-normalizer]] for more on that tool.

## distutils.setup_keywords: Extending the Build System

The `distutils.setup_keywords` group is used to register new keyword arguments for the `setup()` function in `setup.py`. A notable example is [[concepts/cffi]]'s integration:

```ini
[distutils.setup_keywords]
cffi_modules = cffi.setuptools_ext:cffi_modules
```

This registration:
- **Adds `cffi_modules`** as a valid keyword in any project's `setup()` call.
- **Delegates processing** to `cffi.setuptools_ext:cffi_modules`, CFFI's own build machinery.
- **Enables seamless compilation** of C extensions during package installation without manual build configuration.

This pattern demonstrates how entry points allow third-party libraries to hook into Python's build toolchain at well-defined extension points.

## Benefits

- **Decoupled registration**: Plugins and commands activate automatically on install, with no code changes needed in the host package.
- **Discoverability**: Tools can enumerate all registered entry points at runtime.
- **Ecosystem extensibility**: Frameworks like pytest, Sphinx, Flask, and setuptools use entry points to build rich plugin ecosystems.
- **Build system extensibility**: Libraries like CFFI can add custom build steps without forking setuptools.
- **CLI distribution**: Python functions can be exposed as first-class shell commands via `console_scripts`.

## Relationship to pytest Plugins

The `pytest11` group is the canonical mechanism for distributing pytest plugins. When AnyIO registers itself under `pytest11`, pytest automatically loads `anyio.pytest_plugin` at test session start, making async fixtures and markers available without any `conftest.py` changes.

This pattern relates directly to [[concepts/pytest-plugins]], which covers the broader ecosystem of pytest extensibility.

## Related Concepts

- [[concepts/pytest-plugins]] — How pytest discovers and loads plugins via entry points
- [[concepts/python-project-structure]] — Where entry point declarations live in a modern Python project
- [[concepts/cli-entry-points]] — Using `console_scripts` entry points for command-line tools
- [[concepts/python-environment-management]] — How installed package metadata is managed across environments
- [[concepts/cffi]] — C Foreign Function Interface library that uses `distutils.setup_keywords` for build integration
- [[concepts/asyncio]] — The async backend that AnyIO's pytest plugin helps test
- [[concepts/charset-normalizer]] — Encoding detection library exposing a `normalizer` CLI command

See also: [[summaries/top_level]]

See also: [[summaries/arc-clients]]

See also: [[summaries/bash-prompts]]

See also: [[summaries/cli-command-usage]]

See also: [[summaries/cli-commands]]