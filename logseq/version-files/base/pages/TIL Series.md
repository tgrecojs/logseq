## Infinite loops with `for(;;;)`
	- For the last ~3 years, i've spent the vast majority of my time focused on developing secure, distributed systems using none other than JavaScript.
	-
	-
	- on distributed systems.
	- It may come as a surprise to many reade
	-
	- often come across this within the context of #Endo, or #[[agoric sdk]] code.
	- how this works?
		- `for` has 3 parts, each of which is optional.
			- initializes the loop
			  logseq.order-list-type:: number
			- decides whether or not to continue
			  logseq.order-list-type:: number
			- operation that occurs at the end of each iteration.
			  logseq.order-list-type:: number
		- `for(;;)` is the short-form of `for(;true;)`
	- ```javascript
	  // https://github.com/endojs/endo/blob/master/packages/stream/README.md#prime
	  async function *logGenerator() {
	    for (;;) {
	      console.log(yield);
	    }
	  }
	  ```
-
-