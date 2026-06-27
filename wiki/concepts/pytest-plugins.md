---
sources: [summaries/entry_points.md]
brief: Pytest plugins extend test functionality and are auto-discovered via Python entry points under pytest11.
---

# Pytest Plugins

Pytest plugins are Python modules that extend the pytest testing framework with additional fixtures, markers, hooks, and configuration. They can be registered automatically using Python's entry points mechanism, making them active as soon as the package is installed.

## Auto-Discovery via Entry Points

Pytest uses the `pytest11` entry point group to discover and load plugins automatically. Any package that declares an entry point under `[pytest11]` will have its plugin module loaded by pytest at startup — no manual registration required.

Example from [[summaries/entry_points]]:

```ini
[pytest11]
anyio = anyio.pytest_plugin
```

This registers the `anyio.pytest_plugin` module as a pytest plugin under the name `anyio`.

## How It Works

1. **Package Installation**: When a package (e.g., AnyIO) is installed, its `entry_points` metadata is written to the environment.
2. **Pytest Startup**: Pytest queries all installed packages for entries under the `pytest11` group.
3. **Plugin Loading**: Each discovered module is imported and its hooks, fixtures, and markers are activated.
4. **Test Execution**: Tests can now use the features provided by the plugin (e.g., async test support from AnyIO).

## Relation to Python Entry Points

The `pytest11` mechanism is a specific application of the broader [[concepts/python-entry-points]] system, which allows Python packages to advertise components to other packages or tools without tight coupling.

## AnyIO Pytest Plugin

The AnyIO library uses this mechanism to provide:
- Async test support across multiple backends (asyncio, trio)
- Fixtures for running async code in tests
- Markers for selecting the async backend per test

This integration is declared in [[summaries/entry_points]] as the canonical configuration for AnyIO's pytest integration.

## Related Concepts

- [[concepts/python-entry-points]] — The underlying packaging mechanism used for plugin registration
- [[concepts/asyncio]] — One of the async backends AnyIO supports in tests
- [[concepts/python-project-structure]] — Where entry point declarations live in a project
- [[concepts/pytest-plugins]] — Broader ecosystem of pytest extension patterns
