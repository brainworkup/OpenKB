---
sources: [summaries/LICENSE.md, summaries/top_level.md]
brief: Python web frameworks provide tools for building HTTP servers, APIs, and web applications in Python.
---

# Python Web Frameworks

Python web frameworks are libraries and toolkits that simplify the development of web servers, REST APIs, and web applications. They handle common concerns such as routing, request/response handling, middleware, session management, and often WebSocket support.

## Categories

### Synchronous Frameworks
Traditional Python web frameworks operate synchronously, blocking on I/O operations. Examples include Django and Flask. These are well-suited for CPU-bound or moderately concurrent applications.

### Asynchronous Frameworks
Modern async frameworks leverage [[concepts/asyncio]] to handle many concurrent connections without blocking. This makes them ideal for high-throughput APIs, real-time applications, and microservices.

Key examples:
- **aiohttp** — A full-featured async HTTP client/server framework. See [[summaries/top_level]] for details.
- **FastAPI** — High-performance async framework with automatic OpenAPI documentation.
- **Starlette** — Lightweight async framework often used as a foundation for others.
- **Sanic** — Async framework focused on speed.

## Key Features Across Frameworks

- **Routing**: Mapping URL paths to handler functions.
- **Middleware**: Pluggable processing layers for requests and responses.
- **Session & Cookie Management**: Maintaining client state across requests.
- **WebSocket Support**: Real-time bidirectional communication (see [[concepts/websockets]]).
- **Streaming**: Handling large request/response bodies incrementally.
- **Connection Pooling**: Reusing network connections for efficiency (e.g., `ClientSession` in aiohttp).

## Async HTTP in Python

The shift toward async frameworks is driven by [[concepts/asyncio]], Python's built-in event loop framework. Async frameworks avoid the overhead of thread-per-request models, enabling thousands of concurrent connections on modest hardware.

aiohttp is notable for providing **both** an HTTP client and an HTTP server within a single library, making it versatile for:
- Building web APIs and services
- Making concurrent outbound HTTP requests
- Proxying and web scraping at scale
- Real-time applications via WebSockets

## Ecosystem Context

Python web frameworks often integrate with:
- **[[concepts/python-project-structure]]** tooling for packaging and deployment
- **[[concepts/python-environment-management]]** for dependency isolation
- **[[concepts/openai-compatible-api]]** backends for AI-powered web services
- **[[concepts/local-llm-inference]]** servers that expose HTTP APIs

## Related Concepts

- [[concepts/asyncio]] — Underlying async runtime for modern Python web frameworks
- [[concepts/websockets]] — Real-time communication protocol supported by async frameworks
- [[concepts/openai-compatible-api]] — Common API pattern served by Python web frameworks
- [[concepts/python-project-structure]] — Project organization for web framework applications


See also: [[summaries/LICENSE]]