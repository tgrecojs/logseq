- An easy implementation of a Merkle tree for ERC20 pull-based airdrops could involve the following steps:
- Set up a smart contract on the Ethereum blockchain to manage the airdrop.  This contract should include functions for adding addresses to the airdrop list, generating the Merkle tree, and verifying inclusion of an 
  address in the airdrop list.
- Create an array of wallet addresses  eligible for the airdrop, and use a cryptographic library to generate  the hash of each address. These hashes will be the leaves of the Merkle  tree.
- Combine the hashes of each pair of leaves to create the next level of the tree. Repeat this process until only one hash remains, 
  which will be the Merkle root.
- Store the Merkle root on the blockchain, and make it available to users who want to verify their inclusion in the airdrop list.
- Users can use the smart contract's functions to verify their inclusion in the airdrop list by providing their address and the necessary Merkle proof. The smart contract can then use the provided data to verify the inclusion.
- Once the user's address is verified, the smart contract can transfer the airdrop tokens to the user's address.
- ### Instructions
	- Ok so the contract would expose a method for adding their wallet to the merkle tree. It could also expose a function for checking whether or not their wallet address exists within the current merkle tree.
	- Is there anything that the user needs to gain access to after they submit a wallet address? or is storing it all that's required.
	- Once they do so, they would receive back a facet for verifying that their address has been included in the merkle tree. Is this function for verifying
	-
-
- The users can use any ERC20 compatible wallet to interact with the smart contract and claim the tokens.
- It'sworth noting that this is a high-level overview of the process, and  specific implementation details may vary depending on the specific 
  requirements and use case.
- Here are some resources for reference that can be used to implement an ERC20 pull-based airdrop using a Merkle tree:
- A blog article from Hackernoon, titled "Evolution of Airdrop from Common Spam to the Merkle Tree" [(https://hackernoon.com/evolution-of-airdrop-from-common-spam-to-the-merkle-tree-30caa2344170)](https://hackernoon.com/evolution-of-airdrop-from-common-spam-to-the-merkle-tree-30caa2344170), provides an overview of how Merkle tree can be used for airdrops and its advantages.
- Another article from Medium, titled "Merkle Airdrop: One of the Best Airdrop Solution for Token Issues" [(https://medium.com/mochilab/merkle-airdrop-one-of-the-best-airdrop-solution-for-token-issues-e2279df1c5c1)](https://medium.com/mochilab/merkle-airdrop-one-of-the-best-airdrop-solution-for-token-issues-e2279df1c5c1), provides a detailed explanation of how to implement a Merkle tree-based airdrop.
- A Github repository, titled "ERC20 Airdrop" [(https://github.com/dappuniversity/erc20_airdrop)](https://github.com/dappuniversity/erc20_airdrop),
  contains sample code and instructions on how to implement a Merkle 
  tree-based airdrop using Solidity and JavaScript. These resources can 
  provide a good starting point for understanding how to implement an 
  ERC20 pull-based airdrop using a Merkle tree.