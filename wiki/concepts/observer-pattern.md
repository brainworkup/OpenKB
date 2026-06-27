---
sources: [summaries/top_level.md]
brief: A design pattern where objects (observers) register to be notified when another object's state changes.
---

# Observer Pattern

The **Observer Pattern** is a fundamental software design pattern in which an object (the *subject* or *publisher*) maintains a list of dependents (*observers* or *subscribers*) and automatically notifies them of state changes or events. It is a cornerstone of event-driven and reactive programming architectures.

## Core Idea

- A **subject** holds a collection of registered observer callbacks.
- Observers **subscribe** by registering themselves with the subject.
- When a relevant event occurs, the subject **notifies** all observers by invoking their callbacks.
- Observers can typically be added or removed dynamically.

## Key Roles

| Role | Responsibility |
|------|----------------|
| Subject / Publisher | Maintains observer list; fires notifications |
| Observer / Subscriber | Registers interest; implements a callback or handler |
| Event / Signal | The datum or trigger passed from subject to observers |

## Relationship to `aiosignal`

The `aiosignal` library (see [[summaries/top_level]]) is a direct implementation of the Observer Pattern tailored for asynchronous Python. Key parallels:

- The `Signal` object acts as the **subject**, holding a `MutableSequence` of callbacks.
- Registered coroutines or functions are the **observers**.
- Calling `await signal.send()` is the **notification** step, invoking all observers in order.
- Signals can be **frozen** after setup — a common pattern to prevent observer list mutation during dispatch, ensuring consistency.

## Variants and Related Concepts

- **Synchronous Observer**: Callbacks are plain functions called directly.
- **Asynchronous Observer**: Callbacks are coroutines awaited in sequence or concurrently — as in [[summaries/top_level]] (`aiosignal`).
- **Event Bus / Pub-Sub**: A decoupled variant where subjects and observers don't reference each other directly.
- **Reactive Streams**: A more advanced form using backpressure and stream operators.

## Use Cases in Python Ecosystems

- HTTP lifecycle hooks in async web frameworks (e.g., startup/shutdown events).
- Plugin architectures where third-party code reacts to internal application events.
- UI frameworks where widget state changes propagate to bound views.
- Agent and pipeline state change notifications (see [[concepts/agent-pipeline-state-management]]).

## Connection to Event-Driven Architecture

The Observer Pattern is a foundational building block of [[concepts/event-driven-architecture]]. While simple observer implementations couple subject and observer directly, more scalable systems evolve toward full event buses or message brokers that decouple producers from consumers entirely.

## Asyncio Context

In modern Python, async observers registered via libraries like `aiosignal` leverage [[concepts/asyncio]] to ensure non-blocking notification. This allows many observers to be notified efficiently within a single event loop without blocking I/O operations.

## See Also

- [[summaries/top_level]] — `aiosignal` library implementing async observer pattern
- [[concepts/event-driven-architecture]] — broader architectural style built on observer principles
- [[concepts/asyncio]] — Python async runtime underpinning async observer implementations
- [[concepts/agent-pipeline-state-management]] — agent systems that benefit from observer-style event hooks
