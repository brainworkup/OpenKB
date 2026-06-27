---
sources: [summaries/top_level.md]
brief: WebSockets enable persistent, full-duplex communication channels over a single TCP connection for real-time apps.
---

# WebSockets: Real-Time Bidirectional Communication

WebSockets are a communication protocol that provides full-duplex, persistent communication channels over a single TCP connection. Unlike traditional HTTP request-response cycles, WebSockets allow both the client and server to send messages to each other at any time, making them ideal for real-time applications.

## How WebSockets Work

1. **Handshake**: A WebSocket connection begins with an HTTP upgrade request. The client sends an `Upgrade: websocket` header, and the server responds with `101 Switching Protocols`.
2. **Persistent Connection**: Once established, the connection remains open, eliminating the overhead of repeated HTTP handshakes.
3. **Bidirectional Messaging**: Either party (client or server) can push data at any time without waiting for a request.
4. **Frames**: Data is transmitted in lightweight frames, reducing overhead compared to HTTP headers.

## Key Characteristics

- **Low Latency**: No need to re-establish connections per message.
- **Full-Duplex**: Simultaneous two-way communication.
- **Efficient**: Minimal framing overhead compared to polling or long-polling HTTP strategies.
- **Event-Driven**: Both client and server react to incoming messages asynchronously.

## Use Cases

- **Real-Time Chat Applications**: Messages delivered instantly without page refresh.
- **Live Dashboards**: Streaming data updates (e.g., financial tickers, monitoring systems).
- **Collaborative Tools**: Multiple users editing documents simultaneously.
- **Gaming**: Low-latency state synchronization between players.
- **Notifications**: Push-based server-to-client alerts.

## WebSockets in aiohttp

`aiohttp` (see [[summaries/top_level]]) provides native WebSocket support on both the client and server sides, integrated with Python's [[concepts/asyncio]] event loop. This makes it straightforward to build high-concurrency real-time services without blocking I/O.

### Server-Side Example Pattern
```python
async def websocket_handler(request):
    ws = web.WebSocketResponse()
    await ws.prepare(request)
    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            await ws.send_str(f"Echo: {msg.data}")
    return ws
```

### Client-Side Example Pattern
```python
async with aiohttp.ClientSession() as session:
    async with session.ws_connect('ws://example.com/ws') as ws:
        await ws.send_str('Hello')
        async for msg in ws:
            print(msg.data)
```

## Relationship to Async Frameworks

WebSockets pair naturally with asynchronous frameworks because managing many concurrent open connections is a classic I/O-bound problem. [[concepts/asyncio]] allows a single-threaded event loop to handle thousands of simultaneous WebSocket connections efficiently.

## Related Concepts

- [[concepts/asyncio]] — the async foundation enabling non-blocking WebSocket handling
- [[concepts/python-web-frameworks]] — ecosystem context for WebSocket-capable frameworks
- [[summaries/top_level]] — aiohttp, the async HTTP/WebSocket library
