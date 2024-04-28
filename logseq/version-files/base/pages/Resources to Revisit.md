## Concurrency
	- ((662359e1-acb1-45c8-9ebd-5385d667ecaa))
	  collapsed:: true
		- Discusses the internals of `Promise`.
		- **Note:** The following behavior was correct according to the JavaScript spec at the time of writing. Since then, our spec proposal was accepted, and the following "buggy" behavior is now correct.
		  ```javascript
		  const p = Promise.resolve();
		  (async () => {
		  await p; console.log('after:await');
		  })();
		  
		  p.then(() => console.log('tick:a')
		  .then(() => console.log('tick:b'));
		  ```
		  
		  The above program creates a fulfilled promise `p`, and `await`s its result, but also chains two handlers onto it. In which order would you expect the `console.log` calls to execute?
		- Since `p` is fulfilled, you might expect it to print `'after await'` first and then the `'tick'`s. In fact, that’s the behavior you’d get in Node.js 8:
		- ### The `await` bug in Node.js 8
			- Although ==this behavior seems intuitive, it’s not correct according to the specification.== Node.js 10 implements the correct behavior, which is to first execute the chained handlers, and only afterwards continue with the async function.
			- Node.js 10 no longer has the `await` bug.
			- This *“correct behavior”* is arguably not immediately obvious, and was actually surprising to JavaScript developers, so it deserves some explanation. Before we dive into the magical world of promises and async functions, let’s start with some of the foundations.
		- ### Tasks vs. microtasks
		- On a high level there are *tasks* and *microtasks* in JavaScript. ==Tasks handle events like I/O and timers, and execute one at a time. Microtasks implement deferred execution for== `==async==`==/==`==await==` ==and promises, and execute at the end of each task. The microtask queue is always emptied before execution returns to the event loop.==
		- #### The difference between microtasks and tasks
			- For more details, check out Jake Archibald’s explanation of [tasks, microtasks, queues, and schedules in the browser](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/). The task model in Node.js is very similar.
			- According to MDN, ==an async function is a function which operates asynchronously using an implicit promise to return its result.== Async functions are intended to make asynchronous code look like synchronous code, hiding some of the complexity of the asynchronous processing from the developer.
			- The simplest possible async function looks like this:
			- ```javascript
			  async function computeAnswer() {
			    return 42;
			  }
			  ```
			  
			  When called it returns a promise, and you can get to its value like with any other promise.
			  ```javascript
			  const p = computeAnswer();
			  *// → Promise*
			  
			  p.then(console.log);
			  *// prints 42 on the next turn*
			  ```
			  
			  You only get to the value of this promise `p` the next time microtasks are run. In other words, the above program is semantically equivalent to using `Promise.resolve` with the value:
			  
			  ```javascript
			  function computeAnswer() {
			      return Promise.resolve(42);
			  }
			  ```
			  
			  The real power of async functions comes from `await` expressions, which cause the function execution to pause until a promise is resolved, and resume after fulfillment. The value of `await` is that of the fulfilled promise. Here’s an example showing what that means:
			  
			  ```javascript
			  async function fetchStatus(url) {
			    const response = await fetch(url);
			    return response.status;
			  }
			  ```
			  
			  The execution of `fetchStatus` gets suspended on the `await`, and is later resumed when the `fetch` promise fulfills. This is more or less equivalent to chaining a handler onto the promise returned from `fetch`.
			- That handler contains the code following the `await` in the async function.
			- Normally you’d pass a `Promise` to `await`, but you can actually wait on any arbitrary JavaScript value. If the value of the expression following the `await` is not a promise, it’s converted to a promise. That means you can `await 42` if you feel like doing that:
		-
		-
- ### Cryptography
	- {{embed ((662359e1-3b69-43f1-bf61-2e600c194c8b))}}
-