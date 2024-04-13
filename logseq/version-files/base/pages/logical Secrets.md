## Abstract
	- Cryptograhy is only necessary for implementing authentication operations **in the abscense of a mutually trusted secure substrate.**
	- ## Work Plan
	- Implement a public-key system that **derives security from encapsulation rather than encryption.**
	- > Concurrent Logic Programming languages provide such a substrate.
	-
	- ### Intro
		- Two of the issues in security are secrecy and authentication.
		- Capability operating systems and languages with strict encapsulation already provide this for two-party interactions. Providing secure three-party communication is more demanding.
		- 3 Party Authentication
			- passing a message through to an untrusted intermediary
			- {{renderer :mermaid_mnj}}
				- ```mermaid
				  %%{init: { 'logLevel': 'debug', 'theme': 'forest' } }%%
				  
				  graph
				  MsgSend --> Untrusted_Intermediary 
				  Untrusted_Intermediary --> MsgRecipient
				  
				  ```
		- Three-party secrecy
			- is the ability to pass a message through an untrusted intermediary to an intended recipient in such a way that only the intended recipient may read the message.
		- Three party authentication
			- the ability to pass a message through an untrusted intermediary to an intended recipient so that the message contents cannot be faked while preserving the appearance of the origi¬ nal authorship. This corresponds to hu
		-
	-