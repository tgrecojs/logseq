- ## Resources
- https://developer.mozilla.org/en-US/docs/Web/API/Streams_API
- ## Readable Streams
- A readable stream is a data source represented in JavaScript by a [`ReadableStream`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) object that flows from an **underlying source**
- Two types of underlying sources
	- Push Sources
	  logseq.order-list-type:: number
		- constantly push data at you when you've accessed them, and it is up to you to start, pause, or cancel access to the stream.
		  logseq.order-list-type:: number
		- Video Streams, Web sockets
		  logseq.order-list-type:: number
	- Pull Sources
	  logseq.order-list-type:: number
		- require you to explicitly request data from them once connected to.
		  logseq.order-list-type:: number
		- file access via `fetch`
		  logseq.order-list-type:: number
- ### Chunks
- Streams are read in pieces of data called **chunks**.
	- A chunk can be a single byte, or it can be something larger such as a [typed array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays) of a certain size.
	- A single stream can contain chunks of different sizes and types.
- The **chunks** placed in a stream are said to be **enqueued**.
	- This means they are **placed in a queue waiting to be read.**
- An internal queue keeps tracks of chunks that have not been read yet.
- **Reader**
	- The chunks inside of the stream are read by a reader.
	- Readers processes the data one chunk at a time, allowing you to do whatever kind of operation you want to do on it.
- **Consumer**
	- The **reader** plus **the other processing code**.
- **Controller**
	- Each reader has an associated **controller** which controls the stream.
- ### Reading a Stream
	- only **one reader can control a stream at a time**.
- **Locked**
	- a stream that is being read is said to be **locked to a reader**
- **Active Reader**
	- The reader that a stream is being read by.
	-
- ## Writable Streams
- ## [Writable streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Concepts#writable_streams)
- A **writable stream** is a destination into which you can write data, represented in JavaScript by a [`WritableStream`](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream) object.
- This serves as an abstraction over the top of an **underlying sink** — a lower-level I/O sink into which raw data is written.
- **Writer**
	- Data is written to a stream **one chunk at a time** via a **writer**.
	- Chunks can take many different types of forms.
	- Each writeable stream has an **internal queue** that manages the data being written to it.
- Producer
	- **A writer plus associated processing code.**
- **Locked**
	- When a stream is being written to (via a **writer**), **it is said to be locked to that writer**.
	- Only **one writer can be locked to a stream at a time.**
- **Active Writer**
	- **The writer that a stream is currently locked to.**
- **Controller**
	- each writer has an associated controller that allows you to control the stream (for example, to abort it if wished).
	-
-
-
-