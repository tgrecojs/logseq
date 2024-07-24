tags:: #Books, #book-notes
	- #### TIL Why 256 #[[TIL Series]]
		- > SHA-256 always provides an output of 256 bits (32 bytes), which is always encoded as 64 alphanumeric characters in hexadecimal.
		- 256 bits is equal to 32 bytes.
	- ## Hash Functions
		- The output of a hash function is **always the same length**.
		- > SHA-256 always provides an output of 256 bits (32 bytes), which is always encoded as 64 alphanumeric characters in hexadecimal.
		- ### Security Properties of Hash Functions
			- Pre-image resistance
			  logseq.order-list-type:: number
				- Refers to the inability to reverse the hash function in order to decode it.
				  logseq.order-list-type:: number
			- Second Pre-image resistance
			  logseq.order-list-type:: number
				- If given an input and the digest it hashes to, you should not be able to find an additional input that hashes to the same digest.
				  logseq.order-list-type:: number
			- Collision resistance
			  logseq.order-list-type:: number
				- It guarantees that no one should be able to produce two different inputs that hash to the same output
				  logseq.order-list-type:: number
		- ### Understanding the difference between collision resistance and 2nd pre-image resistance
		  collapsed:: true
			- From #MindMac
				- Prompt:
					- **Please explain the difference between collision resistance, pre-image resistance, and seconds pre-image resistance.**
				- Answer
					- Sure, let's take a look at these properties one by one.
					  
					  1) **Collision Resistance**
					  
					  This basically means that it should be hard to find two different inputs that produce the same hash output. In other words, no two separate input values should ideally create the same hash.
					  
					  *Real-world example:* Let's use an example of a school. Assume the student's unique identification number is a hash of their full name. Collision resistance would ensure that two students with different names can't have the same identification number.
					  
					  2) **Pre-image Resistance**
					  
					  This means that if you're given a hash h, it should be hard to find any message m such that H(m) = h. So even if you know the output hash, you can't determine the original input.
					  
					  *Real-world example:* Going back to the school scenario, assume a prankster got hold of a student's ID number (hash) but did not know whose it was (the original input). If the hash function used to create the ID number has pre-image resistance, the prankster cannot figure out the student's name based on his ID number.
					  
					  3) **Second Pre-image Resistance**
					  
					  This property is a variation of Pre-image resistance. Given an input m1, it should be hard to find a different input m2 such that H(m1) = H(m2).
					  
					  *Real-world example:* Back to the school scenario, if a bad student knew a good student's name (m1) and his corresponding ID number (H(m1)), and he wants to forge a new name (m2) that generates the original good student's ID number, he will not be able to do so if the hash function has second pre-image resistance.
					  
					  Essentially, the differences are:
			- Collision resistance prevents anyone from finding two different inputs with the same hash.
			- Pre-image resistance prevents someone from working backwards from a hash to find the input.
			- Second pre-image resistance prevents someone from altering a known input (without changing its hash) to produce a different input with the same hash.
		- ### Additional Security Considerations
			- A hash function must be used *reasonably*.
			  logseq.order-list-type:: number
				- Hashing the world `yes` or `no` is unreasonable. It would be very easy to recreate the conditions needed to produce the exact same digest produced using such values.
				  logseq.order-list-type:: number
			- The size of the digest **does matter**.
			  logseq.order-list-type:: number
				- All cryptographic algorithms must care about the size of their parameters in practice
				  logseq.order-list-type:: number
- ## Book Anotations
-
-
-
- ## Glossary
	- **uniform distribution**
	  logseq.order-list-type:: number
		- Scenario: 
		  logseq.order-list-type:: number
			- we have a hash function that produces outputs of length 2 bits in a uniformly random fashion.
			  logseq.order-list-type:: number
		- Outcome:
		  logseq.order-list-type:: number
			- **It will output 00 25% of the time, 01 25% of the time, and so on**. In turn, not much work is needed to produce a collision.
			  logseq.order-list-type:: number
	- Exclusive OR (**XOR**)
	  logseq.order-list-type:: number
		- [[XOR Notes]]
		  logseq.order-list-type:: number
		- logseq.order-list-type:: number