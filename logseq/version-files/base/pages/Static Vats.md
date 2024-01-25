## Static Vats
	- *Static Vats*
		- **Vats which are defined in the configuration object, and created at startup time.**
	-
- ### Defining Static Vats
	- Defined in the configuration object at startup time.
	- The root object of static vats are made available to the `bootstrap()` method, so they can be wired together as needed.
	- ### Creation Process
	- Static vats are defined by a JS module file which exports a function named buildRootObject. The file may export other names.
	-
	- ### Cavetas
		- The module which exports `buildRootObject` can import other modules as long as they are:
			- pure JS (no native modules, binary libraries)
			  logseq.order-list-type:: number
			- compatible with SES.
			  logseq.order-list-type:: number
			- logseq.order-list-type:: number
		- The source for **all static vats must be available at the time the host application starts**.
	- ### Differentiating between dynamic and static vats
	- ||Static Vats|Dynamic Vats||
	  |--|--|--|--|
	  |Creation Process|Defined within a JS module file which exports a function named **buildRootObject**.|||
	  |||||
	  |||||