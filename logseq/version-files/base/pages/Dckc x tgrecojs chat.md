- Here is a summary of the transcription of the conversation with Dan Connelly on April 13th, focusing on technical discussions about software development practices:
  
  **Topics Discussed**:
  
  1. **Exo Objects**:
	- The discussion centered around the use of Exo objects and particularly the use of the distribution schedule with them.
	- A point of contention was the unnecessary passing of the distribution schedule as an argument to the Exo object's initialization state function, despite a valid reference already existing within the wider lexical scope of the contract.
	- It was identified that passing the distribution schedule argument was an anti-pattern since a valid reference was already available in scope.
	- The takeaway: Reference in-scope variables directly instead of passing them as arguments when they're already accessible within the same scope.
	-
	- **Timers**:
		- The use of absolute vs. relative times in relation to the timers was an area of confusion.
		- The issue arose because the timers were set up at the start of the contract but were being interpreted as if they denoted relative times.
		- The agreed testing method is to create a timer that replicates the expected behavior when the code is running on the blockchain.
	- **Prepare Airdrop Function**:
		- 'prepare_airdrop' function will be called within the context of the token mint contract upon the contract's initiation, which was acknowledged as a valid approach.
	- **State Machines**:
		- The current implementation of state machines was discussed, highlighting its lack of flexibility due to the rigid state transitions.
		- The current design does not easily allow for changes to state transitions or the valid states.
		- The consensus was to keep the state machine simple and inline to expedite deployment without over-engineering it.
	- **Deployment**:
		- A strong emphasis on the urgency to finalize decisions and move forward with getting the software deployed onto the blockchain.
	- ### **Action Items**:
	- Refactor the Exo object implementation to utilize the in-scope reference to the distribution schedule directly.
	  logseq.order-list-type:: number
	- Create a representative timer to test the relative time functionality, ensuring it works correctly on-chain.
	  logseq.order-list-type:: number
	- Validate the 'prepare_airdrop' function as part of the token mint contract initiation process.
	  logseq.order-list-type:: number
	- Construct the state machine in a straightforward, inline manner to allow for a timely deployment.
	  logseq.order-list-type:: number
	- Finalize all decisions regarding the software's design and functionality to prepare for deployment on the blockchain.
	  logseq.order-list-type:: number
	  
	  If you have any questions or need further clarifications, feel free to ask.
- 2. **Anti-Patterns**:
-
-