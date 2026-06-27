---
sources: [summaries/cli-command-management.md, summaries/top_level.md]
brief: gRPC is a high-performance open-source RPC framework by Google using HTTP/2 and Protocol Buffers.
---

# gRPC: Remote Procedure Call Framework

## Overview

**gRPC** (Google Remote Procedure Call) is a modern, open-source, high-performance RPC framework originally developed by Google. It enables efficient communication between distributed services and is widely adopted in microservice architectures.

See also: [[summaries/top_level]]

## Core Characteristics

- **Transport Protocol**: Uses HTTP/2, enabling multiplexed streams, header compression, and lower latency compared to HTTP/1.1.
- **Interface Definition Language (IDL)**: Uses Protocol Buffers (protobuf) for defining service contracts and serializing structured data efficiently.
- **Language Agnostic**: Supports code generation for many languages (Python, Go, Java, C++, etc.), making it suitable for polyglot environments.
- **Streaming Support**: Supports unary, server-side, client-side, and bidirectional streaming RPC patterns.

## Key Features

| Feature | Description |
|---|---|
| Authentication | Built-in support for SSL/TLS and token-based auth |
| Load Balancing | Native client- and proxy-side load balancing |
| Interceptors | Middleware-like hooks for logging, tracing, auth |
| Deadlines/Timeouts | Fine-grained control over call lifetimes |
| Cancellation | Propagates cancellation signals across services |

## Common Use Cases

- **Microservice Communication**: Strongly typed service-to-service calls across a distributed system.
- **Real-time Streaming**: Bidirectional data streams (e.g., audio, sensor data).
- **Internal APIs**: Low-latency internal APIs where REST overhead is undesirable.
- **AI/ML Pipelines**: Efficient communication between model inference servers and application layers.

## Relationship to Other Concepts

gRPC is related to broader patterns of networked service design. In Python contexts it connects to [[concepts/python-networking]] and [[concepts/async-backends]], as gRPC servers can be implemented with asyncio. For service discovery and API design it overlaps with [[concepts/openai-compatible-api]] patterns. Its use of persistent connections and streaming relates to [[concepts/websockets]] as an alternative real-time transport.

For deployments requiring SSL certificate management, [[concepts/ssl-tls-verification]] is relevant.

## gRPC vs. REST

| Aspect | gRPC | REST |
|---|---|---|
| Protocol | HTTP/2 | HTTP/1.1 or HTTP/2 |
| Data Format | Binary (protobuf) | Text (JSON/XML) |
| Contract | Strongly typed (.proto) | Loosely typed (OpenAPI) |
| Streaming | Native | Limited |
| Browser Support | Limited (needs grpc-web) | Universal |

## References

- [[summaries/top_level]] — Source document referencing gRPC as a top-level topic.


See also: [[summaries/cli-command-management]]