---
sources: [summaries/top_level.md]
brief: A pattern for extending function/class behavior without modifying source code, central to Python design.
---

# Decorator Pattern and Python Decorators

The **decorator** is both a structural design pattern in software engineering and a first-class language feature in Python. It enables extending or modifying the behavior of functions, methods, or classes without altering their source code directly.

See also: [[summaries/top_level]]

## The Decorator Design Pattern

In object-oriented programming, the **Decorator Pattern** is a structural pattern that:

- Wraps an object with another object that adds new behavior
- Follows the Open/Closed Principle: open for extension, closed for modification
- Allows behavior to be added to individual objects dynamically at runtime
- Composes behavior through delegation rather than inheritance

### Common Use Cases
- Logging and instrumentation
- Caching and memoization
- Access control and authentication
- Input validation
- Retry logic

## Python Decorators

In Python, decorators are a syntactic feature using the `@` symbol placed above a function or class definition. They are syntactic sugar for wrapping a callable with another callable.

```python
@my_decorator
def my_function():
    pass

# Equivalent to:
my_function = my_decorator(my_function)
```

### Types of Python Decorators

1. **Function decorators** — wrap a function to add pre/post behavior
2. **Class decorators** — wrap or transform an entire class
3. **Method decorators** — applied to methods within a class (e.g., `@staticmethod`, `@classmethod`, `@property`)
4. **Decorator factories** — decorators that accept arguments and return a decorator

### Standard Library Examples
- `@functools.wraps` — preserves metadata of the wrapped function
- `@functools.lru_cache` — memoization via least-recently-used cache
- `@dataclasses.dataclass` — auto-generates class methods
- `@staticmethod`, `@classmethod`, `@property` — built-in method decorators

## Relationship to Other Patterns

Decorators relate closely to:
- The **[[concepts/observer-pattern]]** — both involve wrapping or intercepting behavior
- **[[concepts/iterators]]** — often decorated for lazy evaluation or transformation pipelines
- **[[concepts/python-project-structure]]** — decorators appear throughout well-structured Python projects for routing, validation, and CLI entry points
- **[[concepts/cli-entry-points]]** — CLI frameworks like Click use decorators extensively for defining commands
- **[[concepts/python-entry-points]]** — decorator-based registration patterns are common in plugin systems
- **[[concepts/pytest-plugins]]** — pytest uses decorators such as `@pytest.fixture` and `@pytest.mark` heavily

## Decorators in Frameworks

Many Python frameworks rely on decorators as their primary API surface:

- **Web frameworks** (see [[concepts/python-web-frameworks]]): route registration via `@app.route()`
- **Agent pipelines** (see [[concepts/agent-pipeline-state-management]]): step registration and middleware
- **LLM tooling**: tool registration decorators in agent frameworks

## Summary

The decorator concept, whether as a design pattern or Python language feature, is a foundational tool for composing behavior cleanly, enabling extensible and maintainable software architecture.