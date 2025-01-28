## AnteHandler
	- # AnteDecorator Queue Management System
	- ## Main Purpose
		- The AnteDecorator controls message flow in the Swingset system.
	- ## How it Works
		- It monitors the size of incoming messages
		- Messages go into two queues:
		- Action Queue
		- High Priority Queue
	- ## Queue Management
		- There is a size limit for the Action Queue
		- High Priority Queue items:
		- Skip the size checks
		- Still count in total queue size
		- Can make the queue exceed limits
	- ## Message Processing Steps
		- Messages enter through CheckTx (mempool stage)
		  logseq.order-list-type:: number
		- Messages process through DeliverTx (execution stage)
		  logseq.order-list-type:: number
	- ## Challenge
	- The system can't know at CheckTx:
		- How many messages will be allowed later
		- The size of previous messages in the block
	- ## Solution
	- The system uses a safety buffer:
		- Mempool admission uses a lower limit (around 50%)
		- This creates a hysteresis effect
		- Full queue capacity is available at execution time
		  
		  This design helps prevent queue overflow while maintaining system efficiency.
	-
-
-
- ## Trae
	- I understand you want to create a performance testing script that measures both wallet provisioning and claim transaction times. Here's how we can structure this:
	  
	  ```javascript:/Users/tgreco/dapp-airdrop/nodes-cli-analysis/performance-test.js
	  import { exec } from 'node:child_process';
	  import { promisify } from 'node:util';
	  import { merkleTreeAPI } from '../ui/src/airdrop-data/merkle-tree/index.js';
	  import { TEST_ACCOUNTS } from '../ui/src/generated_keys.js';
	  
	  const execAsync = promisify(exec);
	  
	  const delay = ms => new Promise(resolve => setTimeout(resolve, ms));
	  
	  /**
	   * @typedef {Object} PerformanceMetrics
	   * @property {number} provisionTime - Time taken for wallet provisioning
	   * @property {number} claimTime - Time taken for claim transaction
	   * @property {string} address - Account address
	   * @property {string} iteration - Test iteration number
	   * @property {Error} [error] - Error if any operation failed
	   */
	  
	  /**
	   * Executes a shell command and measures its execution time
	   * @param {string} command - Command to execute
	   * @returns {Promise<{duration: number, output: string}>}
	   */
	  async function measureCommand(command) {
	    const start = performance.now();
	    const { stdout, stderr } = await execAsync(command);
	    const duration = performance.now() - start;
	    return { duration, output: stdout };
	  }
	  
	  /**
	   * Creates the offer JSON for a claim transaction
	   * @param {Object} account - Account details
	   * @returns {string} Stringified offer JSON
	   */
	  function createOfferJSON(account) {
	    const offerBody = {
	      id: `${account.address}-claim-attempt`,
	      invitationSpec: {
	        source: 'contract',
	        instancePath: ['tribblesAirdrop2'],
	        callPipe: [['makeClaimTokensInvitation']],
	        offerArgs: {
	          key: account.pubkey.key,
	          proof: merkleTreeAPI.generateMerkleProof(account.pubkey.key, TEST_ACCOUNTS.map(a => a.pubkey.key)),
	          address: account.address,
	          tier: Math.floor(Math.random() * 5)
	        },
	        proposal: {
	          give: { Fee: { brand: 'IST', value: 5n } }
	        }
	      }
	    };
	    return JSON.stringify(offerBody);
	  }
	  
	  /**
	   * Processes a batch of accounts
	   * @param {Array} accounts - Array of accounts to process
	   * @param {number} iteration - Current iteration number
	   * @returns {Promise<PerformanceMetrics[]>}
	   */
	  async function processBatch(accounts, iteration) {
	    const metrics = [];
	  
	    for (const account of accounts) {
	      try {
	        // Provision wallet
	        const provisionCommand = `agoric wallet provision --account ${account.address} --nickname test-${iteration}-${account.address} --keyring-backend test --spend`;
	        const { duration: provisionTime } = await measureCommand(provisionCommand);
	  
	        // Create and execute claim transaction
	        const offerJSON = createOfferJSON(account);
	        const claimCommand = `agoric wallet --keyring-backend test send --offer '${offerJSON}'`;
	        const { duration: claimTime } = await measureCommand(claimCommand);
	  
	        metrics.push({
	          provisionTime,
	          claimTime,
	          address: account.address,
	          iteration: `${iteration}`,
	        });
	      } catch (error) {
	        metrics.push({
	          provisionTime: 0,
	          claimTime: 0,
	          address: account.address,
	          iteration: `${iteration}`,
	          error,
	        });
	      }
	    }
	  
	    return metrics;
	  }
	  
	  /**
	   * Main test execution function
	   */
	  async function runPerformanceTest() {
	    const BATCH_SIZE = 5;
	    const ITERATIONS = 100;
	    const DELAY_BETWEEN_BATCHES = 5000; // 5 seconds
	    const allMetrics = [];
	  
	    for (let i = 0; i < ITERATIONS; i++) {
	      console.log(`Starting iteration ${i + 1}/${ITERATIONS}`);
	      
	      // Select 5 accounts for this batch
	      const batchAccounts = TEST_ACCOUNTS.slice(i * BATCH_SIZE, (i + 1) * BATCH_SIZE);
	      
	      const batchMetrics = await processBatch(batchAccounts, i + 1);
	      allMetrics.push(...batchMetrics);
	      
	      // Write metrics to file after each batch
	      const metricsJSON = JSON.stringify(allMetrics, null, 2);
	      await fs.promises.writeFile('performance-metrics.json', metricsJSON);
	      
	      // Wait 5 seconds before next batch
	      await delay(DELAY_BETWEEN_BATCHES);
	    }
	  
	    console.log('Performance test completed');
	    return allMetrics;
	  }
	  
	  // Run the test
	  runPerformanceTest().catch(console.error);
	  ```
	  
	  This script:
	  1. Processes accounts in batches of 5
	  2. For each account:
		- Measures wallet provisioning time
		- Measures claim transaction time
		- Records both metrics
		  3. Waits 5 seconds between batches
		  4. Runs for 100 iterations
		  5. Saves metrics to a JSON file
		  
		  The metrics include:
	- Provisioning time
	- Claim transaction time
	- Account address
	- Iteration number
	- Any errors that occurred
	  
	  You can run this with:
	  ```bash
	  node performance-test.js
	  ```
	  
	  The script will create a `performance-metrics.json` file with detailed timing data that you can analyze later.
	  
	  Would you like me to add any additional metrics or modify the test parameters?
	-
