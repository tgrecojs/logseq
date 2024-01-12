- [Zeboot-agenda-Jan-24](https://omnivore.app/me/zeboot-agenda-jan-24-18cfb50c2c2)
  collapsed:: true
  site:: [electriccoin.co](https://electriccoin.co/wp-content/uploads/2024/01/Zeboot-agenda-Jan-24.pdf)
  date-saved:: [[01/11/2024]]
- [Tech/Finance Skill Convergence](https://omnivore.app/me/tech-finance-skill-convergence-18cfa0a0c27)
  collapsed:: true
  site:: [The Diff](https://www.thediff.co/archive/tech-finance-skill-convergence/?ref=the-diff-newsletter)
  author:: Byrne Hobart
  date-saved:: [[01/11/2024]]
  date-published:: [[01/11/2024]]
	- ### Highlights
	  collapsed:: true
		- > (obsessing over latency is useful for cloud computing and for trade execution; being able to tease out tiny signals from huge volumes of data is helpful for systematic investing as well as ad targeting). [⤴️](https://omnivore.app/me/tech-finance-skill-convergence-18cfa0a0c27#410b3433-6b72-47bc-ab30-258378e994b9)
		- > [some Excel products grow to the point that only a single person can understand them, and have to be scrapped when that person leaves](https://www.baseballprospectus.com/news/article/12082/reintroducing-pecota-what-a-long-strange-trip-it-has-been/?ref=thediff.co).
		  > 
		  That last problem isn't intrinsic to Excel, but to programming in general: [⤴️](https://omnivore.app/me/tech-finance-skill-convergence-18cfa0a0c27#974b83e8-a06c-47d5-b02f-e8820b3a10e1)
		- > Scrutability Frontier [⤴️](https://omnivore.app/me/tech-finance-skill-convergence-18cfa0a0c27#2d71d6ae-7790-42d8-805e-c30a700a98d3) 
		  
		  note:: In a general sense, 'scrutability' refers to the degree to which something can be investigated or understood, and 'frontier' refers to an advancing edge or boundary of activity or knowledge. Therefore, a "scrutability frontier" could be understood as the boundary at which our understanding or transparency in a system or phenomena ends.
- [The Protocol Seeking Protocol](https://omnivore.app/me/the-protocol-seeking-protocol-18cf7428727)
  collapsed:: true
  site:: [Dan Finlay’s Blog](https://blog.danfinlay.com/protocol-seeking-protocol/)
  author:: Dan Finlay
  date-saved:: [[01/11/2024]]
  date-published:: [[01/10/2024]]
- [How I take Notes: Mastering the Basics - by Nicola Ballotta](https://omnivore.app/me/how-i-take-notes-mastering-the-basics-by-nicola-ballotta-18cf594cd16)
  collapsed:: true
  site:: [The Hybrid Hacker](https://hybridhacker.email/p/how-i-take-notes-mastering-the-basics)
  author:: Nicola Ballotta
  labels:: [[note-taking]] [[hybrid-hacker]]
  date-saved:: [[01/10/2024]]
  date-published:: [[03/09/2023]]
- [Issue 48 – Bitcoin has "no chance" of going to the moon](https://omnivore.app/me/issue-48-bitcoin-has-no-chance-of-going-to-the-moon-18cf2095d8e)
  collapsed:: true
  site:: [Citation Needed](https://citationneeded.news/issue-48/)
  author:: Molly White
  labels:: [[Read-later]]
  date-saved:: [[01/10/2024]]
  date-published:: [[01/09/2024]]
- [Beyond the shouting match: what is a blockchain, really? -- Dustycloud Brainstorms](https://omnivore.app/me/https-dustycloud-org-blog-what-is-a-blockchain-really-18ce2b8f1d9)
  collapsed:: true
  site:: [dustycloud.org](https://dustycloud.org/blog/what-is-a-blockchain-really/)
  date-saved:: [[01/07/2024]]
  date-published:: [[04/23/2021]]
-
- ## All posts
	- [Merkle Trees](https://omnivore.app/me/merkle-trees-18cfc289cef)
	  collapsed:: true
	  site:: [Omnivore](https://omnivore.app/tgrecojs/merkle-trees-18cfc251d0e)
	  date-saved:: [[01/12/2024]]
		- ### Content
		  collapsed:: true
			- January 12, 2024 • 7 min read
			  
			  Merkle Trees
			  
			  An efficient and verifiable data structure
			  
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
			  
			  Report issues with this page ->
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
	- [How I take Notes: Mastering the Basics - by Nicola Ballotta](https://omnivore.app/me/how-i-take-notes-mastering-the-basics-by-nicola-ballotta-18cf594cd16)
	  collapsed:: true
	  site:: [The Hybrid Hacker](https://hybridhacker.email/p/how-i-take-notes-mastering-the-basics)
	  author:: Nicola Ballotta
	  labels:: [[logseq]] [[note-taking]] [[hybrid-hacker]]
	  date-saved:: [[01/10/2024]]
	  date-published:: [[03/09/2023]]
		- ### Content
		  collapsed:: true
			- \#\#\# A Practical Guide on How to Take Notes. In This First Part, We Will Explore the Basics of Note-taking.
			  
			  I'm honest, if I had to start this essay from a blank page, I'd probably spend hours looking at the blinking cursor without writing a single character. But luckily enough, I've been thinking about this post for weeks now, and guess what? I have all my notes about it, which made starting the piece much more straightforward.
			  
			  When I was younger, precisely in school, **I literally hated taking notes**. I know people who still love to show their school notebooks and diaries, full of notes, schemas, drawings, schedules, and writings. I also still have mine at my parents' house, and you could probably sell them as new.
			  
			  I was so arrogant as to pretend to record everything in my head. Well, I did that for a long time, and I was also good enough at it. But unfortunately, **our brain's capacity is limited** (approximately **2.5 Petabytes of storage** according [Scientific American](https://www.scientificamerican.com/article/what-is-the-memory-capacity/)), and so at one point, my brain started overflowing, losing information. This is when I started taking notes, and this is also where **my life changed**.
			  
			  Someone might think note-taking is just about recording informations, but it's not. I said note-taking literally changed my life, and it was not just a catchy phrase. If you ask yourself why you take notes, the first answers that will come to your mind would probably be "_**to not forget important things**_" or "_**to organize my knowledge**_". And for sure, **these are good reasons to take notes**, but there are also other aspects that probably are not so evident at first glance.
			  
			  Some of the benefits of note-taking, especially if done the right way, include:
			  
			  * 🧠 **Increased memory efficiency**. Writing is a very good way to impress things in your brain and exercise it.
			  * ✍️ **Become better at writing**. Taking notes means writing more, and the more you write, the better you become at it.
			  * 📖 **Become better at reading**. As for writing, reading is a huge part of note-taking.
			  * 🤓 **Improved understanding**. Writing notes means processing information (we will see after how) and processing information helps better understand concepts.
			  * 👮‍♂️ **Increased discipline**. As you may imagine, to be good at all the points I mentioned before, you need to be disciplined, and writing notes is a good way to become it!
			  
			  Another **common misconception is that** **taking structured notes is just for researchers or students**. Nothing could be further from the truth.
			  
			  Taking notes is simply for everyone. It's just a process, **a habit that you establish in your everyday routine** that helps process and keep track of any kind of information. It doesn't matter if you are tracking your research in neuroscience, your fitness progress, your mood, or your cooking recipes; it works the same way.
			  
			  [![](https://proxy-prod.omnivore-image-cache.app/1019x457,sDHjRo1Jy-zMfe6NryEmMkxpQiu5a441sU0nuQaQYknI/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F73d2d758-3490-40f8-a556-e5435efa4302_1019x457.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F73d2d758-3490-40f8-a556-e5435efa4302%5F1019x457.png)
			  
			  There's **an input**, you decide if this input **is relevant** for you, if it is **you process it** and then you store it in a note. Easy peasy.
			  
			  \#\# 📚 A Bit of Theory
			  
			  When you start delving deep into note-taking, **you will discover an entirely new world**.
			  
			  There are entire books about note-taking, methodologies to organize your notes, to grow them, and at one point, to make them yours. It would probably take many issues of this newsletter to go through them all. For the scope of this essay, which is **to share the way I take notes** and **introduce you to this world**, I will just go through some concepts that you will see used in the more practical part.
			  
			  \#\#\# PKM vs PKD
			  
			  Note-taking refers to the action of recording information in a place of your choice, such as a notebook, text file, dedicated software, database, etc. However, it is often put in relation to **Personal Knowledge Management** (PKM).
			  
			  Personal Knowledge Management refers to the **process of** **collecting, organizing, and managing one's own personal knowledge** and information in a way that is useful and accessible.
			  
			  Some people also make a further distinction between Personal Knowledge Management and **Personal Knowledge Development** (PKD). While the former involves collecting and organizing notes, the latter refers to the **process of acquiring, organizing, and integrating new information and experiences into one's existing knowledge base**. It involves actively seeking out new information, reflecting on one's experiences, and making connections between new and old knowledge.
			  
			  \#\#\# P.A.R.A.
			  
			  The P.A.R.A. method is a note-taking and personal knowledge management system created by [Tiago Forte](https://fortelabs.com/), a productivity and organization expert.
			  
			  P.A.R.A. stands for **Projects**, **Areas**, **Resources**, and **Archives**, and the main idea is to organize your notes **based on their actionability**.
			  
			  * 📂 **Projects** \- This section is designated for your ongoing projects. A project refers to a series of tasks that are linked to a specific objective and must be completed within a **defined timeframe**. These tasks represent the **tangible actions** you intend or must undertake.
			  * 📍**Areas** \- This section contains **ongoing activities** that are **still actionable** but **do not have a fixed deadline**, such as work notes, writing, personal finance, and maintaining one's health. These are tasks that demand regular maintenance and attention. As these activities are consistently monitored, new projects may emerge from this section.
			  * ⛏️ **Resources** \- This section serves as a r**epository for information on topics that pique your interest** or prove valuable to you. As the title implies, these are nuggets of knowledge that you wish to retain and consult at a later time.
			  * 🗄️ **Archives** \- This section is designated for **storing items from the other three categories that have already been finished** or are no longer relevant. This includes items such as completed projects, inactive areas, or resources that you no longer wish to actively manage.
			  
			  [![](https://proxy-prod.omnivore-image-cache.app/1019x457,s2ypoZipZ0YGspNkXQD6U3Zvh4roiqo1_orv4pwksWRY/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67fa7065-82f3-47ee-adfc-3de16d299332_1019x457.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67fa7065-82f3-47ee-adfc-3de16d299332%5F1019x457.png)
			  
			  According to Tiago Forte, **these are the four folders you should have** to organize your knowledge and where to put your notes.
			  
			  As you may have noticed, while the Projects and Archives sections are pretty clear sections, the **Area and Resources sections are less defined and could overlap**. Don't worry too much about this; these are just guidelines and inspirations. You can decide how to manage your knowledge!
			  
			  Personally, I use the Area folder for work-related notes, essays, etc. - things that regularly change and are more actionable. While I put most of the things I'm interested in (e.g. Technology, Cooking, Outdoor) in the Resources folder. You will find people who choose where to put things based on how personal they are and others who put just files, schemes, drawings in the Resource folder. **Do whatever you feel is right for you**.
			  
			  \#\#\# Zettelkasten
			  
			  Zettelkasten is **something in between a knowledge management and development system** that was ideated by German sociologist [Niklas Luhmann](https://en.wikipedia.org/wiki/Niklas%5FLuhmann). The name "Zettelkasten" means "slip box" or "index card system" in German, and it refers to a physical or digital collection of notes that are stored and organized in a specific way to aid in idea generation and knowledge organization.
			  
			  [![](https://proxy-prod.omnivore-image-cache.app/1019x457,s8SBE167FS-8exOGcVTi22tNLHeZwEmUOrXh8TdqPe4U/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4e0dded0-9b30-4220-8f03-44f8f68ad820_1019x457.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4e0dded0-9b30-4220-8f03-44f8f68ad820%5F1019x457.png)
			  
			  In a Zettelkasten system, **each note is created on a separate card or digital file**, with a unique identifier, date, and title. The notes are then organized by topic or theme and **connected to other relevant notes through a system of links or tags**. The goal of this system is to allow for flexible and creative idea generation and organization, as well as to enable effective retrieval and synthesis of information.
			  
			  The Zettelkasten methodology has three fundamental concepts:
			  
			  * 📥 **Fleeting Notes** \- These are notes in their most primitive stage. Whenever something sparks your interest, an idea, or something you want to explore further, you should quickly take note of it. For example, "I see a relation between Availability and Accountability in remote working." This should be a **brief but meaningful note**. It's not important how you write it or if it contains typos. You can see it as a reminder that needs to be processed later.
			  * 📚 **Literature Notes** \- These notes are useful when you want to reference something that already exists, such as a theory, idea, or notion. Ideally, even if it's coming from an external source, **you should write this note in your own words**. However, some notes can be copy-pasted.
			  * 🍀 **Permanent Notes** \- This is one of the most important pieces of the Zettelkasten method. While Literature Notes are thoughts coming from third parties, **permanent notes** (also known as Evergreen Notes) **reflect your thoughts**. They could be influenced by Literature Notes, but they should always include your personal thoughts on a particular topic.
			  
			  For Zettelkasten, you should only have these three folders. However, finding notes could become difficult, so how can you find them effectively? In Zettelkasten, there are two more concepts that are crucial:
			  
			  * 🔗 **Links** \- As we saw before, the main idea is to take atomic notes that you can process and elaborate on. But these notes are not very useful alone. That's why linking is one of the core principles of Zettelkasten. Linking multiple notes together can make them much more powerful.
			  * 🏷️ **Tags** \- All your ideas and notes, following the Zettelkasten approach, are scattered in these three folders. That's why you should always use tags. Tags make it easy to find things.
			  
			  I know these are a lot of concepts all together. Please consider that there are entire books discussing Zettelkasten (see the resources section), so it's challenging to condense everything in this article. But let me provide a real example to help you understand better.
			  
			  \#\#\#\# 🪚 A Practical Example
			  
			  Let's take the idea I mentioned earlier. Let’s pretend that while thinking or reading something, I came up with the idea that _Availability and Accountability are related when we talk about remote working_.
			  
			  This is what I would do (I actually did that before writing “[Balancing Availability and Accountability in Remote Working](https://hybridhacker.email/p/balancing-availability-and-accountability)”):
			  
			  * 📥 I take a quick fleeting note about my idea. It should **be meaningful** so I can come back to it later and understand what I was thinking about.
			  * 📚 Meanwhile, I want to have a clearer idea of what Availability and Accountability are, so I start gathering information from the web (or reading a book, or watching videos), to better understand these concepts. When I have a better understanding, I **create a literature note for Accountability and one for Availability**, storing the information I got from third parties there. I also put meaningful tags so I can easily find these notes when needed.
			  * 🍀 After thinking more about my idea, I decide it's time to transform it into something permanent. So I create a permanent/evergreen note. This note **is written in my own words and reflects my idea**, even though I can link the literature notes I created earlier to help contextualize. I also tag this note so it's easier to find when I need it.
			  
			  You see the power of this approach? I hope so, because we are just scratching the surface!
			  
			  \#\# Resources
			  
			  The concepts I explained earlier should be sufficient to understand the basics and approach the more practical guide that we will explore in the next part of this essay. If you wish to dive deeper, here are some interesting resources you can read.
			  
			  📚 **Books**
			  
			  * [How to Take Smart Notes](https://www.amazon.com/How-Take-Smart-Notes-Technique-dp-3982438802/dp/3982438802/) (Sönke Ahrens) - Probably the best and most famous book about Zettelkasten method
			  * [Building a Second Brain](https://www.amazon.com/Building-Second-Brain-Organize-Potential/dp/1982167386) (Tiago Forte) - A must read for notes taking (includes the P.A.R.A method)
			  
			  🔗 **Links**
			  
			  * [Progressive Summarization](https://fortelabs.com/blog/progressive-summarization-a-practical-technique-for-designing-discoverable-notes/) \- How to write better and easily discoverable notes
			  * [Writing Atomic Notes](https://zettelkasten.de/posts/create-zettel-from-reading-notes/) \- Good and practical examples on how to take atomic notes and process them
			  
			  \#\#\#\# 🗣️ Other
			  
			  * <https://www.reddit.com/r/NoteTaking/> \- Subreddit where to exchange informations on how to take notes
			  * <https://www.reddit.com/r/Zettelkasten/> \- Subreddit dedicated to Zettelkasten methodology
			  
			  \#\# ⏩ What to Expect Next
			  
			  I know that all these concepts together, could seem intimidating, but they will be needed to understand the second and more practical part of my note-taking approach. In the next part of “How I take Notes” I will tell you:
			  
			  * How I process notes and transform them into meaningful knowledge
			  * How my PKM workflow works
			  * How I structured my PKM folders
			  * How I use Obsidian to manage my PKM applying the principles I described
			  
			  \#\# ✌️ That’s all folks
			  
			  That's all for today! As always, I would love to hear from my readers (and if you've made it this far, you're definitely one of the bravest). Please don't hesitate to connect with me on [LinkedIn](https://www.linkedin.com/in/nicolaballotta) and send a message. I always respond to everyone!
	- [Organize your Omnivore library with labels](https://omnivore.app/me/organize-your-omnivore-library-with-labels-18a026d6ea3)
	  collapsed:: true
	  site:: [Omnivore Blog](https://blog.omnivore.app/p/organize-your-omnivore-library-with)
	  author:: The Omnivore Team
	  date-saved:: [[08/17/2023]]
	  date-published:: [[04/17/2022]]
		- ### Content
		  collapsed:: true
			- Omnivore provides labels (also known as tags) to help you organize your library. Labels can be added to any saved read, and your library can be filtered based on labels. 
			  
			  On the web if you have a larger screen you can find the labels tool on the left side of the screen. 
			  
			  [ ![](https://proxy-prod.omnivore-image-cache.app/960x711,sd_iEqPoEHqI7WkOUFPZl1axCkvtYhw5KBmUIne0oSQ0/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fa4ec9f3c-baef-464b-8d3a-0b8a384874d3_960x711.gif) ](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fa4ec9f3c-baef-464b-8d3a-0b8a384874d3%5F960x711.gif) 
			  
			  Adding a label from the left menu 
			  
			  For a smaller screen you will find the labels button at the top of the page. 
			  
			  [ ![](https://proxy-prod.omnivore-image-cache.app/1456x1315,sdpGYdxa-cZ3SZWswOh__3ynxRP98SMiWWeOJz6juAzY/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fb59fdeb0-d711-442e-a7c2-4508bedad515_1818x1642.png) ](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fb59fdeb0-d711-442e-a7c2-4508bedad515%5F1818x1642.png) 
			  
			  Article Actions at the top of the Omnivore Reader for smaller screens 
			  
			  iOS users can long press on an item in the library or access the labels modal from the menu in the top right of the reader page. 
			  
			  [ ![](https://proxy-prod.omnivore-image-cache.app/470x0,sOUHLFwk4mORdkuStr6Jl719evRUX-XTauXEp85h4iL4/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F6c99b6c9-7b78-41a8-84b1-a738e832308d_1182x2034.png) ](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F6c99b6c9-7b78-41a8-84b1-a738e832308d%5F1182x2034.png) 
			  
			  Editing labels from the library view on iOS 
			  
			  \#\#  Label searches on iOS 
			  
			  On iOS you can use the label search modal to search for specific labels. This will create an OR search and return all links matching the assigned labels. 
			  
			  [ ![](https://proxy-prod.omnivore-image-cache.app/418x0,s3ZzvKJlkfrGLNHHjmS1HkMp6toHMoHzZXkpZaap0C84/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fc6720208-7551-41cb-a86d-afc58bb160c9_1084x2006.png) ](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fc6720208-7551-41cb-a86d-afc58bb160c9%5F1084x2006.png) 
			  
			  Using labels to filter your search 
			  
			  \#\#  Using Advanced Search to filter your library with labels 
			  
			  Omnivore's advanced search syntax supports searching for multiple labels using AND and OR clauses. You can also negate a label search to find pages that do not have a certain label. 
			  
			  Some examples: 
			  
			  * `label:Newsletter` finds all pages that have the label Newsletter
			  * `label:Cooking,Fitness` finds all your pages with either the Cooking or Fitness labels
			  * `label:Newsletter label:Surfing` finds all pages with both the Newsletter and Surfing labels
			  * l`abel:Coding -label:News` finds all pages with the Coding label that do not have the News label