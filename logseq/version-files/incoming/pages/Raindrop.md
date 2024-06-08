# Articles
	- [9. Applicative · Tom Harding](http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/)
	  title:: 9. Applicative · Tom Harding
	  url:: http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/
	  Tags:: functional programming, fantasyland, applicatives, functors
	  date-saved:: [[06/14/2023]]
	  last-updated:: [[06/06/2024]]
		- ## Highlights
			- > the Apply post
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Fantas, Eel, and Specification · Tom Harding](https://web.archive.org/web/20231219083024/http://www.tomharding.me/fantasy-land/)
	  title:: Fantas, Eel, and Specification · Tom Harding
	  url:: https://web.archive.org/web/20231219083024/http://www.tomharding.me/fantasy-land/
	  Tags:: fantasyland
	  date-saved:: [[06/06/2024]]
	  last-updated:: [[06/06/2024]]
		- ## Highlights
			- > Bifunctor and Profunctor
	- [14: ChainRec · Tom Harding](https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/05/30/fantas-eel-and-specification-14/)
	  title:: 14: ChainRec · Tom Harding
	  url:: https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/05/30/fantas-eel-and-specification-14/
	  Tags:: fantasyland
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/06/2024]]
		- ## Highlights
			- > Happy
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Using Buffers to share data between Node.js and C++ - RisingStack Engineering](https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/)
	  title:: Using Buffers to share data between Node.js and C++ - RisingStack Engineering
	  url:: https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
	  Tags:: 
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/05/2024]]
		- ## Highlights
			- > One of the best things about developing with Node.jsNode.js is an asynchronous event-driven JavaScript runtime and is the most effective when building scalable network applications. Node.js is free of locks, so there's no chance to dead-lock any process. is the ability to move fairly seamlessly between JavaScript and native C++ code – thanks to the V8’s add-on API.
			- > Node.js allows us to move fairly seamlessly between JavaScript and native C++ code.
	- [Implementing Merkle Tree and Patricia Trie](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
	  title:: Implementing Merkle Tree and Patricia Trie
	  url:: https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591
	  Tags:: merkle trees, airdrops
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkel Trees are essentially a tree data structure in which data is stored in the leaf nodes and non leaf nodes store hashes of data with each non-leaf node being the combined hash value of the two nodes below it.
	- [An invitation to category theory - Chalkdust](http://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
	  title:: An invitation to category theory - Chalkdust
	  url:: http://chalkdustmagazine.com/features/an-invitation-to-category-theory/
	  Tags:: category theory, math
	  date-saved:: [[08/16/2023]]
	  last-updated:: [[08/16/2023]]
		- ## Highlights
			- > Martin Kupp
	- [Merkle Tree Construction and Proof-of-Inclusion](https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/)
	  title:: Merkle Tree Construction and Proof-of-Inclusion
	  url:: https://www.derpturkey.com/merkle-tree-construction-and-proof-of-inclusion/
	  Tags:: merkle trees
	  date-saved:: [[12/20/2023]]
	  last-updated:: [[12/20/2023]]
		- ## Highlights
			- > Merkle rootNumber of nodes in the treeList of hashesList of bits
			- > It is a binary tree, where a node can only have zero, one, or two children.
	- [strawman:concurrency [ES Wiki]](https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency)
	  title:: strawman:concurrency [ES Wiki]
	  url:: https://web.archive.org/web/20161026162206/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
	  Tags:: ECMA
	  date-saved:: [[07/26/2023]]
	  last-updated:: [[02/11/2024]]
		- ## Highlights
			- > Promises vs. Monads
			- > JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage.
			- > JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets.
			- > A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.
			- > Transport independence: Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports
			- > An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.
			- > Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops
			  
			  This is the goal(or one of the goals) of this entire thread. To enable rich composition to take place across many computers without introducing attack vectors.
			- > Aggregate objects into process-like units called vats
			- > Objects in one vat can only send asynchronous messages to objects in other vats.
			- > Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object.
			- > A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a turn. A then expression registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s resolution.
			- > The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			- > This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking
			- > The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like alert). Without blocking, conventional deadlock is impossible
			- > The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole
	- [Fantas, Eel, and Specification · Tom Harding](https://web.archive.org/web/20231219083024/http://www.tomharding.me/fantasy-land/)
	  title:: Fantas, Eel, and Specification · Tom Harding
	  url:: https://web.archive.org/web/20231219083024/http://www.tomharding.me/fantasy-land/
	  Tags:: fantasyland
	  date-saved:: [[06/06/2024]]
	  last-updated:: [[06/06/2024]]
		- ## Highlights
			- > Bifunctor and Profunctor
	- [14: ChainRec · Tom Harding](https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/05/30/fantas-eel-and-specification-14/)
	  title:: 14: ChainRec · Tom Harding
	  url:: https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/05/30/fantas-eel-and-specification-14/
	  Tags:: fantasyland
	  date-saved:: [[06/05/2024]]
	  last-updated:: [[06/06/2024]]
		- ## Highlights
			- > Happy