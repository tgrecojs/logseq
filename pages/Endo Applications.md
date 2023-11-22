- The `endo` daemon is a tool for bootstrapping objects, and systems of objects.
- In this sense, `endo` acts a powerbox for objects.
- The experience of looking inside an Endo application is similar to Russian Matroshka doll.
	- In this comparison, we need exchange height and width and instead denote size according to the powers that an object has.
	- Endo applications aren't in the physical world and therefore they do not have a height or a width.
	- ![endo dolls](https://upload.wikimedia.org/wikipedia/commons/7/71/Russian-Matroshka.jpg){:height 412, :width 539}
- `@endo/daemon`
	- `main`
	- The Endo Daemon allows us to bootstrap systems designed to enforce POLA at the deepest level of granularity.
	- It contains access to all of the powers that it may ever need
		- `store`
		  `provide`
		  `inbox`
		  `followInbox`
		  `request,`
		  `resolve,
		  `reject,`
	- execution environment for creating Endo applications.
	- A tool for bootstrapping systems.
- `@endo/daemon`'s `main` module
	- **hands out to each module just enough authority to do the job needed by that particular module.**
- [[How Emily Tamed the Caml]]
- Endo application's implement the [[Powerbox Pattern]]