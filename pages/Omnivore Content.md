- [Horton's “Who Done It?”](https://omnivore.app/me/http-www-erights-org-elib-capability-horton-paper-18c28d56e7f)
  collapsed:: true
  site:: [erights.org](http://www.erights.org/elib/capability/horton/paper/)
  author:: Mark S. Miller
  date-saved:: [[12/01/2023]]
	- ### Highlights
	  collapsed:: true
		- > There are two approaches to protect users from the harm programs can cause, _proactive control_ and _reactive control_. Proactive controls help prevent bad things from happening, or limit the damage when they do. [⤴️](https://omnivore.app/me/http-www-erights-org-elib-capability-horton-paper-18c28d56e7f#053f1782-4d68-42c0-b072-4aabe464c243)
		- > For reactive controls to work well, we must first proactively limit attackers to causing only repairable damage. [⤴️](https://omnivore.app/me/http-www-erights-org-elib-capability-horton-paper-18c28d56e7f#08209e04-daec-42bd-8c0f-d71617579ef5)
		- > Access Control List (ACL) systems support reactive control directly. [⤴️](https://omnivore.app/me/http-www-erights-org-elib-capability-horton-paper-18c28d56e7f#0f89e23e-f6cc-425a-be4a-204af5bb1c1f)
		- > ACL systems presume a program acts on behalf of its “user”. Access is allowed or disallowed by checking whether this operation on this resource is permitted for this user. [⤴️](https://omnivore.app/me/http-www-erights-org-elib-capability-horton-paper-18c28d56e7f#dbe2f850-d439-4cfe-a1fc-e6736790f89e)
- [How To Deconstruct Almost Anything](https://omnivore.app/me/http-www-fudco-com-chip-deconstr-html-18c27847e0f)
  collapsed:: true
  site:: [fudco.com](http://www.fudco.com/chip/deconstr.html)
  date-saved:: [[12/01/2023]]
- [messari.io](https://omnivore.app/me/https-messari-io-report-crypto-fundraising-brief-november-2023-r-18c2233dee2)
  collapsed:: true
  site:: [messari.io](https://messari.io/report/crypto-fundraising-brief-november-2023?referrer=research-reports)
  date-saved:: [[11/30/2023]]
- [What Do You Do About Externalities?](https://omnivore.app/me/https-capitalgains-thediff-co-p-externalities-18c1c8f2f51)
  collapsed:: true
  site:: [Capital Gains](https://capitalgains.thediff.co/p/externalities)
  author:: Byrne Hobart
  date-saved:: [[11/29/2023]]
  date-published:: [[11/29/2023]]
- [Playing with ActivityPub - macwright.com](https://omnivore.app/me/https-macwright-com-2022-12-09-activitypub-html-18c18a4aa6f)
  site:: [macwright.com](https://macwright.com/2022/12/09/activitypub.html)
  author:: Tom MacWright
  date-saved:: [[11/28/2023]]
  date-published:: [[12/08/2022]]
  collapsed:: true
	- ### Highlights
	  collapsed:: true
		- > ActivityPub is the kind of specification that’s so generic that everything implemented on top of it is a particular “flavor” of the specification. There’s an opinionated kind of ActivityPub that Mastodon speaks, which is different from [bookwyrm](https://bookwyrm.social/) or [pixelfed](https://pixelfed.org/). [⤴️](https://omnivore.app/me/https-macwright-com-2022-12-09-activitypub-html-18c18a4aa6f#8ffd756a-8fb7-4b74-9ffd-bab351067f09)
		- > There are many Mastodon hosts and they don’t share any kind of cache so popular posts themselves [have been known to DDoS websites](https://www.jwz.org/blog/2022/11/mastodon-stampede/). [⤴️](https://omnivore.app/me/https-macwright-com-2022-12-09-activitypub-html-18c18a4aa6f#321f3c5d-1e33-481e-be6a-fbee8c6a696e)
- [Locke Lord QuickStudy: Down the Rabbit Hole: Digital Assets in ‎International Arbitration | News, blogs & events | Locke Lord](https://omnivore.app/me/locke-lord-quick-study-down-the-rabbit-hole-digital-assets-in-in-18c0f8d5c31)
  collapsed:: true
  site:: [lockelord.com](https://www.lockelord.com/newsandevents/publications/2022/01/down-the-rabbit-hole)
  date-saved:: [[11/27/2023]]
  date-published:: [[01/27/2022]]
- [Projects We Love: ZKorum – Fission](https://omnivore.app/me/https-fission-codes-blog-projects-we-love-zkorum-18c09ba5698)
  collapsed:: true
  site:: [Fission](https://fission.codes/blog/projects-we-love-zkorum/)
  date-saved:: [[11/25/2023]]
- [How DAT discovers peers](https://omnivore.app/me/super-high-level-18c07ef66d7)
  collapsed:: true
  site:: [blog.mauve.moe](https://blog.mauve.moe/posts/how-dat-discovers-peers)
  labels:: [[DAT]] [[NAT Traversal]] [[DWeb]] [[P2P]]
  date-saved:: [[11/25/2023]]
	- > the DHT can yield false positives and requires participating in a network. To make peer lookup faster, discovery-channel makes use of [dns-discovery](https://github.com/mafintosh/dns-discovery) to talk to a list of centralized DNS servers to find peers. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#a5819326-1e8f-4c83-93c5-443b700ead57)
	- > One of the problems with the DHT, is that items in the DHT don't expire right away, and searching for peers can yield a lot of false-positives. Also, maintining the connection information necessary for staying in the DHT takes up more computational resources. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6318228e-b9d2-453f-a209-0e7c0efbcdb5)
	- > the DHT requires a set of _"bootstrap nodes"_ which are used to find more nodes to start building up your network. These bootstrap nodes are a potential source of failure and DHT clients should attempt to save any nodes they find for later use in order to have a way to bootstrap should the bootstrap nodes go down.
	  > 
	  ##  [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#32b8b315-c354-4919-aff6-6543a5ac9e9b)
	- > discovery-channel uses this module by publishing it's IP/Port combination to the discovery key onto the DHT. Since there is no central authority or single point of failure, publishing on the DHT is resistant to censorship and can survive sketchy networks. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#8788de8e-a0b9-45d3-a933-13cd8eff61bd) 
	  
	  note:: However, this doesn't come without it's own set of problems.
	- > the idea is that each discovery key is sent to nodes whose id is _"close"_ to the key, and peers maintain connections to others that are varying levels of _"closeness"_ to themselves which makes it fast to find the nodes storing keys that you want. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#01e4df3d-e28e-43f9-b4fc-5c7949bc6b23)
	- > The solution for this problem was to get rid of centralized trackers and replace them with a protocol that would split the discovery information amongst all the peers participating in the network. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6a1534d6-8937-4d6f-add5-7f3785b473c7)
	- > Dat isn't the only p2p protocol that relies on peer discovery. Back when applications like BitTorrent and eMule were being developed, they relied a lot on "tracker" servers for announcing peers and searching for them. This resulted in a form of centralization which meant that if a tracker server got taken down or otherwise compromised, the p2p network couldn't function. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6bb22760-03ff-45f0-a86f-9112c11db11e) 
	  
	  note:: Sets the stage for how Kademlia is used to handle routing within distributed systems.
	- > dat-swarm provides a way to join multiple networks and automatically connect to peers, [discovery-channel](https://github.com/maxogden/discovery-channel) is the module responsible for actually finding peers for a given key. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#90b639cb-6e79-4342-8f2e-9496c0d5c741) 
	  
	  note:: This should be a logseq flashcard
	- > discovery-swarm provides an API to **join** networks using a **discovery key**, and then invokes a callback when it finds a peer to connect to.
	  > 
	  ```javascript
	  > 
	  ``` [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#393bae75-892c-49bb-8eb5-5e6cda354b6a)
	- > A lot of the magic from the dat ecosystem comes from [discovery-swarm](https://github.com/mafintosh/discovery-swarm), the module that's responsible for finding peers to replicate with in the dat network. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#1b5f7c6f-a174-4b1d-b33b-fdf2b3c1fe81)
	- ### Highlights
	  collapsed:: true
		- > A lot of the magic from the dat ecosystem comes from [discovery-swarm](https://github.com/mafintosh/discovery-swarm), the module that's responsible for finding peers to replicate with in the dat network. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#1b5f7c6f-a174-4b1d-b33b-fdf2b3c1fe81)
		- > discovery-swarm provides an API to **join** networks using a **discovery key**, and then invokes a callback when it finds a peer to connect to.
		  > 
		  ```javascript
		  > 
		  ``` [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#393bae75-892c-49bb-8eb5-5e6cda354b6a)
		- > dat-swarm provides a way to join multiple networks and automatically connect to peers, [discovery-channel](https://github.com/maxogden/discovery-channel) is the module responsible for actually finding peers for a given key. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#90b639cb-6e79-4342-8f2e-9496c0d5c741) 
		  
		  note:: This should be a logseq flashcard
		- > Dat isn't the only p2p protocol that relies on peer discovery. Back when applications like BitTorrent and eMule were being developed, they relied a lot on "tracker" servers for announcing peers and searching for them. This resulted in a form of centralization which meant that if a tracker server got taken down or otherwise compromised, the p2p network couldn't function. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6bb22760-03ff-45f0-a86f-9112c11db11e) 
		  
		  note:: Sets the stage for how Kademlia is used to handle routing within distributed systems.
		- > The solution for this problem was to get rid of centralized trackers and replace them with a protocol that would split the discovery information amongst all the peers participating in the network. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6a1534d6-8937-4d6f-add5-7f3785b473c7)
		- > the idea is that each discovery key is sent to nodes whose id is _"close"_ to the key, and peers maintain connections to others that are varying levels of _"closeness"_ to themselves which makes it fast to find the nodes storing keys that you want. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#01e4df3d-e28e-43f9-b4fc-5c7949bc6b23)
		- > discovery-channel uses this module by publishing it's IP/Port combination to the discovery key onto the DHT. Since there is no central authority or single point of failure, publishing on the DHT is resistant to censorship and can survive sketchy networks. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#8788de8e-a0b9-45d3-a933-13cd8eff61bd) 
		  
		  note:: However, this doesn't come without it's own set of problems.
		- > the DHT requires a set of _"bootstrap nodes"_ which are used to find more nodes to start building up your network. These bootstrap nodes are a potential source of failure and DHT clients should attempt to save any nodes they find for later use in order to have a way to bootstrap should the bootstrap nodes go down.
		  > 
		  ##  [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#32b8b315-c354-4919-aff6-6543a5ac9e9b)
		- > One of the problems with the DHT, is that items in the DHT don't expire right away, and searching for peers can yield a lot of false-positives. Also, maintining the connection information necessary for staying in the DHT takes up more computational resources. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6318228e-b9d2-453f-a209-0e7c0efbcdb5)
		- > the DHT can yield false positives and requires participating in a network. To make peer lookup faster, discovery-channel makes use of [dns-discovery](https://github.com/mafintosh/dns-discovery) to talk to a list of centralized DNS servers to find peers. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#a5819326-1e8f-4c83-93c5-443b700ead57)
- [How NAT traversal works · Tailscale](https://omnivore.app/me/how-nat-traversal-works-tailscale-18c04f162ce)
  collapsed:: true
  site:: [Tailscale](https://tailscale.com/blog/how-nat-traversal-works/)
  author:: Tailscale
  date-saved:: [[11/25/2023]]
  date-published:: [[08/20/2020]]
- [Joe Duffy - Asynchronous Everything](https://omnivore.app/me/https-joeduffyblog-com-2015-11-19-asynchronous-everything-18bf4c3ff18)
  collapsed:: true
  site:: [joeduffyblog.com](https://joeduffyblog.com/2015/11/19/asynchronous-everything/)
  author:: Joe Duffy
  date-saved:: [[11/21/2023]]
  date-published:: [[11/18/2015]]
- [Safe asset shortage and collateral reuse](https://omnivore.app/me/1813291853-18bff8ea982)
  collapsed:: true
  site:: [econstor.eu](https://www.econstor.eu/bitstream/10419/262212/1/1813291853.pdf)
  author:: Jank, Stephan; Mönch, Emanuel; Schneider, Michael
  date-saved:: [[11/23/2023]]
- [Blockchain Technology and Rehypothecation in the Repo Markets](https://omnivore.app/me/https-agoric-com-blog-guest-post-blockchain-technology-and-rehyp-18bff4c7925)
  collapsed:: true
  site:: [agoric.com](https://agoric.com/blog/guest-post/blockchain-technology-and-rehypothecation-in-the-repo-markets)
  date-saved:: [[11/23/2023]]
- [prof-frisby-notes.md](https://omnivore.app/me/u-0-b-89727-a-89-d-7-11-ee-866-d-57-afe-3-d-7-c-3-d-5-prof-frisb-18bfb3646df)
  collapsed:: true
  site:: [omnivore.app](https://omnivore.app/attachments/u/0b89727a-89d7-11ee-866d-57afe3d7c3d5/prof-frisby-notesmd.pdf)
  date-saved:: [[11/23/2023]]
- [A Scheme Primer](https://omnivore.app/me/a-scheme-primer-18bfa8a865d)
  collapsed:: true
  site:: [spritely.institute](https://spritely.institute/static/papers/scheme-primer.html)
  author:: Christine Lemmer-Webber and the Spritely Institute
  labels:: [[scheme, racket, spritely]] [[scheme]] [[racket]] [[spritely]]
  date-saved:: [[11/23/2023]]
  date-published:: [[07/27/2022]]
- [Classics in the History of Psychology -- Miller (1956)](https://omnivore.app/me/classics-in-the-history-of-psychology-miller-1956-18bf946baf6)
  collapsed:: true
  site:: [psychclassics.yorku.ca](https://psychclassics.yorku.ca/Miller/)
  author:: Dan J. Denis
  date-saved:: [[11/22/2023]]
- [Tech Tuesday - The Turing Machine](https://omnivore.app/me/tech-tuesday-the-turing-machine-18bf9322afd)
  collapsed:: true
  site:: [Mostly Harmless Ideas](https://blog.apiad.net/p/tech-tuesday-the-turing-machine)
  author:: Alejandro Piad Morffis
  date-saved:: [[11/22/2023]]
  date-published:: [[10/24/2023]]
- [Good-To-Know Dev Terms | Developer Portal](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2)
  collapsed:: true
  site:: [tutorials.cosmos.network](https://tutorials.cosmos.network/tutorials/1-tech-terms/)
  labels:: [[cosmos-developers]] [[cosmos-sdk]]
  date-saved:: [[11/18/2023]]
	- ### Highlights
	  collapsed:: true
		- > The [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#ea7d96a2-0872-468d-a5e7-eb93d2dc3c03)
		- > The terms "Cosmos", "interchain ecosystem", and "interchain" can be understood as synonymous. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#65baa980-992a-4ecf-a1ed-5de7b858b678)
		- > Light clients do not track the entire state of a blockchain and also do not contain every transaction/block of a chain. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#b3a7ee77-12ad-4610-be40-85653b48afd2)
		- > In the Tendermint consensus, the light client protocol allows clients to benefit from the same degree of security that full nodes benefit from, while bandwidth requirements are minimized. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#2eb41bb6-ffb6-410d-bc20-403791caf0c8)
		- > A client can receive cryptographic proofs for blockchain states and transactions without having to sync all blocks or even their headers. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#8a7b4160-3900-431d-96eb-dac36d91920a)
		- > The **light client daemon (LCD)** is an HTTP1.1 server exposed by the Cosmos SDK, and its default port is `1317` [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#76fc156d-22a5-49f5-ad33-603b3e6829a5)
		- > Why [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#56d23678-10df-4010-97cb-fb92208ab794)
		- > is [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#c05bab9f-213a-4372-81f9-275f9adb1268)
		- > daemon? [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#9c43ef99-8852-47ed-8621-0118fe56bdc7) 
		  
		  note:: Before SDK v0.40, to get a REST API it was necessary to run another backend service (or daemon (opens new window), a term inherited from Unix), for example using gaiacli rest-server --laddr 0.0.0.0:1317 --node localhost:26657. In Cosmos SDK v0.40, REST was moved inside the node service making it part of the Cosmos SDK, but the term "daemon" stuck, leading to the name light client daemon (LCD).
		- > A **remote procedure call (RPC)** is _a software communication protocol_. The term is often found in distributed computing because RPC is a technique to realize inter-process communication (IPC) by allowing a program to cause a subroutine procedure that is executed in a different address space (a different machine). [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#97d78e48-1be4-4770-a85e-b499000f2fe8)
		- > RPC can be understood as a client-server interaction in which the "caller" is the client, more specifically the requesting program, and the "executor" is the server, more specifically the service-providing program. The interaction is implemented through a request-response message-passing system. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#e620597d-c9a8-4e1d-8e1f-d4af4d696665)
		- > RPC allows calling functions in different address spaces. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#20c1f91f-71dd-481b-99a3-f123612ce657)
		- > However, with [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#f8cfd87d-427c-43ec-9a76-8e64afbab633)
		- > However, with RPC, the developer codes as if the subroutine would be local; the developer does not have to code in details for remote interaction. Thus, with RPCs it is implied that all calling procedures are basically the same, independent of them being local or remote calls. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#e02a5496-abf5-4d30-8c37-0a586e91bc29)
		- > The client has a stub that interfaces with the remote procedure, while the server has a stub to interface with the original request procedure. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#eb4bdb48-c75d-4f94-84ff-b28071dbdefc)
		- > client calls a client stub - a piece of code converting parameters that are passed between client and servers during an RPC. The call is a local procedure call. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#39b3c3d7-0f15-46f9-8b36-26d8fea761e4)
		- > The server also has a stub to interface with the remote procedure.
		  > 
		  1. The client stub packs the procedure parameters into a message. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#f9f5e907-651e-4e63-8ac9-c4a3a0e5e0cd)
		- > Packing procedure parameters is called **marshaling**. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#7f587fb5-b4f3-4a0a-b434-8880db1d9c09)
		- > 1. The client stub then makes a system call to send the message.
		  2. The client's local operating system (OS) sends the message from the client (machine A) to the server (machine B) through the corresponding transport layers.
		  3. The server OS passes the incoming packets to the server stub.
		  4. The server stub unpacks the message and with it the included procedure parameters - this is called **unmarshaling**.
		  5. The server stub calls a server procedure and the procedure is executed.
		  6. Once the procedure is finalized, the output is returned to the server stub.
		  7. The server stub packs the return values into a message.
		  8. The message is sent to the transport layer, which sends the message to the client's transport layer.
		  9. The client stub unmarshals the return parameters and returns them to the original calling client.
		  > 
		  In [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#9f7fa211-18b1-4c32-a676-08341d0d09af)
		- > The Interchain Stack exposes both the CometBFT RPC and the LCD. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#88136f01-206b-445b-8866-4c0476608577)
		- > For each gRPC endpoint defined in a Protobuf `Query` service, the Cosmos SDK offers a corresponding REST endpoint. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#8f611351-4ccb-42ec-ae16-64dfcb51f9a9)
		- > **gRPC-gateway** is a tool to expose gRPC endpoints as REST endpoints. It helps provide APIs in gRPC and RESTful style, reads gRPC service definitions, and generates reverse-proxy servers that can translate a RESTful JSON API into gRPC. [⤴️](https://omnivore.app/me/good-to-know-dev-terms-developer-portal-18be10ae1b2#59203fa4-e79b-48f1-97ea-c2e478cbd4a4)