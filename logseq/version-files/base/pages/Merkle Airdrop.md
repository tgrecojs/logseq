## April 6th
	- - [[GitHub Clean]]
	-
-
- ### Initial Info
	- [[Starting info...]]
- ## Tree Operations
	- What does the term little-endian mean now in particular I'm curious about the meaning of little Indian when used when used when discussing encoding and decoding values
	- #[[Omnivore Content]]
	-
- ## #Cryptography
	- ## Finite Field Operations
		- A finite field is defined as a finite set of numbers and two operations, addition and mulitplication, which satisfy the following:
			- Closed Property 
			  logseq.order-list-type:: number
				- if `a` and `b` are in the set, `a + b` and `a * b` are in the set.
				  logseq.order-list-type:: number
			- Additive Identity Property - 0 exists and has the property `a + 0` = `a`.
			  logseq.order-list-type:: number
			- Mutiplicative identity - 1 exists and `a * 1 = a`
			  logseq.order-list-type:: number
			- logseq.order-list-type:: number
	-
	- ## Hash Functions
		- #### Terms
			- **digest**
				- The output of a hash function is known as the **digest**.
				- A digest holds a summary Aor a representation o fth eoriginal data.
			- **MAC**
		- ### Security Properties of Hash Function
			- Hash functions should satisfy at least one of the following properties:
				- **Preimage resistance**: Given a hash value y for which a corresponding input is not known, it is computationally infeasible 2 to ﬁnd any input x such that y = h(x). This property is also known as one-wayness.
				  logseq.order-list-type:: number
				- **Second-preimage resistance:** Given an input x 1 it is computationally infeasible to ﬁnd another input where x 1 = x 2 such that h(x 1 ) = h(x 2 ). This property is also referred ̸ to as weak collision resistance.
				  logseq.order-list-type:: number
				- **Collision resistance:** It is computationally infeasible to ﬁnd any two inputs x 1 and x2  where x 1 = x 2 such that h(x 1 ) = h(x 2 ). This property is sometimes referred to as ̸ strong collision resistance.
				  logseq.order-list-type:: number
		- ### checksums
			- A function that takes a binary string and divides it into 32-bit blocks while computing the modular sum of those blocks.
			- Benefits: easily computed, offers compression
			- Cons: **NOT SAFE**
				- It is possible to work backwards from the checksum to an input satisfying the sum, therefore it is not **preimage or second-preimage resistant**.
	- MAC
		- Message Authentication Code Algorithm
-
- ## Entropy
- > Min entropy measures how likely you are to guess a value on your first try. If this probability is p, then the min entropy is defined as `-log_2 (p)`.
-