## All posts
	- [Merkle Trees](https://omnivore.app/me/merkle-trees-18cfc251d0e)
	  collapsed:: true
	  site:: [asselinpaul.com](https://asselinpaul.com/merkle.html)
	  labels:: [[logseq]] [[cryptography]] [[merkle-airdrop]]
	  date-saved:: [[01/12/2024]]
		- ### Content
		  collapsed:: true
			- An efficient and verifiable data structure
			  
			  Merkle trees have become increasingly prevalent due to their key role in many decentralised systems. Although we are rarely exposed to them directly, they are worth studying for their elegance alone.
			  
			  \#\# Hash Functions
			  
			  A hash function is a mathematical function which maps anything (represented as a finite string of characters) into an output of fixed length. 
			  
			  This is tremendously useful. Any data has a signature that is significantly smaller than itself.
			  
			  ![hashing a string and an image example](https://proxy-prod.omnivore-image-cache.app/0x0,spSgTngAejCld5PFwb8k28su6seXV0GAKsPoAaDSeMFo/https://asselinpaul.com/img/merkle/hashing.png) 
			  
			  This signature can be used to verify the integrity of data downloaded from potentially dubious sources. Some sites will show the hash for files so that users can check whether the file has been tempered with.
			  
			  ![hashing an original versus a tempered version](https://proxy-prod.omnivore-image-cache.app/0x0,szpHB_wt6In68aZfnkVYnykhljgPg9fB0xQ_3XiUzLVc/https://asselinpaul.com/img/merkle/hashing2.png) 
			  
			  This highlights a key characteristic of hash functions. It must be _nearly_ impossible to modify a file to obtain a different file with the same hash (a hash collision).
			  
			  \#\#\# Properties of hash functions:
			  
			  * their input can be of any size
			  * their output is of fixed-size _e.g:_ 256-bit
			  * they are efficiently computable for an input of size _n_, its hash can be computed in _O(n)_ time
			  
			  \#\# Cryptographic Hash Functions
			  
			  Since Merkle trees are primarily used in settings where cryptography matters, our hash functions will have to abide by the following (stronger) properties:
			  
			  \#\#\# Collision Resistance
			  
			  A collision occurs when two distinct inputs produce the same output:
			  
			  ![example of a hash collision](https://proxy-prod.omnivore-image-cache.app/0x0,sYxeENI99j-1DR8jlwhjj4dZIyJPp-vPYLOVZPG-Vh7U/https://asselinpaul.com/img/merkle/collision.png) 
			  
			  A hash function is collision resistant if nobody can find a collision.
			  
			  It is worth noting that _"nobody can find"_ does not equate to _"does not exist"_. In fact, collisions do exist because the input space is larger than the output space. the input space is infinite whereas the output space is finite 
			  
			  This can be shown using the [pigeon-hole principle](https://en.wikipedia.org/wiki/Pigeonhole%5Fprinciple) , which states that if _n+1_ pigeons must inhabit _n_ pigeon-holes, at least one pigeon-hole must have more than one pigeon inhabitant.
			  
			  ![example of a hash collision](https://proxy-prod.omnivore-image-cache.app/0x0,scLtc4ma9jY61l1VpMmrzSirvrV-BfCZs1KYckKqCJnU/https://asselinpaul.com/img/merkle/io_space.png) 
			  
			  Using the pigeon-hole principle, we can formulate a process which is guaranteed to find a collision:
			  
			  Consider a hash function with a 256-bit output size. Pick 2256+1 distinct values and calculate their respective hashes. Since we picked more values than possible outputs, we are guaranteed that some pair collides when you apply the hash function.
			  
			  This method guarantees that we will cause a collision. We can reduce the number of values to examine while still being fairly likely to observe a collision by making use of the [birthday paradox](https://en.wikipedia.org/wiki/Birthday%5Fproblem) in a cryptographic setting (known as a [birthday attack](https://en.wikipedia.org/wiki/Birthday%5Fattack)). By examining the hashes of 2130+1 randomly chosen inputs, there is a 99.8 percent chance of finding a collision.
			  
			  This method of uncovering hash collisions works for every hash function but takes an impractical amount of time. It would likely take far longer than the age of the universe to find a collision using all the computing power available on earth. _Bitcoin and Cryptocurrency Technologies._ Princeton University Press, p.4
			  
			  For some hash functions, it it easy to produce collisions. Consider the following hash function:
			  
			  H(_x_) = _x_ mod 2256
			  
			  This function returns the last 256 bits of the input (a fixed-size output). It is trivial to find two values that have the same 256 bit ending. **5** and **5 + 2256** would cause a collision for example.
			  
			  We don't know whether such methods exist for other hash methods (_e.g_ [SHA512](https://en.wikipedia.org/wiki/SHA-2)). The general consensus is that these functions are suspected to be collision resistant but no formal proof has ever been made. The hash functions we rely on today have been tested thoroughly by mathematicians and no collision has been found. Occasionally, collisions are found after years of work and particular functions are phased out of cryptographic systems. such was the fate of [MD5](https://en.wikipedia.org/wiki/MD5)
			  
			  \#\#\# Hiding
			  
			  The second property our cryptographic hashes must abide by: given a hash output, it should be impossible to deduce its input.
			  
			  With **y = H(_x_)**, **_x_** can't be deduced from **y**.
			  
			  When the set of potential inputs is fairly restricted, this can be hard. Imagine a problem where you output _true_ or _false_. You then hash the result of the problem. An attacker can deduce whether the output was _true_ or _false_ by computing the hashes of those inputs on his own.
			  
			  In a sense, we want the hash input, **_x_** to be taken from a set that is very spread out. It turns out we can hide even an input that is not spread out by concatenating it with another input that is spread out.
			  
			  A hash function _H_ is said to be hiding if when a secret value _r_ is chosen from a probability distribution that has [_high min-entropy_](https://www.reddit.com/r/crypto/comments/4qwxdz/high%5Fminentropy/), then, given _H(r || x)_, it is infeasible to find x. where || denotes concatenation
			  
			  \#\#\# Puzzle Friendliness
			  
			  Lastly, we require hash functions to be puzzle friendly. This property is the least intuitive of the three.
			  
			  A hash function _H_ is said to be puzzle friendly if for ever every possible _n_\-bit output value, if _k_ is chosen from a distribution with high min-entropy, then it is infeasible to find _x_ such that _H(k || x)_ \= _y_ in time significantly less than 2n.
			  
			  In more concrete terms, if someone wants to have the hash function output **Z** and part of the input(**k**) has been chosen in a suitably random way, then it's very difficult to find the value **x** that makes the hash function output **Z**.
			  
			  \#\#\# An example: [SHA-256](https://en.wikipedia.org/wiki/SHA-2)
			  
			  Having examined the properties of cryptographic hash functions, let's take a look at a commonly used one: **[SHA-256](https://asselinpaul.com/SHA-256)**
			  
			  SHA-256 outputs 256-bit hashes and works by using the _Merkle-Damgård transform_ to convert the underlying fixed-length collision-resistant hash function the compression function into one that accepts arbitrary-length inputs.
			  
			  ![sha_256 diagram](https://proxy-prod.omnivore-image-cache.app/0x0,s9GPlXpfLn7hMwQYgluMR12gDL4y7hzLq-oOc9joSLpg/https://asselinpaul.com/img/merkle/sha_256.png) 
			  
			  The input is padded so that its length is a multiple of 512 bits. It can then be divided into the _n_ message block and processed as shown in the figure above.
			  
			  \#\#\# Recapitulative
			  
			  The hash serves as a fixed-length summary, or unambiguous signature, of a message. It gives us an efficient way to remember things we've seen before and to recognise them again. Whereas an entire file could have been hundreds-of-gigabytes in size, its hash is of fixed length (256 bits in our examples). This greatly reduces storage requirements and enables a great number of practical uses.
			  
			  It is worth noting that different applications of hash functions require varying properties. Although puzzle-friendliness might be required in cryptocurrency systems, it isn't for other applications.
			  
			  \#\# Hash Pointers
			  
			  ![hash pointer diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sFI2GpinpepXqC5RXy9Sfh8vaoPAIB7z2xjwSwCtYYQI/https://asselinpaul.com/img/merkle/hash_pointer.png) 
			  
			  A Hash Pointer is a data-structure that points to data along with a cryptographic hash of the data. In addition to enabling information retrieval, hash pointers can be used to verify the integrity of the data.
			  
			  \#\# Block Chains
			  
			  ![blockchain diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sfhTT81R7SiTcymb6FhD98Wri_vqmxKvWN06-v-EgbJc/https://asselinpaul.com/img/merkle/blockchain.png) 
			  
			  A block chain is a linked list that is built with hash pointers. It is a tamper-evident log because changes to the chain will cause detectable inconsistencies between a data block and the hash pointer to the data block (which cascades right through to the head pointer).
			  
			  ![blockchain cascading error diagram](https://proxy-prod.omnivore-image-cache.app/0x0,s5QiXMI3m1l0mdf4k6ZORhua0eXA47IZSR8iS1kby6DE/https://asselinpaul.com/img/merkle/blockchain2.png) 
			  
			  ![merkle tree diagram](https://proxy-prod.omnivore-image-cache.app/0x0,s1suFq6IGDmoMuCJNhzIa5aVXbSFYKjyyhtk-AtsIvLY/https://asselinpaul.com/img/merkle/merkle.png) 
			  
			  Ralph Merkle had the idea to combine binary trees and hash pointers to create the Merkle tree in 1979\. The leaves contain hash pointers (arranged in pairs) and the hash of each of these blocks is stored in the parent node.
			  
			  We only need to store the root of the tree to verify the integrity of the tree. As in the block chain example, a change at the leaf-level would induce a change to hashes that bubbles up all the way to the root node, effectively making the data-structure tamper-proof.
			  
			  The merkle tree improves over the block chain by enabling concise _proof of membership_. That is, it is efficient to figure out whether a data block is a member of the Merkle Tree.![](https://proxy-prod.omnivore-image-cache.app/0x0,sZbP1Yib1mky0uR-0Qc-nPy5_H6Xtuym1bsUKm0P1ViQ/https://asselinpaul.com/img/merkle/merkle_efficiency.png) Merkle tree efficiency in Bitcoin - [Source](http://chimera.labs.oreilly.com/books/1234000001802/ch07.html\#merkle%5Ftrees) 
			  
			  Proving that a data block is included in the tree only requires showing the blocks in the path from that data block to the root.
			  
			  ![merkle tree membership diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sNSKnHUMCt-Ml8lxPAkXUA6hy3lX8lUt10P_M-7dlmOk/https://asselinpaul.com/img/merkle/membership.png) 
			  
			  If there are _n_ nodes in the tree, only about _log(n)_ items need to be shown. Even with large Merkle trees, we can prove membership in a relatively short time. The space and time requirements are of the order _log(n)_. whereas they are of the order _n_ for block chains 
			  
			  \#\# Uses of Merkle Trees
			  
			  \#\#\# Git
			  
			  The version control system uses Merkle Trees to see which files changed in the tracked directory.
			  
			  ![git merkle](https://proxy-prod.omnivore-image-cache.app/0x0,sOn6gOiiwV75CypwawtBWMIUNF4JyZwyoArSFDE-fxv4/https://asselinpaul.com/img/merkle/git.png) [Source](https://www.slideshare.net/japh44/code-is-not-text-how-graph-technologies-can-help-us-to-understand-our-code-better) 
			  
			  \#\#\# Bitcoin
			  
			  Merkle Trees are used to summarise all the transactions in a particular block, thus providing an efficient way to verify whether a transaction is included in a block.
			  
			  \#\#\# Databases
			  
			  Apache Cassandra, Dynamo and other NoSQL databases use Merkle trees to detect inconsistencies between replicas of databases. [Source](https://wiki.apache.org/cassandra/AntiEntropy) 
			  
			  Merkle trees are very commonly used to achieve consensus in distributed systems (_e.g:_ [IPFS](https://ipfs.io/) uses a Merkle-DAG , [BitTorrent](https://en.wikipedia.org/wiki/BitTorrent), [dat](https://datproject.org/)) or to counter data degradation (_e.g:_ [ZFS](https://en.wikipedia.org/wiki/ZFS)).
			  
			  \#\# Further Reading
			  
			  * Traditional Merkle trees cannot be applied to graphs because graphs are more complicated than trees. This paper from Ashish Kundu IBM T J Watson Research Center and Elisa Bertino Department of Computer Science and CERIAS, Purdue University expands on the subject: [On Hashing Graphs](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.294.8694)
			  * [Merkle Tree Traversal in Log Space and Time](https://link.springer.com/chapter/10.1007/978-3-540-24676-3%5F32) by Michael Szydlo RSA Laboratories presents a technique to efficiently traverse Merkle trees.
			  * [The Swirlds Hashgraph Consensus Algorithm: Fair, Fast, Byzantine Fault Tolerance](http://www.swirlds.com/whitepapers/) by Leemon Baird presents a different hash graph data-structure.
			  
			  \#\#\# Thank you for your time.
			  
			  \#\#\# Sources
			  
			  \#\#\# Colophon
	- [History of Actors (eighty-twenty news)](https://omnivore.app/me/https-eighty-twenty-org-2016-10-18-actors-hopl-ref-blog-danfinla-18d02554ccc)
	  collapsed:: true
	  site:: [eighty-twenty.org](https://eighty-twenty.org/2016/10/18/actors-hopl?ref=blog.danfinlay.com)
	  author:: Tony Garnock-Jones
	  labels:: [[actors]] [[distributed systems]]
	  date-saved:: [[01/13/2024]]
	  date-published:: [[10/17/2016]]
		- ### Content
		  collapsed:: true
			- Yesterday, I presented the history of actors to Christos Dimoulas’[History of Programming Languages](http://www.seas.harvard.edu/courses/cs252/2016fa/)class.
			  
			  Here are the written-out talk notes I prepared beforehand. There is an annotated bibliography at the end.
			  
			  ---
			  
			  \#\# Introduction
			  
			  Today I’m going to talk about the actor model. I’ll first put the model in context, and then show three different styles of actor language, including two that aim to be realistic programming systems.
			  
			  I’m going to draw on a few papers:
			  
			  * 2016 survey by Joeri De Koster, Tom Van Cutsem, and Wolfgang De Meuter, for taxonomy and common terminology (SPLASH AGERE 2016)
			  * 1990 overview by Gul Agha, who has contributed hugely to the study of actors (Comm. ACM 1990)
			  * 1997 operational semantics for actors by Gul Agha, Ian Mason, Scott Smith, and Carolyn Talcott.
			  * brief mention of some of the content of the original 1973 paper by Carl Hewitt, Peter Bishop, and Richard Steiger. (IJCAI 1973)
			  * Joe Armstrong’s 2003 dissertation on Erlang.
			  * 2005 paper on the language E by Mark Miller, Dean Tribble, and Jonathan Shapiro. (TGC 2005)
			  
			  I’ve put an annotated bibliography at the end of this file.
			  
			  \#\# Actors in context: Approaches to concurrent programming
			  
			  One way of classifying approaches is along a spectrum of private vs. shared state.
			  
			  * Shared memory, threads, and locking: very limited private state, almost all shared
			  * Tuple spaces and my own research, Syndicate: some shared, some private and isolated
			  * Actors: almost all private and isolated, just enough shared to do routing
			  
			  Pure functional “data parallelism” doesn’t fit on this chart - it lacks shared mutable state entirely.
			  
			  \#\# The actor model
			  
			  The actor model, then is
			  
			  * asynchronous message passing between entities (actors)  
			   * with guaranteed delivery  
			   * addressing of messages by actor identity
			  * a thing called the “ISOLATED TURN PRINCIPLE”  
			   * no shared mutable state; strong encapsulation; no global mutable state  
			   * no interleaving; process one message at a time; serializes state access  
			   * liveness; no blocking
			  
			  In the model, each actor performs “turns” one after the other: a turn is taking a waiting message, interpreting it, and deciding on both a state update for the actor and a collection of actions to take in response, perhaps sending messages or creating new actors.
			  
			  A turn is a sequential, atomic block of computation that happens in response to an incoming message.
			  
			  What the actor model buys you:
			  
			  * modular reasoning about state in the overall system, and modular reasoning about local state change within an actor, because state is private, and access is serialized; only have to consider interleavings of messages
			  * no “deadlocks” or data races (though you can get “datalock” and other global non-progress in some circumstances, from logical inconsistency and otherwise)
			  * flexible, powerful capability-based security
			  * failure isolation and fault tolerance
			  
			  It’s worth remarking that actor-like concepts have sprung up several times independently. Hewitt and many others invented and developed actors in the 1970s, but there are two occasions where actors seem to have been independently reinvented, as far as I know.
			  
			  One is work on a capability-based operating system, KeyKOS, in the 1980s which involved a design very much like Hewitt’s actors, feeding into research which led ultimately to the language E.
			  
			  The other is work on highly fault-tolerant designs for telephone switches, also in the 1980s, which culminated in the language Erlang.
			  
			  Both languages are clearly actor languages, and in both cases, apparently the people involved were unaware of Hewitt’s actor model at the time.
			  
			  \#\# Terminology
			  
			  There are two kinds of things in the actor model: _messages_, which are data sent across some medium of communication, and _actors_, which are stateful entities that can only affect each other by sending messages back and forth.
			  
			  Messages are completely immutable data, passed by copy, which may contain references to other actors.
			  
			  Each actor has
			  
			  * private state; analogous to instance variables
			  * an interface; which messages it can respond to
			  
			  Together, the private state and the interface make up the actor’s_behaviour_, a key term in the actor literature.
			  
			  In addition, each actor has
			  
			  * a mailbox; inbox, message queue
			  * an address; denotes the mailbox
			  
			  Within this framework, there has been quite a bit of variety in how the model appears as a concrete programming language.
			  
			  De Koster et al. classify actor languages into
			  
			  * The Classic Actor Model (create, send, become)
			  * Active Objects (OO with a thread per object; copying of passive data between objects)
			  * Processes (raw Erlang; receive, spawn, send)
			  * Communicating Event-Loops (no passive data; E; near/far refs; eventual refs; batching)
			  
			  We see instances of the actor model all around us. The internet’s IP network is one example: hosts are the actors, and datagrams the messages. The web can be seen as another: a URL denotes an actor, and an HTTP request a message. Seen in a certain light, even Unix processes are actor-like (when they’re single threaded, if they only use fds and not shm). It can be used as a structuring principle for a system architecture even in languages like C or C++ that have no hope of enforcing any of the invariants of the model.
			  
			  \#\# Rest of the talk
			  
			  For the rest of the talk, I’m going to cover the classic actor model using Agha’s presentations as a guide; then I’ll compare it to E, a communicating event-loop actor language, and to Erlang, a process actor language.
			  
			  \#\# Classic Actor Model
			  
			  The original 1973 actor paper by Hewitt, Bishop and Steiger in the International Joint Conference on Artificial Intelligence, is incredibly far out!
			  
			  It’s a position paper that lays out a broad and colourful research vision. It’s packed with amazing ideas.
			  
			  The heart of it is that Actors are proposed as a universal programming language formalism ideally suited to building artificial intelligence.
			  
			  The goal really was A.I., and actors and programming languages were a means to that end.
			  
			  It makes these claims that actors bring great benefits in a huge range of areas:
			  
			  * foundations of semantics
			  * logic
			  * knowledge-based programming
			  * intentions (software contracts)
			  * study of expressiveness of programming languages
			  * teaching of computation
			  * extensible, modular programming
			  * privacy and protection
			  * synchronization constructs
			  * resource management
			  * structured programming
			  * computer architecture
			  
			  (It’s amazingly “full-stack”: computer architecture!?)
			  
			  In each of these areas, you can see what they were going for. In some, actors have definitely been useful; in others, the results have been much more modest.
			  
			  In the mid-to-late 70s, Hewitt and his students Irene Greif, Henry Baker, and Will Clinger developed a lot of the basic theory of the actor model, inspired originally by SIMULA and Smalltalk-71\. Irene Greif developed the first operational semantics for it as her dissertation work and Will Clinger developed a denotational semantics for actors.
			  
			  In the late 70s through the 80s and beyond, Gul Agha made huge contributions to the actor theory. His dissertation was published as a book on actors in 1986 and has been very influential. He separated the actor model from its A.I. roots and started treating it as a more general programming model. In particular, in his 1990 Comm. ACM paper, he describes it as a foundation for CONCURRENT OBJECT-ORIENTED PROGRAMMING.
			  
			  Agha’s formulation is based around the three core operations of the classic actor model:
			  
			  * `create`: constructs a new actor from a template and some parameters
			  * `send`: delivers a message, asynchronously, to a named actor
			  * `become`: within an actor, replaces its behaviour (its state & interface)
			  
			  The classic actor model is a UNIFORM ACTOR MODEL, that is, everything is an actor. Compare to uniform object models, where everything is an object. By the mid-90s, that very strict uniformity had fallen out of favour and people often worked with _two-layer_ languages, where you might have a functional core language, or an object-oriented core language, or an imperative core language with the actor model part being added in to the base language.
			  
			  I’m going to give a simplified, somewhat informal semantics based on his 1997 work with Mason, Smith and Talcott. I’m going to drop a lot of details that aren’t relevant here so this really will be simplified.
			  
			  ```coq
			  e  :=  λx.e  |  e e  |  x  | (e,e) | l | ... atoms, if, bool, primitive operations ...
			    |  create e
			    |  send e e
			    |  become e
			  
			  l  labels, PIDs; we'll use them like symbols here
			  
			  ```
			  
			  and we imagine convenience syntax
			  
			  ```ocaml
			  e1;e2              to stand for   (λdummy.e2) e1
			  let x = e1 in e2   to stand for   (λx.e2) e1
			  
			  match e with p -> e, p -> e, ...
			    to stand for matching implemented with if, predicates, etc.
			  
			  ```
			  
			  We forbid programs from containing literal process IDs.
			  
			  ```coq
			  v  :=  λx.e  |  (v,v)  |  l
			  
			  R  :=  []  |  R e  |  v R  |  (R,e)  |  (v,R)
			    |  create R
			    |  send R e  |  send v R
			    |  become R
			  
			  ```
			  
			  Configurations are a pair of a set of actors and a multiset of messages:
			  
			  ```makefile
			  m  :=  l <-- v
			  
			  A  :=  (v)l  |  [e]l
			  
			  C  :=  < A ... | m ... >
			  
			  ```
			  
			  The normal lambda-calculus-like reductions apply, like beta:
			  
			  ```jboss-cli
			    < ... [R[(λx.e) v]]l ... | ... >
			  --> < ... [R[e{v/x}  ]]l ... | ... >
			  
			  ```
			  
			  Plus some new interesting ones that are actor specific:
			  
			  ```jboss-cli
			    < ... (v)l ...    | ... l <-- v' ... >
			  --> < ... [v v']l ... | ...          ... >
			  
			    < ... [R[create v]]l       ... | ... >
			  --> < ... [R[l'      ]]l (v)l' ... | ... >   where l' fresh
			  
			    < ... [R[send l' v]]l ... | ... >
			  --> < ... [R[l'       ]]l ... | ... l' <-- v >
			  
			    < ... [R[become v]]l       ... | ... >
			  --> < ... [R[nil     ]]l' (v)l ... | ... >   where l' fresh
			  
			  ```
			  
			  Whole programs `e` are started with
			  
			  where `a` is an arbitrary label.
			  
			  Here’s an example - a mutable cell.
			  
			  ```xl
			  Cell = λcontents . λmessage . match message with
			                                (get, k) --> become (Cell contents);
			                                             send k contents
			                                (put, v) --> become (Cell v)
			  
			  ```
			  
			  Notice that when it gets a `get` message, it first performs a `become`in order to quickly return to `ready` state to handle more messages. The remainder of the code then runs alongside the ready actor. Actions after a `become` can’t directly affect the state of the actor anymore, so even though we have what looks like multiple concurrent executions of the actor, there’s no sharing, and so access to the state is still serialized, as needed for the isolated turn principle.
			  
			  ```livecodeserver
			  < [let c = create (Cell 0) in
			   send c (get, create (λv . send c (put, v + 1)))]a | >
			  
			  < [let c = l1 in
			   send c (get, create (λv . send c (put, v + 1)))]a
			  (Cell 0)l1 | >
			  
			  < [send l1 (get, create (λv . send l1 (put, v + 1)))]a
			  (Cell 0)l1 | >
			  
			  < [send l1 (get, l2)]a
			  (Cell 0)l1
			  (λv . send l1 (put, v + 1))l2 | >
			  
			  < [l1]a
			  (Cell 0)l1
			  (λv . send l1 (put, v + 1))l2 | l1 <-- (get, l2) >
			  
			  < [l1]a
			  [Cell 0 (get, l2)]l1
			  (λv . send l1 (put, v + 1))l2 | >
			  
			  < [l1]a
			  [become (Cell 0); send l2 0]l1
			  (λv . send l1 (put, v + 1))l2 | >
			  
			  < [l1]a
			  [send l2 0]l3
			  (Cell 0)l1
			  (λv . send l1 (put, v + 1))l2 | >
			  
			  < [l1]a
			  [l2]l3
			  (Cell 0)l1
			  (λv . send l1 (put, v + 1))l2 | l2 <-- 0 >
			  
			  < [l1]a
			  [l2]l3
			  (Cell 0)l1
			  [send l1 (put, 0 + 1)]l2 | >
			  
			  < [l1]a
			  [l2]l3
			  (Cell 0)l1
			  [l1]l2 | l1 <-- (put, 1) >
			  
			  < [l1]a
			  [l2]l3
			  [Cell 0 (put, 1)]l1
			  [l1]l2 | >
			  
			  < [l1]a
			  [l2]l3
			  [become (Cell 1)]l1
			  [l1]l2 | >
			  
			  < [l1]a
			  [l2]l3
			  [nil]l4
			  (Cell 1)l1
			  [l1]l2 | >
			  
			  ```
			  
			  (You could consider adding a garbage collection rule like
			  
			  ```jboss-cli
			    < ... [v]l ... | ... >
			  --> < ...      ... | ... >
			  
			  ```
			  
			  to discard the final value at the end of an activation.)
			  
			  Because at this level all the continuations are explicit, you can encode patterns other than sequential control flow, such as fork-join.
			  
			  For example, to start two long-running computations in parallel, and collect the answers in either order, multiplying them and sending the result to some actor k’, you could write
			  
			  ```smali
			  let k = create (λv1 . become (λv2 . send k' (v1 * v2))) in
			  send task1 (req1, k);
			  send task2 (req2, k)
			  
			  ```
			  
			  Practically speaking, both Hewitt’s original actor language, PLASMA, and the language Agha uses for his examples in the 1990 paper, Rosette, have special syntax for ordinary RPC so the programmer needn’t manipulate continuations themselves.
			  
			  So that covers the classic actor model. Create, send and become. Explicit use of actor addresses, and lots and lots of temporary actors for inter-actor RPC continuations.
			  
			  Before I move on to Erlang: remember right at the beginning I told you the actor model was
			  
			  * asynchronous message passing, and
			  * the isolated turn principle
			  
			  ?
			  
			  The isolated turn principle requires _liveness_ \- you’re not allowed to block indefinitely while responding to a message!
			  
			  But here, we can:
			  
			  ```swift
			  let d = create (λc . send c c) in
			  send d d
			  
			  ```
			  
			  Compare this with
			  
			  ```armasm
			  letrec beh = (λc . become beh; send c c) in
			  let d = create beh in
			  send d d
			  
			  ```
			  
			  These are both degenerate cases, but in different ways: the first becomes inert very quickly and the actor `d` is never returned to an idle/ready state, while the second spins uselessly forever.
			  
			  Other errors we could make would be to fail to send an expected reply to a continuation.
			  
			  One thing the semantics here rules out is interaction with other actors before doing a `become`; there’s no way to have a waiting continuation perform the `become`, because by that time you’re in a different actor. In this way, it sticks to the very letter of the Isolated Turn Principle, by forbidding “blocking”, but there are other kinds of things that can go wrong to destroy progress.
			  
			  Even if we require our behaviour functions to be total, we can still get _global_ nontermination.
			  
			  So saying that we “don’t have deadlock” with the actor model is very much oversimplified, even at the level of simple formal models, let alone when it comes to realistic programming systems.
			  
			  In practice, programmers often need “blocking” calls out to other actors before making a state update; with the classic actor model, this can be done, but it is done with a complicated encoding.
			  
			  \#\# Erlang: Actors for fault-tolerant systems
			  
			  Erlang is an example of what De Koster et al. call a Process-based actor language.
			  
			  It has its origins in telephony, where it has been used to build telephone switches with fabled “nine nines” of uptime. The research process that led to Erlang concentrated on high-availability, fault-tolerant software. The reasoning that led to such an actor-like system was, in a nutshell:
			  
			  * Programs have bugs. If part of the program crashes, it shouldn’t corrupt other parts. Hence strong isolation and shared-nothing message-passing.
			  * If part of the program crashes, another part has to take up the slack, one way or another. So we need crash signalling so we can detect failures and take some action.
			  * We can’t have all our eggs in one basket! If one machine fails at the hardware level, we need to have a nearby neighbour that can smoothly continue running. For that redundancy, we need distribution, which makes the shared-nothing message passing extra attractive as a unifying mechanism.
			  
			  Erlang is a two-level system, with a functional language core equipped with imperative actions for asynchronous message send and spawning new processes, like Agha’s system.
			  
			  The difference is that it lacks `become`, and instead has a construct called `receive`.
			  
			  Erlang actors, called processes, are ultra lightweight threads that run sequentially from beginning to end as little functional programs. As it runs, no explicit temporary continuation actors are created: any time it uses receive, it simply blocks until a matching message appears.
			  
			  After some initialization steps, these programs typically enter a message loop. For example, here’s a mutable cell:
			  
			  ```erlang
			  mainloop(Contents) ->
			  receive
			    {get, K} -> K ! Contents,
			                mainloop(Contents);
			    {put, V} -> mainloop(V)
			  end.
			  
			  ```
			  
			  A client program might be
			  
			  ```crystal
			  Cell = spawn(fun mainloop(0) end),
			  Cell ! {get, self()},
			  receive
			  V -> ...
			  end.
			  
			  ```
			  
			  Instead of using `become`, the program performs a tail call which returns to the `receive` statement as the last thing it does.
			  
			  Because `receive` is a statement like any other, Erlang processes can use it to enter substates:
			  
			  ```routeros
			  mainloop(Service) ->
			  receive
			    {req, K} -> Service ! {subreq, self()},
			                receive
			                  {subreply, Answer} -> K ! {reply, Answer},
			                                        mainloop(Service)
			                end
			  end.
			  
			  ```
			  
			  While the process is blocked on the inner `receive`, it only processes messages matching the patterns in that inner receive, and it isn’t until it does the tail call back to `mainloop` that it starts waiting for `req` messages again. In the meantime, non-matched messages queue up waiting to be `receive`d later.
			  
			  This is called “selective receive” and it is difficult to reason about. It doesn’t _quite_ violate the letter of the Isolated Turn Principle, but it comes close. (`become` can be used in a similar way.)
			  
			  The goal underlying “selective receive”, namely changing the set of messages one responds to and temporarily ignoring others, is important for the way people think about actor systems, and a lot of research has been done on different ways of selectively enabling message handlers. See Agha’s 1990 paper for pointers toward this research.
			  
			  One unique feature that Erlang brings to the table is crash signalling. The jargon is “links” and “monitors”. Processes can ask the system to make sure to send them a message if a monitored process exits. That way, they can perform RPC by
			  
			  * monitoring the server process
			  * sending the request
			  * if a reply message arrives, unmonitor the server process and continue
			  * if an exit message arrives, the service has crashed; take some corrective action.
			  
			  This general idea of being able to monitor the status of some other process was one of the seeds of my own research and my language Syndicate.
			  
			  So while the classic actor model had create/send/become as primitives, Erlang has spawn/send/receive, and actors are processes rather than event-handler functions. The programmer still manipulates references to actors/processes directly, but there are far fewer explicit temporary actors created compared to the “classic model”; the ordinary continuations of Erlang’s functional fragment take on those duties.
			  
			  \#\# E: actors for secure cooperation
			  
			  The last language I want to show you, E, is an example of what De Koster et al. call a Communicating Event-Loop language.
			  
			  E looks and feels much more like a traditional object-oriented language to the programmer than either of the variations we’ve seen so far.
			  
			  ```applescript
			  def makeCell (var contents) {
			    def getter {
			        to get() { return contents }
			    }
			    def setter {
			        to set(newContents) {
			            contents := newContents
			        }
			    }
			    return [getter, setter]
			  }
			  
			  ```
			  
			  The mutable cell in E is interestingly different: It yields _two_values. One specifically for setting the cell, and one for getting it. E focuses on security and securability, and encourages the programmer to hand out objects that have the minimum possible authority needed to get the job done. Here, we can safely pass around references to`getter` without risking unauthorized update of the cell.
			  
			  E uses the term “vat” to describe the concept closest to the traditional actor. A vat has not only a mailbox, like an actor, but also a call stack, and a heap of local objects. As we’ll see, E programmers don’t manipulate references to vats directly. Instead, they pass around references to objects in a vat’s heap.
			  
			  This is interesting because in the actor model messages are addressed to a particular actor, but here we’re seemingly handing out references to something finer grained: to individual objects _within_ an actor or vat.
			  
			  This is the first sign that E, while it uses the basic create/send/become-style model at its core, doesn’t expose that model directly to the programmer. It layers a special E-specific protocol on top, and only lets the programmer use the features of that upper layer protocol.
			  
			  There are two kinds of references available: near refs, which definitely point to objects in the local vat, and far refs, which may point to objects in a different vat, perhaps on another machine.
			  
			  To go with these two kinds of refs, there are two kinds of method calls: _immediate_ calls, and _eventual_ calls.
			  
			  receiver.method(arg, …)
			  
			  receiver <- method(arg, …)
			  
			  It is an error to use an immediate call on a ref to an object in a different vat, because it blocks during the current turn while the answer is computed. It’s OK to use eventual calls on any ref at all, though: it causes a message to be queued in the target vat (which might be our own), and a _promise_ is immediately returned to the caller.
			  
			  The promise starts off unresolved. Later, when the target vat has computed and sent a reply, the promise will become resolved. A nifty trick is that even an unresolved promise is useful: you can _pipeline_them. For example,
			  
			  ```ruby
			  def r1 := x <- a()
			  def r2 := y <- b()
			  def r3 := r1 <- c(r2)
			  
			  ```
			  
			  would block and perform multiple network round trips in a traditional simple RPC system; in E, there is a protocol layered on top of raw message sending that discusses promise creation, resolution and use. This protocol allows the system to send messages like
			  
			  ```smalltalk
			  "Send a() to x, and name the resulting promise r1"
			  "Send b() to y, and name the resulting promise r2"
			  "When r1 is known, send c(r2) to it"
			  
			  ```
			  
			  Crucial here is that the protocol, the language of discourse between actors, allows the expression of concepts including the notion of a send to happen at a future time, to a currently-unknown recipient.
			  
			  The protocol and the E vat runtimes work together to make sure that messages get to where they need to go efficiently, even in the face of multiple layers of forwarding.
			  
			  Each turn of an E vat involves taking one message off the message queue, and dispatching it to the local object it denotes. Immediate calls push stack frames on the stack as usual for object-oriented programming languages; eventual calls push messages onto the message queue. Execution continues until the stack is empty again, at which point the turn concludes and the next turn starts.
			  
			  One interesting problem with using promises to represent cross-vat interactions is how to do control flow. Say we had
			  
			  ```armasm
			  def r3 := r1 <- c(r2)  // from earlier
			  if (r3) {
			  myvar := a();
			  } else {
			  myvar := b();
			  }
			  
			  ```
			  
			  By the time we need to make a decision which way to go, the promise r3 may not yet be resolved. E handles this by making it an error to depend immediately on the value of a promise for control flow; instead, the programmer uses a `when` expression to install a handler for the event of the resolution of the promise:
			  
			  ```livescript
			  when (r3) -> {
			  if (r3) {
			    myvar := a();
			  } else {
			    myvar := b();
			  }
			  }
			  
			  ```
			  
			  The test of r3 and the call to a() or b() and the update to myvar are_delayed_ until r3 has resolved to some value.
			  
			  This looks like it violates the Isolated Turn Principle! It seems like we now have some kind of interleaving. But what’s going on under the covers is that promise resolution is done with an incoming message, queued as usual, and when that message’s turn comes round, the when clause will run.
			  
			  Just like with classic actors and with Erlang, managing these multiple-stage, stateful interactions in response to some incoming message is generally difficult to do. It’s a question of finding a balance between the Isolated Turn Principle, and its commitment to availability, and encoding the necessary state transitions without risking inconsistency or deadlock.
			  
			  Turning to failure signalling, E’s vats are not just the units of concurrency but also the units of partial failure. An uncaught exception within a vat destroys the whole vat and invalidates all remote references to objects within it.
			  
			  While in Erlang, processes are directly notified of failures of other processes as a whole, E can be more fine-grained. In E, the programmer has a convenient value that represents the outcome of a transaction: the promise. When a vat fails, or a network problem arises, any promises depending on the vat or the network link are put into a special state: they become _broken promises_. Interacting with a broken promise causes the contained exception to be signalled; in this way, broken promises propagate failure along causal chains.
			  
			  If we look under the covers, E seems to have a “classic style” model using create/send/become to manage and communicate between whole vats, but these operations aren’t exposed to the programmer. The programmer instead manipulates two-part “far references” which denote a vat along with an object local to that vat. Local objects are created frequently, like in regular object-oriented languages, but vats are created much less frequently; and each vat’s stack takes on duties performed in “classic” actor models by temporary actors.
			  
			  \#\# Conclusion
			  
			  I’ve presented three different types of actor language: the classic actor model, roughly as formulated by Agha et al.; the process actor model, represented by Erlang; and the communicating event-loop model, represented by E.
			  
			  The three models take different approaches to reconciling the need to have structured local data within each actor in addition to the more coarse-grained structure relating actors to each other.
			  
			  The classic model makes everything an actor, with local data largely deemphasised; Erlang offers a traditional functional programming model for handling local data; and E offers a smooth integration between an imperative local OO model and an asynchronous, promise-based remote OO model.
			  
			  ---
			  
			  \#\# Annotated Bibliography
			  
			  \#\#\# Early work on actors
			  
			  * **1973\. Carl Hewitt, Peter Bishop, and Richard Steiger, “A universal modular ACTOR formalism for artificial intelligence,” in Proc. International Joint Conference on Artificial Intelligence**  
			  This paper is a position paper from which we can understand the motivation and intentions of the research into actors; it lays out a very broad and colourful research vision that touches on a huge range of areas from computer architecture up through programming language design to teaching of computer science and approaches to artificial intelligence.  
			  The paper presents a _uniform actor model_; compare and contrast with uniform object models offered by (some) OO languages. The original application of the model is given as PLANNER-like AI languages.  
			  The paper claims benefits of using the actor model in a huge range of areas:  
			   * foundations of semantics  
			   * logic  
			   * knowledge-based programming  
			   * intentions (contracts)  
			   * study of expressiveness  
			   * teaching of computation  
			   * extensible, modular programming  
			   * privacy and protection  
			   * synchronization constructs  
			   * resource management  
			   * structured programming  
			   * computer architecture  
			  The paper sketches the idea of a contract (called an “intention”) for ensuring that invariants of actors (such as protocol conformance) are maintained; there seems to be a connection to modern work on “monitors” and Session Types. The authors write:  
			  > The intention is the CONTRACT that the actor has with the outside world.  
			  Everything is super meta! Actors can have intentions! Intentions are actors! Intentions can have intentions! The paper presents the beginnings of a reflective metamodel for actors. Every actor has a scheduler and an “intention”, and may have monitors, a first-class environment, and a “banker”.  
			  The paper draws an explicit connection to capabilities (in the security sense); Mark S. Miller at<http://erights.org/history/actors.html> says of the Actor work that it included “prescient” statements about Actor semantics being “a basis for confinement, years before Norm Hardy and the Key Logic guys”, and remarks that “the Key Logic guys were unaware of Actors and the locality laws at the time, \[but\] were working from the same intuitions.”  
			  There are some great slogans scattered throughout, such as “Control flow and data flow are inseparable”, and “Global state considered harmful”.  
			  The paper does eventually turn to a more nuts-and-bolts description of a predecessor language to PLASMA, which is more fully described in Hewitt 1976.  
			  When it comes to formal reasoning about actor systems, the authors here define a partial order - PRECEDES - that captures some notion of causal connection. Later, the paper makes an excursion into epistemic modal reasoning.  
			  Aside: the paper discusses continuations; Reynolds 1993 has the concept of continuations as firmly part of the discourse by 1973, having been rediscovered in a few different contexts in 1970-71 after van Wijngaarden’s 1964 initial description of the idea. See J. C. Reynolds, “The discoveries of continuations,” LISP Symb. Comput., vol. 6, no. 3–4, pp. 233–247, 1993.  
			  In the “acknowledgements” section, we see:  
			  > Alan Kay whose FLEX and SMALL TALK machines have influenced our work. Alan emphasized the crucial importance of using intentional definitions of data structures and of passing messages to them. This paper explores the consequences of generalizing the message mechanism of SMALL TALK and SIMULA-67; the port mechanism of Krutar, Balzer, and Mitchell; and the previous CALL statement of PLANNER-71 to a universal communications mechanism.
			  * **1975\. Irene Greif. PhD dissertation, MIT EECS.**  
			  Specification language for actors; per Baker an “operational semantics”. Discusses “continuations”.
			  * **1976\. C. Hewitt, “Viewing Control Structures as Patterns of Passing Messages,” AIM-410**  
			  AI focus; actors as “agents” in the AI sense; recursive decomposition: “each of the experts can be viewed as a society that can be further decomposed in the same way until the primitive actors of the system are reached.”  
			  > We are investigating the nature of the _communication mechanisms_\[…\] and the conventions of discourse  
			  More concretely, examines “how actor message passing can be used to understand control structures as patterns of passing messages”.  
			  > \[…\] there is no way to decompose an actor into its parts. An actor is defined by its behavior; not by its physical representation!  
			  Discusses PLASMA (“PLAnner-like System Modeled on Actors”), and gives a fairly detailed description of the language in the appendix. Develops “event diagrams”.  
			  Presents very Schemely factorial implementations in recursive and iterative (tail-recursive accumulator-passing) styles. During discussion of the iterative factorial implementation, Hewitt remarks that `n` is not closed over by the `loop` actor; it is “not an acquaintance” of `loop`. Is this the beginning of the line of thinking that led to Clinger’s “safe-for-space” work?  
			  Everything is an actor, but some of the actors are treated in an awfully structural way: the trees, for example, in section V.4 on Generators:  
			  ```clojure  
			  (non-terminal:  
			  (non-terminal: (terminal: A) (terminal: B))  
			  (terminal: C))  
			  ```  
			  Things written with this `keyword:` notation look like structures. Their _reflections_ are actors, but as structures, they are subject to pattern-matching; I am unsure how the duality here was thought of by the principals at the time, but see the remarks regarding “packages” in the appendix.
			  * **1977\. C. Hewitt and H. Baker, “Actors and Continuous Functionals,” MIT A.I. Memo 436A**  
			  Some “laws” for communicating processes; “plausible restrictions on the histories of computations that are physically realizable.” Inspired by physical intuition, discusses the history of a computation in terms of a partial order of events, rather than a sequence.  
			  > The actor model is a formalization of these ideas \[of Simula/Smalltalk/CLU-like active data processing messages\] that is independent of any particular programming language.  
			  > Instances of Simula and Smalltalk classes and CLU clusters are actors”, but they are _non-concurrent_. The actor model is broader, including concurrent message-passing behaviour.  
			  Laws about (essentially) lexical scope. Laws about histories (finitude of activation chains; total ordering of messages inbound at an actor; etc.), including four different ordering relations. “Laws of locality” are what Miller was referring to on that erights.org page I mentioned above; very close to the capability laws governing confinement.  
			  Steps toward denotational semantics of actors.
			  * **1977\. Russell Atkinson and Carl Hewitt, “Synchronization in actor systems,” Proc. 4th ACM SIGACT-SIGPLAN Symp. Princ. Program. Lang., pp. 267–280.**  
			  Introduces the concept of “serializers”, a “generalization and improvement of the monitor mechanism of Brinch-Hansen and Hoare”. Builds on Greif’s work.
			  * **1981\. Will Clinger’s PhD dissertation. MIT**
			  
			  \#\#\# Actors as Concurrent Object-Oriented Programming
			  
			  * **1986\. Gul Agha’s book/dissertation.**
			  * **1990\. G. Agha, “Concurrent Object-Oriented Programming,” Commun. ACM, vol. 33, no. 9, pp. 125–141**  
			  Agha’s work recast the early “classic actor model” work in terms of_concurrent object-oriented programming_. Here, he defines actors as “self-contained, interactive, independent components of a computing system that communicate by asynchronous message passing”, and gives the basic actor primitives as `create`, `send to`, and `become`. Examples are given in the actor language Rosette.  
			  This paper gives an overview and summary of many of the important facets of research on actors that had been done at the time, including brief discussion of: nondeterminism and fairness; patterns of coordination beyond simple request/reply such as transactions; visualization, monitoring and debugging; resource management in cases of extremely high levels of potential concurrency; and reflection.  
			  > The customer-passing style supported by actors is the concurrent generalization of continuation-passing style supported in sequential languages such as Scheme. In case of sequential systems, the object must have completed processing a communication before it can process another communication. By contrast, in concurrent systems it is possible to process the next communication as soon as the replacement behavior for an object is known.  
			  Note that the sequential-seeming portions of the language are defined in terms of asynchronous message passing and construction of explicit continuation actors.
			  * **1997\. G. A. Agha, I. A. Mason, S. F. Smith, and C. L. Talcott, “A Foundation for Actor Computation,” J. Funct. Program., vol. 7, no. 1, pp. 1–72**  
			  Long paper that carefully and fully develops an operational semantics for a concrete actor language based on lambda-calculus. Discusses various equivalences and laws. An excellent starting point if you’re looking to build on a modern approach to operational semantics for actors.
			  
			  \#\#\# Erlang: Actors from requirements for fault-tolerance / high-availability
			  
			  * **2003\. J. Armstrong, “Making reliable distributed systems in the presence of software errors,” Royal Institute of Technology, Stockholm**  
			  A good overview of Erlang: the language, its design intent, and the underlying philosophy. Includes an evaluation of the language design.
			  
			  \#\#\# E: Actors from requirements for secure interaction
			  
			  * **2005\. M. S. Miller, E. D. Tribble, and J. Shapiro, “Concurrency Among Strangers,” in Proc. Int. Symp. on Trustworthy Global Computing, pp. 195–229.**  
			  As I summarised this paper for a seminar class on distributed systems: “The authors present E, a language designed to help programmers manage_coordination_ of concurrent activities in a setting of distributed, mutually-suspicious objects. The design features of E allow programmers to take control over concerns relevant to distributed systems, without immediately losing the benefits of ordinary OO programming.”  
			  E is a canonical example of the “communicating event loops” approach to Actor languages, per the taxonomy of the survey paper listed below. It combines message-passing and isolation in an interesting way with ordinary object-oriented programming, giving a two-level language structure that has an OO flavour.  
			  The paper does a great job of explaining the difficulties that arise when writing concurrent programs in traditional models, thereby motivating the actor model in general and the features of E in particular as a way of making the programmer’s job easier.
			  
			  \#\#\# Taxonomy of actors
			  
			  * **2016\. J. De Koster, T. Van Cutsem, and W. De Meuter, “43 Years of Actors: A Taxonomy of Actor Models and Their Key Properties,” Software Languages Lab, Vrije Universiteit Brussel, VUB-SOFT-TR-16-11**  
			  A very recent survey paper that offers a taxonomy for classifying actor-style languages. At its broadest, actor languages are placed in one of four groups:  
			   * The Classic Actor Model (create, send, become)  
			   * Active Objects (OO with a thread per object; copying of passive data between objects)  
			   * Processes (raw Erlang; receive, spawn, send)  
			   * Communicating Event-Loops (E; near and far references; eventual references; batching)  
			  Different kinds of “futures” or “promises” also appear in many of these variations in order to integrate asynchronous message_reception_ with otherwise-sequential programming.
	- [blockchain-new-paper](https://omnivore.app/me/u-7-e-8-dd-540-b-132-11-ee-b-631-eb-976-daebc-17-blockchain-new--18cfd24bdf0)
	  collapsed:: true
	  site:: [omnivore.app](https://omnivore.app/attachments/u/7e8dd540-b132-11ee-b631-eb976daebc17/blockchain-new-paper.pdf)
	  author:: MCA-46
	  date-saved:: [[01/12/2024]]
		- ### Content
		  collapsed:: true
			-