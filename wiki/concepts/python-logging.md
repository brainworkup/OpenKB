---
sources: [summaries/top_level.md]
brief: Python's built-in logging framework for recording runtime events at configurable severity levels.
---

# Python Logging

Python's standard `logging` module provides a flexible framework for emitting log messages from applications and libraries. It is part of the Python standard library and requires no additional installation.

## Core Concepts

### Log Levels

Messages are categorized by severity, from lowest to highest:

| Level | Numeric Value | Typical Use |
|----------|---------------|-------------------------------|
| DEBUG | 10 | Detailed diagnostic info |
| INFO | 20 | General operational messages |
| WARNING | 30 | Unexpected but non-fatal |
| ERROR | 40 | Serious problems |
| CRITICAL | 50 | Fatal errors |

### Key Components

- **Logger**: The primary interface used by application code (`logging.getLogger(name)`).
- **Handler**: Sends log records to a destination (console, file, network, etc.).
- **Formatter**: Controls the layout/format of log output strings.
- **Filter**: Provides fine-grained control over which records are passed to handlers.

### Basic Usage

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

logger.debug('Detailed info')
logger.info('Application started')
logger.warning('Disk space low')
logger.error('File not found')
logger.critical('System failure')
```

### Custom Formatters

Formatters accept format strings using `LogRecord` attributes:

```python
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
```

## Extensions and Enhancements

The standard `logging` module is designed to be extensible. Third-party packages build on top of it to add features:

- **colorlog**: Adds ANSI color codes to log output, making different severity levels visually distinct in terminal environments. See [[summaries/top_level]] for details on this package.
- **structlog**: Provides structured (JSON-style) logging.
- **loguru**: A modern alternative with simpler configuration.

## Best Practices

- Use `logging.getLogger(__name__)` in library code to create hierarchical loggers.
- Never use `print()` for diagnostic output in production code — use logging instead.
- Set log levels per handler to allow different verbosity for console vs. file output.
- Avoid `logging.basicConfig()` in library code; leave configuration to the application.

## Terminal Output and Color

By default, Python logging emits plain text. Tools like `colorlog` enhance readability by mapping log levels to ANSI terminal colors, leveraging [[concepts/terminal-output-formatting]] capabilities of modern terminals.

## Related Concepts

- [[concepts/python-utilities]] — Ecosystem of small Python libraries that enhance the developer experience, including logging tools.
- [[concepts/terminal-output-formatting]] — ANSI escape codes and colored terminal output used by packages like `colorlog`.
- [[concepts/cli-entry-points]] — Command-line tools that rely on logging for operational feedback.
- [[concepts/python-project-structure]] — How logging configuration fits into overall project layout.
