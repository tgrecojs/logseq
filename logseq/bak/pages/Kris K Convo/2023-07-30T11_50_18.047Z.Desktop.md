- Like, `makeCapTP` is very abstract. It knows nothing about transport or message framing. It just sends and receives plain-old-JavaScript objects as messages.
	- makeMessageCapTP adapts makeCapTP so that it’s using async iterators of POJO messages both coming and going, which is an opinion™ about how streams should look.
	- makeCapTP can stand directly on top of EventEmitter (node) or EventTarget (web) (MessagePort, Worker) streams. But…
- It’s my preference to introduce this middle layer of abstraction, using async iterators, because then I can separately adapt all kinds of platform-specific streams to the async iterator pattern, and also separately have utility methods that work on async iterators which I can stand on any of these.
- `@endo/stream` provides some abstract utilities for async iterators, like map and pump.
- Notably, async generator functions can be either sources or sinks for async iterators, so programming at that layer flushes well with the design of the language.
- Then there’s another architectural layer for framing messages. Netstring is a protocol for delimiting frames from a stream of bytes. There’s also LP32. WebSocket provides message framing, so this layer is not necessary for that transport protocol. Being pluggable makes that layer flexible.
- Then there’s the transport layer (UNIX domain socket, Windows named pipe, TCP, TLS, &c) which I’ve chosen to adapt to async iterators everywhere. `@endo/stream-node` is one such adapter. The Pet Daemon’s power adapter turns sockets and pipes into async iterators of chunked byte streams.
- So the architecture layering is: `MessageTransport(Transport > Message Framing) > Encoding (JSON/CBOR/Syrup) > CapTP(Agoric/OCapn)`
-
tags:: #[[Endo]]

-