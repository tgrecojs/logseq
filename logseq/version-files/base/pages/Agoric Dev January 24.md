## Zoe Invitation Types
	- # Invitation Types Analysis Notes
	- ## AgoricContractInvitationSpec
	  1. **Source Type:** 'agoricContract'
	  2. **Description:** Invitation source is a chain of calls starting with an agoricName
	  3. **Key Components:**
		- Starts with instancePath lookup within agoricNames
		- Uses callPipe for executing calls on preceding results
		- Final result expected to return an Invitation
	- ## ContractInvitationSpec
	  1. **Source Type:** 'contract'
	  2. **Description:** Source is a contract that takes an Instance to look up in zoe
	  3. **Key Components:**
		- Requires Instance
		- Uses publicInvitationMaker
		- Optional invitationArgs
	- ## PurseInvitationSpec
	  1. **Source Type:** 'purse'
	  2. **Description:** Invitation is already in Zoe "invitation" purse requiring query
	  3. **Key Components:**
		- Uses find/query invitation by kvs mechanism
		- Requires Instance and description
	- ## ContinuingInvitationSpec
	  1. **Source Type:** 'continuing'
	  2. **Description:** Continuing invitation where offer result from previous invitation had invitationMakers property
	  3. **Key Components:**
		- References previousOffer
		- Requires invitationMakerName
		- Optional invitationArgs
		  
		  *Would you like me to capture any additional details or organize these notes differently?*
-
-
	- In the Agoric codebase, "coalesce" generally refers to the process of combining or merging multiple state updates into a single, coherent state representation. Let me explain how it works in the context you've shared:
	  
	  1. In <mcfile name="wallet.js" path="/Users/tgreco/agoric-sdk/packages/agoric-cli/src/lib/wallet.js"></mcfile>, the <mcsymbol name="coalesceWalletState" filename="wallet.js" path="/Users/tgreco/agoric-sdk/packages/agoric-cli/src/lib/wallet.js" startline="111" type="function"></mcsymbol> function:
	- Takes a history of wallet updates (through the `follower` parameter)
	- Processes these updates in chronological order (oldest first)
		- Combines them into a single, final state representation
			- The process works like this:
				- ```javascript
				  // 1. Collect historical updates
				  const history = [];
				  for await (const followerElement of iterateReverse(follower)) {
				  // Skip errors
				  if ('error' in followerElement) {
				    continue;
				  }
				  history.push(followerElement.value);
				  }
				  // 2. Apply updates in chronological order
				  
				  const coalescer = makeWalletStateCoalescer(invitationBrand);
				  for (const record of history.reverse()) {
				  coalescer.update(record);
				  }
				  ```
			- This coalesced state is important because:
				- The wallet maintains a history of all changes
					- To know the current state, you need to apply all these changes in order.
					- Coalescing gives you the final result of applying all updates
						- You can see this in action in the <mcfile name="wallet.js" path="/Users/tgreco/agoric-sdk/packages/agoric-cli/src/commands/wallet.js"></mcfile> `show` command, where both the coalesced state and current state are used to generate a summary:
							- ```javascript
							  const coalesced = await coalesceWalletState(follower);
							  const current = await getCurrent(opts.from, { readLatestHead });
							  const summary = summarize(current, coalesced, agoricNames);
							  ```
						- The comment in <mcfile name="inter.js" path="/Users/tgreco/agoric-sdk/packages/agoric-cli/src/commands/inter.js"></mcfile> `// coalesceWalletState should do this` suggests that the hardening of offer statuses should ideally happen during the coalescing process, rather than at the point of use.
						- Think of coalescing like maintaining a checkbook:
						- Each transaction is an update
					- Instead of looking at every transaction to know your balance
						- You can "coalesce" them into a final balance that represents the end state
						- This final state incorporates all the individual changes that happened over time
- tags:: #[[agoric-sdk]], [[Merkle Airdrop]], [[Zoe]]
- id:: 67946fc8-2826-4a5c-a4f9-e042e1c927b9