-
- # Zoe Invitation Types
	- ## `AgoricContractInvitationSpec`
		- **Source Type:** 'agoricContract'
		- **Description:** Invitation source is a chain of calls starting with an agoricName
		- **Key Components:**
			- Starts with instancePath lookup within agoricNames
			- Uses callPipe for executing calls on preceding results
			- Final result expected to return an Invitation
		- ```typescript
		  /**
		   * source of invitation is a chain of calls starting with an agoricName
		   * - the start of the pipe is a lookup of instancePath within agoricNames
		   * - each entry in the callPipe executes a call on the preceding result
		   * - the end of the pipe is expected to return an Invitation
		   */
		  export type AgoricContractInvitationSpec = {
		      source: 'agoricContract';
		      instancePath: string[];
		      callPipe: Array<[methodName: string, methodArgs?: any[]]>;
		  };
		  ```
	- ## `ContractInvitationSpec`
		- **Source Type:** 'contract'
		- **Description:** Source is a contract that takes an Instance to look up in zoe
		- **Key Components:**
			- Requires Instance
			- Uses publicInvitationMaker
			- Optional invitationArgs
			- ```typescript
			  /**
			    * source is a contract (in which case this takes an Instance to look up in zoe)
			    */
			  export type ContractInvitationSpec = {
			    source: 'contract';
			    instance: Instance;
			    publicInvitationMaker: string;
			    invitationArgs?: any[];
			  };
			  ```
	- ## `PurseInvitationSpec`
		- **Source Type:** 'purse'
		- **Description:** Invitation is already in Zoe "invitation" purse requiring query
		- **Key Components:**
			- Uses find/query invitation by kvs mechanism
			- Requires Instance and description
		- ```typescript
		  /**
		   * the invitation is already in your Zoe "invitation" purse so we need to query it
		   * - use the find/query invitation by kvs thing
		   */
		  export type PurseInvitationSpec = {
		      source: 'purse';
		      instance: Instance;
		      description: string;
		  };
		  
		  
		  ```
	- ## ContinuingInvitationSpec
		- **Source Type:** 'continuing'
		- **Description:** Continuing invitation where offer result from previous invitation had invitationMakers property
		- **Key Components:**
			- References previousOffer
			- Requires invitationMakerName
			- Optional invitationArgs
		- ```typescript
		  
		  /**
		   * source is a contract (in which case this takes an Instance to look up in zoe)
		   */
		  export type ContractInvitationSpec = {
		      source: 'contract';
		      instance: Instance;
		      publicInvitationMaker: string;
		      invitationArgs?: any[];
		  };
		  ```
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