- ## HandledPromise
	-
- ### Security (`isSafePromise`)
	-
	- ```javascript
	  /**
	     * If the promise `p` is safe, then during the evaluation of the
	     * expressopns `p.then` and `await p`, `p` cannot mount a reentrancy attack.
	     * Unfortunately, due to limitations of the current JavaScript standard,
	     * it seems impossible to prevent `p` from mounting a reentrancy attack
	     * during the evaluation of `isSafePromise(p)`, and therefore during
	     * operations like `HandledPromise.resolve(p)` that call
	     * `isSafePromise(p)` synchronously.
	     *
	     * The `@endo/marshal` package defines a related notion of a passable
	     * promise, i.e., one for which which `passStyleOf(p) === 'promise'`. All
	     * passable promises are also safe. But not vice versa because the
	     * requirements for a promise to be passable are slightly greater. A safe
	     * promise must not override `then` or `constructor`. A passable promise
	     * must not have any own properties. The requirements are otherwise
	     * identical.
	     *
	     * @param {Promise} p
	     * @returns {boolean}
	     */
	    const isSafePromise = p => {
	      return (
	        isFrozen(p) &&
	        getPrototypeOf(p) === Promise.prototype &&
	        Promise.resolve(p) === p &&
	        getOwnPropertyDescriptor(p, 'then') === undefined &&
	        getOwnPropertyDescriptor(p, 'constructor') === undefined
	      );
	    };
	  ```
		-
-