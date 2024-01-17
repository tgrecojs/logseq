-
- # January 12th Meeting
- ### Proposal, Evaluation Tool, Developer Roles, Issuer, Admin, Creator
  
  Patrick and Thomas discussed a proposal, the details of which were not clearly stated. They also talked about an evaluation tool and its potential usefulness. They touched upon the role of developers and the possibility of auto-generating some aspects of the system. They also discussed the role of an issuer and the creation of a brand. Towards the end, they mentioned the roles of an admin and a creator in the system. However, the conversation was not very clear, making it difficult to provide a precise summary.
- ### Token Deposit and Airdrop Process Discussion
  
  Thomas and Patrick engaged in a technical conversation about a process involving token deposits and airdrops. Thomas explained the mechanics of the process, including the use of an issuer kit to create it and the handling of deposit airdrop payments. They also discussed the implementation of the code, acknowledging some potential issues. Thomas confirmed that the deposit pass facet and the air dropped purch were functioning as expected. Patrick showed agreement and raised a question about the possibility of incorporating certain elements into the existing code.
- ### Creation and Operations of a Deposit Asset
  
  Thomas and Patrick discussed the creation of a deposit asset and the associated operations. Thomas detailed the process of updating the signature and the use of the Merkel proof or tree route. They also discussed the inclusion proof and the role of the wallet address. Thomas explained the hashing process of the address and amount for each node in the tree, and how these hashes are combined to create the Merkel root. They concluded that the proof and inclusion proof would be provided as arguments to the claim method, with the wallet address potentially determinable from the proof.
- ### Token Claim Process Discussed
  
  Thomas and Patrick discussed a process related to tokens and wallets. They explored how to send tokens to a wallet and what an invitation to claim tokens looks like. Patrick explained that users need a smart wallet to claim an invitation and that the first claim cannot be made without one. They also discussed the possibility of checking if a user has a smart wallet before delivering an invitation. The discussion concluded without a clear decision on the next steps.
- ### Smart Contract Interaction and Token Distribution
  
  Thomas and Patrick discussed the process of interacting with a Zoe smart contract. Thomas explained that a transaction needs to be signed and Patrick clarified that the message "wallet action" needs to be called. They also discussed the possibility of public facet claim interactions. Thomas mentioned that Alice would receive a token and Curse would hold the tokens. They also touched on the topic of verification, with Thomas noting that verification isn't currently in place. They ended the discussion by agreeing on the need to send something to a specific address and to check if the provided address matched the one in the proof.
- ### Workload and Proof Validation Discussion
  
  Thomas and Patrick discussed the potential unnecessary work involved in creating a clearer flow. Patrick suggested it might not be a crazy amount of work. Thomas then introduced a hash function and expressed their desire to get comfortable with passing off a reference to a function. They also touched on the topic of powers, with Thomas highlighting the possibility of someone not holding a valid proof. Patrick agreed with Thomas's points.
- ### Smart Wall Limitations and Future Changes
  
  Patrick and Thomas discussed the limitations of Smart Wall, a system for handling claims. Patrick explained that public facets must be invitation makers and cannot be read-only methods, as read methods are not free. They also mentioned a convention where public facets must return a facet with specific keys. They also touched on potential future changes to Smart Wall, noting that everyone is waiting for changes that could potentially remove these limitations.
- ### Call Pipe Discussion and Proposal Stage
  
  Patrick and Thomas discussed the intricacies of a call pipe and its role in their project. Thomas expressed confusion about the call pipe's function, which Patrick clarified, explaining its role in connecting different parts of their process. They also discussed the potential need for a simpler method for Thomas's team. Patrick emphasized that their method involves getting a collateral manager and passing its result into another operation. By the end of their discussion, they had reached a proposal stage, with Thomas agreeing to investigate further.
- tags:: [[Merkle Airdrop]]
-
-