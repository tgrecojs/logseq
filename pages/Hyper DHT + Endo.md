## Questions
	- I have a runlet that is creating a new server using hyper-dht. I'm wonering
		- Is there a specific port I should be creating instances on? It looks the DHT constructor takes a UDX option.
		- ```js
		  {
		    // A list of bootstrap nodes
		    bootstrap: [ 'bootstrap-node.com:24242', ... ],
		    // Optionally pass in array of { host, port } to add to the routing table if you know any peers
		    nodes: [{ host, port }, ...],
		    // Optionally pass a port you prefer to bind to instead of a random one
		    port: 0,
		    // Optionally pass a host you prefer to bind to instead of all networks
		    host: '0.0.0.0',
		    // Optionally pass a UDX instance on which sockets will be created.
		    udx,
		    // dht-rpc will automatically detect if you are firewalled. If you know that you are not set this to false
		    firewalled: true
		  }
		  
		  ```
	- Host property
		- It looks like theres an opportunity to have an EndoHost (?)
	-
	-
	- Process ID
		-