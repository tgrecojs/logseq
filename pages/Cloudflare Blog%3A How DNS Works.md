hypothesis-uri:: https://www.cloudflare.com/learning/dns/what-is-dns/
hypothesis-title:: What Is DNS? | How DNS Works | Cloudflare
hypothesis-naming-scheme:: 0.2.0

-
- 📌 DNS translates domain names to IP addresses so browsers can load Internet resources.
  hid:: 3EfpUh-GEe60VDc1yILjog
  updated:: 2023-07-11T01:04:17.608340+00:00
- 📌 DNS servers eliminate the need for humans to memorize IP addresses such as 192.168.1.1 (in IPv4), or more complex newer alphanumeric IP addresses such as 2400:cb00:2048:1::c629:d7a2 (in IPv6).
  hid:: 9EMqih-GEe6oVjdZ_RTxsw
  updated:: 2023-07-11T01:04:57.851502+00:00
- 📌 The process of DNS resolution involves converting a hostname (such as www.example.com) into a computer-friendly IP address (such as 192.168.1.1).
  hid:: EBhn1B-HEe6dgZ9PW28KXQ
  updated:: 2023-07-11T01:05:44.550571+00:00
- ### 4 types of DNS servers involved in loading a web page:
	- hid:: a-QW_B-IEe6d8NfgSiEoYw
	  updated:: 2023-07-11T01:15:28.048312+00:00
	- 1. DNS Recursor
		- The recursor can be thought of as a librarian who is asked to go find a particular book somewhere in a library.
		- designed to receive queries from client machines through applications such as web browsers.
		- Typically the recursor is then responsible for making additional requests in order to satisfy the client’s DNS query.
		  2. Root Server
		- 1st step in resolving human readable host names into IP addresses.
		- typically **serves as a reference to other, more specific locations.**
		- Can be thought of as an index in a library that points to different racks of books. 
		  3. TLD nameserver
		- The nameserver is the next step in the search for a specific IP address.
		- The nameservers **hosts the last portion of a hostname**
			- example.com - TLD server is "com"
			- example.xyz - TLD server is "xyz"
		- Can be thought of as **a specific rack of books.**
		  4. Authoritative nameserver
		- last stop on on the nameserver query.
		- translates (or resolves) a specific name into it's definition.
		- it will return the IP address for the requested hostname back to the DNS Recursor (the librarian) that made the initial request. #[[dns]] #[[resolver]] #[[TLD]]
- 📌 The recursive resolver is the computer that responds to a recursive request from a client and takes the time to track down the DNS record. It does this by making a series of requests until it reaches the authoritative DNS nameserver for the requested record (or times out or returns an error if no record is found).
  hid:: sqr2yB-IEe6ciCftvRFsXg
  updated:: 2023-07-11T01:17:26.792019+00:00
- 📌 recursive DNS resolvers do not always need to make multiple requests in order to track down the records needed to respond to a client; #[[DNS]] #[[caching]]
  hid:: 1TquQB-IEe65E9_oJjgs_A
  updated:: 2023-07-11T01:18:24.803324+00:00
  📝 ***caching** is a data persistence process that helps short-circuit the necessary requests by serving the requested resource record earlier in the DNS lookup.*
- 📌 an authoritative DNS server is a server that actually holds, and is responsible for, DNS resource records. #[[DNS]]
	- this is the server at the bottom of the DNS lookup chain.
- ### Authoritative
	- has total control.
	- holds the ability to provide (or restrict) access to a web server.
	- it not reliant on any other DNS server in the look up process.
	- **The single source of truth.** It can satisfy queries from its own data without needing to query another source.
	- ### DNS Record Request Sequence
	- ![dns. sequence](https://cf-assets.www.cloudflare.com/slt3lc6tev37/6Cxvsc4NOvmU4pPkKbkDmP/a7588a4c8a3c187e9175a40fa1b3d548/dns_record_request_sequence_authoritative_nameserver.png)
- 📌 in instances where the query is for a subdomain such as foo.example.com or blog.cloudflare.com, an additional nameserver will be added to the sequence after the authoritative nameserver, which is responsible for storing the subdomain’s CNAME record. 
  hid:: 99BAkB-JEe6rpt93nSz9FA
  updated:: 2023-07-11T01:28:36.091723+00:00
  📝 ## CNAME DNS Record Request Sequence
  ![CNAME DNS Record Request Sequence](https://cf-assets.www.cloudflare.com/slt3lc6tev37/1O1o3jhs0ztWsD00k8RLIJ/f33c1793a7e21cb92678c1f35ef1b245/dns_record_request_sequence_cname_subdomain.png)
- 📌 Different DNS recursive resolvers such as Google DNS, OpenDNS, and providers like Comcast all maintain data center installations of DNS recursive resolvers. These resolvers allow for quick and easy queries through optimized clusters of DNS-optimized computer systems, but they are fundamentally different than the nameservers hosted by Cloudflare.
  hid:: V8r-Dh-KEe6mTsd-hpNKMQ
  updated:: 2023-07-11T01:29:13.322640+00:00
- #### 8 steps in a DNS lookup:
	- 1. A user types ‘example.com’ into a web browser and the query travels into the Internet and is received by a DNS recursive resolver.
	- 2. The resolver then queries a DNS root nameserver (.).
	- 3. The root server then responds to the resolver with the address of a Top Level Domain (TLD) DNS server (such as .com or .net), which stores the information for its domains. When searching for example.com, our request is pointed toward the .com TLD.
	- 4. The resolver then makes a request to the .com TLD.
	- 5. The TLD server then responds with the IP address of the domain’s nameserver, example.com.
	- 6. Lastly, the recursive resolver sends a query to the domain’s nameserver.
	- 7. The IP address for example.com is then returned to the resolver from the nameserver
	- .8. The DNS resolver then responds to the web browser with the IP address of the domain requested initially.
- **Once the 8 steps of the DNS lookup have returned the IP address for example.com, the browser is able to make the request for the web page:**
	- 1. The browser makes a HTTP request to the IP address.
	- 2. The server at that IP returns the webpage to be rendered in the browser (step 10).
	- ##### Complete DNS Lookup and Webpage Query
	  ![dns and webpage query](https://cf-assets.www.cloudflare.com/slt3lc6tev37/1NzaAqpEFGjqTZPAS02oNv/bf7b3f305d9c35bde5c5b93a519ba6d5/what_is_a_dns_server_dns_lookup.png)
- 📌 The DNS resolver is the first stop in the DNS lookup, and it is responsible for dealing with the client that made the initial request.
  hid:: AR_x7h-LEe65Iuv8ktqgRQ
  updated:: 2023-07-11T01:33:57.445965+00:00
- 📌 The resolver starts the sequence of queries that ultimately leads to a URL being translated into the necessary IP address.
  hid:: BMfbzB-LEe6mMIs-q8V6Nw
  updated:: 2023-07-11T01:34:03.555748+00:00