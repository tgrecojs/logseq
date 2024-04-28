## Deployment Notes
-
- `BootstrapPowers`
	- A `BootstrapPowers` object is composed of several *promise spaces*. A promise space is a `{ produce, consume }` pair where:
		- `consume[name]` is a promise associated with a specific name.
		- `produce[name].resolve(value)` resolves the promise associated with the same name by providing a value.
- #### Accessing Zoe
	- [placeholder for description of promise space in which Zoe exists]
-
- ## Strategy
	- **Deploy airdropCampagin and use an IST, Toy Atom, etc purse to mimic meme token.**
	- Focus 1 hour towards getting things setup.
	- Ask cooney for 30 mins to try and deploy this in AM.
	-
	- ### Two Contracts
		- tokenMint
		  logseq.order-list-type:: number
		- airdropCampaign
		  logseq.order-list-type:: number
	- ### tokenMint
		- Brand`  to `agoricNames.brand`
		- `Issuer` to `agoricNames.issuer`
	- airdropCampaign
		- doesn't need issuer-related powers.
		- timer
		- agoricNames
		- board
		- storage
- tags:: #[[Merkle Airdrop]]
-