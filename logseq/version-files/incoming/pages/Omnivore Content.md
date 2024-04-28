## All posts
	- [Faster async functions and promises · V8](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88)
	  site:: [v8.dev](https://v8.dev/blog/fast-async)
	  date-saved:: [[04/20/2024]]
	  date-published:: [[11/11/2018]]
	  id:: 662359e1-acb1-45c8-9ebd-5385d667ecaa
		- ### Content
			- Published · Tagged with [ECMAScript](https://v8.dev/blog/tags/ecmascript) [benchmarks](https://v8.dev/blog/tags/benchmarks) [presentations](https://v8.dev/blog/tags/presentations)
			  
			  Asynchronous processing in JavaScript traditionally had a reputation for not being particularly fast. To make matters worse, debugging live JavaScript applications — in particular Node.js servers — is no easy task, _especially_ when it comes to async programming. Luckily the times, they are a-changin’. This article explores how we optimized async functions and promises in V8 (and to some extent in other JavaScript engines as well), and describes how we improved the debugging experience for async code.
			  
			  **Note:** If you prefer watching a presentation over reading articles, then enjoy the video below! If not, skip the video and read on.
			  
			  \#\# A new approach to async programming [\#](\#a-new-approach-to-async-programming)
			  
			  \#\#\# From callbacks to promises to async functions [\#](\#from-callbacks-to-promises-to-async-functions)
			  
			  Before promises were part of the JavaScript language, callback-based APIs were commonly used for asynchronous code, especially in Node.js. Here’s an example:
			  
			  ```moonscript
			  function handler(done) {
			  validateParams((error) => {
			    if (error) return done(error);
			    dbQuery((error, dbResults) => {
			      if (error) return done(error);
			      serviceCall(dbResults, (error, serviceResults) => {
			        console.log(result);
			        done(error, serviceResults);
			      });
			    });
			  });
			  }
			  ```
			  
			  The specific pattern of using deeply-nested callbacks in this manner is commonly referred to as _“callback hell”_, because it makes the code less readable and hard to maintain.
			  
			  Luckily, now that promises are part of the JavaScript language, the same code could be written in a more elegant and maintainable manner:
			  
			  ```javascript
			  function handler() {
			  return validateParams()
			    .then(dbQuery)
			    .then(serviceCall)
			    .then(result => {
			      console.log(result);
			      return result;
			    });
			  }
			  ```
			  
			  Even more recently, JavaScript gained support for [async functions](https://developers.google.com/web/fundamentals/primers/async-functions). The above asynchronous code can now be written in a way that looks very similar to synchronous code:
			  
			  ```javascript
			  async function handler() {
			  await validateParams();
			  const dbResults = await dbQuery();
			  const results = await serviceCall(dbResults);
			  console.log(results);
			  return results;
			  }
			  ```
			  
			  With async functions, the code becomes more succinct, and the control and data flow are a lot easier to follow, despite the fact that the execution is still asynchronous. (Note that the JavaScript execution still happens in a single thread, meaning async functions don’t end up creating physical threads themselves.)
			  
			  \#\#\# From event listener callbacks to async iteration [\#](\#from-event-listener-callbacks-to-async-iteration)
			  
			  Another asynchronous paradigm that’s especially common in Node.js is that of [ReadableStreams](https://nodejs.org/api/stream.html\#stream%5Freadable%5Fstreams). Here’s an example:
			  
			  ```coffeescript
			  const http = require('http');
			  http.createServer((req, res) => {
			  let body = '';
			  req.setEncoding('utf8');
			  req.on('data', (chunk) => {
			    body += chunk;
			  });
			  req.on('end', () => {
			    res.write(body);
			    res.end();
			  });
			  }).listen(1337);
			  
			  ```
			  
			  This code can be a little hard to follow: the incoming data is processed in chunks that are only accessible within callbacks, and the end-of-stream signaling happens inside a callback too. It’s easy to introduce bugs here when you don’t realize that the function terminates immediately and that the actual processing has to happen in the callbacks.
			  
			  Fortunately, a cool new ES2018 feature called [async iteration](http://2ality.com/2016/10/asynchronous-iteration.html) can simplify this code:
			  
			  ```javascript
			  const http = require('http');
			  http.createServer(async (req, res) => {
			  try {
			    let body = '';
			    req.setEncoding('utf8');
			    for await (const chunk of req) {
			      body += chunk;
			    }
			    res.write(body);
			    res.end();
			  } catch {
			    res.statusCode = 500;
			    res.end();
			  }
			  }).listen(1337);
			  
			  ```
			  
			  Instead of putting the logic that deals with the actual request processing into two different callbacks — the `'data'` and the `'end'` callback — we can now put everything into a single async function instead, and use the new `for await…of` loop to iterate over the chunks asynchronously. We also added a `try-catch` block to avoid the `unhandledRejection` problem[\[1\]](\#fn1).
			  
			  You can already use these new features in production today! Async functions are **fully supported starting with Node.js 8 (V8 v6.2 / Chrome 62)**, and async iterators and generators are **fully supported starting with Node.js 10 (V8 v6.8 / Chrome 68)**!
			  
			  \#\# Async performance improvements [\#](\#async-performance-improvements)
			  
			  We’ve managed to improve the performance of asynchronous code significantly between V8 v5.5 (Chrome 55 & Node.js 7) and V8 v6.8 (Chrome 68 & Node.js 10). We reached a level of performance where developers can safely use these new programming paradigms without having to worry about speed.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/600x371,scjbrUsZZZ-V8wTrWYGGligcVsUyx-EjkpqOMG9UCFWU/https://v8.dev/_img/fast-async/doxbee-benchmark.svg)
			  
			  The above chart shows the [doxbee benchmark](https://github.com/v8/promise-performance-tests/blob/master/lib/doxbee-async.js), which measures performance of promise-heavy code. Note that the charts visualize execution time, meaning lower is better.
			  
			  The results on the [parallel benchmark](https://github.com/v8/promise-performance-tests/blob/master/lib/parallel-async.js), which specifically stresses the performance of [Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global%5FObjects/Promise/all), are even more exciting:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/600x371,sqST2H23KA5wSbP9KaGaMA2JKPoTlQnvvss1Aag4oGLQ/https://v8.dev/_img/fast-async/parallel-benchmark.svg)
			  
			  We’ve managed to improve `Promise.all` performance by a factor of **8×**.
			  
			  However, the above benchmarks are synthetic micro-benchmarks. The V8 team is more interested in how our optimizations affect [real-world performance of actual user code](https://v8.dev/blog/real-world-performance).
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/600x371,s18y0PuT9nAffkQ1FewAnLVlCILaPVYUS4BQZ469odXE/https://v8.dev/_img/fast-async/http-benchmarks.svg)
			  
			  The above chart visualizes the performance of some popular HTTP middleware frameworks that make heavy use of promises and `async` functions. Note that this graph shows the number of requests/second, so unlike the previous charts, higher is better. The performance of these frameworks improved significantly between Node.js 7 (V8 v5.5) and Node.js 10 (V8 v6.8).
			  
			  These performance improvements are the result of three key achievements:
			  
			  * [TurboFan](https://v8.dev/docs/turbofan), the new optimizing compiler 🎉
			  * [Orinoco](https://v8.dev/blog/orinoco), the new garbage collector 🚛
			  * a Node.js 8 bug causing `await` to skip microticks 🐛
			  
			  When we [launched TurboFan](https://v8.dev/blog/launching-ignition-and-turbofan) in [Node.js 8](https://medium.com/the-node-js-collection/node-js-8-3-0-is-now-available-shipping-with-the-ignition-turbofan-execution-pipeline-aa5875ad3367), that gave a huge performance boost across the board.
			  
			  We’ve also been working on a new garbage collector, called Orinoco, which moves garbage collection work off the main thread, and thus improves request processing significantly as well.
			  
			  And last but not least, there was a handy bug in Node.js 8 that caused `await` to skip microticks in some cases, resulting in better performance. The bug started out as an unintended spec violation, but it later gave us the idea for an optimization. Let’s start by explaining the buggy behavior:
			  
			  **Note:** The following behavior was correct according to the JavaScript spec at the time of writing. Since then, our spec proposal was accepted, and the following "buggy" behavior is now correct.
			  
			  ```typescript
			  const p = Promise.resolve();
			  (async () => {
			  await p; console.log('after:await');
			  })();
			  
			  p.then(() => console.log('tick:a'))
			  .then(() => console.log('tick:b'));
			  
			  ```
			  
			  ==The above program creates a fulfilled promise== `==p==`==, and== `==await==`==s its result, but also chains two handlers onto it.== In which order would you expect the `console.log` calls to execute?
			  
			  ==Since== `==p==` ==is fulfilled, you might expect it to print== `=='after====:====await===='==` ==first and then the== `=='tick===='==`==s.== In fact, that’s the behavior you’d get in Node.js 8:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/960x446,sg4Yb4z0ft6c6MYW9h0rzkD_--mDvXwXSv1sfW7zxLFQ/https://v8.dev/_img/fast-async/await-bug-node-8.svg)
			  
			  The `await` bug in Node.js 8
			  
			  Although ==this behavior seems intuitive, it’s not correct according to the specification.== Node.js 10 implements the correct behavior, which is to first execute the chained handlers, and only afterwards continue with the async function.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/960x446,snKJMsNL80MdvKJOiHXmmMW4GKIWEvVJW5I3XNgrEgNw/https://v8.dev/_img/fast-async/await-bug-node-10.svg)
			  
			  Node.js 10 no longer has the `await` bug
			  
			  This _“correct behavior”_ is arguably not immediately obvious, and was actually surprising to JavaScript developers, so it deserves some explanation. Before we dive into the magical world of promises and async functions, let’s start with some of the foundations.
			  
			  \#\#\# Tasks vs. microtasks [\#](\#tasks-vs.-microtasks)
			  
			  On a high level there are _tasks_ and _microtasks_ in JavaScript. ==Tasks handle events like I/O and timers, and execute one at a time. Microtasks implement deferred execution for== `==async==`==/==`==await==` ==and promises, and execute at the end of each task. The microtask queue is always emptied before execution returns to the event loop.==
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/833x286,sCtdKLezkHV4xCAUCFMwg5kMJDr02yWu9b4HqTWUCU4Q/https://v8.dev/_img/fast-async/microtasks-vs-tasks.svg)
			  
			  The difference between microtasks and tasks
			  
			  For more details, check out Jake Archibald’s explanation of [tasks, microtasks, queues, and schedules in the browser](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/). The task model in Node.js is very similar.
			  
			  According to MDN, ==an async function is a function which operates asynchronously using an implicit promise to return its result.== Async functions are intended to make asynchronous code look like synchronous code, hiding some of the complexity of the asynchronous processing from the developer.
			  
			  The simplest possible async function looks like this:
			  
			  ```ada
			  async function computeAnswer() {
			  return 42;
			  }
			  ```
			  
			  When called it returns a promise, and you can get to its value like with any other promise.
			  
			  ```cpp
			  const p = computeAnswer();
			  // → Promise
			  p.then(console.log);
			  // prints 42 on the next turn
			  
			  ```
			  
			  You only get to the value of this promise `p` the next time microtasks are run. In other words, the above program is semantically equivalent to using `Promise.resolve` with the value:
			  
			  ```ada
			  function computeAnswer() {
			  return Promise.resolve(42);
			  }
			  ```
			  
			  The real power of async functions comes from `await` expressions, which cause the function execution to pause until a promise is resolved, and resume after fulfillment. The value of `await` is that of the fulfilled promise. Here’s an example showing what that means:
			  
			  ```qml
			  async function fetchStatus(url) {
			  const response = await fetch(url);
			  return response.status;
			  }
			  ```
			  
			  The execution of `fetchStatus` gets suspended on the `await`, and is later resumed when the `fetch` promise fulfills. This is more or less equivalent to chaining a handler onto the promise returned from `fetch`.
			  
			  ```arcade
			  function fetchStatus(url) {
			  return fetch(url).then(response => response.status);
			  }
			  ```
			  
			  That handler contains the code following the `await` in the async function.
			  
			  Normally you’d pass a `Promise` to `await`, but you can actually wait on any arbitrary JavaScript value. If the value of the expression following the `await` is not a promise, it’s converted to a promise. That means you can `await 42` if you feel like doing that:
			  
			  ```javascript
			  async function foo() {
			  const v = await 42;
			  return v;
			  }
			  const p = foo();
			  // → Promise
			  
			  p.then(console.log);
			  // prints `42` eventually
			  
			  ```
			  
			  More interestingly, `await` works with any [“thenable”](https://promisesaplus.com/), i.e. any object with a `then` method, even if it’s not a real promise. So you can implement funny things like an asynchronous sleep that measures the actual time spent sleeping:
			  
			  ```javascript
			  class Sleep {
			  constructor(timeout) {
			    this.timeout = timeout;
			  }
			  then(resolve, reject) {
			    const startTime = Date.now();
			    setTimeout(() => resolve(Date.now() - startTime),
			               this.timeout);
			  }
			  }
			  (async () => {
			  const actualTime = await new Sleep(1000);
			  console.log(actualTime);
			  })();
			  
			  ```
			  
			  Let’s see what V8 does for `await` under the hood, following the [specification](https://tc39.es/ecma262/\#await). Here’s a simple async function `foo`:
			  
			  ```javascript
			  async function foo(v) {
			  const w = await v;
			  return w;
			  }
			  ```
			  
			  When called, it wraps the parameter `v` into a promise and suspends execution of the async function until that promise is resolved. Once that happens, execution of the function resumes and `w` gets assigned the value of the fulfilled promise. This value is then returned from the async function.
			  
			  \#\#\# `await` under the hood [\#](\#await-under-the-hood)
			  
			  First of all, V8 marks this function as _resumable_, which means that execution can be suspended and later resumed (at `await` points). Then it creates the so-called `implicit_promise`, which is the promise that is returned when you invoke the async function, and that eventually resolves to the value produced by the async function.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/960x470,sndmOuuwLN9OMBRxeF4ASbIhJs7lkJkuWjEBMM9iU_ro/https://v8.dev/_img/fast-async/await-under-the-hood.svg)
			  
			  Comparison between a simple async function and what the engine turns it into
			  
			  Then comes the interesting bit: the actual `await`. First the value passed to `await` is wrapped into a promise. Then, handlers are attached to this wrapped promise to resume the function once the promise is fulfilled, and execution of the async function is suspended, returning the `implicit_promise` to the caller. Once the `promise` is fulfilled, execution of the async function is resumed with the value `w` from the `promise`, and the `implicit_promise` is resolved with `w`.
			  
			  In a nutshell, the initial steps for `await v` are:
			  
			  1. Wrap `v` — the value passed to `await` — into a promise.
			  2. Attach handlers for resuming the async function later.
			  3. Suspend the async function and return the `implicit_promise` to the caller.
			  
			  Let’s go through the individual operations step by step. Assume that the thing that is being `await`ed is already a promise, which was fulfilled with the value `42`. Then the engine creates a new `promise` and resolves that with whatever’s being `await`ed. This does deferred chaining of these promises on the next turn, expressed via what the specification calls a [PromiseResolveThenableJob](https://tc39.es/ecma262/\#sec-promiseresolvethenablejob).
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/814x543,sSFxtdkRi9TGgRpreNBDIuPeV9PlhvD8k5nv20HMs9Lg/https://v8.dev/_img/fast-async/await-step-1.svg)
			  
			  Then the engine creates another so-called `throwaway` promise. It’s called _throwaway_ because nothing is ever chained to it — it’s completely internal to the engine. This `throwaway` promise is then chained onto the `promise`, with appropriate handlers to resume the async function. This `performPromiseThen` operation is essentially what [Promise.prototype.then()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global%5FObjects/Promise/then) does, behind the scenes. Finally, execution of the async function is suspended, and control returns to the caller.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/814x543,sAQVNKOvn-Tv-2w1MsCBGRxHI7shUhRrq7FdACtw4aYc/https://v8.dev/_img/fast-async/await-step-2.svg)
			  
			  Execution continues in the caller, and eventually the call stack becomes empty. Then the JavaScript engine starts running the microtasks: it runs the previously scheduled [PromiseResolveThenableJob](https://tc39.es/ecma262/\#sec-promiseresolvethenablejob), which schedules a new [PromiseReactionJob](https://tc39.es/ecma262/\#sec-promisereactionjob) to chain the `promise` onto the value passed to `await`. Then, the engine returns to processing the microtask queue, since the microtask queue must be emptied before continuing with the main event loop.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/814x543,sjLDc8ljRa8FtwwurHpKpJpjtG86NYkUqPDxKv-qrFRc/https://v8.dev/_img/fast-async/await-step-3.svg)
			  
			  Next up is the [PromiseReactionJob](https://tc39.es/ecma262/\#sec-promisereactionjob), which fulfills the `promise` with the value from the promise we’re `await`ing — `42` in this case — and schedules the reaction onto the `throwaway` promise. The engine then returns to the microtask loop again, which contains a final microtask to be processed.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/814x543,sz1TvYu6y1WsxRyjKyWCGcSTawr5LrCfGiPW8h2JDZDA/https://v8.dev/_img/fast-async/await-step-4-final.svg)
			  
			  Now this second [PromiseReactionJob](https://tc39.es/ecma262/\#sec-promisereactionjob) propagates the resolution to the `throwaway` promise, and resumes the suspended execution of the async function, returning the value `42` from the `await`.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/958x465,sE2DjEGKn-Ny8LgcXV6F_0pOBmCAfVFerOA8i4LD2Rn4/https://v8.dev/_img/fast-async/await-overhead.svg)
			  
			  Summary of the overhead of `await`
			  
			  Summarizing what we’ve learned, for each `await` the engine has to create **two additional** promises (even if the right hand side is already a promise) and it needs **at least three** microtask queue ticks. Who knew that a single `await` expression resulted in _that much overhead_?!
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/451x215,swSGgaqMxyf1MjfIBdcOfsJATQg_7lT8UaGo0gEdIuB0/https://v8.dev/_img/fast-async/await-code-before.svg)
			  
			  Let’s have a look at where this overhead comes from. The first line is responsible for creating the wrapper promise. The second line immediately resolves that wrapper promise with the `await`ed value `v`. These two lines are responsible for one additional promise plus two out of the three microticks. That’s quite expensive if `v` is already a promise (which is the common case, since applications normally `await` on promises). In the unlikely case that a developer `await`s on e.g. `42`, the engine still needs to wrap it into a promise.
			  
			  As it turns out, there’s already a [promiseResolve](https://tc39.es/ecma262/\#sec-promise-resolve) operation in the specification that only performs the wrapping when needed:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/950x376,spkJW4BzRx_Aij8iLmGLKCmz8iE3VDNtgxeNPXWXa3vM/https://v8.dev/_img/fast-async/await-code-comparison.svg)
			  
			  This operation returns promises unchanged, and only wraps other values into promises as necessary. This way you save one of the additional promises, plus two ticks on the microtask queue, for the common case that the value passed to `await` is already a promise. This new behavior is already [enabled by default in V8 v7.2](https://v8.dev/blog/v8-release-72\#async%2Fawait). For V8 v7.1, the new behavior can be enabled using the `--harmony-await-optimization` flag. We’ve [proposed this change to the ECMAScript specification](https://github.com/tc39/ecma262/pull/1250) as well.
			  
			  Here’s how the new and improved `await` works behind the scenes, step by step:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/792x545,sPmuR6h_2tKMfi8m2nYZgv1_hCWwB8Ve8naKUPTIZmko/https://v8.dev/_img/fast-async/await-new-step-1.svg)
			  
			  Let’s assume again that we `await` a promise that was fulfilled with `42`. Thanks to the magic of [promiseResolve](https://tc39.es/ecma262/\#sec-promise-resolve) the `promise` now just refers to the same promise `v`, so there’s nothing to do in this step. Afterwards the engine continues exactly like before, creating the `throwaway` promise, scheduling a [PromiseReactionJob](https://tc39.es/ecma262/\#sec-promisereactionjob) to resume the async function on the next tick on the microtask queue, suspending execution of the function, and returning to the caller.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/648x548,sL8xY6Rt3-hLPDFWMoPNnFVM5ANs4l47WDu0saH9v6Os/https://v8.dev/_img/fast-async/await-new-step-2.svg)
			  
			  Then eventually when all JavaScript execution finishes, the engine starts running the microtasks, so it executes the [PromiseReactionJob](https://tc39.es/ecma262/\#sec-promisereactionjob). This job propagates the resolution of `promise` to `throwaway`, and resumes the execution of the async function, yielding `42` from the `await`.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/956x383,sP0u3uIK3cUSRQiYw-jgcUal1sdwH_RHbFw9zaQsJa1w/https://v8.dev/_img/fast-async/await-overhead-removed.svg)
			  
			  Summary of the reduction in `await` overhead
			  
			  This optimization avoids the need to create a wrapper promise if the value passed to `await` is already a promise, and in that case we go from a minimum of **three** microticks to just **one** microtick. This behavior is similar to what Node.js 8 does, except that now it’s no longer a bug — it’s now an optimization that is being standardized!
			  
			  It still feels wrong that the engine has to create this `throwaway` promise, despite being completely internal to the engine. As it turns out, the `throwaway` promise was only there to satisfy the API constraints of the internal `performPromiseThen` operation in the spec.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/937x198,s0tKybKI3-ADKUg-LO0e3MAuBMSmcnsQ_mqDBs2w750I/https://v8.dev/_img/fast-async/await-optimized.svg)
			  
			  This was recently addressed in an [editorial change](https://github.com/tc39/ecma262/issues/694) to the ECMAScript specification. Engines no longer need to create the `throwaway` promise for `await` — most of the time[\[2\]](\#fn2).
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/940x356,sqQRV5c3DMpVic6x572DTqJXF-toti5I13Zya9ywLg-s/https://v8.dev/_img/fast-async/node-10-vs-node-12.svg)
			  
			  Comparison of `await` code before and after the optimizations
			  
			  Comparing `await` in Node.js 10 to the optimized `await` that’s likely going to be in Node.js 12 shows the performance impact of this change:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/600x371,shooN8URgg3DQDUYf3qzMLQsC66B3zDD55AKtSb01Pwo/https://v8.dev/_img/fast-async/benchmark-optimization.svg)
			  
			  **`async`/`await` outperforms hand-written promise code now**. The key takeaway here is that we significantly reduced the overhead of async functions — not just in V8, but across all JavaScript engines, by patching the spec.
			  
			  **Update:** As of V8 v7.2 and Chrome 72, `--harmony-await-optimization` is enabled by default. [The patch](https://github.com/tc39/ecma262/pull/1250) to the ECMAScript specification was merged.
			  
			  \#\# Improved developer experience [\#](\#improved-developer-experience)
			  
			  In addition to performance, JavaScript developers also care about the ability to diagnose and fix problems, which is not always easy when dealing with asynchronous code. [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools) supports _async stack traces_, i.e. stack traces that not only include the current synchronous part of the stack, but also the asynchronous part:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/877x369,s-Vx14wcO0nM9wpQLgY_9YBMCTOeEP1ELG_7-w3LcRP4/https://v8.dev/_img/fast-async/devtools.png)
			  
			  This is an incredibly useful feature during local development. However, this approach doesn’t really help you once the application is deployed. During post-mortem debugging, you’ll only see the `Error\#stack` output in your log files, and that doesn’t tell you anything about the asynchronous parts.
			  
			  We’ve recently been working on [_zero-cost async stack traces_](https://bit.ly/v8-zero-cost-async-stack-traces) which enrich the `Error\#stack` property with async function calls. “Zero-cost” sounds exciting, doesn’t it? How can it be zero-cost, when the Chrome DevTools feature comes with major overhead? Consider this example where `foo` calls `bar` asynchronously, and `bar` throws an exception after `await`ing a promise:
			  
			  ```javascript
			  async function foo() {
			  await bar();
			  return 42;
			  }
			  async function bar() {
			  await Promise.resolve();
			  throw new Error('BEEP BEEP');
			  }
			  
			  foo().catch(error => console.log(error.stack));
			  
			  ```
			  
			  Running this code in Node.js 8 or Node.js 10 results in the following output:
			  
			  ```angelscript
			  $ node index.js
			  Error: BEEP BEEP
			    at bar (index.js:8:9)
			    at process._tickCallback (internal/process/next_tick.js:68:7)
			    at Function.Module.runMain (internal/modules/cjs/loader.js:745:11)
			    at startup (internal/bootstrap/node.js:266:19)
			    at bootstrapNodeJSCore (internal/bootstrap/node.js:595:3)
			  ```
			  
			  Note that although the call to `foo()` causes the error, `foo` is not part of the stack trace at all. This makes it tricky for JavaScript developers to perform post-mortem debugging, independent of whether your code is deployed in a web application or inside of some cloud container.
			  
			  The interesting bit here is that the engine knows where it has to continue when `bar` is done: right after the `await` in function `foo`. Coincidentally, that’s also the place where the function `foo` was suspended. The engine can use this information to reconstruct parts of the asynchronous stack trace, namely the `await` sites. With this change, the output becomes:
			  
			  ```angelscript
			  $ node --async-stack-traces index.js
			  Error: BEEP BEEP
			    at bar (index.js:8:9)
			    at process._tickCallback (internal/process/next_tick.js:68:7)
			    at Function.Module.runMain (internal/modules/cjs/loader.js:745:11)
			    at startup (internal/bootstrap/node.js:266:19)
			    at bootstrapNodeJSCore (internal/bootstrap/node.js:595:3)
			    at async foo (index.js:2:3)
			  ```
			  
			  In the stack trace, the topmost function comes first, followed by the rest of the synchronous stack trace, followed by the asynchronous call to `bar` in function `foo`. This change is implemented in V8 behind the new `--async-stack-traces` flag. **Update**: As of V8 v7.3, `--async-stack-traces` is enabled by default.
			  
			  However, if you compare this to the async stack trace in Chrome DevTools above, you’ll notice that the actual call site to `foo` is missing from the asynchronous part of the stack trace. As mentioned before, this approach utilizes the fact that for `await` the resume and suspend locations are the same — but for regular [Promise\#then()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global%5FObjects/Promise/then) or [Promise\#catch()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global%5FObjects/Promise/catch) calls, this is not the case. For more background, see Mathias Bynens’s explanation on [why await beats Promise\#then()](https://mathiasbynens.be/notes/async-stack-traces).
			  
			  \#\# Conclusion [\#](\#conclusion)
			  
			  We made async functions faster thanks to two significant optimizations:
			  
			  * the removal of two extra microticks, and
			  * the removal of the `throwaway` promise.
			  
			  On top of that, we’ve improved the developer experience via [_zero-cost async stack traces_](https://bit.ly/v8-zero-cost-async-stack-traces), which work with `await` in async functions and `Promise.all()`.
			  
			  And we also have some nice performance advice for JavaScript developers:
			  
			  * favor `async` functions and `await` over hand-written promise code, and
			  * stick to the native promise implementation offered by the JavaScript engine to benefit from the shortcuts, i.e. avoiding two microticks for `await`.
			  
			  ---
			  
			  1. Thanks to [Matteo Collina](https://twitter.com/matteocollina) for pointing us to [this issue](https://github.com/mcollina/make-promises-safe/blob/master/README.md\#the-unhandledrejection-problem). [↩︎](\#fnref1)
			  2. V8 still needs to create the `throwaway` promise if [async\_hooks](https://nodejs.org/api/async%5Fhooks.html) are being used in Node.js, since the `before` and `after` hooks are run within the _context_ of the `throwaway` promise. [↩︎](\#fnref2)
		- ### Highlights
		  id:: 662359e1-b250-4c93-bcad-8183782b1153
			- > The above program creates a fulfilled promise `p`, and `await`s its result, but also chains two handlers onto it. [⤴️](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88#2d6adec7-89ae-4585-a407-49bab57b6820)
			- > Since `p` is fulfilled, you might expect it to print `'after:await'` first and then the `'tick'`s. [⤴️](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88#827c4371-4dfe-4fec-947e-e19d98990135)
			- > this behavior seems intuitive, it’s not correct according to the specification. [⤴️](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88#991ffafd-8440-4c5f-9cbf-b66327b1dca1)
			- > the correct behavior, which is to first execute the chained handlers, and only afterwards continue with the async function. [⤴️](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88#89e65f7d-da21-4d44-bdec-2f946f4c89e5)
			- > Tasks handle events like I/O and timers, and execute one at a time. Microtasks implement deferred execution for `async`/`await` and promises, and execute at the end of each task. The microtask queue is always emptied before execution returns to the event loop. [⤴️](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88#6e92b315-a28b-4c4b-a6f6-a97b8894ebfc)
			- > an async function is a function which operates asynchronously using an implicit promise to return its result. [⤴️](https://omnivore.app/me/faster-async-functions-and-promises-v-8-18efa045a88#11ebb6eb-841e-40ee-b18c-4802b401c0c8)
	- [Web screenshots as digital assets | Pacific Lane](https://omnivore.app/me/web-screenshots-as-digital-assets-pacific-lane-18ef9e976e9)
	  collapsed:: true
	  site:: [Pacific Lane](https://pacific-lane.com/posts/web-screenshots-as-digital-assets/)
	  date-saved:: [[04/20/2024]]
		- ### Content
		  collapsed:: true
			- Let's imagine that every business today could easily download a free [digital wallet](https://en.wikipedia.org/wiki/Digital%5Fwallet) that instead of storing [cryptocurrency](https://ethereum.org/wallets), stores irrefutable proof of financial facts like:
			  
			  1. This transaction is approved by the counterparty
			  2. This transaction will occur on a specific date
			  3. This transaction has already occurred & the funds have moved
			  
			  What could a business possibly do with such an app?
			  
			  A lot, as it turns out.
			  
			  Due to the invasive nature of many [invoice factoring](https://en.wikipedia.org/wiki/Factoring%5F%28finance%29), [invoice financing](https://en.wikipedia.org/wiki/Debtor%5Ffinance), and [line of credit](https://en.wikipedia.org/wiki/Line%5Fof%5Fcredit) business relationships; ==when it comes time for a business to apply for funding via one, some, or all of these methods; that business must often "give away the keys to the kingdom" as part of the deal.==
			  
			  [NerdWallet](https://www.nerdwallet.com/article/small-business/invoice-factoring) gives some indication of what we are talking about when we describe these relationships as "invasive" and "giving away the keys":
			  
			  > Loss of direct control. With invoice factoring, your factoring company works directly with your customers. This means you must trust your factoring company to deal with your customers in a responsible and fair manner, and hope they don’t affect your relationships in a negative way.
			  
			  And in [this NerdWallet piece](https://www.nerdwallet.com/article/small-business/invoice-financing), too:
			  
			  > To qualify for invoice financing, you should have creditworthy customers who have a history of paying on time. In general, the creditworthiness and reputation of your customers will play a larger role in the underwriting process, making it easier to qualify for invoice financing over other business loan options.
			  
			  What those descriptions boil down to in practice today is that businesses applying for funds often create adjunct financial web site login credentials for their funders, or even more common, just let their funders "borrow" their own primary web site login credentials so these benefactors can monitor the stability of things like recurring revenues or the veracity of things like [UCC-1 claims](https://www.wolterskluwer.com/en/expert-insights/the-courts-are-clear-ucc-1s-must-be-authorized-to-be-effective).
			  
			  Funders are manually logging into web sites like this _all the time_, so they can see that certain money claimed to be owed really is owed or that certain obligations are likely to be paid. They are doing this for the simple reason that they must satisfy themselves that the transactions attested by the applying business (their prospect or customer) are in fact real.
			  
			  These activities of credential borrowing and logging in are usually manual in nature (humans typing at keyboards), often against the terms of service of the web sites themselves, and these days, generally likely to present a seriously interruptive pain in the neck for everyone involved since facilities like [MFA](https://en.wikipedia.org/wiki/Multi-factor%5Fauthentication) are now commonplace and things like [passkeys](https://www.passkeys.com/) (aka _WebAuthn_) are gaining prevalence.
			  
			  As for the use of adjunct logins, most [vendor portals](https://en.wikipedia.org/wiki/Enterprise%5Fportal) do not support them and only some banks support them. When it comes to portals, Amazon's [Vendor Central](https://boostontime.com/blog/create-multiple-user-on-amazon-vendor-central/) is one notable exception, since as you can see at the page linked, Amazon has for some time supported multiple users for the same Vendor Central business account. This feature allows a business user's account and their funders' snooping accounts to be kept separate.
			  
			  But for the bulk of funder login management, imagine the example of your funder needing to directly access, e.g., your [Target supplier portal](https://target.suppliergateway.com/MyAccount/Login) accounts receivable confirmations or your [Capital One Buiness Banking](https://www.capitalone.com/small-business/bank/) account details every few days or even every day, and pestering you for your latest MFA code every time they trigger one. Such practices are, in fact, commonplace in the world of small and medium-sized business finance.
			  
			  And even if funders are given adjunct logins, their prospects and customers then have to self-manage access to the attendant user accounts, e.g., when a funder denies their application or they add or change funding entities. Over time these businesses seeking funds are likely to encounter a host of other headaches ancillary to the straightforward need of the funders simply to verify incoming revenue and other financial facts from customers of their customers.
			  
			  It's worth noting at this point that there's a simple reason why businesses can't simply send to funders their own financial [screenshots](https://en.wikipedia.org/wiki/Screenshot) that they snap themselves: **fraud**. There is far too much risk of fabricated or adulterated screenshots showing up within the funders' own customer bases.
			  
			  Not only are there [several](https://www.seattletimes.com/business/no-oversight-inside-a-boom-time-startup-fraud-and-its-unraveling/) [recent](https://kildare-nationalist.ie/2024/01/12/two-kildare-men-receive-suspended-sentence-for-invoice-fraud-of-e120k/) [examples](https://bossip.com/1313737/bossip-exclusive-music-producer-teddy-riley-wins-1-9-mill-fraud-suit/) of this happening, it only stands to reason after a little thought. Eventually bad actors spoil the integrity of any well-meaning verification process based on a [circle of trust](http://www.sergiohicke.com/blog/the-circle-of-trust-how-do-you-give-trust) and common decency. Like Al Pacino said in that mediocre 2002 movie [S1m0ne](https://en.wikipedia.org/wiki/Simone%5F%282002%5Ffilm%29), _our ability to manufacture fraud now exceeds our ability to detect it._
			  
			  But must it always?
			  
			  Let's take a step back and be prepared to breathe a sigh of relief. It looks like, thank goodness, a promising new tool is coming online that holds the potential to provide a fundamentally better way for small and medium-sized businesses to prove what's true without having to engage in one giant [kludge](https://en.wikipedia.org/wiki/Kludge) made up of myriad manual web site logins.
			  
			  Ironically, but perhaps unsurprisingly, this tool is being underwritten by the fraud-ridden cryptocurrency sector. We should definitely look past that fact though, and leverage just the new technical capabilities coming down the pike. As we'll see, doing so will let us in theory serve [all the businesses that need reliable funding](https://www.finder.com/business-loans/business-loan-statistics), and in [scalable](https://en.wikipedia.org/wiki/Scalability) ways.
			  
			  Let's take look at [TLSNotary](https://tlsnotary.org/).
			  
			  ---
			  
			  \#\#\# TLSNotary - What is it?
			  
			  **TLSNotary** is an open source project that describes itself this way:
			  
			  > Export data from any web application and prove facts about it without compromising on privacy...proofs of authenticity for any data on the web...Using our protocol you can securely prove...You received a money transfer...
			  
			  et cetera.
			  
			  In truth, TLSNotary picks up where the now deprecated [PageSigner](https://github.com/tlsnotary/PageSigner) Chrome extensions and Firefox addons left off. We're going to start this section by looking at the older PageSigner, because its functionality was very similar to today's TLSNotary such that both projects share an espirit de corps. PageSigner (brought online [c. 2014](https://web.archive.org/web/20141001000000%2A/tlsnotary.org)) described itself like this:
			  
			  > PageSigner allows you to "notarize" web pages. Think of it as a cryptographically secure webpage screenshot - it's different from an ordinary screenshot in that it can't be edited in Photoshop; it really proves that you received that data from the server. You can then share that proof with anyone you like.
			  
			  That sounds a lot like the TLSNotary description we just referenced above, doesn't it?
			  
			  Below is a [demo video from 2015](https://yt.cdaut.de/watch?v=zQXF0iXEBSM) that shows the PageSigner Firefox addon in action. Before you watch it, it's worth noting that back when PageSigner was an active project, it relied on an older protocol _also_ called "TLSNotary." That's why you'll hear reference to TLSNotary in the video, but that act of mentioning is not referring to the same [Rust](https://www.rust-lang.org/)\-based project we'll be discussing further down in this post. Take a look:
			  
			  To emphasize the point, _PageSigner_ in its currently archived state is not likely to work very well or work at all today. You probably shouldn't bother with it. The associated browser extensions and addons probably would not even load in any recent version of Chrome or Firefox, and if they did, you would likely encounter a host of blocking issues.
			  
			  Instead, let yourself imagine—nay, embrace— what the modern, working version of TLSNotary could do for business financing, once it's ready, with the client either implemented in something like [Tauri](https://tauri.app/) or [Electron](https://www.electronjs.org/), or perhaps as a completely [headless browser](https://en.wikipedia.org/wiki/Headless%5Fbrowser) client that periodically reaches out to those same myriad web sites to grab data, but without requiring the funder to be manually in the mix as an intermediary.
			  
			  The resulting "screenshots" (this is an oversimplification, but good enough for the purposes of this post) would be effectively cryptographically signed by the [TLS certificates](https://en.wikipedia.org/wiki/Public%5Fkey%5Fcertificate\#TLS/SSL%5Fserver%5Fcertificate) of the visited web sites themselves, so to the degree funders trust Amazon's TLS certificates (which they most likely do), they should equivalently trust that the screenshots their customers are sending to them are unadulterated, i.e., no one added a nine or a bunch of zeros to the result before sending it along.
			  
			  The analogy of a digital [promissory note](https://en.wikipedia.org/wiki/Promissory%5Fnote) is by no means a perfect one, but TLSNotary arguably enables a new strain of cryptographically secure proof that would share some similarities:
			  
			  > The term note payable is commonly used in accounting (as distinguished from accounts payable) or commonly as just a "note", it is internationally defined by the Convention providing a uniform law for bills of exchange and promissory notes, but regional variations exist. A banknote is frequently referred to as a promissory note, as it is made by a bank and payable to bearer on demand. Mortgage notes are another prominent example...If the promissory note is unconditional and readily saleable, it is called a negotiable instrument.
			  
			  Perhaps it is better to just contend that TLSNotary may soon be enabling a new kind of [digital asset](https://en.wikipedia.org/wiki/Digital%5Fasset) that has nothing to do with cryptocurrency or blockchains, further distinguished from those things by having tangible value, since this asset irrevocably proves some money is on the way from, e.g. Amazon, or that a business really did pay off its previous loan. These digital artifacts, whatever the best name is, should logically be stored in a digital wallet, and then "pulled out" and shown to any funder who wants to see them.
			  
			  Hopefully someone will bring a verison of the above into reality, soon after TLSNotary is ready for production.
		- ### Highlights
		  collapsed:: true
			- > when it comes time for a business to apply for funding via one, some, or all of these methods; that business must often "give away the keys to the kingdom" as part of the deal. [⤴️](https://omnivore.app/me/web-screenshots-as-digital-assets-pacific-lane-18ef9e976e9#b9f25eda-913e-43f2-8628-695cd89a0966)
	- [TLS Oracles: Liberating Private Web Data with Cryptography | by Bastian Wetzel | Apr, 2024 | Medium](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821)
	  site:: [Medium](https://bwetzel.medium.com/tls-oracles-liberating-private-web-data-with-cryptography-e66e5fad7c34)
	  author:: Bastian Wetzel
	  date-saved:: [[04/20/2024]]
	  date-published:: [[04/16/2024]]
	  id:: 662359e1-3b69-43f1-bf61-2e600c194c8b
	  collapsed:: true
		- ### Content
		  collapsed:: true
			- \#\# TL;DR
			  
			  Communications over the Internet are secured by Transport Layer Security (TLS), a widely used security protocol. ==TLS enables privacy protection and data integrity between a client and a server, but it does not allow the client to prove the authenticity of the data to a third party.== This problem could be addressed by having the client authenticate a particular website to pass data directly to a third party website (e.g., Bob could allow his company’s payroll record, which stores verified birth dates, to share data with a third party). However, this would force the client to share more data than necessary (providing the exact birth date instead of the fact that a certain age limit is met), and such a solution would require the addition of new web infrastructures.
			  
			  ==Recent work introduces TLS oracles that address this challenge by cryptographically validating the data.== Acting as a bridge between Web2 and Web3, TLS oracles could liberate private web data and facilitate their integration into Web3 smart contracts.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/700x661,soTcrH_yAv5uWITL_BBryApBcR7Ie9n3RtFgjbaoyAwU/https://miro.medium.com/v2/resize:fit:1400/1*TkMlAxJh3BynvfhxbTpJ2g.png)
			  
			  Projects working on TLS Oracles (not comprehensive; as of April 16, 2024)
			  
			  The illustration above provides a non-exhaustive overview of projects exploring TLS oracles and complementary services. As the space is still young and rapidly evolving, it’s likely that this illustration could change considerably in just a few months.
			  
			  This blog post begins with a brief introduction to TLS, followed by an explanation of TLS oracles and their use cases. Subsequently, it will offer an overview of ongoing research efforts and projects in this field.
			  
			  \#\# **Transport Layer Security (TLS)**
			  
			  \#\# What is TLS?
			  
			  As outlined by [Cloudflare](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/), ==Transport Layer Security is a widely adopted security protocol designed to ensure privacy and data security in Internet communications.== Its predominant application involves encrypting communication between web applications and servers, such as web browsers accessing websites. ==At the core of TLS there is a process called== ==[TLS handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)====, in which the server and the client cryptographically secure their communication channel (i.e., when a user accesses a website using TLS, the TLS handshake begins between the user’s device and the web server)==.
			  
			  The latest iteration, [TLS 1.3](https://www.cloudflare.com/learning/ssl/why-use-tls-1.3/), was published in 2018.
			  
			  \#\# **Why is TLS important?**
			  
			  In the past, data was often transmitted unencrypted over the internet. Without TLS, sensitive data such as logins, credit card details and personal information are vulnerable to interception. This vulnerability extends to various online activities, including browsing habits, email exchanges, chats and conference calls. Implementing TLS in client and server applications guarantees the encryption of transmitted data with robust algorithms, protecting it from unauthorized access by third parties.
			  
			  \#\# **TLS Oracles**
			  
			  As previously mentioned, TLS doesn’t allow a client to prove to a third party that certain data authentically came from a particular website. As a result, data use is often restricted to its point of origin.
			  
			  Recent advancements have introduced TLS oracles as a solution to this problem by cryptographically confirming the origin of digital content. As also described by [Ernstberger et al](https://eprint.iacr.org/2024/447.pdf), ==these oracles extend a standard TLS connection with a third party (through a three-party handshake) and ensure that the third party remains unaware of the content exchanged between a client and a server.== The third party acts solely as a verifier, validating the exchanged data. This allows the client to prove that certain data authentically came from a particular website. Using Zero-Knowledge Proofs (ZKPs), the client can optionally prove statements about such data without revealing the data itself. Using the example described in the introduction, Bob could use the company’s payroll account to prove that a certain age requirement is met without revealing the actual date of birth.
			  
			  \#\# **Use cases of TLS Oracles**
			  
			  TLS oracles enable Web3 dApps to take advantage of data analytics and model intelligence from the traditional Internet economy and enable new use cases such as the following, as also outlined by [PADO Labs](https://docs.padolabs.org/category/use-cases) and [zkPass](https://zkpass.gitbook.io/zkpass/introduction/use-cases):
			  
			  * **Identity Verification**: Lightweight and reusable identity verification
			  * **Social Networking**: Interoperable and composable social graphs
			  * **Provable Assets**: Privacy-preserving attestation of crypto asset holdings
			  * **Crypto On & Off Ramps**: Reliable on-ramp and off-ramp with off-chain payment attestations
			  * **Undercollateralized DeFi Lending Protocol**: DeFi lending protocol combining on-chain and off-chain credit for lower collateralized borrowing opportunities, enhancing capital efficiency
			  * **Medical and Healthcare**: Facilitating secure and efficient sharing of medical records and health information between patients, healthcare providers, and other stakeholders
			  * **Insurance Claims**: ZKPs from private data during web sessions for insurance policy eligibility verification, enabling automatic claim settlement without manual review
			  
			  \#\# **Projects working on TLS Oracles**
			  
			  \#\# **Initial approaches with known limitations**
			  
			  ==Initial approaches to TLS oracles have known limitations==, as also mentioned by [Zhang et al](https://dl.acm.org/doi/pdf/10.1145/3372297.3417239). These methods either only work with outdated versions of TLS that do not provide privacy from the oracle (e.g., initial versions of [TLSNotary](https://tlsnotary.org/)), or depend on trusted hardware (e.g., [Town Crier](https://dl.acm.org/doi/pdf/10.1145/2976749.2978326); [zkPortal](https://zkportal.io/)). Other oracles rely on the cooperation of servers and require the installation of TLS extensions (e.g., [TLS-N](https://eprint.iacr.org/2017/578.pdf); [ROSEN](https://dl.acm.org/doi/pdf/10.1145/3474123.3486763)) or changes in the application layer logic (e.g. [draft-cavage-http-signatures-11](https://datatracker.ietf.org/doc/html/draft-cavage-http-signatures-11); [draft-yasskin-http-origin-signed-responses-05](https://datatracker.ietf.org/doc/html/draft-yasskin-http-origin-signed-responses-05)). Server-supported oracle methods have two main problems. First, they compromise compatibility with the legacy system, which is a major hurdle to widespread adoption. Second, such solutions offer only limited exportability, as web servers retain sole discretion over what data can be exported and can censor export attempts at their own will.
			  
			  \#\# **Recent projects where the verifier acts as an external entity**
			  
			  One of the earliest contributors to TLS Oracles is [TLSNotary](https://tlsnotary.org/). The TLSNotary protocol consists of 3 steps: ==First, the prover (i.e. client) requests data from a server over TLS while cooperating with the verifier in secure and privacy-preserving Multi-Party Computation (MPC). Second, the prover selectively discloses the data to the verifier. Third, the verifier verifies the data. In 2022, TLSNotary was rebuilt from the ground up.== This renewed version of the TLSNotary protocol offers enhanced security, privacy, and performance. It currently supports TLS 1.2\. TLS 1.3 support will be added in 2024\. [Shinkai Protocol](https://download.shinkai.com/Shinkai%5FProtocol%5FWhitepaper.pdf) is another protocol that takes great inspiration from the TLSNotary protocol, while adding several advancements on top specific to their AI network use case.
			  
			  [DECO](https://www.deco.works/) introduced the first TLS oracle for TLS version 1.2 without server-side adaptation (back then, TLSNotary did not support TLS 1.2). In DECO, most of client’s computation is emulated using a maliciously secure Two-Party Computation (2PC) protocol.They use an implementation of the authenticated garbling, a state-of-the-art malicious 2PC framework.
			  
			  [PADO Labs](https://padolabs.org/)introduced the garble-then-prove technique to achieve the same security requirement as DECO without using any heavy mechanism like generic malicious 2PC. Their end-to-end implementation shows 14× improvement in communication and an order of magnitude improvement in computation compared to DECO.
			  
			  Whilst PADO Labs’ protocol can be extended to TLS 1.3, the above protocols do not provide explicit security support for TLS 1.3\. Therefore, [DIDO](https://eprint.iacr.org/2023/1056.pdf) and [DiStefano](https://eprint.iacr.org/2023/1063.pdf) propose an optimized solution for TLS 1.3 websites. While DIDO uses semi-honest 2PC, DiStefano relies on maliciously secure 2PC, similar to DECO. The authors of DiStefano note that the use of semi-honest 2PC must be applied carefully to prevent loss of security.
			  
			  Other projects, such as [zkPass](https://zkpass.org/) and [Crunch](https://github.com/MarcusWentz/ethdenver2024), whose whitepapers do not mention the research outlined above, are also noteworthy.zkPass enables users to selectively and privately validate their private web data to the Web3 world. Crunch is an ETHDenver hackathon winner that focuses on reducing the cost of the MPC (to allow for more participants and thus greater security) by shifting as much of the security burden as possible from the MPC component to zero-knowledge Risc0 proofs. They support TLS 1.3.
			  
			  \#\# **Recent projects where the verifier acts as a proxy in between the client and the server**
			  
			  The proxy model, as also outlined by [Zhang et al](https://dl.acm.org/doi/pdf/10.1145/3372297.3417239), offers an alternative approach where the verifier serves as a proxy between the prover and the server (as opposed to an external entity). In this setup, the prover sends/receives messages to/from the server through the verifier. This alternative protocol presents a distinct trade-off between performance and security. It boasts high efficiency as it minimizes cryptographic operations post-handshake. However, it necessitates additional network assumptions (e.g., reliable connectivity between the proxy and the server throughout the session) and thus only withstands weaker network adversaries. For instance, the client could potentially execute a man-in-the-middle (MiTM) attack between itself and the server.
			  
			  The authors of DECO note that their model is efficient for small requests, but can be expensive for large records. Therefore, they describe an extension to their protocol where the verifier acts as a proxy between the server and the client to make it more efficient.
			  
			  [ORIGO](https://eprint.iacr.org/2024/447.pdf) provides a proposal similar to the proxy mode of DECO. They facilitate a TLS oracle that does not demand for any communication intensive 2PC. Therefore, they achieve constant communication, independent of the size of the data requested from an application server. They leverage that in TLS 1.3.
			  
			  [Janus](https://eprint.iacr.org/2023/1377.pdf) is also a TLS oracle designed specifically for TLS 1.3 within the proxy framework. Janus utilizes handshake and record layer 2PC, ensuring security against MiTM attacks in the proxy mode (as opposed to ORIGO which works under the assumption of a weaker network adversary and thus offers no MiTM security). Yet, the utilization of 2PC requires communication in size linear to the number of gates in the garbled circuit and is therefore not as scalable.
			  
			  [Reclaim Protocol](https://www.reclaimprotocol.org/) introduces an HTTP proxy to oversee the handshake and data transmission process. Every webpage accessed is routed through a randomly selected HTTP proxy witness. Notably, the HTTP proxy witness is not a MiTM attack. However, there are several trust assumptions involved, as users must install a mobile app to generate proofs using the HTTP proxy witness.
			  
			  \#\# **Limitations of TLS Oracles**
			  
			  As with most emerging technologies, TLS oracles have limitations in terms of scalability. As a result, TLS oracles are very effective at selectively verifying small amounts of private data, but are limited in their applicability to larger sensitive data sets. The proxy model is an alternative approach that allows for better scalability, but relies on more network assumptions to withstand a MiTM attack.
			  
			  \#\# **Complementary Services to TLS Oracles**
			  
			  To realize the vision of integrating Internet data into Web3 smart contracts and enhance their capabilities, TLS oracle providers are collaborating with other service providers as described below.
			  
			  \#\# **Attestation registries**
			  
			  Attestations are shared with attestation registries like [Verax](https://www.ver.ax/), [EAS](https://attest.sh/), [BAS](https://doc.bascan.io/) or [Sign Protocol](https://sign.global/). These registries serve as infrastructures, enabling the creation of diverse attestation schemas and assisting users in generating and distributing attestations (both on-chain or off-chain) to other on-chain entities.
			  
			  Taking the example of Sign protocol, it facilitates partners like PADO Labs or zkPass to:
			  
			  * Seamlessly integrate validation results onto any chain and bridge these results to other chains subsequent to the initial attestation
			  * Encrypt and permanently store the captured TLS session and zero-knowledge proof for archival purposes and future retrieval
			  * Enable any smart contract to access, correctly decode, and utilize validation results
			  
			  \#\# **ZK Coprocessors**
			  
			  TLS Oracles focus on harnessing the value of off-chain user data. In numerous scenarios, users’ off-chain data can be combined with on-chain data to meet business requirements, a common practice in on-chain scenarios. For instance, to engage with a permissioned DeFi protocol, Bob may need to establish a comprehensive profile comprising both off-chain identity records (e.g., the fulfillment of a certain age requirement) and on-chain data (e.g., historic trading volume).
			  
			  Furthermore, potential collaborations with other off-chain verification services, such as [KryptonZK](https://www.kryptonzk.com/https:/www.kryptonzk.com/), which offers zero-knowledge based verifiable analytics on private big data, would also be possible. Let’s take an example of a DeFi lending protocol where Bob needs to show that his average savings over the last months have been enough to access a P2P loan. Bob can attest the source of his financial data with a TLS oracle, and that the computed statistic (analytics), the average of his savings, has been computed correctly with an off-chain verification service like KryptonZK.
			  
			  \#\# **Closing Words**
			  
			  While TLS ensures secure access to web data, its limitation in validating data authenticity to a third party restricts the utilization of that data. TLS oracles, however, bridge this gap by cryptographically affirming the origin of digital content, liberating private data from the confines of centralized servers and expanding its integration into Web3 smart contracts.
			  
			  Projects like TLSNotary, DECO, and others have pioneered innovative approaches to TLS oracles, employing techniques such as three-party handshake, secure multi-party computation (MPC) and zero-knowledge proofs (ZKPs) to enhance security and privacy. These advancements open doors to diverse applications across various sectors, including identity verification, social networking, DeFi lending, or insurance claims settlement.
			  
			  Despite the progress made, TLS oracles still encounter limitations in validating larger data sets. However, ongoing research and development efforts continue to address these challenges and expand the possibilities of TLS oracles in the Web3 ecosystem.
			  
			  _Acknowledgements: This article expands upon the content presented in prior writings, such as_ [_Cloudflare_](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/)_,_ [_Ernstberger et al_](https://eprint.iacr.org/2024/447.pdf)_, and_ [_Zhang et al_](https://dl.acm.org/doi/pdf/10.1145/3372297.3417239)_._
			  
			  _Many thanks to_ [_Emanuele Ragnoli_](https://www.linkedin.com/in/emanuele-ragnoli-a33b57/) _for conversations around this topic._
			  
			  _If you have a project working on TLS oracles, or if you are using TLS oracles, please reach out!_
			  
			  _You could follow me on_ [_Twitter_](https://twitter.com/bastian%5Fwetzel) _and_ [_LinkedIn_](https://www.linkedin.com/in/bastian-wetzel/)_._
		- ### Highlights
		  collapsed:: true
			- > TLS enables privacy protection and data integrity between a client and a server, but it does not allow the client to prove the authenticity of the data to a third party. [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#41dfffe7-b0c4-44ae-9f01-43ecef5d8117)
			- > Recent work introduces TLS oracles that address this challenge by cryptographically validating the data. [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#16a2f814-2c08-4c0f-b393-a1dc3a4dbdde)
			- > Transport Layer Security is a widely adopted security protocol designed to ensure privacy and data security in Internet communications. [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#a18c4ef8-fbc6-4806-8bda-e922b7bdc7e5)
			- >  At the core of TLS there is a process called TLS handshake, in which the server and the client cryptographically secure their communication channel (i.e., when a user accesses a website using TLS, the TLS handshake begins between the user’s device and the web server) [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#424f51ad-b811-402d-baf8-917794a7fe9e)
			- > these oracles extend a standard TLS connection with a third party (through a three-party handshake) and ensure that the third party remains unaware of the content exchanged between a client and a server. [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#4165f554-d959-49e1-9486-f5410493cb20)
			- > Initial approaches to TLS oracles have known limitations [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#4669a01f-c2e6-471f-a93c-fd4b0b95f295)
			- > First, the prover (i.e. client) requests data from a server over TLS while cooperating with the verifier in secure and privacy-preserving Multi-Party Computation (MPC). Second, the prover selectively discloses the data to the verifier. Third, the verifier verifies the data. In 2022, TLSNotary was rebuilt from the ground up. [⤴️](https://omnivore.app/me/tls-oracles-liberating-private-web-data-with-cryptography-by-bas-18ef9c7b821#e6b0c83b-8348-4a64-a265-b22a64bc6b25)
		-
		-
		- tags::: #TLS, #TLSNotary
	- [What is Hindley-Milner? (and why is it cool?) - Code Commit](https://omnivore.app/me/what-is-hindley-milner-and-why-is-it-cool-code-commit-18ef3cb7dba)
	  collapsed:: true
	  site:: [web.archive.org](https://web.archive.org/web/20120630042941/http://www.codecommit.com/blog/scala/what-is-hindley-milner-and-why-is-it-cool)
	  date-saved:: [[04/18/2024]]
	  date-published:: [[12/28/2008]]
		- ### Content
		  collapsed:: true
			- \#\# [What is Hindley-Milner? (and why is it cool?)](https://web.archive.org/web/20120630042941/http://www.codecommit.com/blog/scala/what-is-hindley-milner-and-why-is-it-cool)
			  
			  Anyone who has taken even a cursory glance at the vast menagerie of programming languages should have at least heard the phrase “Hindley-Milner”. F\#, one of the most promising languages ever to emerge from the forbidding depths of Microsoft Research, makes use of this mysterious algorithm, as do Haskell, OCaml and ML before it. There is even some research being undertaken to find a way to apply the power of HM to optimize dynamic languages like Ruby, JavaScript and Clojure.
			  
			  However, despite widespread application of the idea, I have yet to see a decent layman’s-explanation for what the heck everyone is talking about. How does the magic actually work? Can you always trust the algorithm to infer the right types? Further, why is Hindley-Milner is better than (say) Java? So, while those of you who actually know what HM is are busy recovering from your recent aneurysm, the rest of us are going to try to figure this out.
			  
			  \#\#\# Ground Zero
			  
			  Functionally speaking, Hindley-Milner (or “Damas-Milner”) is an algorithm for inferring value types based on use. It literally formalizes the intuition that a type can be deduced by the functionality it supports. Consider the following bit of _psuedo_\-Scala (not a flying toy):
			  
			  | def foo(s: String) \= s.length   // note: no explicit types def bar(x, y) \= foo(x) \+ y |
			  | ---------------------------------------------------------------------------------------- |
			  
			  Just looking at the definition of `bar`, we can easily see that its type _must_ be `(String, Int)=>Int`. As humans, this is an easy thing for us to intuit. We simply look at the body of the function and see the two uses of the `x` and `y` parameters. `x` is being passed to `foo`, which expects a `String`. Therefore, `x` must be of type `String` for this code to compile. Furthermore, `foo` will return a value of type `Int`. The `+` method on class `Int` expects an `Int` parameter; thus, `y` must be of type `Int`. Finally, we know that `+` returns a new value of type `Int`, so there we have the return type of `bar`.
			  
			  This process is almost exactly what Hindley-Milner does: it looks through the body of a function and computes a _constraint set_ based on how each value is used. This is what we were doing when we observed that `foo` expects a parameter of type `String`. Once it has the constraint set, the algorithm completes the type reconstruction by unifying the constraints. If the expression is well-typed, the constraints will yield an unambiguous type at the end of the line. If the expression is not well-typed, then one (or more) constraints will be contradictory or merely unsatisfiable given the available types.
			  
			  \#\#\# Informal Algorithm
			  
			  The easiest way to see how this process works is to walk it through ourselves. The first phase is to derive the constraint set. We start by assigning each value (`x` and `y`) a _fresh_ type (meaning “non-existent”). If we were to annotate `bar` with these type variables, it would look something like this:
			  
			  | def bar(x: X, y: Y) \= foo(x) \+ y |
			  | ---------------------------------- |
			  
			  The type names are not significant, the important restriction is that they do not collide with any “real” type. Their purpose is to allow the algorithm to unambiguously reference the yet-unknown type of each value. Without this, the constraint set cannot be constructed.
			  
			  Next, we drill down into the body of the function, looking specifically for operations which impose some sort of type constraint. This is a depth-first traversal of the AST, which means that we look at operations with higher-precedence first. Technically, it doesn’t matter what order we look; I just find it easier to think about the process in this way. The first operation we come across is the dispatch to the `foo` method. We know that `foo` is of type `String=>Int`, and this allows us to add the following constraint to our set:
			  
			  _X_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `String`
			  
			  The next operation we see is `+`, involving the `y` value. Scala treats all operators as method dispatch, so this expression actually means “`foo(x).+(y)`. We already know that `foo(x)` is an expression of type `Int` (from the type of `foo`), so we know that `+` is defined as a method on class `Int` with type `Int=>Int` (I’m actually being a bit hand-wavy here with regards to what we do and do not know, but that’s an unfortunate consequence of Scala’s object-oriented nature). This allows us to add another constraint to our set, resulting in the following:
			  
			  _X_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `String`
			  
			  _Y_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `Int`
			  
			  The final phase of the type reconstruction is to _unify_ all of these constraints to come up with real types to substitute for the _X_ and _Y_ type variables. Unification is literally the process of looking at each of the constraints and trying to find a single type which satisfies them all. Imagine I gave you the following facts:
			  
			  * Daniel is tall
			  * Chris is tall
			  * Daniel is red
			  * Chris is blue
			  
			  Now, consider the following constraints:
			  
			  _Person1_ is tall
			  
			  _Person1_ is red
			  
			  Hmm, who do you suppose _Person1_ might be? This process of combining a constraint set with some given facts can be mathematically formalized in the guise of unification. In the case of type reconstruction, just substitute “types” for “facts” and you’re golden.
			  
			  In our case, the unification of our set of type constraints is fairly trivial. We have exactly one constraint per value (`x` and `y`), and both of these constraints map to concrete types. All we have to do is substitute “`String`” for “_X_” and “`Int`” for “_Y_” and we’re done.
			  
			  To really see the power of unification, we need to look at a slightly more complex example. Consider the following function:
			  
			  | def baz(a, b) \= a(b) :: b |
			  | -------------------------- |
			  
			  This snippet defines a function, `baz`, which takes a function and some other parameter, invoking this function passing the second parameter and then “cons-ing” the result onto the second parameter itself. We can easily derive a constraint set for this function. As before, we start by coming up with type variables for each value. Note that in this case, we not only annotate the parameters but also the return type. I sort of skipped over this part in the earlier example since it only sufficed to make things more verbose. Technically, this type is always inferred in this way.
			  
			  | def baz(a: X, b: Y): Z = a(b) :: b |
			  | ---------------------------------- |
			  
			  The first constraint we should derive is that `a` must be a function which takes a value of type _Y_ and returns some fresh type _Y’_ (pronounced like “_why prime_“). Further, we know that `::` is a function on class `List[A]` which takes a new element `A` and produces a new `List[A]`. Thus, we know that _Y_ and _Z_ must both be `List[Y']`. Formalized in a constraint set, the result is as follows:
			  
			  
			  _X_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) (_Y_`=>`_Y’_ )
			  
			  _Y_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `List[`_Y'_ `]`  
			  
			  _Z_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `List[`_Y'_ `]`
			  
			  Now the unification is not so trivial. Critically, the _X_ variable depends upon _Y_, which means that our unification will require at least one step:
			  
			  
			  _X_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) ( `List[`_Y'_ `]` `=>`_Y’_ )
			  
			  _Y_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `List[`_Y'_ `]`  
			  
			  _Z_ ![\mapsto](https://proxy-prod.omnivore-image-cache.app/0x0,s93ddAFGyjj8oGfFvocovu7WfaWU-wu8MCsrvffkQlbA/https://web.archive.org/web/20120630042941im_/http://alt1.artofproblemsolving.com/Forum/latexrender/pictures/2/2/3/22341972d49b9a3a22a4ef9220996604cf49ad12.gif) `List[`_Y'_ `]`
			  
			  This is the same constraint set as before, except that we have substituted the known mapping for _Y_ into the mapping for _X_. This substitution allows us to eliminate _X_, _Y_ and _Z_ from our inferred types, resulting in the following typing for the `baz` function:
			  
			  | def baz(a: List\[Y'\]\=>Y', b: List\[Y'\]): List\[Y'\] \= a(b) :: b |
			  | ------------------------------------------------------------------- |
			  
			  Of course, this still isn’t valid. Even assuming that `Y'` were valid Scala syntax, the type checker would complain that no such type can be found. This situation actually arises surprisingly often when working with Hindley-Milner type reconstruction. Somehow, at the end of all the constraint inference and unification, we have a type variable “left over” for which there are no known constraints.
			  
			  The solution is to treat this unconstrained variable as a type parameter. After all, if the parameter has no constraints, then we can just as easily substitute any type, including a generic. Thus, the final revision of the `baz` function adds an unconstrained type parameter “`A`” and substitutes it for all instances of _Y’_ in the inferred types:
			  
			  | def baz\[A\](a: List\[A\]\=>A, b: List\[A\]): List\[A\] \= a(b) :: b |
			  | -------------------------------------------------------------------- |
			  
			  \#\#\# Conclusion
			  
			  …and that’s all there is to it! Hindley-Milner is really no more complicated than all of that. One can easily imagine how such an algorithm could be used to perform far more complicated reconstructions than the trivial examples that we have shown.
			  
			  Hopefully this article has given you a little more insight into how Hindley-Milner type reconstruction works under the surface. This variety of type inference can be of immense benefit, reducing the amount of syntax required for type safety down to the barest minimum. Our “`bar`” example actually started with (coincidentally) Ruby syntax and showed that it still had all the information we needed to verify type-safety. Just a bit of information you might want to keep around for the next time someone suggests that all statically typed languages are overly-verbose.
	- [So You Still Don't Understand Hindley-Milner? Part 2 - Amit's Blog](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-2-amit-s-blog-18ef3ca1e6a)
	  collapsed:: true
	  site:: [Amit's (old) blog](https://legacy-blog.akgupta.ca/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-2/)
	  author:: Amit Kumar Gupta
	  labels:: [[fp]] [[functional programming]] [[hindley-milner]]
	  date-saved:: [[04/18/2024]]
	  date-published:: [[06/06/2013]]
		- ### Content
		  collapsed:: true
			- In [Part 1](https://legacy-blog.akgupta.ca/blog/2013/05/14/so-you-still-dont-understand-hindley-milner/), we said what the building blocks of the Hindley-Milner formalization would be, and in this post we'll thoroughly define them, and actually formulate the formalization:
			  
			  \#\# Formalizing the concept of an expression
			  
			  We'll give a [recursive definition](https://en.wikipedia.org/wiki/Recursive%5Fdefinition) of what an expression is; in other words, we'll ==state what the most basic kind of expression is, we'll say how to create new, more complex expressions out of existing expressions, and we'll say that only things made in this way are valid expressions.==  
			  
			  1. ==Variables are valid expressions.==
			  2. ==If== _==e==_ ==is any expression, and== _==x==_ ==is any variable, then== _==λ==_ _==x==_==.==_==e==_ ==is an expression.== Here it helps to think of e as typically (thought not necessarily) a more complex expression involving _x_, ==e.g.== _==x==_==2====+ 2====, and then== _==λ==_ _==x==_==.==_==e==_ ==as the anonymous function that takes an input== _==x==_ ==and returns the result of evaluating the expression== _==e==_ ==with the given value of== _==x==_==.== In other words, think of it like this:  
			  | 1 | function(x) { return x^2 \+ 2; } |  
			  | - | -------------------------------- |
			  3. ==If== _==f==_ ==and== _==e==_ ==are valid expressions, then== _==f==_==(==_==e==_==)== ==is a valid expression.== This is called Application, for obvious reasons.
			  4. If _x_ is a variable, and _e_1 and _e_0 are valid expressions, then substituting every occurrence of _x_ in _e_0 for _e_1 yields a valid expression. So, for example, if _e_1 is the expression _x_2 \+ 2, and _e_0 is the expression _y_ / 3, then if we let _x_ \= _e_0 in _e_1, we get the expression (_y_ / 3)2 \+ 2.  
			  \[NB: This last clause is redundant and not officially a part of the Lambda Calculus definition of an expression, as substituting _e_0 for _x_ in _e_1 is equivalent to applying the abstraction _λ_ _x_. _e_1 to _e_0. It's added to support something called [let-polymorphism](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner\#Let-polymorphism).\]
			  
			  And nothing else is a valid expression.
			  
			  Aside: anyone paying close attention will wonder, wait, how can I make any useful expressions out of this? How do I even get _x_2 \+ 2, or in fact 2 for that matter, out of the above? Heck, what about 0? There is nothing in the rules above which _obviously_ yield the expression 0. The solution is to create expressions in the Lambda Calculus which behave like 0, 1, …, + , × , − , /  when interpreted correctly. In other words, we have to encode numbers, numerical operations, strings, etc. into patterns we can create with the Lambda syntax. [This blog post](https://palmstroem.blogspot.com/2012/05/lambda-calculus-for-absolute-dummies.html) has a nice little section on numbers and numerical operations. This is a great feature of the Lambda Calculus: we have a simple syntax which we can define recursively in 4 simple clauses, and this therefore allows us to prove many things about it inductively in 4 main steps, yet the language itself has the expressive power to capture numbers, strings, and all the types and operations we could ever care about.
			  
			  \#\# Formalizing statements about types
			  
			  Let _e_ be any expression, that is, "_e_" is a variable in our meta-language which stands for any expression in our base language, like any of the following:
			  
			  | 1 2 3 | x Math.pow(x,2) \[1,2\].forEach ( function(x) { print(x); } ) |
			  | ----- | ------------------------------------------------------------- |
			  
			  Then if _t_ is any type, we can express "_e_ is of type _t_" by
			  
			  
			  _e_: _t_  
			  
			  Just like _e_, _t_ is a variable in our meta-language, and it can stand for any type in the base language, like Int, String, etc. And just like _e_, _t_ doesn't necessarily need to stand for any one type in particular.
			  
			  One can give a formal definition for what counts as a type, just as we did for expressions above. However the abstraction gets fairly twisted, so we'll leave it at that. I should just point out a few two key things to keep in mind:
			  
			  1. If _s_ and _t_ are types, then so is _t_ → _s_; it's the type of a function with _t_ inputs and _s_ outputs.
			  2. If _r_ is a type, possibly made up of other types (just as _t_ → _s_ is made up of _t_ and _s_, which could each potentially have been made up of other types), and _α_ is a variable for a type, then ∀ _α_. _r_ is a type. That makes no sense without an example:
			  
			  | 1 | function (x) { return x; } |
			  | - | -------------------------- |
			  
			  This function is type String → String. But it's also Int → Int. In fact, for any type _t_, it's type _t_ → _t_. We're gonna say that it has type ∀ _t_. _t_ → _t_. Each of the types String → String, _t_ → _t_, are "monotypes". ∀ _t_. _t_ → _t_ is a "polytype". The identity function above has the abstract polytype ∀ _t_. _t_ → _t_ which, in practice, means that for every real type _t_, it has type _t_ → _t_. If all of this has been sinking in, then we can compactly express this as:
			  
			  
			  _λ_ _x_. _x_: ∀ _α_. _α_ → _α_  
			  
			  \#\# Formalizing statements about statements about types
			  
			  Now we're going to want to formalize a bunch of rules for how we can go from some knowledge of expressions and their types to inferring types of more expressions. Remember how propositional calculus formalized Modus Ponens? We're going to do something similar. For instance, say we want to formalize the following piece of reasoning:
			  
			  > Suppose I've already been able to infer that a variable `duck` has type `Animal`.  
			  > Suppose furthermore that I've inferred that `speak` is a method of type `Animal -> String`.  
			  > Then I can infer that `speak(duck)` has type String.
			  > 
			  > And any reasoning of this form is valid type inference.
			  
			  We'll formalize that as follows:
			  
			  Γ⊢e0:τ→τ′ Γ⊢e1:τ\_
			  
			  Γ ⊢ _e_0(_e_1): _τ_ʹ
			  
			  That rule has the name \[App\] (for application), and it's one of the ones pictured in [that StackOverflow question](https://stackoverflow.com/questions/12532552/what-part-of-milner-hindley-do-you-not-understand). We'll talk about it and the rest of the rules in the next post. For now, let's first get a handle on all the symbols you see above:
			  
			  * Γ , this will stand for the collection of statements we already know, or perhaps, the statements we're assuming. More generally, Γ  should just be thought of as some collection of statements (about expressions and their types). And of course, there's nothing special about the letter Γ ; capital greek letters are commonly used for sets of statements however.
			  * ⊢ , the "turnstile", denotes that something can be inferred. For instance, Γ ⊢ _x_: _t_ says that if we take the statements in Γ  as our assumptions/axioms/current knowledge, then we can infer that _x_ has type _t_.
			  * ∈ , "epsilon", denotes membership. _x_: _t_ ∈ Γ  says that the statement _x_: _t_ is a member of Γ .
			  * That long horizontal bar; this line tells us that we can make the conclusion below the line if the things above the line are taken as premises in the argument. It lets us express things like, "if we can infer such and such, then we can infer such and such", e.g.
			  
			  If we can infer that _y_ has type _σ_ from Γ , then we can infer _x_ has type _τ_ from Γ .
			  
			  Next up:
			  
			  * [Part 3](https://legacy-blog.akgupta.ca/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-3/), where we put this all together and make sense of the inference rules used by the HM algorithm.
			  
			  \#\# Comments
		- ### Highlights
		  collapsed:: true
			- > state what the most basic kind of expression is, we'll say how to create new, more complex expressions out of existing expressions, and we'll say that only things made in this way are valid expressions. [⤴️](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-2-amit-s-blog-18ef3ca1e6a#ee8d68de-0d5d-4343-af51-e6eee871c768)
			- > Variables are valid expressions. [⤴️](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-2-amit-s-blog-18ef3ca1e6a#4bd9ee86-6d54-4f99-819c-841f41e9639e)
			- > If _e_ is any expression, and _x_ is any variable, then _λ_ _x_. _e_ is an expression. [⤴️](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-2-amit-s-blog-18ef3ca1e6a#5debf092-5f90-4869-9ec1-50ed0f50bb93)
			- > e.g. _x_2 \+ 2, and then _λ_ _x_. _e_ as the anonymous function that takes an input _x_ and returns the result of evaluating the expression _e_ with the given value of _x_. [⤴️](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-2-amit-s-blog-18ef3ca1e6a#488df998-f754-4f53-8aa5-fe2d9519c992)
			- > If _f_ and _e_ are valid expressions, then _f_(_e_) is a valid expression. [⤴️](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-2-amit-s-blog-18ef3ca1e6a#412835d4-2f0d-450b-ac2f-c7680547642a)
	- [So you still don't understand Hindley-Milner? Part 1 - Amit's Blog](https://omnivore.app/me/so-you-still-don-t-understand-hindley-milner-part-1-amit-s-blog-18ef3c9c5a5)
	  collapsed:: true
	  site:: [Amit's (old) blog](https://legacy-blog.akgupta.ca/blog/2013/05/14/so-you-still-dont-understand-hindley-milner/)
	  author:: Amit Kumar Gupta
	  labels:: [[fp]] [[functional programming]] [[hindley-milner]]
	  date-saved:: [[04/18/2024]]
	  date-published:: [[05/13/2013]]
		- ### Content
		  collapsed:: true
			- I was out for drinks with [Josh Long](https://twitter.com/starbuxman) and some other friends from work, when he found out I "speak math." He had come across [this StackOverflow question](https://stackoverflow.com/questions/12532552/what-part-of-milner-hindley-do-you-not-understand) and asked me what it meant:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/0x0,s4Nf4IR2I_UpMtc2FJk6U9CRUSl6GQZE-kNT2mCuSHnk/https://i.stack.imgur.com/hZhjl.png)
			  
			  Before we figure out what it means, let's get an idea for why we care in the first place. Daniel Spiewak's blog post (link broken) gives a really nice explanation of the purpose of the HM algorithm, in addition to an in-depth example of its application:
			  
			  > Functionally speaking, Hindley-Milner (or “Damas-Milner”) is an algorithm for inferring value types based on use. It literally formalizes the intuition that a type can be deduced by the functionality it supports.
			  
			  Okay, so we want to formalize an algorithm for inferring types of any given expression. In this post, I'm going to touch on what it means to formalize something, then describe the building blocks of the HM formalization. In [Part 2](https://legacy-blog.akgupta.ca/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-2/), I'll flesh out the building blocks of the formalization. Finally in [Part 3](https://legacy-blog.akgupta.ca/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-3/), I'll translate that StackOverflow question.  
			  
			  \#\# What it means to formalize
			  
			  Okay, so we want to talk about expressions. Arbitrary expressions. In an arbitrary language. And we want to talk about inferring types of these expressions. And we want to figure out rules for how we can infer types. And then we're going to want to make an algorithm that uses these rules to infer types. So we're going to need a meta-language. A language to talk about expressions in an arbitrary programming language. This meta-language should:
			  
			  * Be abstract and generic, so that it allows us to reason about statements of type inference purely based on their _form_ (hence, _formalization_), without having to worry about their content.
			  * Give a precise, unambiguous, yet intuitive definition for what an expression is.
			  * Give those definitions in terms of a small number of uncontroversial primitive concepts.
			  * Similarly give definitions for types, the idea that an expression has a type, and the idea that we can infer that a given expression has a given type.
			  * Lend itself to a simple, terse symbolic representation, e.g. rather than saying "the expression formed by applying the first expression to the second expression has the type of a function from strings to some type we don't care to specify in the current context" we could simply say "_e_1(_e_2): String → _t_".
			  * Be easily translated to something a computer can understand and implement, so we can ultimately automate type inference.
			  
			  To make all that a little more concrete, let's look at a really quick example of a formalization. If, instead of formalizing a language for talking about inferring types of expressions in an arbitrary programming language, what if we wanted to formalize a language for talking about truths of sentences in arbitrary natural languages? Without formalization, we might say something like
			  
			  > Suppose I know that if it's raining, Bob will carry an umbrella.  
			  > And suppose I also know that it's raining.  
			  > Then, I can conclude that Bob will carry an umbrella.
			  > 
			  > And any argument that takes this form is a valid way to reason.
			  
			  Propositional Calculus formalizes that whole things as a rule known as Modus Ponens:
			  
			  where _A_ and _B_ are variables representing propositions (a.k.a. sentences or clauses) in an arbitrary natural language.
			  
			  Okay, so let's enumerate the building blocks of the HM formalization:
			  
			  \#\# Building blocks of the formalization
			  
			  We will need:
			  
			  1. A formal way to talk about expressions. This formalization should meet the criteria enumerated above; for this purpose we use the **Lambda Calculus**. I'll be explaining that in a minute, but there's nothing crazy going on here.
			  2. A formal way to talk about types, and a formal way to talk about expressions and types together. After all, the purpose of the HM algorithm is to be able to deduce statements of the form "expression _e_ has type _t_".
			  3. A formal set of rules for deriving statements about expression types from other such statements. Rules along the lines of: "if I can already demonstrate that some expression has this type, and another expression has that type, then this third expression has this other type". _Such a set of rules is exactly what you're seeing in that SO question_. I'll be translating this in full detail.
			  4. An algorithm that intelligently uses the deduction rules to get from a starting point to deducing/inferring a desired conclusion statement: "the expression _e_ that I'm interested in has type _t_". This is the "algorithm" part for the "HM algorithm", and that's not something I'll be going into in these posts.
			  
			  Onward, ho!
			  
			  * [Part 2](https://legacy-blog.akgupta.ca/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-2/), wherein we thoroughly define points 1 and 2 above, and demystify the mathematical syntax.
			  * [Part 3](https://legacy-blog.akgupta.ca/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-3/), wherein we translate the type inference rules in point 3 above.
			  
			  \#\# Comments
	- [strawman:concurrency [ES Wiki]](https://omnivore.app/me/strawman-concurrency-es-wiki-18eedf4b141)
	  collapsed:: true
	  site:: [web.archive.org](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman%25253Aconcurrency)
	  date-saved:: [[04/17/2024]]
	  date-published:: [[06/16/2013]]
		- ### Content
		  collapsed:: true
			- The Wayback Machine - https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
			  
			  * [Communicating Event-Loop Concurrency and Distribution](\#communicating%5Fevent-loop%5Fconcurrency%5Fand%5Fdistribution)  
			   * [Concurrency Model and Promises](\#concurrency%5Fmodel%5Fand%5Fpromises)  
			   * [Vats](\#vats)  
			   * [Promises and Promise States](\#promises%5Fand%5Fpromise%5Fstates)  
			   * [Eventual Operations](\#eventual%5Foperations)  
			   * [Fundamental Static Q Methods](\#fundamental%5Fstatic%5Fq%5Fmethods)  
			   * [Promise methods](\#promise%5Fmethods)  
			   * [Syntactic Sugar](\#syntactic%5Fsugar)  
			   * [Non-fundamental Static Q Conveniences](\#non-fundamental%5Fstatic%5Fq%5Fconveniences)  
			         * [Q.delay](\#q.delay)  
			         * [Q.race](\#q.race)  
			         * [Q.all](\#q.all)  
			         * [Q.join](\#q.join)  
			         * [Q.memoize](\#q.memoize)  
			         * [Q.async](\#q.async)  
			         * [Q.defer()](\#q.defer)
			  * [Examples](\#examples)  
			   * [Infinite Queue](\#infinite%5Fqueue)  
			   * [Spawn](\#spawn)  
			   * [Vat.evalLater() as Async-PGAS](\#vat.evallater%5Fas%5Fasync-pgas)  
			   * [Open Vat](\#open%5Fvat)  
			   * [there](\#there)  
			   * [Map-Reduce Lite](\#map-reduce%5Flite)  
			   * [AMD Loader Lite](\#amd%5Floader%5Flite)
			  * [See](\#see)
			  
			  \#\# Communicating Event-Loop Concurrency and Distribution
			  
			  On both the browser and the server, JavaScript’s de-facto concurrency model is increasingly “shared nothing” communicating event loops. JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage. JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets. And server-side JavaScript has a rapidly growing role as the counterparty of these protocols, and increasingly uses event loops to service them. 
			  
			  This strawman consists of several major parts, not all of which need be accepted together.
			  
			  1. **Reality:** Codifying and formalizing JavaScript’s de-facto concurrency model as a de-jure model.
			  2. **Promises:** A way to  
			   * (**Q(p).post(), Q(p).get()**) Make asynchronous requests of objects that may not be synchronously reachable, such as remote objects.  
			   * (**Q(p).then()**) Ease the burden of local event loop programming, by reifying the ability to register a callback as a first class value.  
			   * (**[Q.async, yield](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:async%5Ffunctions "strawman:async_functions"):**) for implicit registration of shallow continuations on promises.
			  3. **Syntactic sugar**  
			   * **The infix “`!`” operator:** An eventual analog of “`.`“, for making eventual requests look more like immediate requests.
			  4. (**Q.makeFar()** and **Q.makeRemote()**) A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.  
			   * **Transport independence:** Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports.
			  5. (**Vat()**) An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.  
			   * **Worker independence:** Using `Vat`  
			   API  
			    as an abstraction layer around worker spawning on the browser or process spawning on the server.
			  6. (**Vat.evalLater(), there()**) Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops  
			   * **Symmetric Mobile Code:** Generalizes from the current use of JavaScript as mobile code sent only from server and only to browsers.
			  
			  \#\# Concurrency Model and Promises
			  
			  Aggregate objects into process-like units called _vats_. Objects in one vat can only send asynchronous messages to objects in other vats. _Promises_ represent such references to potentially remote objects. _Eventual message sends_ queue _eventual-deliveries_ in the work queue of the vat hosting the target object. A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a _turn_. A _then expression_ registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s _resolution_. The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			  
			  This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking. The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like `alert`). Without blocking, conventional deadlock is impossible. Of course, less conventional forms of race condition and deadlock bugs [remain](https://web.archive.org/web/20160404122250/http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html "http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html").
			  
			  \#\# Vats
			  
			  Partition the JavaScript reference graph into separate units, corresponding to prior concepts variously called vats, workers, processes, tanks, or grains. We adopt the “_vat_” terminology here for expository purposes. Vats are only asynchronously coupled to each other, and represent the minimal possible unit of concurrency, transparent distribution, orthogonal persistence, migration, partial failure, resource control, preemptive termination/deallocation, and defense from denial of service. Each vat consists of 
			  
			  * a single sequential thread of control,
			  * a single call-return stack,
			  * a single fifo queue holding _eventual-deliveries_,
			  * an internal object heap,
			  * and incoming and outgoing _remote references_.
			  
			  A vat’s thread of control dequeues the next eventual-delivery from the queue and processes it to completion before proceeding to the next. When the queue is empty, the vat is idle. 
			  
			  const vat = Vat(); //makes a new vat, as an object local to the creating vat.
			  // A Vat has an ''evalLater'' method that evaluates a Program in a turn of that vat.
			  // The ''evalLater'' method returns a promise for the evaluation's completion value.
			  const funP = vat.evalLater('' + function fun(x, y) { return x + y; }); // see below
			  const sumP = funP ! (3, 5); // sumP will eventually resolve to 8, unless...
			  const doneP = vat.terminateAsap(new Error('die')); // that vat is terminated before ''sumP'' is resolved.
			  // If the vat is terminated first, then ''sumP'' resolves to a //rejected// problem, with
			  // (Error: die) as its alleged reason for rejection.
			  // Once the vat is terminated, ''doneP'' will eventually resolve to ''true''.
			  
			  The vat object that represents a new vat is local to the creating vat, so that a vat may be terminated without waiting for that vat’s eventual-delivery queue to drain. 
			  
			  The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole. We intend that WebWorkers can be implemented in terms of vats and vice versa. However, when vats are built on WebWorkers, in the absence of some kind of weak reference and gc notification mechanism, it is probably impossible to arrange for the collection of distributed garbage. Even with them, [much more](https://web.archive.org/web/20160404122250/http://erights.org/history/original-e/dgc/index.html "http://erights.org/history/original-e/dgc/index.html") is needed to enable collection of distributed cyclic garbage. On the other hand, when vats are provided more primitively, multiple vats within an address space can be jointly within the purview of a single concurrent garbage collector, enabling full gc among these co-resident vats. However, truly distributed vats would still be faced with these same distributed garbage collection worries.
			  
			  The “` '' + function... `” trick above depends on [function\_to\_string](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:function%5Fto%5Fstring "harmony:function_to_string") to actually pass a string which is the program source for the function, while nevertheless having the function itself appear in the spawning program as code rather than as a literal string. This helps IDEs, refactoring tools, etc. A vat’s `evalLater` method evaluates that string as a program in a safe scope – a scope containing only the standard global variables such as `Object`, `Array`, etc. Except for these, the source passed in should be _closed_ – should not contain free references to any other variables. If the function is closed but for these standard globals, and these standard globals are not shadowed or replaced in the spawning context, then an IDE’s scope analysis of the code remains accurate.
			  
			  \#\# Promises and Promise States
			  
			  We introduce a new opaque type of object, the _Promise_ to represent potentially remote references. A normal JavaScript direct reference may only designate an object within the same vat. Only promises may designate objects in other vats. A promise may be in one of several states:
			  
			  [![](https://proxy-prod.omnivore-image-cache.app/0x0,s3MtXvPSlVBOuVroCwRCi7mcdELJGNC1uEMf9KMjOjhE/https://web.archive.org/web/20160404122250im_/http://wiki.ecmascript.org/lib/exe/fetch.php?w=&h=&cache=cache&media=strawman:refstates3.png)](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/lib/exe/detail.php?id=strawman%3Aconcurrency&cache=cache&media=strawman:refstates3.png "strawman:refstates3.png") (TODO: Revise diagram to replace “unresolved” with “pending” and “broken” with “rejected”.) 
			  
			  * _pending_ – when it is not yet determined what object the promise designates,  
			   * _pending-local_ – when the right to determine what the promise designates resides in the same vat,  
			   * _pending-remote_ – when that right is either in flight between vats or resides in a remote vat,
			  * _fulfilled_ – resolved to successfully designate some object,  
			   * _near_ – resolved to a direct reference to a local object,  
			   * _far_ – resolved to designate a remote object,
			  * _rejected_ – will never designate an object, for an alleged reason represented by an associated error.
			  
			  A promise may transition from _pending_ to any state. Additionally a promise can transition from _far_ to _rejected_. A _resolved_ promise can designate any non-promise value including primitives, `null`, and `undefined`. Primitives, `null`, `undefined`, and some objects are pass-by-copy. All other objects are pass-by-reference. A promise resolved to designate a pass-by-copy value is always near, i.e., it always designates a local copy of the value.
			  
			  If a function `foo` immediately returns either `X` or a promise which it later fulfills with `X`, we say that `foo` **_reveals_** `X`. Unless stated otherwise, we implicitly elide the error conditions from such statements. For the more explicit statement, append: _“or it throws, or it does not terminate, or it rejects the returned promise, or it never resolves the returned promise.”_ Put another way, such a function returns a **_reference_** to `X`, where by _reference_ we mean either `X` or a promise for `X`.
			  
			  \#\# Eventual Operations
			  
			  The existing JavaScript infix `.` (dot or _now_) operator enables synchronous interaction with the local object designated by a direct reference. We introduce a corresponding infix `!` (bang or _eventually_) operator for corresponding asynchronous interaction with objects eventually designated by either direct references or promises.
			  
			  Abstract Syntax: 
			  
			  Expression : ...
			      Expression ! [ Expression ] Arguments    // eventual send
			      Expression ! Arguments                   // eventual call
			      Expression ! [ Expression ]              // eventual get
			      Expression ! [ Expression ] = Expression // eventual put
			      delete Expression ! [ Expression ]       // eventual delete
			  
			  The `...` means “and all the normal right hand sides of this production. By “abstract” here I mean the distinction that must be preserved by parsing, i.e., in an [ast](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:ast "strawman:ast"), but without explaining the precedence and associativity which explains how this is unambiguously parsed. In all cases, the eventual form of an expression queues a eventual-delivery recording the need to perform the corresponding immediate form in the vat hosting the (eventually) designated object. The eventual form immediately evaluates to a promise for the result of eventually performing this eventual-delivery.
			  
			  function add(x, y) { return x + y; }
			  const sumP = add ! (3, 5); //sumP resolves in a later turn to 8.
			  
			  Attempted Concrete Syntax: 
			  
			  MemberExpression : ...
			      MemberExpression [nlth] ! [ Expression ]
			      MemberExpression [nlth] ! IdentifierName
			  CallExpression : ...
			      CallExpression [nlth] ! [ Expression ] Arguments
			      CallExpression [nlth] ! IdentifierName Arguments
			      MemberExpression [nlth] ! Arguments
			      CallExpression [nlth] ! Arguments
			      CallExpression [nlth] ! [ Expression ]
			      CallExpression [nlth] ! IdentifierName
			  UnaryExpression : ...
			      delete CallExpression [nlth] ! [ Expression ]
			      delete CallExpression [nlth] ! IdentifierName
			  LeftHandSideExpression :
			      Identifier
			      CallExpression [ Expression ]
			      CallExpression . IdentifierName
			      CallExpression [nlth] ! [ Expression ]
			      CallExpression [nlth] ! IdentifierName
			  
			  “`[nlth]`” above is short for “`[No LineTerminator here]`“, in order to unambiguously distinguish infix from prefix bang in the face of automatic semicolon insertion. 
			  
			  \#\# Fundamental Static Q Methods
			  
			  | Q(target) \-> targetP                         | Lifts the target argument into a promise designating the same object. If target is already a promise, then that promise is returned. (A promise for promise for T simplifies into a promise for T. Category theorists will be more pleased than Type theorists ;).)                                |
			  | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
			  | Q.reject(reason) \-> rejectedP                | Makes and returns a fresh _rejected_ promise recording (a sanitized form of) reason as the alleged reason for rejection. reason should generally be an immutable pass-by-copy Error object.                                                                                                        |
			  | Q.promise(f(resolve,reject)\->()) \-> promise | Makes a fresh promise, where the promise is initially _pending-local_, and the argument function f is called with resolve and reject functions for resolving this promise.                                                                                                                         |
			  | Q.isPromise(target) \-> boolean               | Is target a promise? If not, then using target as a target in the various promise operations is still equivalent to using Q(target), i.e., the promise operations will automatically lift all values to promises.                                                                                  |
			  | Q.makeFar(handler, nextSlotP) \-> promise     | Makes a resolved _far_ reference, which can only [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to _rejected_.                                                                          |
			  | Q.makeRemote(handler, nextSlotP) \-> promise  | Makes an _pending-remote_ promise, which can [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to anything.                                                                                |
			  | Q.ahorten(target1) \-> target2                | Returns the currently most resolved form of target1\. If target1 is a _fulfilled_ promise, return its resolution. If target1 is a promise that is following promise target2, then return target2. If target1 is a terminal _pending_ or _rejected_ promise, or a non-promise, then return target1. |
			  
			  Additional non-fundamental static Q convenience methods appear below.
			  
			  \#\# Promise methods
			  
			  Assuming `p` is a promise 
			  
			  | p.get(name) \-> valueP                    | Returns a promise for the result of eventually getting the value of the name property of target.                                                                                                                                               |
			  | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
			  | p.post(opt\_name, args) \-> resultP       | Eventually invoke the named method of target with these args. Returns a promise for what the result will be.                                                                                                                                   |
			  | p.send(opt\_name, ...args) \-> resultP    | p.send(m, a, b) is equivalent to p.post(m, \[a,b\])                                                                                                                                                                                            |
			  | p.fcall(...args) \-> resultP              | p.fcall(a, b) is equivalent to p.post(void 0, \[a,b\])                                                                                                                                                                                         |
			  | p.put(name, value) \-> voidP              | Eventually set the value of the name property of target to value. Return a promise-for-undefined, used for indicating completion.                                                                                                              |
			  | p.delete(name) \-> trueP                  | Eventually delete the name property of target. Returns a promise for the boolean result.                                                                                                                                                       |
			  | p.then(success, opt\_failure) \-> resultP | Registers functions success and opt\_failure to be called back in a later turn once target is _resolved_. If _fulfilled_, call success(resolution). Else if _rejected_, call opt\_failure(reason). Return a promise for the callback’s result. |
			  | p.end()                                   | If p resolves to _rejected_, log the reason to wherever uncaught exceptions go on this platform, e.g., onerror(reason).                                                                                                                        |
			  
			  
			  \#\# Syntactic Sugar
			  
			  | Abstract Syntax  | Expansion          | Simple Case  | Expansion            | JSON/RESTful equiv        |
			  | ---------------- | ------------------ | ------------ | -------------------- | ------------------------- |
			  | x ! \[i\](y, z)  | Q(x).send(i, y, z) | x ! p(y, z)  | Q(x).send(’p’, y, z) | POST https://...q=p {...} |
			  | x ! (y, z)       | Q(x).fcall(y, z)   | \-           | \-                   | POST https://... {...}    |
			  | x ! \[i\]        | Q(x).get(i)        | x ! p        | Q(x).get(’p’)        | GET https://...q=p        |
			  | x ! \[i\] = v    | Q(x).put(i, v)     | x ! p = v    | Q(x).put(’p’, v)     | PUT https://...q=p {...}  |
			  | delete x ! \[i\] | Q(x).delete(i)     | delete x ! p | Q(x).delete(’p’)     | DELETE https://...q=p     |
			  
			  
			  \#\# Non-fundamental Static Q Conveniences
			  
			  \#\#\# Q.delay
			  
			  Reveal the `answer` sometime after `millis` milliseconds have elapsed.
			  
			  Q.delay = function(millis, answer = undefined) {
			    return Q.promise(resolve => {
			      setTimeout(() => resolve(answer), millis);
			    });
			  };
			  
			  \#\#\# Q.race
			  
			  Given a list of promises, returns a promise for the resolution of whichever promise we notice has completed first.
			  
			  Q.race = function(answerPs) {
			    return Q.promise((resolve,reject) => {
			      for (answerP of answerPs) {
			        Q(answerP).then(resolve,reject);
			      };
			    });
			  };
			  
			  We can compose `Q.race`, `Q.delay`, and `Q.reject` to timeout eventual requests.
			  
			  var answer = Q.race([bob ! foo(carol), 
			                       Q.delay(5000, Q.reject(new Error("timeout")))]);
			  
			  \#\#\# Q.all
			  
			  Often it’s useful to collect several promised answers, in order to react either when _all_ the answers are ready or when _any_ of the promises becomes _rejected_.
			  
			  Q.all = function(answerPs) {
			    let countDown = answerPs.length;
			    const answers = [];
			    if (countDown === 0) { return Q(answers); }
			    return Q.promise((resolve,reject) => {
			      answerPs.forEach((answerP, index) => {
			        Q(answerP).then(answer => {
			          answers[index] = answer;
			          if (--countDown === 0) { resolve(answers); }
			        }, reject);
			      });
			    });
			  };
			  
			  We can compose `Q.all`, `then`, and [destructuring](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:destructuring "harmony:destructuring") to delay until several operands are revealed
			  
			  var sumP = Q.all([xP, yP]).then(([x, y]) => (x + y);
			  
			  \#\#\# Q.join
			  
			  Join is our eventual equality operation. Any messages sent on the join of `xP` and `yP` are only delivered if `xP` and `yP` both reveal the same target, in which case these messages are eventually delivered to that target and this joined promise itself eventually becomes fulfilled to designate that target. Otherwise, all these messages are discarded with the usual rejected promise contagion.
			  
			  Q.join = function(xP, yP) {
			    return Q.all([xP, yP]).then(([x, y]) => {
			      if (Object.is(x, y)) {
			        return x;
			      } else {
			        throw new Error("not the same");
			      }
			    });
			  };
			  
			  \#\#\# Q.memoize
			  
			  `Q.memoize` of a one argument function returns a new similar one argument function, except that it (eventually) calls the original function no more than once for each such argument. The memo function immediately returns a promise for what the original function will eventually return. Equivalence of arguments is determined by the optional memoMap passed in, which defaults to a new WeakMap() if absent. (Passing a memoMap in also allows the caller to seed the mapping with some prior associations.)
			  
			  The difference from a traditional synchronous memoizer is that the original function is called _eventually_ after a promise for its result is _already_ memoized, enabling cycles to work. For example, if memoF === memoize(f) and f(x) calls memoF(x), then an outer call to memoF(x) schedules an eventual call to f(x) which makes an inner call to memoF(x). Both outer and inner calls to memoF(x) returns a promise for what f(x) will eventually return.
			  
			   Q.memoize = function(oneArgFuncP, memoMap = WeakMap()) {
			  
			     return function oneArgMemo(arg) {
			       if (memoMap.has(arg)) {
			         return memoMap.get(arg);
			       } else {
			         const resultP = oneArgFuncP ! (arg);
			         memoMap.set(arg, resultP);
			         return resultP;
			       }
			     }
			   };
			  
			  \#\#\# Q.async
			  
			  \#\#\# Q.defer()
			  
			  (Will likely be deprecated)
			  
			  Q.defer = function() {
			    const deferred = {};
			    deferred.promise = Q.promise((resolve,reject) => {
			      deferred.resolve = resolve;
			      deferred.reject = reject;
			    });
			    return deferred;
			  };
			  
			  \#\# Examples
			  
			  \#\# Infinite Queue
			  
			  function makeQueue() {
			    let rear;
			    let front = Q.promise(r => { rear = r; });
			    return Object.freeze({
			      enqueue: function(elem) {
			        let nextRear;
			        const nextTail = Q.promise(r => { nextRear = r; });
			        rear({head: elem, tail: nextTail});
			        rear = nextRear;
			      },
			      dequeue: function() {
			        const result = front ! head;
			        front = front ! tail;
			        return result;
			      }
			    });
			  }
			  
			  Calling `queue.dequeue()` will return a promise for the next element that has or will be enqueued.
			  
			  \#\# Spawn
			  
			  The following `spawn` function is a simple abstraction built on `Vat` and `then` that captures a simple common case:
			  
			  function spawn(src) {
			    const vat = Vat();
			    const resultP = vat.evalLater(src);
			    Q(resultP).then(function(_) {
			      vat.terminateAsap(new Error('done')); 
			    });
			    return resultP;
			  }
			  
			  const sumP = spawn('3+5'}); // sumP eventually resolves to 8.
			  
			  The argument string to `spawn` is evaluated in a new Vat spawned for that purpose. Spawn returns a promise for what that string will evaluate to. Once that promise resolves, the spawned vat is shut down.
			  
			  \#\# Vat.evalLater() as Async-PGAS
			  
			  In the Async-PGAS language X10, the “at” statement is defined as
			  
			  // x10 grammar, not javascript or proposed javascript
			  Statement :
			    at ( PlaceExpression ) Statement
			  
			  The “at” statement first evaluates the PlaceExpression to a place, which is analogous to a vat. It then evaluates the Statement at that place. The statement evaluates with the lexical scope containing the “at” statement, so the locality of the values bound to these identifiers is the locality they have at that place rather than at the location containing the “at” statement. Although the argument to `Vat.evalLater` must be a closed expression (modulo whitelisted globals), we can get the same effect, a bit more verbosely, by passing in these bindings as an explicit eventual call to a closed function.
			  
			  For example, the X10-ish program
			  
			  const x = 6;
			  let ultimateP;
			  at (place) { ultimateP = x*7; }
			  
			  can be expressed using `aVat.evalLater()` as
			  
			  const x = 6;
			  let ultimateP = place.evalLater(x => x*7) ! (x);
			  
			  \#\# Open Vat
			  
			  The `makeOpenVat` function makes an `OpenVat` function. Each `OpenVat` function is like the `Vat` function, in that both make an return a new vat instance. Each `OpenVat` function additionally maintains a side table mapping from all incoming remote promises to the evaluation function of the open vat made by this `OpenVat` function. The reason we call such vats _open_ is that, given a remote promise into such a vat and the `OpenVat` function that made that vat, one can thereby inject new code into that vat.
			  
			  TODO: explain the membrane variation used below.
			  
			  function makeOpenVat() {
			    const wm = WeakMap();
			  
			    function OpenVat() {
			      const vat = Vat();
			      const openVat = Object.freeze({
			        evalLater: makeMembraneX(
			          vat.evalLater,
			          { registerRemote: function(remote) {
			              wm.set(remote, openVat.evalLater); }}),
			        terminateAsap: vat.terminateAsap
			      });
			      return openVat;
			    }
			    OpenVat.evalAt = function(p, src) {
			      return wm.get(p)(src);
			    };
			    return OpenVat;
			  }
			  
			  \#\# there
			  
			  The `there(p, ...)` function is analogous to `Q(p).then(...)`, except instead of merely relocating the execution of the callback in time till after `p` is resolved, it further relocates it in spacetime, to where and when p is near. Like `Q(p).then(...)`, `there` immediately returns a promise for the eventual outcome. We do not likewise relocate the errback, so that we can still notify it and it can still react on the requesting side to a partition between the requestor and `p`‘s host.
			  
			  function there(p, callback, opt_errback) {
			    var callbackP = OpenVat.evalAt(p, '' + callback);
			    return (callbackP ! (p)).then(
			      v => v,
			      opt_errback);
			  }
			  
			  \#\# Map-Reduce Lite
			  
			  Given an initial result value, a list of potentially remote promises to elements, a closed mobile `mapper` function from elements to mapped result values, and an associative / commutative `reducer` function from pairs of references to result values to a new reference to a result value, `mapReduce` immediately returns a promise for the result of mapping all the elements where they live, and reducing all of these results together with the initial result value to a result.
			  
			  I call this “Map-Reduce Lite” because, unlike a production map-reduce infrastructure, the following `mapReduce` does all reductions on the spawning machine, which is therefore a scaling bottleneck, and has none of the fault-tolerance. Here, any failure causes the overall map-reduce to fail, i.e., the returned promise becomes a _rejected_ promise. The mapper and reduction functions are like the conventional functional programming variety, rather than the map-reduce variety which arranged for group-by keys.
			  
			  /**
			   * Type/Guard syntax below is only a placeholder, not a serious proposal.
			   * @param initValue ::T2
			   * @param elemPs    ::Array[Ref[T1]]  // i.e., Array[T1 | Promise[T1]]
			   * @param mapper    ::(T1 -> T2)      // closed mobile function
			   * @param reducer   ::(T2 x T2 -> T2)
			   * @reveals         ::T2              // i.e., @returns ::Ref[T2]
			   */
			  function mapReduce(initValue, elemPs, mapper, reducer) {
			    let countDown = elemPs.length;
			    if (countDown === 0) { return initValue; }
			    let result = initValue;
			  
			    return Q.promise((resolve, reject) => {
			      elemPs.forEach(elemP => {
			        const mappedP = there(elemP, mapper);
			        Q(mappedP).then(mapped => {
			          result = reducer(result, mapped);
			          if (--countDown === 0) { resolve(result); }
			        }, reject);
			      });
			    });
			  }
			  
			  \#\# AMD Loader Lite
			  
			  This is a minimal Asynchronous Module Definition (AMD) Loader for a subset of the [AMD API](https://web.archive.org/web/20160404122250/https://github.com/amdjs/amdjs-api/wiki/AMD "https://github.com/amdjs/amdjs-api/wiki/AMD") specification. In this subset, `define` is called with exactly two arguments, a `dependencies` list of module names, and a factory function with one parameter per dependency.
			  
			   function makeSimpleAMDLoader(fetch, moduleMap = Map()) {
			     var loader;
			  
			     function rawLoad(id) {
			       return Q(fetch(id)).then(src => {
			         var result = Q.reject(new Error('"define" not called by: ' + id));
			         function define(deps, factory) {
			           result = Q.all(deps.map(loader)).then(imports => {
			             return factory(...imports);
			           });
			         }
			         define.amd = {lite: true};
			  
			         Function('define', src)(define);
			         return result;
			       });
			     }
			     return loader = Q.memoize(rawLoad, moduleMap);
			   }
			  
			  If module “W” depends on modules “X”, “Y”, and “Z”, then only once the promises for the “X”, “Y”, and “Z” modules have all been fulfilled will the “W” factory function be called with these module instances. The result of calling this factory function will then become the “W” module instance.
			  
			  What it means to _be_ the source for the “W” module is that `fetch(”W”)` will eventually return that source string. For example, a given `fetch` function might fetch it from `https://example.com/prefix/W.js`.
			  
			  // At https://example.com/prefix/W.js
			  define(['X', 'Y', 'Z'], function(X, Y, Z) {
			    return X(Y, Z);
			  })
			  
			  Since the memo mapping we need is from module names, which are strings rather than objects, we need to provide an explicit memoMap argument to `Q.memoize`, which should be a map that accepts strings as keys.
			  
			  Although the cycle tolerance of `Q.memoize` is generally useful, here it hurts. Because `define` won’t call the factory function until all (`Q.all`) of the dependencies are fulfilled, any cyclic AMD dependencies cause an undetected deadlock. Still, in the naive absence of this cycle tolerance, such cyclic dependencies would have instead caused an infinite regress which is even worse. Better would be cycle detection and rejection, which we leave as an exercise for the reader. 
			  
			  \#\# See
	- [strawman:concurrency [ES Wiki]](https://omnivore.app/me/strawman-concurrency-es-wiki-18d971e3b8d)
	  collapsed:: true
	  site:: [web.archive.org](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman%253Aconcurrency)
	  date-saved:: [[02/11/2024]]
	  date-published:: [[06/16/2013]]
		- ### Content
		  collapsed:: true
			- The Wayback Machine - https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
			  
			  * [Communicating Event-Loop Concurrency and Distribution](\#communicating%5Fevent-loop%5Fconcurrency%5Fand%5Fdistribution)  
			   * [Concurrency Model and Promises](\#concurrency%5Fmodel%5Fand%5Fpromises)  
			   * [Vats](\#vats)  
			   * [Promises and Promise States](\#promises%5Fand%5Fpromise%5Fstates)  
			   * [Eventual Operations](\#eventual%5Foperations)  
			   * [Fundamental Static Q Methods](\#fundamental%5Fstatic%5Fq%5Fmethods)  
			   * [Promise methods](\#promise%5Fmethods)  
			   * [Syntactic Sugar](\#syntactic%5Fsugar)  
			   * [Non-fundamental Static Q Conveniences](\#non-fundamental%5Fstatic%5Fq%5Fconveniences)  
			         * [Q.delay](\#q.delay)  
			         * [Q.race](\#q.race)  
			         * [Q.all](\#q.all)  
			         * [Q.join](\#q.join)  
			         * [Q.memoize](\#q.memoize)  
			         * [Q.async](\#q.async)  
			         * [Q.defer()](\#q.defer)
			  * [Examples](\#examples)  
			   * [Infinite Queue](\#infinite%5Fqueue)  
			   * [Spawn](\#spawn)  
			   * [Vat.evalLater() as Async-PGAS](\#vat.evallater%5Fas%5Fasync-pgas)  
			   * [Open Vat](\#open%5Fvat)  
			   * [there](\#there)  
			   * [Map-Reduce Lite](\#map-reduce%5Flite)  
			   * [AMD Loader Lite](\#amd%5Floader%5Flite)
			  * [See](\#see)
			  
			  \#\# Communicating Event-Loop Concurrency and Distribution
			  
			  On both the browser and the server, JavaScript’s de-facto concurrency model is increasingly “shared nothing” communicating event loops. JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage. JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets. And server-side JavaScript has a rapidly growing role as the counterparty of these protocols, and increasingly uses event loops to service them. 
			  
			  This strawman consists of several major parts, not all of which need be accepted together.
			  
			  1. **Reality:** Codifying and formalizing JavaScript’s de-facto concurrency model as a de-jure model.
			  2. **Promises:** A way to  
			   * (**Q(p).post(), Q(p).get()**) Make asynchronous requests of objects that may not be synchronously reachable, such as remote objects.  
			   * (**Q(p).then()**) Ease the burden of local event loop programming, by reifying the ability to register a callback as a first class value.  
			   * (**[Q.async, yield](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:async%5Ffunctions "strawman:async_functions"):**) for implicit registration of shallow continuations on promises.
			  3. **Syntactic sugar**  
			   * **The infix “`!`” operator:** An eventual analog of “`.`“, for making eventual requests look more like immediate requests.
			  4. (**Q.makeFar()** and **Q.makeRemote()**) A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.  
			   * **Transport independence:** Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports.
			  5. (**Vat()**) An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.  
			   * **Worker independence:** Using `Vat`  
			   API  
			    as an abstraction layer around worker spawning on the browser or process spawning on the server.
			  6. (**Vat.evalLater(), there()**) Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops  
			   * **Symmetric Mobile Code:** Generalizes from the current use of JavaScript as mobile code sent only from server and only to browsers.
			  
			  \#\# Concurrency Model and Promises
			  
			  Aggregate objects into process-like units called _vats_. ==Objects in one vat can only send asynchronous messages to objects in other vats.== _==Promises==_ ==represent such references to potentially remote objects.== _==Eventual message sends==_ ==queue== _==eventual-deliveries==_ ==in the work queue of the vat hosting the target object.== A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a _turn_. A _then expression_ registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s _resolution_. The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
			  
			  This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking. The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like `alert`). Without blocking, conventional deadlock is impossible. Of course, less conventional forms of race condition and deadlock bugs [remain](https://web.archive.org/web/20160404122250/http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html "http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html").
			  
			  \#\# Vats
			  
			  Partition the JavaScript reference graph into separate units, corresponding to prior concepts variously called vats, workers, processes, tanks, or grains. We adopt the “_vat_” terminology here for expository purposes. Vats are only asynchronously coupled to each other, and represent the minimal possible unit of concurrency, transparent distribution, orthogonal persistence, migration, partial failure, resource control, preemptive termination/deallocation, and defense from denial of service. Each vat consists of 
			  
			  * a single sequential thread of control,
			  * a single call-return stack,
			  * a single fifo queue holding _eventual-deliveries_,
			  * an internal object heap,
			  * and incoming and outgoing _remote references_.
			  
			  A vat’s thread of control dequeues the next eventual-delivery from the queue and processes it to completion before proceeding to the next. When the queue is empty, the vat is idle. 
			  
			  const vat = Vat(); //makes a new vat, as an object local to the creating vat.
			  // A Vat has an ''evalLater'' method that evaluates a Program in a turn of that vat.
			  // The ''evalLater'' method returns a promise for the evaluation's completion value.
			  const funP = vat.evalLater('' + function fun(x, y) { return x + y; }); // see below
			  const sumP = funP ! (3, 5); // sumP will eventually resolve to 8, unless...
			  const doneP = vat.terminateAsap(new Error('die')); // that vat is terminated before ''sumP'' is resolved.
			  // If the vat is terminated first, then ''sumP'' resolves to a //rejected// problem, with
			  // (Error: die) as its alleged reason for rejection.
			  // Once the vat is terminated, ''doneP'' will eventually resolve to ''true''.
			  
			  The vat object that represents a new vat is local to the creating vat, so that a vat may be terminated without waiting for that vat’s eventual-delivery queue to drain. 
			  
			  The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole. We intend that WebWorkers can be implemented in terms of vats and vice versa. However, when vats are built on WebWorkers, in the absence of some kind of weak reference and gc notification mechanism, it is probably impossible to arrange for the collection of distributed garbage. Even with them, [much more](https://web.archive.org/web/20160404122250/http://erights.org/history/original-e/dgc/index.html "http://erights.org/history/original-e/dgc/index.html") is needed to enable collection of distributed cyclic garbage. On the other hand, when vats are provided more primitively, multiple vats within an address space can be jointly within the purview of a single concurrent garbage collector, enabling full gc among these co-resident vats. However, truly distributed vats would still be faced with these same distributed garbage collection worries.
			  
			  The “` '' + function... `” trick above depends on [function\_to\_string](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:function%5Fto%5Fstring "harmony:function_to_string") to actually pass a string which is the program source for the function, while nevertheless having the function itself appear in the spawning program as code rather than as a literal string. This helps IDEs, refactoring tools, etc. A vat’s `evalLater` method evaluates that string as a program in a safe scope – a scope containing only the standard global variables such as `Object`, `Array`, etc. Except for these, the source passed in should be _closed_ – should not contain free references to any other variables. If the function is closed but for these standard globals, and these standard globals are not shadowed or replaced in the spawning context, then an IDE’s scope analysis of the code remains accurate.
			  
			  \#\# Promises and Promise States
			  
			  We introduce a new opaque type of object, the _Promise_ to represent potentially remote references. A normal JavaScript direct reference may only designate an object within the same vat. Only promises may designate objects in other vats. A promise may be in one of several states:
			  
			  [![](https://proxy-prod.omnivore-image-cache.app/0x0,s3MtXvPSlVBOuVroCwRCi7mcdELJGNC1uEMf9KMjOjhE/https://web.archive.org/web/20160404122250im_/http://wiki.ecmascript.org/lib/exe/fetch.php?w=&h=&cache=cache&media=strawman:refstates3.png)](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/lib/exe/detail.php?id=strawman%3Aconcurrency&cache=cache&media=strawman:refstates3.png "strawman:refstates3.png") (TODO: Revise diagram to replace “unresolved” with “pending” and “broken” with “rejected”.) 
			  
			  * _pending_ – when it is not yet determined what object the promise designates,  
			   * _pending-local_ – when the right to determine what the promise designates resides in the same vat,  
			   * _pending-remote_ – when that right is either in flight between vats or resides in a remote vat,
			  * _fulfilled_ – resolved to successfully designate some object,  
			   * _near_ – resolved to a direct reference to a local object,  
			   * _far_ – resolved to designate a remote object,
			  * _rejected_ – will never designate an object, for an alleged reason represented by an associated error.
			  
			  A promise may transition from _pending_ to any state. Additionally a promise can transition from _far_ to _rejected_. A _resolved_ promise can designate any non-promise value including primitives, `null`, and `undefined`. Primitives, `null`, `undefined`, and some objects are pass-by-copy. All other objects are pass-by-reference. A promise resolved to designate a pass-by-copy value is always near, i.e., it always designates a local copy of the value.
			  
			  If a function `foo` immediately returns either `X` or a promise which it later fulfills with `X`, we say that `foo` **_reveals_** `X`. Unless stated otherwise, we implicitly elide the error conditions from such statements. For the more explicit statement, append: _“or it throws, or it does not terminate, or it rejects the returned promise, or it never resolves the returned promise.”_ Put another way, such a function returns a **_reference_** to `X`, where by _reference_ we mean either `X` or a promise for `X`.
			  
			  \#\# Eventual Operations
			  
			  The existing JavaScript infix `.` (dot or _now_) operator enables synchronous interaction with the local object designated by a direct reference. We introduce a corresponding infix `!` (bang or _eventually_) operator for corresponding asynchronous interaction with objects eventually designated by either direct references or promises.
			  
			  Abstract Syntax: 
			  
			  Expression : ...
			      Expression ! [ Expression ] Arguments    // eventual send
			      Expression ! Arguments                   // eventual call
			      Expression ! [ Expression ]              // eventual get
			      Expression ! [ Expression ] = Expression // eventual put
			      delete Expression ! [ Expression ]       // eventual delete
			  
			  The `...` means “and all the normal right hand sides of this production. By “abstract” here I mean the distinction that must be preserved by parsing, i.e., in an [ast](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:ast "strawman:ast"), but without explaining the precedence and associativity which explains how this is unambiguously parsed. In all cases, the eventual form of an expression queues a eventual-delivery recording the need to perform the corresponding immediate form in the vat hosting the (eventually) designated object. The eventual form immediately evaluates to a promise for the result of eventually performing this eventual-delivery.
			  
			  function add(x, y) { return x + y; }
			  const sumP = add ! (3, 5); //sumP resolves in a later turn to 8.
			  
			  Attempted Concrete Syntax: 
			  
			  MemberExpression : ...
			      MemberExpression [nlth] ! [ Expression ]
			      MemberExpression [nlth] ! IdentifierName
			  CallExpression : ...
			      CallExpression [nlth] ! [ Expression ] Arguments
			      CallExpression [nlth] ! IdentifierName Arguments
			      MemberExpression [nlth] ! Arguments
			      CallExpression [nlth] ! Arguments
			      CallExpression [nlth] ! [ Expression ]
			      CallExpression [nlth] ! IdentifierName
			  UnaryExpression : ...
			      delete CallExpression [nlth] ! [ Expression ]
			      delete CallExpression [nlth] ! IdentifierName
			  LeftHandSideExpression :
			      Identifier
			      CallExpression [ Expression ]
			      CallExpression . IdentifierName
			      CallExpression [nlth] ! [ Expression ]
			      CallExpression [nlth] ! IdentifierName
			  
			  “`[nlth]`” above is short for “`[No LineTerminator here]`“, in order to unambiguously distinguish infix from prefix bang in the face of automatic semicolon insertion. 
			  
			  \#\# Fundamental Static Q Methods
			  
			  | Q(target) \-> targetP                         | Lifts the target argument into a promise designating the same object. If target is already a promise, then that promise is returned. (A promise for promise for T simplifies into a promise for T. Category theorists will be more pleased than Type theorists ;).)                                |
			  | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
			  | Q.reject(reason) \-> rejectedP                | Makes and returns a fresh _rejected_ promise recording (a sanitized form of) reason as the alleged reason for rejection. reason should generally be an immutable pass-by-copy Error object.                                                                                                        |
			  | Q.promise(f(resolve,reject)\->()) \-> promise | Makes a fresh promise, where the promise is initially _pending-local_, and the argument function f is called with resolve and reject functions for resolving this promise.                                                                                                                         |
			  | Q.isPromise(target) \-> boolean               | Is target a promise? If not, then using target as a target in the various promise operations is still equivalent to using Q(target), i.e., the promise operations will automatically lift all values to promises.                                                                                  |
			  | Q.makeFar(handler, nextSlotP) \-> promise     | Makes a resolved _far_ reference, which can only [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to _rejected_.                                                                          |
			  | Q.makeRemote(handler, nextSlotP) \-> promise  | Makes an _pending-remote_ promise, which can [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to anything.                                                                                |
			  | Q.ahorten(target1) \-> target2                | Returns the currently most resolved form of target1\. If target1 is a _fulfilled_ promise, return its resolution. If target1 is a promise that is following promise target2, then return target2. If target1 is a terminal _pending_ or _rejected_ promise, or a non-promise, then return target1. |
			  
			  Additional non-fundamental static Q convenience methods appear below.
			  
			  \#\# Promise methods
			  
			  Assuming `p` is a promise 
			  
			  | p.get(name) \-> valueP                    | Returns a promise for the result of eventually getting the value of the name property of target.                                                                                                                                               |
			  | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
			  | p.post(opt\_name, args) \-> resultP       | Eventually invoke the named method of target with these args. Returns a promise for what the result will be.                                                                                                                                   |
			  | p.send(opt\_name, ...args) \-> resultP    | p.send(m, a, b) is equivalent to p.post(m, \[a,b\])                                                                                                                                                                                            |
			  | p.fcall(...args) \-> resultP              | p.fcall(a, b) is equivalent to p.post(void 0, \[a,b\])                                                                                                                                                                                         |
			  | p.put(name, value) \-> voidP              | Eventually set the value of the name property of target to value. Return a promise-for-undefined, used for indicating completion.                                                                                                              |
			  | p.delete(name) \-> trueP                  | Eventually delete the name property of target. Returns a promise for the boolean result.                                                                                                                                                       |
			  | p.then(success, opt\_failure) \-> resultP | Registers functions success and opt\_failure to be called back in a later turn once target is _resolved_. If _fulfilled_, call success(resolution). Else if _rejected_, call opt\_failure(reason). Return a promise for the callback’s result. |
			  | p.end()                                   | If p resolves to _rejected_, log the reason to wherever uncaught exceptions go on this platform, e.g., onerror(reason).                                                                                                                        |
			  
			  
			  \#\# Syntactic Sugar
			  
			  | Abstract Syntax  | Expansion          | Simple Case  | Expansion            | JSON/RESTful equiv        |
			  | ---------------- | ------------------ | ------------ | -------------------- | ------------------------- |
			  | x ! \[i\](y, z)  | Q(x).send(i, y, z) | x ! p(y, z)  | Q(x).send(’p’, y, z) | POST https://...q=p {...} |
			  | x ! (y, z)       | Q(x).fcall(y, z)   | \-           | \-                   | POST https://... {...}    |
			  | x ! \[i\]        | Q(x).get(i)        | x ! p        | Q(x).get(’p’)        | GET https://...q=p        |
			  | x ! \[i\] = v    | Q(x).put(i, v)     | x ! p = v    | Q(x).put(’p’, v)     | PUT https://...q=p {...}  |
			  | delete x ! \[i\] | Q(x).delete(i)     | delete x ! p | Q(x).delete(’p’)     | DELETE https://...q=p     |
			  
			  
			  \#\# Non-fundamental Static Q Conveniences
			  
			  \#\#\# Q.delay
			  
			  Reveal the `answer` sometime after `millis` milliseconds have elapsed.
			  
			  Q.delay = function(millis, answer = undefined) {
			    return Q.promise(resolve => {
			      setTimeout(() => resolve(answer), millis);
			    });
			  };
			  
			  \#\#\# Q.race
			  
			  Given a list of promises, returns a promise for the resolution of whichever promise we notice has completed first.
			  
			  Q.race = function(answerPs) {
			    return Q.promise((resolve,reject) => {
			      for (answerP of answerPs) {
			        Q(answerP).then(resolve,reject);
			      };
			    });
			  };
			  
			  We can compose `Q.race`, `Q.delay`, and `Q.reject` to timeout eventual requests.
			  
			  var answer = Q.race([bob ! foo(carol), 
			                       Q.delay(5000, Q.reject(new Error("timeout")))]);
			  
			  \#\#\# Q.all
			  
			  Often it’s useful to collect several promised answers, in order to react either when _all_ the answers are ready or when _any_ of the promises becomes _rejected_.
			  
			  Q.all = function(answerPs) {
			    let countDown = answerPs.length;
			    const answers = [];
			    if (countDown === 0) { return Q(answers); }
			    return Q.promise((resolve,reject) => {
			      answerPs.forEach((answerP, index) => {
			        Q(answerP).then(answer => {
			          answers[index] = answer;
			          if (--countDown === 0) { resolve(answers); }
			        }, reject);
			      });
			    });
			  };
			  
			  We can compose `Q.all`, `then`, and [destructuring](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:destructuring "harmony:destructuring") to delay until several operands are revealed
			  
			  var sumP = Q.all([xP, yP]).then(([x, y]) => (x + y);
			  
			  \#\#\# Q.join
			  
			  Join is our eventual equality operation. Any messages sent on the join of `xP` and `yP` are only delivered if `xP` and `yP` both reveal the same target, in which case these messages are eventually delivered to that target and this joined promise itself eventually becomes fulfilled to designate that target. Otherwise, all these messages are discarded with the usual rejected promise contagion.
			  
			  Q.join = function(xP, yP) {
			    return Q.all([xP, yP]).then(([x, y]) => {
			      if (Object.is(x, y)) {
			        return x;
			      } else {
			        throw new Error("not the same");
			      }
			    });
			  };
			  
			  \#\#\# Q.memoize
			  
			  `Q.memoize` of a one argument function returns a new similar one argument function, except that it (eventually) calls the original function no more than once for each such argument. The memo function immediately returns a promise for what the original function will eventually return. Equivalence of arguments is determined by the optional memoMap passed in, which defaults to a new WeakMap() if absent. (Passing a memoMap in also allows the caller to seed the mapping with some prior associations.)
			  
			  The difference from a traditional synchronous memoizer is that the original function is called _eventually_ after a promise for its result is _already_ memoized, enabling cycles to work. For example, if memoF === memoize(f) and f(x) calls memoF(x), then an outer call to memoF(x) schedules an eventual call to f(x) which makes an inner call to memoF(x). Both outer and inner calls to memoF(x) returns a promise for what f(x) will eventually return.
			  
			   Q.memoize = function(oneArgFuncP, memoMap = WeakMap()) {
			  
			     return function oneArgMemo(arg) {
			       if (memoMap.has(arg)) {
			         return memoMap.get(arg);
			       } else {
			         const resultP = oneArgFuncP ! (arg);
			         memoMap.set(arg, resultP);
			         return resultP;
			       }
			     }
			   };
			  
			  \#\#\# Q.async
			  
			  \#\#\# Q.defer()
			  
			  (Will likely be deprecated)
			  
			  Q.defer = function() {
			    const deferred = {};
			    deferred.promise = Q.promise((resolve,reject) => {
			      deferred.resolve = resolve;
			      deferred.reject = reject;
			    });
			    return deferred;
			  };
			  
			  \#\# Examples
			  
			  \#\# Infinite Queue
			  
			  function makeQueue() {
			    let rear;
			    let front = Q.promise(r => { rear = r; });
			    return Object.freeze({
			      enqueue: function(elem) {
			        let nextRear;
			        const nextTail = Q.promise(r => { nextRear = r; });
			        rear({head: elem, tail: nextTail});
			        rear = nextRear;
			      },
			      dequeue: function() {
			        const result = front ! head;
			        front = front ! tail;
			        return result;
			      }
			    });
			  }
			  
			  Calling `queue.dequeue()` will return a promise for the next element that has or will be enqueued.
			  
			  \#\# Spawn
			  
			  The following `spawn` function is a simple abstraction built on `Vat` and `then` that captures a simple common case:
			  
			  function spawn(src) {
			    const vat = Vat();
			    const resultP = vat.evalLater(src);
			    Q(resultP).then(function(_) {
			      vat.terminateAsap(new Error('done')); 
			    });
			    return resultP;
			  }
			  
			  const sumP = spawn('3+5'}); // sumP eventually resolves to 8.
			  
			  The argument string to `spawn` is evaluated in a new Vat spawned for that purpose. Spawn returns a promise for what that string will evaluate to. Once that promise resolves, the spawned vat is shut down.
			  
			  \#\# Vat.evalLater() as Async-PGAS
			  
			  In the Async-PGAS language X10, the “at” statement is defined as
			  
			  // x10 grammar, not javascript or proposed javascript
			  Statement :
			    at ( PlaceExpression ) Statement
			  
			  The “at” statement first evaluates the PlaceExpression to a place, which is analogous to a vat. It then evaluates the Statement at that place. The statement evaluates with the lexical scope containing the “at” statement, so the locality of the values bound to these identifiers is the locality they have at that place rather than at the location containing the “at” statement. Although the argument to `Vat.evalLater` must be a closed expression (modulo whitelisted globals), we can get the same effect, a bit more verbosely, by passing in these bindings as an explicit eventual call to a closed function.
			  
			  For example, the X10-ish program
			  
			  const x = 6;
			  let ultimateP;
			  at (place) { ultimateP = x*7; }
			  
			  can be expressed using `aVat.evalLater()` as
			  
			  const x = 6;
			  let ultimateP = place.evalLater(x => x*7) ! (x);
			  
			  \#\# Open Vat
			  
			  The `makeOpenVat` function makes an `OpenVat` function. Each `OpenVat` function is like the `Vat` function, in that both make an return a new vat instance. Each `OpenVat` function additionally maintains a side table mapping from all incoming remote promises to the evaluation function of the open vat made by this `OpenVat` function. The reason we call such vats _open_ is that, given a remote promise into such a vat and the `OpenVat` function that made that vat, one can thereby inject new code into that vat.
			  
			  TODO: explain the membrane variation used below.
			  
			  function makeOpenVat() {
			    const wm = WeakMap();
			  
			    function OpenVat() {
			      const vat = Vat();
			      const openVat = Object.freeze({
			        evalLater: makeMembraneX(
			          vat.evalLater,
			          { registerRemote: function(remote) {
			              wm.set(remote, openVat.evalLater); }}),
			        terminateAsap: vat.terminateAsap
			      });
			      return openVat;
			    }
			    OpenVat.evalAt = function(p, src) {
			      return wm.get(p)(src);
			    };
			    return OpenVat;
			  }
			  
			  \#\# there
			  
			  The `there(p, ...)` function is analogous to `Q(p).then(...)`, except instead of merely relocating the execution of the callback in time till after `p` is resolved, it further relocates it in spacetime, to where and when p is near. Like `Q(p).then(...)`, `there` immediately returns a promise for the eventual outcome. We do not likewise relocate the errback, so that we can still notify it and it can still react on the requesting side to a partition between the requestor and `p`‘s host.
			  
			  function there(p, callback, opt_errback) {
			    var callbackP = OpenVat.evalAt(p, '' + callback);
			    return (callbackP ! (p)).then(
			      v => v,
			      opt_errback);
			  }
			  
			  \#\# Map-Reduce Lite
			  
			  Given an initial result value, a list of potentially remote promises to elements, a closed mobile `mapper` function from elements to mapped result values, and an associative / commutative `reducer` function from pairs of references to result values to a new reference to a result value, `mapReduce` immediately returns a promise for the result of mapping all the elements where they live, and reducing all of these results together with the initial result value to a result.
			  
			  I call this “Map-Reduce Lite” because, unlike a production map-reduce infrastructure, the following `mapReduce` does all reductions on the spawning machine, which is therefore a scaling bottleneck, and has none of the fault-tolerance. Here, any failure causes the overall map-reduce to fail, i.e., the returned promise becomes a _rejected_ promise. The mapper and reduction functions are like the conventional functional programming variety, rather than the map-reduce variety which arranged for group-by keys.
			  
			  /**
			   * Type/Guard syntax below is only a placeholder, not a serious proposal.
			   * @param initValue ::T2
			   * @param elemPs    ::Array[Ref[T1]]  // i.e., Array[T1 | Promise[T1]]
			   * @param mapper    ::(T1 -> T2)      // closed mobile function
			   * @param reducer   ::(T2 x T2 -> T2)
			   * @reveals         ::T2              // i.e., @returns ::Ref[T2]
			   */
			  function mapReduce(initValue, elemPs, mapper, reducer) {
			    let countDown = elemPs.length;
			    if (countDown === 0) { return initValue; }
			    let result = initValue;
			  
			    return Q.promise((resolve, reject) => {
			      elemPs.forEach(elemP => {
			        const mappedP = there(elemP, mapper);
			        Q(mappedP).then(mapped => {
			          result = reducer(result, mapped);
			          if (--countDown === 0) { resolve(result); }
			        }, reject);
			      });
			    });
			  }
			  
			  \#\# AMD Loader Lite
			  
			  This is a minimal Asynchronous Module Definition (AMD) Loader for a subset of the [AMD API](https://web.archive.org/web/20160404122250/https://github.com/amdjs/amdjs-api/wiki/AMD "https://github.com/amdjs/amdjs-api/wiki/AMD") specification. In this subset, `define` is called with exactly two arguments, a `dependencies` list of module names, and a factory function with one parameter per dependency.
			  
			   function makeSimpleAMDLoader(fetch, moduleMap = Map()) {
			     var loader;
			  
			     function rawLoad(id) {
			       return Q(fetch(id)).then(src => {
			         var result = Q.reject(new Error('"define" not called by: ' + id));
			         function define(deps, factory) {
			           result = Q.all(deps.map(loader)).then(imports => {
			             return factory(...imports);
			           });
			         }
			         define.amd = {lite: true};
			  
			         Function('define', src)(define);
			         return result;
			       });
			     }
			     return loader = Q.memoize(rawLoad, moduleMap);
			   }
			  
			  If module “W” depends on modules “X”, “Y”, and “Z”, then only once the promises for the “X”, “Y”, and “Z” modules have all been fulfilled will the “W” factory function be called with these module instances. The result of calling this factory function will then become the “W” module instance.
			  
			  What it means to _be_ the source for the “W” module is that `fetch(”W”)` will eventually return that source string. For example, a given `fetch` function might fetch it from `https://example.com/prefix/W.js`.
			  
			  // At https://example.com/prefix/W.js
			  define(['X', 'Y', 'Z'], function(X, Y, Z) {
			    return X(Y, Z);
			  })
			  
			  Since the memo mapping we need is from module names, which are strings rather than objects, we need to provide an explicit memoMap argument to `Q.memoize`, which should be a map that accepts strings as keys.
			  
			  Although the cycle tolerance of `Q.memoize` is generally useful, here it hurts. Because `define` won’t call the factory function until all (`Q.all`) of the dependencies are fulfilled, any cyclic AMD dependencies cause an undetected deadlock. Still, in the naive absence of this cycle tolerance, such cyclic dependencies would have instead caused an infinite regress which is even worse. Better would be cycle detection and rejection, which we leave as an exercise for the reader. 
			  
			  \#\# See
		- ### Highlights
		  collapsed:: true
			- > Objects in one vat can only send asynchronous messages to objects in other vats. [⤴️](https://omnivore.app/me/strawman-concurrency-es-wiki-18d971e3b8d#a7e7b673-aed5-4f4c-b423-b2592dfc6385)
			- > _Promises_ represent such references to potentially remote objects. _Eventual message sends_ queue _eventual-deliveries_ in the work queue of the vat hosting the target object. [⤴️](https://omnivore.app/me/strawman-concurrency-es-wiki-18d971e3b8d#e472ac14-e40a-4977-bf32-89b2b23e2dae)
	- [Notes on Functional Programming III: Functor, Applicative & Monad || Chrysanthium](https://omnivore.app/me/notes-on-functional-programming-iii-functor-applicative-monad-ch-18d38448346)
	  collapsed:: true
	  site:: [chrysanthium.com](https://chrysanthium.com/notes-on-functional-programming-iii)
	  author:: akullpp
	  labels:: [[functional programming]] [[fp]] [[javascript]]
	  date-saved:: [[01/23/2024]]
		- ### Content
		  collapsed:: true
			- \#\#\#\#\#\# 2017/03
			  
			  Preliminary: [The Fantasy Land](https://github.com/fantasyland/fantasy-land) algebra is a specification which many good functional libraries implement and covers additional laws of algebraic structures which I won't cover.
			  
			  \#\# Functor
			  
			  A **functor** is just a container for values with a `map` method that applies a function to the values while consistently returning the new values in the same container.
			  
			  According to this definition an array is a functor:
			  
			  ```angelscript
			  const xs = [1, 2, 3]
			  // Array.isArray(xs) === true
			  const ys = xs.map((x) => x)
			  // Array.isArray(ys) === true
			  ```
			  
			  You have an array with values, apply a function via `map` to each of the values and get an array with values back.
			  
			  Unfortunately, not all data structures have a map function so you might need to write a wrapper which could be as simple as:
			  
			  ```pgsql
			  class Wrapper {
			  constructor(value) {
			    this.value = value
			  }
			  
			  map(f) {
			    return new Wrapper(f(this.value))
			  }
			  }
			  ```
			  
			  \#\#\# Advantages of a functor
			  
			  **A functor enables generalized behavior**
			  
			  It allows you to map over values without being tied to a specific structure.
			  
			  Furthermore, all advantages of `map` over `for` loops apply to which is mainly compactness and expressiveness.
			  
			  \#\# Applicative
			  
			  An **applicative** is an extension of a functor and is used to be able to apply functors to each other. In order to do this, it wraps a function around the value which is wrapped by the functor. The required `ap` method is used to automatically apply the already partially applied and wrapped function via `map` to the wrapped value of another functor:
			  
			  ```pgsql
			  Wrapper.prototype.ap = function (wrapped) {
			  return wrapped.map(this.value)
			  }
			  ```
			  
			  Additionally, an applicative must provide an `of` method which is used to create an instance with default minimal context:
			  
			  ```pgsql
			  Wrapper.of = (x) => new Wrapper(x)
			  ```
			  
			  > Functors which only have an `ap` method are called Apply. Functors which only have an `of` method are called Pointed Functors. If they have both they are called Applicatives.
			  
			  We will use `of` to write a new `map` method:
			  
			  ```pgsql
			  Wrapper.prototype.map = function (f) {
			  return Wrapper.of(f(this.value);
			  };
			  ```
			  
			  Let's assume we have the following setup:
			  
			  ```pgsql
			  const add = (x) => (y) => x + y
			  const wrapperOne = Wrapper.of(1)
			  const wrapperTwo = Wrapper.of(2)
			  const wrapperOneAdd = wrapperOne.map(add)
			  ```
			  
			  where `add` is a curried function and `wrapperOneAdd` is an object of type `Wrapper` with the wrapped value of `y => 1 + y`.
			  
			  To be explicit: The first box contains the integer `1`. The second box the result, i.e. `y => 1 + y`, of the function `add` which was applied to the boxed `1`. So we are two levels deep now.
			  
			  Using the `ap` method would result in a new wrapped value:
			  
			  ```pgsql
			  wrapperOneAdd.ap(wrapperTwo)
			  // Wrapper {value: 3}
			  ```
			  
			  The wrapped partial function `y => 1 + x` is applied to the unwrapped value of `2`, i.e. `y => 1 + 2` which is then returned as a wrapped value `3`.
			  
			  Keep in mind the box is always of the same type, the values not necessarily.
			  
			  \#\#\# Advantages of an Applicative
			  
			  An applicative is best used if you have several tasks which don't depend on each other, e.g. if you have several independent calls you could write the following interface:
			  
			  ```routeros
			  Task.of(renderPage)
			  .ap(Http.get('orders'))
			  .ap(Http.get('billing'));
			  .ap(Http.get('ads'));
			  .ap(Http.get('tracking'));
			  ```
			  
			  **An applicative promotes simplicity**
			  
			  Generally speaking it creates a simple interface for complex code. However the real advantages can only be grasped if a specific applicative is used. General applicatives like the one described above are rather rare. Further information will be provided in the monad section.
			  
			  \#\# Monads
			  
			  A **monad** applies a function which returns a wrapped value to a wrapped value. The main advantage over applicatives is that they run sequentially by providing the `chain` method. Here is an implementation of a general monad:
			  
			  ```cs
			  class Monad {
			  static of(value) {
			    return new Monad(value)
			  }
			  
			  constructor(value) {
			    this.value = value
			  }
			  
			  map(f) {
			    return Monad.of(f(this.value))
			  }
			  
			  ap(monad) {
			    return monad.map(this.value)
			  }
			  
			  chain(f) {
			    return this.map(f).value
			  }
			  }
			  ```
			  
			  Normally you would also implement the `join` method which is a straight forward helper method to return the value which helps us to flatten monads when chained:
			  
			  ```kotlin
			  join() {
			  return this.value;
			  }
			  
			  chain(f) {
			  return this.map(f).join();
			  }
			  ```
			  
			  However, just like general applicatives, a general implementation of a monad doesn't make much sense. One of the most common practical examples of a specific monad is the **Maybe monad**:
			  
			  ```scala
			  class Maybe extends Monad {
			  static of(value) {
			    return new Maybe(value)
			  }
			  
			  constructor(value) {
			    super(value)
			  }
			  
			  isNothing() {
			    return this.value === null || this.value === undefined
			  }
			  
			  map(f) {
			    return this.isNothing() ? Maybe.of(null) : Maybe.of(f(this.value))
			  }
			  }
			  ```
			  
			  It just checks whether the wrapped value is `null` or `undefined` and if it is, returns a wrapped `null`. Why is this interesting? Well, first of all it avoids pesky null checks. Additionally, it provides safety from runtime errors when chaining several methods where one may fail to return a value:
			  
			  ```javascript
			  const prop = (p) => (o) => o[p]
			  
			  const getUsername = (account) =>
			  Maybe.of(account).map(prop('personal')).map(prop('user')).map(prop('name'))
			  
			  // Might be retrieved async!
			  const user = {
			  personal: {
			    user: {
			      name: 'John Doe',
			    },
			  },
			  }
			  
			  getUsername(user)
			  // Maybe { value: 'John Doe' }
			  ```
			  
			  If one property in the path wouldn't exist, we wouldn't get an error but a `Maybe` with value `null`.
			  
			  Right now you probably think you have a déjà vu and yes you are correct _Promise_ is a monad.
			  
			  \#\#\# Advantages of a monad
			  
			  In general monads are concise and expressive with the ability to encapsulate side-effects.
			  
			  The advantages of specific monads are as numerous as their implementations ranging from the simple forms of _State_ ensuring the correct state flow, _Sequence_ where `;` can be a monad for control flow, _Maybe_ handling null checks, _Promise_ handling asynchronicity up to the complex ones of _Probability Distribution_ or _Transaction_ for databases.
			  
			  \#\# Links
			  
			  * [Notes on Functional Programming I: First-class, Pure, Curried Functions](https://chrysanthium.com/notes-on-functional-programming-i)
			  * [Notes on Functional Programming II: Composition & Point-free Style](https://chrysanthium.com/notes-on-functional-programming-ii)
	- [How DAT discovers peers](https://omnivore.app/me/super-high-level-18c07ef66d7)
	  collapsed:: true
	  site:: [blog.mauve.moe](https://blog.mauve.moe/posts/how-dat-discovers-peers)
	  labels:: [[DAT]] [[NAT Traversal]] [[DWeb]] [[P2P]]
	  date-saved:: [[11/25/2023]]
		- ### Content
		  collapsed:: true
			- There are a lot of guides and introductions to Dat that focus on using dat to transfer files, the replication protocol, and how the data is transferred between peers. However, that's just half of the story. ==A lot of the magic from the dat ecosystem comes from== ==[discovery-swarm](https://github.com/mafintosh/discovery-swarm)====, the module that's responsible for finding peers to replicate with in the dat network.==
			  
			  ==discovery-swarm provides an API to== **==join==** ==networks using a== **==discovery key==**==, and then invokes a callback when it finds a peer to connect to.==
			  
			  ```javascript
			  // Taken from the discovery-swarm example
			  var swarm = require('discovery-swarm')
			  
			  var sw = swarm()
			  
			  sw.listen(666)
			  sw.join('cool stuff') // can be any id/name/hash
			  
			  sw.on('connection', function (connection) {
			  console.log('found + connected to peer')
			  })
			  
			  ```
			  
			  This seems easy enough, but it feels a little magical. Under the hood, it combines several high level modules to get the job done.
			  
			  \#\# [Peer discovery](\#peer-discovery)
			  
			  While ==dat-swarm provides a way to join multiple networks and automatically connect to peers,== ==[discovery-channel](https://github.com/maxogden/discovery-channel)== ==is the module responsible for actually finding peers for a given key.== It's what advertises the local ip/port to the network and searchs for peer identifiers.
			  
			  \#\# [Decentralized peer lookup - The DHT](\#decentralized-peer-lookup---the-dht)
			  
			  ==Dat isn't the only p2p protocol that relies on peer discovery. Back when applications like BitTorrent and eMule were being developed, they relied a lot on "tracker" servers for announcing peers and searching for them. This resulted in a form of centralization which meant that if a tracker server got taken down or otherwise compromised, the p2p network couldn't function.==
			  
			  ==The solution for this problem was to get rid of centralized trackers and replace them with a protocol that would split the discovery information amongst all the peers participating in the network.== The particular approach is called a Distributed Hash Table. It's a key-value store that automatically stores keys on nodes in the network. The most popular algorythm for determening which peers should store which data is called [Kademlia](https://en.wikipedia.org/wiki/Kademlia). I'm not going to go too far into it, but ==the idea is that each discovery key is sent to nodes whose id is== _=="close"==_ ==to the key, and peers maintain connections to others that are varying levels of== _=="closeness"==_ ==to themselves which makes it fast to find the nodes storing keys that you want.==
			  
			  This functionality is implemented in the [bittorrent-dht](https://github.com/webtorrent/bittorrent-dht) module which was originally made for the popular [webtorrent](https://webtorrent.io/) project.
			  
			  ==discovery-channel uses this module by publishing it's IP/Port combination to the discovery key onto the DHT. Since there is no central authority or single point of failure, publishing on the DHT is resistant to censorship and can survive sketchy networks.==
			  
			  One of the problems with the DHT, is that items in the DHT don't expire right away, and searching for peers can yield a lot of false-positives. Also, maintining the connection information necessary for staying in the DHT takes up more computational resources. As well, ==the DHT requires a set of== _=="bootstrap nodes"==_ ==which are used to find more nodes to start building up your network. These bootstrap nodes are a potential source of failure and DHT clients should attempt to save any nodes they find for later use in order to have a way to bootstrap should the bootstrap nodes go down.==
			  
			  \#\# [Faster than the DHT - DNS! (and MDNS)](\#faster-than-the-dht---dns-and-mdns)
			  
			  The next module, is a two-parter. As I mentioned before, ==the DHT can yield false positives and requires participating in a network. To make peer lookup faster, discovery-channel makes use of== ==[dns-discovery](https://github.com/mafintosh/dns-discovery)== ==to talk to a list of centralized DNS servers to find peers.==
			  
			  DNS is the technology used to resolve website hostnames like `www.example.com` to IP addresses. It works by having a network of DNS "servers" that talk to each other about which domains map to which IP addresses. Clients wanting to connect to a website will then issue DNS queries to the server by sending UDP packets asking for a _"query"_, and getting a result back over the same UDP socket. The query contains a _"type"_ of record, and the _"name"_ to search for.
			  
			  discovery-swarm makes use of the [txt](https://github.com/mafintosh/dns-discovery/blob/master/index.js\#L392) _"type"_ , and generates a fake name by using the discovery key as a subdomain on a configurable domain. `txt` records are used for encoding freeform data that applications can make use of. For example, you could tell dns-discovery to use the `example.local` domain, and when searching for the `foobar` discovery key, it will make DNS queries for `foobar.example.local`. It tells the DNS server to track it's own IP/Port by adding the [additionals](https://github.com/mafintosh/dns-discovery/blob/master/index.js\#L396) attribute to it's query with some custom data specifing whether it's announcing itself, looking up a peer, or un-announcing itself. The logic for how it encodes this information is [here](https://github.com/mafintosh/dns-discovery/blob/master/index.js\#L719).
			  
			  Using DNS is great for when you want speedy responses and to have a way to invalidate stored data (when you're leaving a network, for example). However, this introduces a point of centralization, and if the DNS servers get taken out, it prevents peers from being able to discover each other.
			  
			  \#\# [Working without internet!](\#working-without-internet)
			  
			  The internet is great for P2P applications, but sometimes you don't have the luxery of connecting to any computer in the world, or sometimes the computers you want to connect to are sitting right beside you and don't need any fancy distributed discovery. dns-discovery supports this scenario by making use of something called [Multicast DNS](https://en.wikipedia.org/wiki/Multicast%5FDNS) to find peers on the local network. It works by sending _"multicast UDP packets"_ to the local network containing what's essentially a DNS request, which might be picked up by any computers that are listening for it. The computers that are listening on the _"name"_ in the DNS query will then broadcast a DNS response with their local IP/Port which can be used to connect to them. This functionality is implemented in the [multicast-dns](https://github.com/mafintosh/multicast-dns) module. dns-discovery starts listening on MDNS requests, and has logic for responding to them [here](https://github.com/mafintosh/dns-discovery/blob/master/index.js\#L149).
			  
			  \#\# [How does discovery-swarm put this together?](\#how-does-discovery-swarm-put-this-together)
			  
			  When you initiate a discovery-swarm instance, you can specify the TCP port you want to listen to, the UDP port for DNS/MDNS queries, the DHT bootstrap nodes to use, the DNS server list, and the fake domain to use for DNS/MDNS discovery. Specifying or not specifying or not specifying these arguments enables and disables parts of the discovery features. Therese some other little knobs to toggle, but that's the main stuff that matters.
			  
			  After specifying the discovery options, you need to start joining networks. The `discovery key` is modified by being optionall [hashed](https://github.com/maxogden/discovery-channel/blob/master/index.js\#L104), and truncated to be only [20 hex characters](https://github.com/maxogden/discovery-channel/blob/master/index.js\#L105). The 20 character limit is there to make it fit the Kademlia discovery key limitations.
			  
			  It announces itself wherever needed, and starts periodically querying for peer information. Once it finds a peer, it will attempt to establish a connection to it. Peers will likewise attempt to establish connections to it once they discover it.
			  
			  At this point, discovery-swarm has a duplex node stream between the two peers. Here is where two things can happen. By default, discovery-swarm will attempt to do a "handshake" between the two peers by exchanging their "ids" that they generated upon initialization. This lets the peers know who they're connected to and prevents them from opening more than one connection to each other. This can be overrided by providing a custom `stream` handler in the initialization options which returns an `EventEmitter` that will emit a `handshake` event with the peer ID itself.
			  
			  From here it will happily run and establish new connections once existing ones close and optionally prevent new ones once it reaches a limit.
			  
			  The discovery-swarm instance can be used to join multiple networks, and is able to handle incoming connections from multiple peers.
			  
			  \#\# [How does Dat make use of it?](\#how-does-dat-make-use-of-it)
			  
			  discovery-swarm has a lot of options, and if they're not compatible, peers can't find each other. Luckily the dat project provides a module, [dat-swarm-defaults](https://github.com/datproject/dat-swarm-defaults), which allows different projects in the Dat ecosystem to easily configure discovery so that they can all connect to each other.
			  
			  Here's what it looks like
			  
			  ```1c
			  var DAT_DOMAIN = 'dat.local'
			  var DEFAULT_DISCOVERY = [
			  'discovery1.datprotocol.com',
			  'discovery2.datprotocol.com'
			  ]
			  var DEFAULT_BOOTSTRAP = [
			  'bootstrap1.datprotocol.com:6881',
			  'bootstrap2.datprotocol.com:6881',
			  'bootstrap3.datprotocol.com:6881',
			  'bootstrap4.datprotocol.com:6881'
			  ]
			  
			  var DEFAULT_OPTS = {
			  dns: {server: DEFAULT_DISCOVERY, domain: DAT_DOMAIN},
			  dht: {bootstrap: DEFAULT_BOOTSTRAP}
			  }
			  
			  ```
			  
			  Beaker takes these defaults, and ads some more [options](https://github.com/beakerbrowser/beaker-core/blob/master/dat/library.js\#L111)
			  
			  ```yaml
			  discoverySwarm(swarmDefaults({
			    id: networkId,
			    hash: false,
			    utp: true,
			    tcp: true,
			    dht: false,
			    stream: createReplicationStream
			  }))
			  
			  ```
			  
			  As you can see from the code snippit, beaker is disabling the hashing and the DHT! This means that when you're advertising discovery keys, you can just take the first 20 hex characters, and not worry about any of the rest of the processing. This also means that the DHT isn't being used for peer discovery, so the only source of peers is DNS and MDNS, which in most cases will mean only DNS is used to find peers over the internet.
			  
			  The `createReplicationStream` argument is creating a replication stream for [hyperdrive](https://github.com/mafintosh/hyperdrive) instances, which is the module used to replicate dat archives. The dat replication protocol contains a handshake, so the discovery swarm handshake isn't being used.
			  
			  \#\# [Steps for a MVP discovery swarm](\#steps-for-a-mvp-discovery-swarm)
			  
			  Hopefully, this article has shown you the main pieces of discovery-swarm and can help you create your own implementation in other environments.
			  
			  The minimal amount needed is to use the DNS discovery, and pipe connections into the hypercore protocol to replicate hyperdrives.
			  
			  I've started work on [a module](https://github.com/RangerMauve/discovery-swarm-stream) that will allow browsers to make use of a discovery-swarm proxy over a websocket. This will allow applications to find peers and connect to them (almost) directly and allow people to experiment with p2p features outside of beaker, and experiment with cutting edge stuff like hyperdb even within beaker.
			  
			  Hopefully this has been informative, feel free to hit me up [on fritter](dat://fritter.hashbase.io/user/dat://3df8868d5c3420d7acdf72d17129e4569cf83723092314ea6b260d112797d8c8), or [twitter](https://mobile.twitter.com/RangerMauve) if you have any questions or want to brainstorm more about bringing discovery to other platforms.
			  
			  * RangerMauve
		- ### Highlights
		  collapsed:: true
			- > A lot of the magic from the dat ecosystem comes from [discovery-swarm](https://github.com/mafintosh/discovery-swarm), the module that's responsible for finding peers to replicate with in the dat network. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#1b5f7c6f-a174-4b1d-b33b-fdf2b3c1fe81)
			- > discovery-swarm provides an API to **join** networks using a **discovery key**, and then invokes a callback when it finds a peer to connect to.
			  > 
			  ```javascript
			  > 
			  ``` [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#393bae75-892c-49bb-8eb5-5e6cda354b6a)
			- > dat-swarm provides a way to join multiple networks and automatically connect to peers, [discovery-channel](https://github.com/maxogden/discovery-channel) is the module responsible for actually finding peers for a given key. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#90b639cb-6e79-4342-8f2e-9496c0d5c741)
			- > Dat isn't the only p2p protocol that relies on peer discovery. Back when applications like BitTorrent and eMule were being developed, they relied a lot on "tracker" servers for announcing peers and searching for them. This resulted in a form of centralization which meant that if a tracker server got taken down or otherwise compromised, the p2p network couldn't function. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6bb22760-03ff-45f0-a86f-9112c11db11e)
			- > The solution for this problem was to get rid of centralized trackers and replace them with a protocol that would split the discovery information amongst all the peers participating in the network. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6a1534d6-8937-4d6f-add5-7f3785b473c7)
			- > the idea is that each discovery key is sent to nodes whose id is _"close"_ to the key, and peers maintain connections to others that are varying levels of _"closeness"_ to themselves which makes it fast to find the nodes storing keys that you want. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#01e4df3d-e28e-43f9-b4fc-5c7949bc6b23)
			- > discovery-channel uses this module by publishing it's IP/Port combination to the discovery key onto the DHT. Since there is no central authority or single point of failure, publishing on the DHT is resistant to censorship and can survive sketchy networks. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#8788de8e-a0b9-45d3-a933-13cd8eff61bd)
			- > One of the problems with the DHT, is that items in the DHT don't expire right away, and searching for peers can yield a lot of false-positives. Also, maintining the connection information necessary for staying in the DHT takes up more computational resources. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#6318228e-b9d2-453f-a209-0e7c0efbcdb5)
			- > the DHT requires a set of _"bootstrap nodes"_ which are used to find more nodes to start building up your network. These bootstrap nodes are a potential source of failure and DHT clients should attempt to save any nodes they find for later use in order to have a way to bootstrap should the bootstrap nodes go down.
			  > 
			  ##  [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#32b8b315-c354-4919-aff6-6543a5ac9e9b)
			- > the DHT can yield false positives and requires participating in a network. To make peer lookup faster, discovery-channel makes use of [dns-discovery](https://github.com/mafintosh/dns-discovery) to talk to a list of centralized DNS servers to find peers. [⤴️](https://omnivore.app/me/super-high-level-18c07ef66d7#a5819326-1e8f-4c83-93c5-443b700ead57)
	- [Chainable monads in JavaScript – Marco Faustinelli's Muzietto Blog](https://omnivore.app/me/chainable-monads-in-java-script-marco-faustinelli-s-muzietto-blo-18eea36cd08)
	  collapsed:: true
	  site:: [Marco Faustinelli's Muzietto Blog](https://faustinelli.wordpress.com/2013/08/14/chainable-monads-in-javascript/)
	  author:: Muzietto
	  labels:: [[faustinelli]]
	  date-saved:: [[04/17/2024]]
	  date-published:: [[08/14/2013]]
		- ### Content
		  collapsed:: true
			- _**Gist**: the chainable monads presented here are JavaScript functions that allow idiomatic expressions such as_
			  
			  _`monad1.bind(function1)` ———> `monad2`_
			  
			  _Since JavaScript is typeless, there’s not really any reason to talk about monad\[A\] rather than monad\[B\], but the raison d’être of monads remains. JavaScript monads are created and manipulated by running convenient functions. The first monad in the chain usually comes from the run of a ‘unit’ function. This article shows all this in the case of the clueless Maybe monad. For more professionally relevant talk, you may want to go asap to the next article._
			  
			  After acquiring some proficiency in functional programming using JavaScript, I’ve come back lately to an old interest of mine: monads. For those who happen to read here and to share my basic dilemma: “what the hell do we need monads for, as productive professional programmers?” I’d like to anticipate that yes! I have eventually found one single application very, very likely to solve a serious, tough problem of mine at work. But it is pretty complicated and involves the state monad. More details are given in the next article.
			  
			  This article is not meant as an introduction to monads. It is just a presentation of one possible way to express them using advanced JavaScript concepts. I will suppose that things like map, flatMap (aka bind) and unit are already known and clear to you. Otherwise, the best introduction to monads I can suggest is “Monads are Elephants”, part 1 and 2\. Understanding the relationship between flatMap and for-comprehensions and being able to juggle functions between those two representations is imho necessary to obtain anything useful from monads.
			  
			  Going back to JavaScript: even though monads are mostly seen as typed objects, they don’t have to be. Whilst in Scala we have to make sure we use a f : String -> Monad\[Boolean\] as argument for the bind method of a Monad\[String\], in JavaScript we may forget about this, accept the risks of typelessness and enjoy a much readable, expressive syntax.
			  
			  I found a pair of good, useful pages around showing examples of monads in JavaScript (\[1\]\[2\]). The basic criterion I have used to pick my favorite ones is that they present code which creates monads as result of the plain execution of named functions, avoiding “new”, prototypes and other pseudo class-like JS muck.
			  
			  Another thing the pages I’ve mentioned here have in common (but which instead I find a lesser idea) is that they express the various unit, map, flatMap/bind etc. as binary factory methods operating on pairs of monads and mapping functions which are given as parameters to the factory; for example:
			  
			  `bind(monad,flatmappingFunction);`
			  
			  Alas, this becomes pretty nested, convoluted and difficult to read as soon as you begin to bind more than a pair of monads to each other. That’s why I prefer to see monads as entities with own capabilities, so that I may write:
			  
			  `monad.bind(functionA).bind(functionB).bind(functionC)`
			  
			  knowing that `monad.bind(functionA)` will create a new monad instance with its own bind method, ready to accept functionB as its parameter and build a third monad instance, and so on.
			  
			  Let’s stress once more that in this schema monads are always the product of the execution of plain named functions. There is no “new” involved nor prototypes to inherit from; “this” is mostly the running function (and when not, it’s always under the strict supervision of apply :-)). The only objects in the game are JS literals returned at the end of the various runs.
			  
			  Also relevant to observe here is that my schema uses the module pattern(\[3\]). The named function returns an object, which I call Monad.maybe or Monad.state to avoid littering the global object (caveat: I haven’t checked whether Monad namespaces exist already in any browser and what are their attributes!), and exposes the whole palette of methods for the given monad. So there is indeed a factory, but you only need Monad.<xxxx>.unit to start the dances.
			  
			  All the code shown here is available at the Github repository named _geiesmonads_ (user Muzietto). The next part of this article is dedicated to the Maybe monad (aka Option), which I personally find unfit for any practical usage in our day jobs (ok, ok: no more “if null”, big deal!). I am presenting it here only to illustrate the concept of chained JavaScript monads with a truly simple example. The real thing I am pursuing is the State monad, with its I/O management facets. But that’s a harder nut to crack and comes only in the next article, where you may want to jump right now, if you are already acquainted with Maybes and are looking for something really professionally relevant.
			  
			  \#\#\# Chainable Maybe monad in JavaScript
			  
			  So, here is my instanceable/chainable Maybe monad. Again, the only producer of monads (here and in other types of monad that I will present later) is unit. Producing a monad means attaching the various map/flatMap/unit/flatten/bind methods to the result function/object/monad/youNameIt that will be returned by unit. I’ve managed to have bind delegate to unit the preparation of the monad result of its binding. So, the monad builder is always unit.
			  
			  You may want to compare this implementation to the Maybe monad from \[2\].
			  
			  `var myMaybeMonad = function() { var unit = function(value){ var result = function(optionalBindingFunction){if (optionalBindingFunction) { // synt sugarreturn bind.apply(result,[optionalBindingFunction]); }else {return value; } }; result.bind = bind; result.flatten = flatten; result.unit = unit; result.instanceOf = instanceOf; result.map = map;return result; };`
			  
			  var map = function(fab){return bind(lift(fab));};
			  
			  // to be used in the context of the monad  
			  var flatten = function(){  
			  if (this()) {return this();}  
			  else { // a None is an empty function  
			  return function(){};  
			  }  
			  };
			  
			  // famb(flatten(ma)) –> mb  
			  var bind = function(famb){  
			  // runs in the context of the monad…  
			  return unit(famb(flatten.apply(this)))();  
			  };
			  
			  var instanceOf = function(){  
			  return ‘maybe’;  
			  };
			  
			  // fab –> famb  
			  var lift = function(fab){  
			  return function(x) {  
			  return unit(fab(x));  
			  };  
			  };
			  
			  return {  
			  unit: unit,  
			  lift: lift,  
			  map: map,  
			  flatten: flatten,  
			  instanceOf: instanceOf,  
			  bind: bind  
			  };  
			  }();
			  
			  Monad.maybe = myMaybeMonad;
			  
			  Monad.maybe.unit deserves a little explanation. It has here two roles:
			  
			  1. canonical: factory method for Maybes, given a value to wrap inside
			  2. special: syntactic sugar to allow to remove all explicit invocations to bind
			  
			  The second role makes the two following instructions identical:
			  
			  `monad.bind(functionA).bind(functionA) ` and ` monad(functionA)(functionB)`
			  
			  Please note the sugary business is realized in the case of this Maybe monad, but not in the State monad presented in next article.
			  
			  The example usage is taken straight from \[2\]. We need a sample function that transforms values contained in the maybes, like
			  
			  `var more = function(value){return 'more ' + value;};`
			  
			  First we need to lift it and make it useable by bind/flatMap:
			  
			  `var mMore = Monad.maybe.lift(more);`
			  
			  As you can see in the code of lift, this wraps more into a:
			  
			  `function(x) {return unit(more(x));}`
			  
			  Given a Maybe named coffee and containing, surprisingly, the string “coffee”, we bind it with mMore writing:
			  
			  `coffee.bind(mMore)` or `coffee(mMore)`
			  
			  and we get a Maybe which, once flattened, returns “more coffee”.  
			  [![mission_accomplished](https://proxy-prod.omnivore-image-cache.app/0x0,sTAS-L34JIDJMocsK0Fmh2qJxKaKdzElnmicSOMlEgcQ/https://faustinelli.wordpress.com/wp-content/uploads/2013/08/mission_accomplished.jpg?w=490)](https://faustinelli.wordpress.com/wp-content/uploads/2013/08/mission%5Faccomplished.jpg)
			  
			  \[1\] <http://igstan.ro/posts/2011-05-02-understanding-monads-with-javascript.html>  
			  \[2\] <http://jabberwocky.eu/2012/11/02/monads-for-dummies/>  
			  \[3\] <http://www.yuiblog.com/blog/2007/06/12/module-pattern/>
			  
			  \#\# Post navigation
	- [#1 - lists, cons, car, cdr in JS](https://omnivore.app/me/functional-programming-in-java-script-playing-with-lists-cons-ca-18eea3570a0)
	  collapsed:: true
	  site:: [Marco Faustinelli's Muzietto Blog](https://faustinelli.wordpress.com/2013/08/14/functional-programming-in-javascript-playing-with-lists-cons-car-and-cdr/)
	  author:: Muzietto
	  labels:: [[faustinelli]]
	  date-saved:: [[04/16/2024]]
	  date-published:: [[08/14/2013]]
		- ### Content
		  collapsed:: true
			- _**Gist**: JavaScript allows to describe lists as functions in a very natural way. By putting in place a few short albeit sophisticated functions called `cons`, `car `(aka `head`) and `cdr `(aka `tail`) a complete system for handling lists (`create`, `sort`, `filter`, etc.) as binary trees is shown here. Recursion of calls lies at the basis of all; any sizeable list in this system requires a huge call stack and, unfortunately, this seems to go against the way the various JavaScript engines are implemented. Therefore, this has to remain an exercise._
			  
			  In the last couple of years, my studies about functional programming have shifted focus, moved away from Java and gotten into JavaScript. This was caused firstly by me going to work for a web agency and having to become proficient in that pesky language (well, only in its good parts :-))
			  
			  Using JavaScript and its first-order functions, it is perfectly legitimate to stay away from classes, types and stuff. Being just you and the functions makes it just right to program …functionally.
			  
			  My most relevant products so far are all on [GitHub ](https://github.com/muzietto "Muzietto on GitHub")(user Muzietto):
			  
			  * _geieslists_: a complete (experimental !) library for managing lists built purely from primitive functions, starting from `cons`, `car`/`head`, `cdr`/`tail`, and ending up with high-level list operations like `splice `or `mergesort`
			  * _littleFunkyJavaScripter_: a thorough rewrite of the Little Schemer in JavaScript, using geieslists as engine to build and manage the unavoidable lists. I still have to “do” chapter 10, but the rest of the book is all there, especially the supercool Y-combinator.
			  * _seasonedFunkyJavaScripter_: still work in progress. Same as above with regards to the Seasoned Schemer. I just got till chapter 13 and got stuck over `call-with-current-continuation`. I hope I’ll be able to proceed further in the future: continuations are insanely interesting.
			  * _geiesmonads_: maybe and state monad in JavaScript – examples of usage. In the case of the state monad you can find there the embryo for [a whole system of modular, chainable operations](https://faustinelli.wordpress.com/2013/08/14/handling-io-with-the-state-monad-in-javascript/ "Handling I/O with the state monad in Javascript").
			  * _geiesfolds_: Fold right and then left in JavaScript. Lots of [examples ](https://faustinelli.wordpress.com/2010/04/22/foldleft-foldright/ "FoldLeft, FoldRight")of usages.
			  
			  (_geieslists_ reads _j-s-lists_ in Italian, how funny…)
			  
			  \#\#\# Example list computation in JavaScript
			  
			  I am presenting a simple example using geieslists. Let’s evaluate the head of a two-elements list.
			  
			  Inside geieslists any list (in the best LISP tradition) is the concatenation of an element `car `and another list `cdr`. This concatenation takes place through the application of a function named `cons `(they tell me that’s an abbreviation of _constructor_), which binds those two values. The resulting `cons `is obviously a list, just like `cdr `was. A list of two elements `'a'` and `'b'` is created recursively starting from a value `EMPTY_LIST` (we’ll see later what an `EMPTY_LIST` is) as:
			  
			  `list = cons('a',cons('b',EMPTY_LIST))`
			  
			  \#\#\#\# Off with her head!
			  
			  When we want to extract the head (aka `car`) of that list (aka `cons`), we apply over that `cons `a function named `head `(or `car`); `head `passes to `cons `a third function (in this case anonymous) which, given two arguments, simply returns the first one. Those two arguments happen to be the original two values given to cons in the previous paragraph. In detail, the expression of head is:
			  
			  ```actionscript
			  head = function(cons){
			    return cons(function(first,second){
			      return first;
			    });
			  };
			  ```
			  
			  In other words the head of a list comes from the application of function head to list; that’s the result of running:
			  
			  ```lisp
			  head(list) = (function(cons){
			    return cons(function(first,second){
			      return first;
			    });
			  )(list)
			  ```
			  
			  If we search instead the `tail`, we apply over the same `cons `a function named `tail `(or `cdr`); `tail `passes to `cons `a new anonymous binary function that receives the two original values of the `cons `and returns the second one. In short, the tail of a list is the result of running:
			  
			  ```lisp
			  tail(list) = (function(cons){
			    return cons(function(first,second){
			      return second;
			    });
			  )(list)
			  ```
			  
			  \#\#\#\# Happy the cons: he lives a simple life
			  
			  The missing piece is `cons`. `cons `is basically a binary function with a single simple goal in life: to allow other binary functions to access its two original arguments and use them as **their** own arguments. In a way, `cons `does nothing with its arguments; it just wraps them and keeps them warm and ready for other functions to use. In plain JavaScript:
			  
			  ```actionscript
			  cons = function(first, second) {
			    return function(anotherBinaryFunction){
			      return anotherBinaryFunction(first, second);
			    };
			  };
			  ```
			  
			  Technically `cons `is a function that receives two arguments, runs and returns as a result a function which in turn is ready to receive another function as its parameter. A bit dizzying, first time I saw it.
			  
			  This means that if we pass to the function `cons(first, second)` another function as parameter, the value `cons(first, second)(anotherBinaryFunction)` will be whatever returns from running `anotherBinaryFunctions `using `cons`‘ arguments as parameters. In detail:
			  
			  ```lisp
			  cons(first, second)(anotherBinaryFunction) =
			    anotherBinaryFunction(first, second);
			  ```
			  
			  In the case of the two-elements list we’d have:
			  
			  ```lisp
			  cons('a', cons('b', EMPTY_LIST))(anotherBinaryFunction) =
			    anotherBinaryFunction('a', cons('b', EMPTY_LIST));
			  ```
			  
			  If we want to extract the head of this two-elements list and we recall the definition of `head`, the substitution goes:
			  
			  ```lisp
			  cons('a', cons('b', EMPTY_LIST))(head) =
			    headInnerBinaryFunction('a', cons('b', EMPTY_LIST)) =
			    (function(x, y) { return x; })('a', cons('b', EMPTY_LIST)) = 'a';
			  ```
			  
			  This makes a lot of sense when told this way, but things ain’t that easy. In the code of our program we don’t write:
			  
			  `cons('a', cons('b', EMPTY_LIST))(head)`
			  
			  but we write instead:
			  
			  `head(cons('a', cons('b', EMPTY_LIST)))`
			  
			  What makes the magic happen, so that:
			  
			  `head(cons('a', cons(‘b’, EMPTY_LIST))) = 'a'`
			  
			  like if there was no `cons` in between ?
			  
			  \#\#\#\# Head’s Version
			  
			  Here is how the matter is seen from the point of view of head. I guess it is better to use here a simpler lambda notation (Scala, Java8) that allows to see clearly input parameters and returned values.
			  
			  In this notation `cons`, `head `and `tail `become respectively (please compare with the JavaScript implementations given previously):
			  
			  ```livescript
			  cons = (x, y) -> w -> w(x, y)
			  head = c -> c((x, y) -> x)
			  tail = c -> c((x, y) -> y)
			  ```
			  
			  Here `c` is a shortcut/reminder that the operand of `head `and `tail `is a `cons`. We saw that:
			  
			  ```lisp
			  head(list) = (c -> c((x, y) -> x))(cons('a', cons('b', EMPTY_LIST)))
			  ```
			  
			  NB: I am putting in **bold** the two brackets that delimit head.
			  
			  For the sake of clarity, let’s substitute:
			  
			  `cons('b', EMPTY_LIST) = TAIL`
			  
			  Substituting inside list the first usage of `cons `with its definition we obtain:
			  
			  ```lisp
			  head(list) = (c -> c((x, y) -> x))(cons('a', TAIL)) =
			  (c -> c((x, y) -> x))(w -> w('a', TAIL))
			  ```
			  
			  At this point we see that technically `head `receives its `cons`, which is nothing else than a closure carrying the whole list. Because of the definition of `head`, parameter `c` (the `cons`) becomes `c(fun)`, where `fun `is that convenient inner binary function that always returns the first of its own parameters: that’s where `head `got its name from. The steps are (please follow carefully the various substitutions):
			  
			  ```livescript
			  head(list) = (c -> c((x, y) -> x))(w -> w('a', TAIL)) =
			  (w -> w('a', TAIL))((x, y) -> x)) =
			  ((x, y) -> x))('a', TAIL) = 'a'
			  ```
			  
			  So, in a way, `head `and `cons `pass each other the sherry…
			  
			  Basically, `cons `and `head `(or `tail`) plug into each other: `cons `takes a function and gives it its own context; `head `takes a function and gives it a visitor that operates in its context. In a way, `head `is more _refined_; `cons `is more _generic_. `head `is the specialist, and so is `tail`.
			  
			  There is lot to understand here, a lot more to dig and discover:
			  
			  * are there other possible list specialists besides `head `and `tail`?
			  * what are the characteristics that make these functions pluggable in each other?
			  * what are other problems (not necessarily involving lists) that may be solved by pairs of such crafted functions?
			  
			  The key must be in the signatures, but it’s the bodies that express the intent. Gotta play some more with these concepts.
			  
			  **UPDATE** All the playing resulted in [another post](https://faustinelli.wordpress.com/2014/02/07/an-fp-pattern-what-is-it-whats-its-name/ "An FP pattern? What is it? What’s its name?"), which investigates whether this may be a known FP pattern.
			  
			  \#\#\#\# Full of Emptiness
			  
			  The last bit is the definition of `EMPTY_LIST`.
			  
			  `EMPTY_LIST` is something that, whenever encountered, signals that it’s time to stop processing the list. It can be any value. But here we are used to have `cdr `as a function, so it’s nicer that `EMPTY_LIST` be a function (provided that we use always exactly the same instance!). We like the fixed-point function because it reminds us of the Y-combinator, so we say:
			  
			  `EMPTY_LIST = function(x) { return x; }`
			  
			  \#\#\# The problem with the stack of the function calls
			  
			  At the heart of this system we have recursive calls to the cons function. This creates very soon a huge call stack for each list you need. It comes out that javaScript engines are not optimized for recurring on closures \[1\] (why is that?).
			  
			  This impacts both the waste of memory created by each list and the running time of a mid-sized cons operation. It is interesting to measure the impact on execution speed by using the benchmark page \[1\]
			  
			  The pages I owe credit to for building geieslists are primarily three:
			  
			  * <http://matt.might.net/articles/js-church/>
			  * <http://blairmitchelmore.com/javascript/cons-car-and-cdr>
			  * <http://dankogai.typepad.com/blog/2006/03/lambda%5Fcalculus.html>
			  
			  \[1\] see <http://spencertipping.com/js-instabench/> – please compare:  
			  – allocating small structures -> 1M (int, int) pairs as array literals  
			  – allocating small structures -> 1M (int, int) pairs as binary CPS closures  
			  – accessing small structures -> 1M (int, int) pairs as array literals  
			  – accessing small structures -> 1M (int, int) pairs as binary CPS closures
			  
			  \#\# Post navigation
	- [Persistence and Upgrade](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e)
	  collapsed:: true
	  site:: [erights.org](http://www.erights.org/data/serial/jhu-paper/upgrade.html)
	  author:: Mark S. Miller
	  date-saved:: [[04/16/2024]]
		- ### Content
		  collapsed:: true
			- ---
			  
			  (\*\*\* To be written)
			  
			  \#\# In Praise of Manual Persistence
			  
			  Computers crash too often. We wish our applications, data, and activities to last much longer. ==To achieve this, traditionally, programmers had to
			        encode their application's representations twice -- once as a live runtime
			        data structure and once as schema: as a file or database format to be
			        saved to disk. This was the pain of== _==manual persistence==_==.== Transparent orthogonal persistence was first conceived as a way to avoid this error-prone redundancy, essentially by masking the crash \[KeyKOS, EROS, PJama\]. A process running in such a system proceeds essentially as if no crash had happened. Such a process has an easy immortality, and the original persistence problem is solved.
			  
			  ==The== _==upgrade==_ ==problem occurs when we wish our application data
			        to survive a different kind of trauma -- the upgrade of the code the application
			        instantiates.== When using manual persistence, the upgrade problem is properly the _==schema evolution==_ ==problem -- to design, along with a new release
			        of an application, means for converting its persistent schema from an
			        old representation to the new one.== For example, later releases of an editor typically know how to read documents written by earlier releases.
			  
			  However, ==if one uses transparent orthogonal persistence== _==instead==_==of manual persistence, then the entire runtime representation becomes
			        the schema that needs to evolve.== This amplifies the difficulty of the upgrade problem, often to a fatal extent. Manual persistence provides a source of great leverage for upgrade, and one that's easy to miss: By encoding their representation twice, programmers naturally bring to each encoding those concerns specific to the purpose of that encoding, often without thinking about this dichotomy explicitly.
			  
			  _Do you, Programmer,_ 
			  _take this Object to be part of the persistent state of your application,_ 
			  _to have and to hold,_ 
			  _through maintenance and iterations,_ 
			  _for past and future versions,_ 
			  _as long as the application shall live?_
			  
			  \--Arturo Bejar \[ref StateBundles\]
			  
			  Normally, ==we view the design of the runtime representation of our application
			        as the "real" one, and wish the persistent state to be derived from it.
			        But as this quote from Arturo suggests, the kind of commitment one needs
			        to invest in persistent state isn't appropriate for runtime data structures==, and shouldn't be. Runtime data structures are often delicate complex machines in motion, with many complex distributed consistency assumptions between the parts, designed to interact efficiently with an ongoing world of users or devices, and encoding meaning in ways that are largely undocumented. The complexity of this runtime world traditionally relies on the program itself staying constant while the process is executing. 
			  
			  By contrast, when programmers design schema for manual persistence, Arturo's question is properly uppermost on their mind. These schema are stable representations designed with little redundancy, few opportunities for distributed inconsistency, with little penalty for inefficient representations, encoding only the essential application state that needs to survive across time, and where this encoding is much more likely to be well documented. Runtime representations emphasize the operational, whereas schema emphasize the declarative.
			  
			  Once the customers of an application accumulate their own privately held persistent state from this application, such as their own private documents, then Arturo's question becomes unavoidable. To release a new version of an application without losing old customers, one must enable those customers to revive their old state into an instantiation of the new version of the application reliably -- with no per-instance programmer intervention.
			  
			  Smalltalk, with its easy support for live upgrade, is not a counter-example. This support cannot be made reliable, and is instead designed for programmers-as-customers who know how to recover from inconsistencies.
			  
			  If the programmers were using only transparent orthogonal persistence to give the application's data long life, then this upgrade problem resembles maintenance on an operational (though suspended) machine whose workings may be largely mysterious. Worse, since upgrades must happen in an automated way on customer data without programmers present, it more closely resembles building an upgrade-robot that will reliably perform this maintenance on any possible state such a machine may be in. With machines of great complexity, the feasible changes will usually only be minor tweaks and adjustments, not major design changes. The difficulty of upgrade will place a severe limit on the speed with which a vendor will be able to improve their program. This kind of persistence indeed provides a process with easy immortality, but only as a living fossil.
			  
			  If, on the other hand, the programmers were using manual persistence (whether through foresight, habit, or lack of an alternative), then, when they wish to release a new version, the total number of semantically significant cases in the schema should usually be small enough that they can each be thought about carefully, in order to see how to convert its meaning into the closest appropriate meaning in the application's new version. The upgrade-robot arrives with parameterized blueprints (the new version of the program) for building a new running machine (instantiating a new running process). The schema provides the arguments needed to complete the blueprint. The old machine is scrapped and a new machine is freshly built around these arguments. 
			  
			  As another analogy, if the runtime representation is the application instance's phenotype, then the schema is the instance's genotype. Biological evolution works partially because it operates only on the genotype, where a genotype unfolds into the vastly more complex phenotype via the indirect operational process of embryology. Like an ephemeral live process instantiating an application (ie, a vat incarnation), each phenotype operates only from a fixed snapshot of its genotype. Evolution only happens in the transition between generations. While we needn't take these analogies too seriously, they can significantly aid our intuitions.
			  
			  Having made the greater initial investment in engineering two representations, the programmers using manual-persistence will then be able to improve their application much faster without losing their customers, perhaps overtaking the head start of the harder-to-evolve but faster time-to-market singly represented alternative.
			  
			  The first step in dealing with the schema evolution problem is to mostly avoid the problem by saving vastly smaller schema.
			  
			  \*\*\* Basic _**E**_ orientation, including Vats, distributed objects, live refs and SturdyRefs, and object-capability security.
			  
			  \*\*\* Assumption of per-vat persistence by _**E**_ computational model.
			  
			  \#\#  Manual Revival as Zero-Delta Upgrade
			  
			  Note that none of the above discussion assumes that transparent orthogonal persistence and largely manual persistence are exclusive options. A system may well use both: transparent orthogonal persistence to mask crashes efficiently \[KeyKOS, EROS\], and largely manual persistence only when upgrading. A future _**E**_\-on-EROS may very well operate in this mode. In this scenario, performance need not be a goal of the largely manual system.
			  
			  In the absence of support for high speed transparent orthogonal persistence, a system may very well use largely manual persistence mechanism for both purposes. Each post-crash revival is then a degenerate zero-delta upgrade: Each revival runs through the upgrade-supporting logic each time, even when no upgrade is actually occurring. The current _**E**_, running on Java running on stock OSes, operates in this mode. Performance therefore should be a goal of _**E**_'s persistence mechanisms, but is not at this time.
			  
			  \#\#  Mechanism / Policy Separation
			  
			  Of course, by definition, anything a program does is automated, so what do we even mean by "manual" persistence? We are not arguing against automation, abstraction, and reuse. Rather the issue is whether to build a primitively provded inescapable comprehensive solution _vs._ a toolkit of reusable tools from which one can roll one's own solution, or several co-existing ones. When one can design a single solution adequate for the needed range of uses, often one should, as the uniformity of a single comprehensive solution can bring great benefits. When one size doesn't fit all, we should instead turn to the tradition of _mechanism / policy_ separation. A toolkit can serve as the mechanisms out of which one may build a variety of persistence systems embodying a (limited) range of policy choices.
			  
			  What we mean by "manual" persistence is that the _**E**_ kernel does not itself provide a primitive persistence system, but rather provides primitive tools out of which persistence systems may be fashioned. The _**E**_ system as a whole does provide a default persistence system built "manually" from these tools, but this has the status of library code rather than fundamental primitives. Multiple such libraries can coexist, and the default one is in no sense special.
			  
			  \*\*\* incoherent notes here to the end of this file. Do Not Read \*\*\*\* 
			  
			  The tool most central to such a toolkit is a serialization / unserialization system. _**E**_ currently uses Java's serialization streams, which has a rather flexible and mature set of customization hooks for building streams embodying a wide range of serialization policies. Most of the goals of our toolkit are already achieved by Java's serialization design, so this paper proceeds from there.
			  
			  Here, we wish to support a range of compromises between the explicitness and separation of concerns of manual persistence _vs._ the economy of expression provided by automating aspects of persistence. Why a range of compromises? Why not try to find one good compromise and just build that? Because there are too many kinds of persistence policies that plausibly need to be supported.
			  
			  * What to save/restore _vs._ what to reconstruct _vs._ what to reconnect. (More on this below.)
			  * Where to save persistent state? Files? Databases?
			  * When to revive saved state? On process (vat) revival, or faulting on-demand?
			  * Fail-stop vs. best efforts. When problems are hit, either saving or restoring, should one give up or make due?
			  * What is a consistent state, and how does one obtain access such a state? Does such a state include messages in flight?
			  * Transactions: When is a saved state a basis for commitment? How does one abort and fall back to a previous state? As separate subsystems asynchronously snapshot, how is consistency recovered when they revive from different times?  
			  Faced with such a variety, we use the traditional answer: mechanism / policy separation. We have built persistence support into _**E**_ in two layers:  
			  A set of building blocks from which an application developer can (within limits) build a persistence system embodying those policies that serve their needs, including a fully manual system if desired. These mechanisms must not allow an unprivileged persistence subsystem from violating any of _**E**_'s security properties, while allowing for operation that's reasonable for a subsystem holding a given set of authorities.
			  * One example persistence system, built only from these building blocks, embodying a set of policy choices that won't be suitable for all applications, but is nevertheless designed to be widely reused.  
			  \#\#  7.6\. Persistence and Mutual Suspicion
			  
			  ---
			  
			  Unless stated otherwise, all text on this page which is either unattributed or by Mark S. Miller is hereby placed in the public domain.
		- ### Highlights
		  collapsed:: true
			- > To achieve this, traditionally, programmers had to encode their application's representations twice -- once as a live runtime data structure and once as schema: as a file or database format to be saved to disk. This was the pain of _manual persistence_. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#424acfd1-c2cb-434e-8e56-fed0779ce32d)
			- > The _upgrade_ problem occurs when we wish our application data to survive a different kind of trauma -- the upgrade of the code the application instantiates. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#cb11d24b-f63f-492d-ba8c-d27e374cc3b2)
			- > When using manual persistence, the upgrade problem is properly the _schema evolution_ problem -- to design, along with a new release of an application, means for converting its persistent schema from an old representation to the new one. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#ab321696-59bc-456e-9a7d-f24d474c3cdd)
			- > For example, later releases of an editor typically know how to read documents written by earlier releases. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#8c5dc357-2932-4dc4-898e-c0d557b33674)
			- > if one uses transparent orthogonal persistence _instead_ of manual persistence, then the entire runtime representation becomes the schema that needs to evolve. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#5ff7e3e7-ec94-465c-a917-d6cf877d7e06)
			- > This amplifies the difficulty of the upgrade problem, often to a fatal extent. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#c40813fa-e17a-4e03-94e0-14a6581e6eb2)
			- > Manual persistence provides a source of great leverage for upgrade, and one that's easy to miss: By encoding their representation twice, programmers naturally bring to each encoding those concerns specific to the purpose of that encoding, often without thinking about this dichotomy explicitly. [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#ea405d00-3f3b-40f3-bacc-ff0b00f9e4e1)
			- > we view the design of the runtime representation of our application as the "real" one, and wish the persistent state to be derived from it. But as this quote from Arturo suggests, the kind of commitment one needs to invest in persistent state isn't appropriate for runtime data structures [⤴️](https://omnivore.app/me/persistence-and-upgrade-18ee98ac81e#54fd14ba-997b-47f9-886e-db07396cbe20)
	- [How to Choose the Right Parameters for Argon2 – Twelve 21](https://omnivore.app/me/how-to-choose-the-right-parameters-for-argon-2-twelve-21-18ec11c593e)
	  collapsed:: true
	  site:: [web.archive.org](https://web.archive.org/web/20231019140616/https://www.twelve21.io/how-to-choose-the-right-parameters-for-argon2/)
	  author:: Bryan Burman
	  date-saved:: [[04/09/2024]]
	  date-published:: [[06/06/2019]]
		- ### Content
		  collapsed:: true
			- In [my last post](https://web.archive.org/web/20231019140616/https://www.twelve21.io/how-to-securely-store-a-password/), I discussed four cryptographic hashing functions that are suitable for password storage. I pointed out that Argon2 (in particular Argon2id) is the hashing function that leading cryptographers around the world recommend. Simply choosing to use Argon2 for password storage is only half of the battle.
			  
			  Argon2 accepts three parameters you can use to tune the algorithm to your needs. In this post, I’ll discuss these three parameters and hopefully give you the tools and knowledge to show you how to choose the best parameters for your scenario.
			  
			  \#\# What’s the Goal?
			  
			  As I’ve discussed [in ad nauseam](https://web.archive.org/web/20231019140616/https://www.twelve21.io/the-case-for-the-web-authentication-webauthn-protocol/), storing passwords in a system poses countless security threats. Storing strong, cryptographic hashes of passwords in a database is one mitigation against an attacker who is able to breach your system. But it doesn’t stop here.
			  
			  There is a delicate balance between execution time and required resources when calculating a password hash. Your goal is to maximize the time it takes to calculate a hash to prevent an attacker from performing a brute force attack. However, your goal is to also minimize the cost to your system so that your product doesn’t come to a grinding halt when someone wants to login.
			  
			  Argon2 allows you to manipulate three parameters in an effort to find this sweet spot:
			  
			  * The number of iterations, or passes, that the algorithm will make over memory, an increase in such yields additional protection against tradeoff attacks.
			  * The amount of memory to utilize, with more memory protecting against customized hardware attacks.
			  * The degree of parallelism, or the number of concurrent threads, that the algorithm will utilize to compute the hash.
			  
			  \#\# Choosing Correct Parameters
			  
			  \#\#\# Choose the Execution Time
			  
			  Before you can begin to choose the parameters for Argon2, you need to consider what the execution time of the algorithm needs to be. This will unfortunately create some tension with [recommended UX response times](https://web.archive.org/web/20231019140616/https://www.nngroup.com/articles/response-times-3-important-limits/). Unfortunately (and fortunately) for the user, security really needs to win out here.
			  
			  The Argon2 documentation recommends 0.5 seconds for authentication. [Libsodium’s documentation](https://web.archive.org/web/20231019140616/https://libsodium.gitbook.io/doc/password%5Fhashing/the%5Fargon2i%5Ffunction) recommends 1 second for web applications and 5 seconds for desktop applications. I personally wouldn’t recommend anything more than 1 second or your users will hate you and logins will end up DoS’ing your application. Anything less than 0.5 seconds is a certain security failure.
			  
			  \#\#\# Choose the Degree of Parallelism
			  
			  According to the Argon2 documentation, the degree of parallelism you choose should be dependent upon the number of CPU cores on your authentication server. Simply double the number of cores to determine the degree of parallelism. For example, if your authentication server CPU has four cores, then you should choose a degree of parallelism of eight.
			  
			  \#\#\# Choose the Amount of Memory
			  
			  The cost of computing a password hash should be “[dominated by memory-related costs](https://web.archive.org/web/20231019140616/https://www.youtube.com/watch?v=z-VqQzwN4bs)“. The more memory you throw at Argon2, the more resources the algorithm takes up and the more computationally expensive the process. The only way that an attacker can negate this is to purchase more memory. The hardware requirements quickly make the cost prohibitive.
			  
			  Because of this, your goal should be to start calibrating Argon2 with the largest memory usage possible. Then, work your way down until you achieve the execution time required. Once you find the memory usage required to achieve the execution time, you are at a good starting point for the next step.
			  
			  \#\#\# Choose the Number of Iterations
			  
			  Now that you have identified parallelism and memory usage, simply increase the number of iterations until the algorithm exceeds execution time. For example, if your maximum execution time is 2 seconds, 2 iterations cost you 1.9 seconds, and 3 iterations cost you 2.1 seconds, choose 2 iterations.
			  
			  From what I can find, there isn’t a documented minimum number of iterations. The Argon2 documentation does allude that a single iteration with higher memory usage is acceptable. Some contributors to the algorithm recommend a minimum of 2 iterations. Argon2’s benchmark utility utilizes 3 iterations. Libsodium’s implementation utilizes 4 iterations for sensitive operations. And the OWASP password storage cheat sheet is out there in left field by recommending upwards of 40 iterations.
			  
			  Unfortunately, there is no clear guidance and I am no cryptographer. Libsodium, however, is held in quite high respect, so a minimum number of 4 iterations is probably conservative. Thus, if Argon2 completes the password hashing algorithm in an acceptable amount of time using less iterations, decrease the memory usage and then increase the number of iterations until you reach the execution time limit. Repeat this process until you surpass 4 iterations.
			  
			  \#\# An Easier Way
			  
			  I’ve taken the liberty to throw together a small .NET Core command line application that eases the burden of choosing the correct parameters. You can download the code from GitHub [here](https://web.archive.org/web/20231019140616/https://github.com/bburman/Twelve21.PasswordStorage).
			  
			  Simply execute the following commands to clone the repo and perform the Argon2 calibration function:
			  
			  git clone https://github.com/bburman/Twelve21.PasswordStorage.git
			  cd ./Twelve21.PasswordStorage
			  dotnet build
			  cd ./Twelve21.PasswordStorage
			  dotnet run a2c
			  
			  By default, the application will use a degree of parallelism based on the number of CPU cores. It will use a minimum of two iterations and a maximum of 1 second for execution time. Use the -h parameter to see a list of options that you can use to fine tune the algorithm. The application will run and show you the best results, similar to:
			  
			  Best results:
			  M =  256 MB, T =    2, d = 8, Time = 0.732 s
			  M =  128 MB, T =    6, d = 8, Time = 0.99 s
			  M =   64 MB, T =   12, d = 8, Time = 0.968 s
			  M =   32 MB, T =   24, d = 8, Time = 0.896 s
			  M =   16 MB, T =   49, d = 8, Time = 0.973 s
			  M =    8 MB, T =   96, d = 8, Time = 0.991 s
			  M =    4 MB, T =  190, d = 8, Time = 0.977 s
			  M =    2 MB, T =  271, d = 8, Time = 0.973 s
			  M =    1 MB, T =  639, d = 8, Time = 0.991 s
			  
			  It’s important to note that this application needs to executed on the server under the configuration for which you intend to perform authentication. The above was executed on a business-grade company laptop, not an application server. Results will vary and you need to select parameters that will match your production environment.
			  
			  \#\# An Alternative Method
			  
			  Alternatively, you can utilize a library, such as Libsodium. A popular library, such as this, has reasonable parameters built into the platform. This not only reduces the choices that you have to make as a developer, but also prohibits someone from making a blatant mistake. Using a library comes with the added bonus that, if a library is ever updated, simply pulling in the update will yield more accurate parameters.
			  
			  The downside is that, if you know what you are doing, it’s difficult to get the fine-grained control over the algorithm that you might need. Still, choosing reasonable defaults from a library such as Libsodium is a much better alternative to using blatantly bad parameter choices.
			  
			  \#\# Conclusion
			  
			  This blog post discussed the parameters that the Argon2 algorithm takes as input. It also discussed the trade-off between these parameters with regards to execution time. Generally a higher execution time will yield a tougher password hash to brute-force, but it could also cripple your system. Hopefully this post helps you determine parameters that are optimized for your environment while also making it more difficult for an attacker.
			  
			  In a future post, we’ll discuss how to properly use Argon2 from both .NET and Java applications. So stick around. And, as always, if you have questions or comments, please leave me a comment below.
	- [Introduction to Peer-to-Peer Networks](https://omnivore.app/me/introduction-to-peer-to-peer-networks-18eb65b604d)
	  collapsed:: true
	  site:: [Lifewire](https://www.lifewire.com/introduction-to-peer-to-peer-networks-817421)
	  author:: Bradley Mitchell
	  date-saved:: [[04/06/2024]]
	  date-published:: [[05/04/2008]]
		- ### Content
		  collapsed:: true
			- Most home networks are hybrid P2P networks
			  
			  Peer-to-peer networking is an approach to [computer networking](https://www.lifewire.com/what-is-computer-networking-816249) in which all computers share equivalent responsibility for processing data. Peer-to-peer networking (also known as peer networking) differs from client-server networking, where specific devices have responsibility for providing or serving data, and other devices consume or otherwise act as clients of those servers.
			  
			  \#\#  Characteristics of a Peer Network 
			  
			  Peer-to-peer networking is common on small [local area networks](https://www.lifewire.com/what-is-lan-4684071) (LANs), particularly home networks. Both wired and wireless home networks can be configured as peer-to-peer environments.
			  
			  Computers in a peer-to-peer network run the same [networking protocols](https://www.lifewire.com/definition-of-protocol-network-817949) and software. Peer network devices are often situated physically near one another, typically in homes, small businesses, and schools. Some peer networks, however, use the internet and are geographically dispersed worldwide.
			  
			  Home networks that use [broadband routers](https://www.lifewire.com/what-is-a-broadband-router-816301) are hybrid peer-to-peer and client-server environments. The router provides centralized internet connection sharing, but file, printer, and other resource sharing are managed directly between the local computers involved.
			  
			  \#\#  Peer-to-Peer and P2P Networks 
			  
			  Internet-based peer-to-peer networks became popular in the 1990s due to the development of [P2P](https://www.lifewire.com/torrent-file-2622839) file-sharing networks such as Napster. Technically, many P2P networks are not pure peer networks but rather hybrid designs as they use central servers for some functions such as search.
			  
			  \#\#  Peer-to-Peer and Ad-Hoc Wi-Fi Networks 
			  
			  [Wi-Fi](https://www.lifewire.com/what-is-wi-fi-2377430) (wireless) networks support [ad-hoc](https://www.lifewire.com/ad-hoc-mode-in-wireless-networking-816560) connections between devices. Ad-hoc Wi-Fi networks are pure peer-to-peer compared to those that use wireless routers as an intermediate device. Devices that form ad-hoc networks require no infrastructure to communicate.
			  
			  \#\#  Benefits of a Peer-to-Peer Network 
			  
			  P2P networks are robust. If one attached device goes down, the network continues. In client-server networks, when the server goes down, it takes the entire network with it.
			  
			  Computers in peer-to-peer [workgroups](https://www.lifewire.com/definition-of-workgroup-816285) can be configured to allow [sharing of files](https://www.lifewire.com/file-sharing-on-computer-networks-817371), [printers](https://www.lifewire.com/windows-file-and-printer-sharing-818221), and other resources across all devices. Peer networks allow data to be shared in both directions, whether for downloads to a computer or uploads from a computer.
			  
			  On the internet, peer-to-peer networks handle a high volume of file-sharing traffic by distributing the load across many computers. Because they do not rely exclusively on central servers, P2P networks scale better and are more resilient than client-server networks in case of failures or traffic bottlenecks.
			  
			  Peer-to-peer networks are relatively easy to expand. As the number of devices in the network increases, the power of the P2P network increases, as each additional computer is available for processing data.
			  
			  \#\#  Security Concerns 
			  
			  Like client-server networks, peer-to-peer networks are vulnerable to security attacks.
			  
			  * Because each device participates in routing traffic through the network, hackers can easily launch denial of service attacks.
			  * P2P software acts as server and client, which makes peer-to-peer networks more vulnerable to remote attacks than client-server networks.
			  * Data that is corrupt can be shared on P2P networks by modifying files that are on the network to introduce malicious code.
			  
			  Thanks for letting us know!
			  
			  Get the Latest Tech News Delivered Every Day
			  
			  [Subscribe](\#)
	- [Smart Wallet Dapp Architecture | Agoric Documentation](https://omnivore.app/me/https-docs-agoric-com-guides-getting-started-contract-rpc-html-18e58ac3d09)
	  collapsed:: true
	  site:: [docs.agoric.com](https://docs.agoric.com/guides/getting-started/contract-rpc)
	  labels:: [[agoric]]
	  date-saved:: [[03/19/2024]]
	  date-published:: [[03/18/2024]]
		- ### Content
		  collapsed:: true
			- The [Agoric Platform](https://docs.agoric.com/guides/platform/) consists of smart contracts and services such as [Zoe](https://docs.agoric.com/guides/zoe/) running in a [Hardened JavaScript](https://docs.agoric.com/guides/js-programming/hardened-js.html) VM running on top of a Cosmos SDK consensus layer. Clients interact with the consensus layer by making queries and submitting messages in signed transactions. In the Smart Wallet Architecture, dapps consist of
			  
			  * Hardened JavaScript smart contracts
			  * clients that can submit offers and query status via the consensus layer
			  
			  ![smart wallet dapp sequence diagram](https://proxy-prod.omnivore-image-cache.app/0x0,skEfV5hzXuxJxYAULHxTjYpnsytOu-KnxNsZHMBrZ3xc/https://docs.agoric.com/assets/sw-dapp-arch.fMM6JZGd.svg)
			  
			  1. A client formats an offer, signs it, and broadcasts it.
			  2. The offer is routed to the `walletFactory` contract, which finds (or creates) the `smartWallet` object associated with the signer's address and uses it to execute the offer.
			  3. The `smartWallet` calls `E(zoe).offer(...)` and monitors the status of the offer, emitting it for clients to query.
			  4. Zoe escrows the payments and forwards the proposal to the contract indicated by the offer.
			  5. The contract tells Zoe how to reallocate assets.
			  6. Zoe ensures that the reallocations respect offer safety and then provides payouts accordingly.
			  7. The client's query tells it that the payouts are available.
			  
			  \#\# Signing and Broadcasting Offers [​](\#signing-and-broadcasting-offers)
			  
			  One way to sign and broadcast offers is with the `agd tx ...` command. For example:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
			  
			  ```routeros
			  agd tx swingset wallet-action --allow-spend "$ACTION" \
			  --chain-id=agoriclocal --from=acct1
			  ```
			  
			  Another is using a wallet signing UI such as Keplr via the [Keplr API](https://docs.keplr.app/api/).
			  
			  Given sufficient care with key management, a [cosmjs SigningStargateClient](https://cosmos.github.io/cosmjs/latest/stargate/classes/SigningStargateClient.html) or any other client that can deliver a [agoric.swingset.MsgWalletSpendAction](https://github.com/Agoric/agoric-sdk/blob/mainnet1B/golang/cosmos/proto/agoric/swingset/msgs.proto\#L70) to a [Cosmos SDK endpoint](https://docs.cosmos.network/main/core/grpc%5Frest) works.
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)proto
			  
			  ```protobuf
			  message MsgWalletSpendAction {
			    bytes owner = 1;
			    string spend_action = 2;
			  }
			  ```
			  
			  \#\# Querying VStorage [​](\#querying-vstorage)
			  
			  [VStorage](https://github.com/Agoric/agoric-sdk/tree/master/golang/cosmos/x/vstorage\#readme) (for "Virtual Storage") is a key-value store that is read-only for clients of the consensus layer. From within the JavaScript VM, it is accessed via a `chainStorage` API with a node at each key that is write-only; a bit like a `console`.
			  
			  ![vstorage query diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sIqAE0XWp_PMqp_lvytreIJing2t19kAsBKdrcPZZjpQ/https://docs.agoric.com/assets/vstorage-brand-q.SCB_G9qx.svg)
			  
			  The protobuf definition is [agoric.vstorage.Query](https://github.com/Agoric/agoric-sdk/blob/mainnet1B/golang/cosmos/proto/agoric/vstorage/query.proto\#L11):
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)proto
			  
			  ```protobuf
			  service Query {
			  // Return an arbitrary vstorage datum.
			  rpc Data(QueryDataRequest) returns (QueryDataResponse) {
			    option (google.api.http).get = "/agoric/vstorage/data/{path}";
			  }
			  
			  // Return the children of a given vstorage path.
			  rpc Children(QueryChildrenRequest)
			    returns (QueryChildrenResponse) {
			      option (google.api.http).get = "/agoric/vstorage/children/{path}";
			  }
			  }
			  ```
			  
			  We can issue queries using, `agd query ...`:
			  
			  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
			  
			  ```routeros
			  $ agd query vstorage children 'published.agoricNames'
			  
			  children:
				- brand
				- installation
				- instance
				  ...
				  ```
				  
				  The [Agoric CLI](https://docs.agoric.com/guides/agoric-cli/) `follow` command supports vstorage query plus some of the marshalling conventions discussed below:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
				  
				  ```prolog
				  $ agoric follow -lF :published.agoricNames.brand
				  [
				  [
				  "BLD",
				  slotToVal("board0566","Alleged: BLD brand"),
				  ],
				  [
				  "IST",
				  slotToVal("board0257","Alleged: IST brand"),
				  ],
				  ...
				  ]
				  ```
				  
				  vstorage viewer by p2p
				  
				  The [vstorage-viewer](https://github.com/p2p-org/p2p-agoric-vstorage-viewer) contributed by p2p is often _very_ handy:
				  
				  [![vstorage viewer screenshot](https://proxy-prod.omnivore-image-cache.app/0x0,sGme0IEveRijb5RUHKEcxKvKJ9wmZrafFxNSL20MEE8s/https://user-images.githubusercontent.com/150986/259798595-40cd22f0-fa01-43a9-b92a-4f0f4813a4f6.png)](https://p2p-org.github.io/p2p-agoric-vstorage-viewer/\#https://devnet.rpc.agoric.net/%7Cpublished,published.agoricNames%7C)
				  
				  \#\# Specifying Offers [​](\#specifying-offers)
				  
				  Recall that for an agent within the JavaScript VM, [E(zoe).offer(...)](https://docs.agoric.com/reference/zoe-api/zoe.html\#e-zoe-offer-invitation-proposal-paymentpkeywordrecord-offerargs) takes an `Invitation` and optionally a `Proposal` with `{ give, want, exit }`, a `PaymentPKeywordRecord`, and `offerArgs`; it returns a `UserSeat` from which we can [getPayouts()](https://docs.agoric.com/reference/zoe-api/user-seat.html\#e-userseat-getpayouts).
				  
				  ![Zoe API diagram, simplified](https://proxy-prod.omnivore-image-cache.app/0x0,sW1nJfLsMAsWaDsM8LnRkj5aTbmggbzlQhHdV53Jdvfg/https://docs.agoric.com/assets/zoe-simp.NTJeh880.svg)
				  
				  In the Smart Wallet architecture, a client uses an `OfferSpec` to tell its `SmartWallet` how to conduct an offer. It includes an `invitationSpec` to say what invitation to pass to Zoe. For example:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```php
				  /** @type {import('@agoric/smart-wallet').InvitationSpec} */
				  const invitationSpec = {
				  source: 'contract',
				  instance,
				  publicInvitationMaker: 'makeBattleInvitation',
				  invitationArgs: ['troll'],
				  };
				  ```
				  
				  Here the `SmartWallet` calls `E(zoe).getPublicFacet(instance)` and then uses the `publicInvitationMaker` and `invitationArgs` to call the contract's public facet.
				  
				  ![InvitationSpec sequence diagram](https://proxy-prod.omnivore-image-cache.app/0x0,skUHEvBvDb3zL4Mk3If4UPVjf_qtNeR6pU9E0PI43hxs/https://docs.agoric.com/assets/inv-spec.5-1ou8Wm.svg)
				  
				  InvitationSpec Usage
				  
				  Supposing `spec` is an `InvitationSpec`, its `.source` is one of:
				  
				  * `==purse==` ==- to make an offer with an invitation that is already in the Invitation purse of the smart wallet and agrees with== `==spec==` ==on== `==.instance==` ==and== `==.description==` ==properties.== For example, in [dapp-econ-gov](https://github.com/Agoric/dapp-econ-gov), committee members use invitations sent to them when the committee was created.
				  * `contract` \- the smart wallet makes an invitation by calling a method on the public facet of a specified instance: `E(E(zoe).getPublicFacet(spec.instance)[spec.publicInvitationMaker](...spec.invitationArgs)`
				  * `agoricContract` \- for example, from [dapp-inter](https://github.com/Agoric/dapp-inter):
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```prolog
				  {
				  source: 'agoricContract',
				  instancePath: ['VaultFactory'],
				  callPipe: [
				   ['getCollateralManager', [toLock.brand]],
				   ['makeVaultInvitation'],
				  ],
				  }
				  ```
				  
				  The smart wallet finds the instance using `E(agoricNames).lookup('instance', ...spec.instancePath)` and makes a chain of calls specified by `spec.callPipe`. Each entry in the callPipe is a `[methodName, args?]` pair used to execute a call on the preceding result. The end of the pipe is expected to return an Invitation.
				  
				  * `continuing` \- For example, `dapp-inter` uses the following `InvitationSpec` to adjust a vault:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```css
				  {
				  source: 'continuing',
				  previousOffer: vaultOfferId,
				  invitationMakerName: 'AdjustBalances',
				  }
				  ```
				  
				  In this continuing offer, the smart wallet uses the `spec.previousOffer` id to look up the `.invitationMakers` property of the result of the previous offer. It uses `E(invitationMakers)[spec.invitationMakerName](...spec.invitationArgs)` to make an invitation.
				  
				  The client fills in the proposal, which instructs the `SmartWallet` to withdraw corresponding payments to send to Zoe.
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```php
				  /** @type {import('@agoric/smart-wallet').BridgeAction} */
				  const action = harden({
				  method: 'executeOffer',
				  offer: {
				  id: 'battle7651',
				  invitationSpec,
				  proposal: {
				    give: { Gold: AmountMath.make(brands.gold, 100n) },
				  },
				  },
				  });
				  ```
				  
				  But recall the `spend_action` field in `MsgWalletSpendAction` is a string. In fact, the expected string in this case is of the form:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```capnproto
				  t.regex(spendAction, /^{"body":"\#.*","slots":\["board123","board32342"\]}$/);
				  const goldStuff =
				  '\\"brand\\":\\"$1.Alleged: Gold Brand\\",\\"value\\":\\"+100\\"';
				  t.true(spendAction.includes(goldStuff));
				  ```
				  
				  We recognize `"method":"executeOffer"` and such, but `body:`, `slots:`, and `$1.Alleged: Gold Brand` need further explanation.
				  
				  \#\#\# Marshalling Amounts and Instances [​](\#marshalling-amounts-and-instances)
				  
				  To start with, amounts include `bigint`s. The `@endo/marshal` API handles those:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```reasonml
				  const m = makeMarshal(undefined, undefined, smallCaps);
				  
				  const stuff = harden([1, 2, 3n, undefined, NaN]);
				  const capData = m.toCapData(stuff);
				  t.deepEqual(m.fromCapData(capData), stuff);
				  ```
				  
				  To marshal brands and instances, recall from the discussion of [marshal in eventual send](https://docs.agoric.com/guides/js-programming/eventual-send.html\#e-and-marshal-a-closer-look) how remotables are marshalled with a translation table.
				  
				  The [Agoric Board](https://docs.agoric.com/reference/repl/board.html) is a well-known name service that issues plain string identifiers for object identities and other passable _keys_ (that is: passable values excluding promises and errors). Contracts and other services can use its table of identifiers as a marshal translation table:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```dart
				  /** @type {Record<string, Brand>} */
				  const brands = {
				  gold: asset.gold.brand,
				  victory: asset.victory.brand,
				  };
				  
				  // explicitly register brand using the board API
				  const victoryBrandBoardId = await E(theBoard).getId(brands.victory);
				  t.is(victoryBrandBoardId, 'board0371');
				  
				  // When the publishing marshaler needs a reference marker for something
				  // such as the gold brand, it issues a new board id.
				  const pubm = E(theBoard).getPublishingMarshaller();
				  const brandData = await E(pubm).toCapData(brands);
				  t.deepEqual(brandData, {
				  body: `\#${JSON.stringify({
				  gold: '$0.Alleged: Gold Brand',
				  victory: '$1.Alleged: Victory Brand',
				  })}`,
				  slots: ['board0592', 'board0371'],
				  });
				  ```
				  
				  To reverse the process, clients can mirror the on-chain board translation table by synthesizing a remotable for each reference marker received:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```javascript
				  const makeBoardContext = () => {
				  const synthesizeRemotable = (_slot, iface) =>
				  Far(iface.replace(/^Alleged: /, ''), {});
				  
				  const { convertValToSlot, convertSlotToVal } = makeTranslationTable(
				  slot => Fail`unknown id: ${slot}`,
				  synthesizeRemotable,
				  );
				  const marshaller = makeMarshal(convertValToSlot, convertSlotToVal, smallCaps);
				  
				  /** Read-only board work-alike. */
				  const board = harden({
				  getId: convertValToSlot,
				  getValue: convertSlotToVal,
				  });
				  
				  return harden({
				  board,
				  marshaller,
				  /**
				   * Unmarshall capData, synthesizing a Remotable for each boardID slot.
				   *
				   * @type {(cd: import("@endo/marshal").CapData<string>) => unknown }
				   */
				  ingest: marshaller.fromCapData,
				  });
				  };
				  ```
				  
				  Now we can take results of vstorage queries for `Data('published.agoricNames.brand')` and `Data('published.agoricNames.instance')` unmarshal ("ingest") them:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```routeros
				  const clientContext = makeBoardContext();
				  
				  const brandQueryResult = {
				  body: `\#${JSON.stringify({
				  gold: '$1.Alleged: Gold Brand',
				  victory: '$0.Alleged: Victory Brand',
				  })}`,
				  slots: ['board0371', 'board32342'],
				  };
				  const brands = clientContext.ingest(brandQueryResult);
				  const { game1: instance } = clientContext.ingest(instanceQueryResult);
				  ```
				  
				  And now we have all the pieces of the `BridgeAction` above. The marshalled form is:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```css
				  t.deepEqual(clientContext.marshaller.toCapData(action), {
				  body: `\#${JSON.stringify({
				  method: 'executeOffer',
				  offer: {
				    id: 'battle7651',
				    invitationSpec: {
				      instance: '$0.Alleged: Instance',
				      invitationArgs: ['troll'],
				      publicInvitationMaker: 'makeBattleInvitation',
				      source: 'contract',
				    },
				    proposal: {
				      give: {
				        Gold: { brand: '$1.Alleged: Gold Brand', value: '+100' },
				      },
				    },
				  },
				  })}`,
				  slots: ['board123', 'board32342'],
				  });
				  ```
				  
				  We still don't quite have a single string for the `spend_action` field. We need to `stringify` the `CapData`:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```reasonml
				  const spendAction = JSON.stringify(
				  clientContext.marshaller.toCapData(action),
				  );
				  ```
				  
				  And now we have the `spend_action` in the expected form:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)js
				  
				  ```capnproto
				  t.regex(spendAction, /^{"body":"\#.*","slots":\["board123","board32342"\]}$/);
				  const goldStuff =
				  '\\"brand\\":\\"$1.Alleged: Gold Brand\\",\\"value\\":\\"+100\\"';
				  t.true(spendAction.includes(goldStuff));
				  ```
				  
				  The wallet factory can now `JSON.parse` this string into `CapData` and unmarshal it using a board marshaller to convert board ids back into brands, instances, etc.
				  
				  \#\# Smart Wallet VStorage Topics [​](\#smart-wallet-vstorage-topics)
				  
				  Each smart wallet has a node under `published.wallet`:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
				  
				  ```gcode
				  $ agd query vstorage children published.wallet
				  children:
				- agoric1h4d3mdvyqhy2vnw2shq4pm5duz5u8wa33jy6cl
				- agoric1qx2kqqdk80fdasldzkqu86tg4rhtaufs00na3y
				- agoric1rhul0rxa2z829a6xkrvuq8m8wjwekyduv7dzfj
				  ...
				  ```
				  
				  Smart wallet clients should start by getting the **current** state at `published.${ADDRESS}.current` and then subscribe to **updates** at `published.${ADDRESS}`. For example, we can use `agoric follow -lF` to get the latest `.current` record:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
				  
				  ```dts
				  $ agoric follow -lF :published.wallet.agoric1h4d3mdvyqhy2vnw2shq4pm5duz5u8wa33jy6cl.current
				  {
				  liveOffers: [],
				  offerToPublicSubscriberPaths: [
				  [
				    "openVault-1691526589332",
				    {
				      vault: "published.vaultFactory.managers.manager0.vaults.vault2",
				    },
				  ],
				  ],
				  offerToUsedInvitation: [
				  [
				    "openVault-1691526589332",
				    {
				      brand: slotToVal("board0074","Alleged: Zoe Invitation brand"),
				      value: [
				        {
				          description: "manager0: MakeVault",
				          handle: slotToVal(null,"Alleged: InvitationHandle"),
				          installation: slotToVal("board05815","Alleged: BundleIDInstallation"),
				          instance: slotToVal("board00360","Alleged: InstanceHandle"),
				        },
				      ],
				    },
				  ],
				  ],
				  purses: [
				  {
				    balance: {
				      brand: slotToVal("board0074"),
				      value: [],
				    },
				    brand: slotToVal("board0074"),
				  },
				  ],
				  }
				  ```
				  
				  Then we can use `agoric follow` without any options to get a stream of updates as they appear.
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
				  
				  ```css
				  agoric follow :published.wallet.agoric1h4d3mdvyqhy2vnw2shq4pm5duz5u8wa33jy6cl
				  ...
				  {
				  status: {
				  id: "closeVault-1691526597848",
				  invitationSpec: {
				    invitationMakerName: "CloseVault",
				    previousOffer: "openVault-1691526589332",
				    source: "continuing",
				  },
				  numWantsSatisfied: 1,
				  payouts: {
				    Collateral: {
				      brand: slotToVal("board05557","Alleged: ATOM brand"),
				      value: 13000000n,
				    },
				    Minted: {
				      brand: slotToVal("board0257","Alleged: IST brand"),
				      value: 215000n,
				    },
				  },
				  proposal: {
				    give: {
				      Minted: {
				        brand: slotToVal("board0257"),
				        value: 5750000n,
				      },
				    },
				    want: {},
				  },
				  result: "your vault is closed, thank you for your business",
				  },
				  updated: "offerStatus",
				  }
				  ```
				  
				  Note that status updates are emitted at several points in the handling of each offer:
				  
				  * when the `getOfferResult()` promise settles
				  * when the `numWantsSatisfied()` promise settles
				  * when the payouts have been deposited.
				  
				  And we may get `balance` updates at any time.
				  
				  The data published via vstorage are available within the JavaScript VM via the [getPublicTopics](https://github.com/Agoric/agoric-sdk/blob/mainnet1B/packages/smart-wallet/src/smartWallet.js\#L585) API.
				  
				  The [CurrentWalletRecord](https://github.com/Agoric/agoric-sdk/blob/mainnet1B/packages/smart-wallet/src/smartWallet.js\#L71-L76) type is:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)ts
				  
				  ```qml
				  {
				  purses: Array<{brand: Brand, balance: Amount}>,
				  offerToUsedInvitation: Array<[ offerId: string, usedInvitation: Amount ]>,
				  offerToPublicSubscriberPaths: Array<[ offerId: string, publicTopics: { [subscriberName: string]: string } ]>,
				  liveOffers: Array<[import('./offers.js').OfferId, import('./offers.js').OfferStatus]>,
				  }
				  ```
				  
				  And [UpdateRecord](https://github.com/Agoric/agoric-sdk/blob/mainnet1B/packages/smart-wallet/src/smartWallet.js\#L80-L83) is:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)ts
				  
				  ```groovy
				   { updated: 'offerStatus', status: import('./offers.js').OfferStatus }
				  | { updated: 'balance'; currentAmount: Amount }
				  | { updated: 'walletAction'; status: { error: string } }
				  ```
				  
				  Both of those types include [OfferStatus](https://github.com/Agoric/agoric-sdk/blob/mainnet1B/packages/smart-wallet/src/offers.js\#L21C14-L26C5) by reference:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)ts
				  
				  ```groovy
				  import('./offers.js').OfferSpec & {
				  error?: string,
				  numWantsSatisfied?: number
				  result?: unknown | typeof UNPUBLISHED_RESULT,
				  payouts?: AmountKeywordRecord,
				  }
				  ```
				  
				  \#\# VBank Assets and Cosmos Bank Balances [​](\#vbank-assets-and-cosmos-bank-balances)
				  
				  Note that balances of assets such as **IST** and **BLD** are already available via consensus layer queries to the Cosmos SDK [bank module](https://docs.cosmos.network/main/modules/bank).
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
				  
				  ```1c
				  $ agd query bank balances agoric1h4d3mdvyqhy2vnw2shq4pm5duz5u8wa33jy6cl -o json | jq .balances
				  [
				  {
				  "denom": "ibc/BA313C4A19DFBF943586C0387E6B11286F9E416B4DD27574E6909CABE0E342FA",
				  "amount": "100000000"
				  },
				  {
				  "denom": "ubld",
				  "amount": "10000000"
				  },
				  {
				  "denom": "uist",
				  "amount": "215000"
				  }
				  ]
				  ```
				  
				  They are not published redundantly in vstorage and nor does the smart wallet emit `balance` updates for them.
				  
				  To get the correspondence between certain cosmos denoms (chosen by governance) and their ERTP brands, issuers, and display info such as `decimalPlaces`, see `published.agoricNames.vbankAsset`:
				  
				  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHXaFhfIMHoH6ID5HuubDsR6O8ANJZtIYcOUz63x2L40/data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(128,128,128,1)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E)sh
				  
				  ```dts
				  agoric follow -lF :published.agoricNames.vbankAsset
				  [
				  [
				  "ibc/BA313C4A19DFBF943586C0387E6B11286F9E416B4DD27574E6909CABE0E342FA",
				  {
				    brand: slotToVal("board05557","Alleged: ATOM brand"),
				    denom: "ibc/BA313C4A19DFBF943586C0387E6B11286F9E416B4DD27574E6909CABE0E342FA",
				    displayInfo: {
				      assetKind: "nat",
				      decimalPlaces: 6,
				    },
				    issuer: slotToVal("board02656","Alleged: ATOM issuer"),
				    issuerName: "ATOM",
				    proposedName: "ATOM",
				  },
				  ],
				  [
				  "ubld",
				  {
				    brand: slotToVal("board0566","Alleged: BLD brand"),
				    denom: "ubld",
				    displayInfo: {
				      assetKind: "nat",
				      decimalPlaces: 6,
				    },
				    issuer: slotToVal("board0592","Alleged: BLD issuer"),
				    issuerName: "BLD",
				    proposedName: "Agoric staking token",
				  },
				  ],
				  [
				  "uist",
				  {
				    brand: slotToVal("board0257","Alleged: IST brand"),
				    denom: "uist",
				    displayInfo: {
				      assetKind: "nat",
				      decimalPlaces: 6,
				    },
				    issuer: slotToVal("board0223","Alleged: IST issuer"),
				    issuerName: "IST",
				    proposedName: "Agoric stable token",
				  },
				  ],
				  ...
				  ]
				  ```
				  - ### Highlights
				  collapsed:: true
				  - > `purse` \- to make an offer with an invitation that is already in the Invitation purse of the smart wallet and agrees with `spec` on `.instance` and `.description` properties. [⤴️](https://omnivore.app/me/https-docs-agoric-com-guides-getting-started-contract-rpc-html-18e58ac3d09#21856ba1-e523-4526-a184-3f053e6b0c91)
				  - [draft-toomim-httpbis-linked-json-00](https://omnivore.app/me/draft-toomim-httpbis-linked-json-00-18e49ca8a6e)
				  collapsed:: true
				  site:: [raw.githubusercontent.com](https://raw.githubusercontent.com/braid-org/braid-spec/master/draft-toomim-httpbis-linked-json-00.txt)
				  labels:: [[CRDTs]] [[Braid Specification]] [[JSON]] [[JSON-LD]]
				  date-saved:: [[03/16/2024]]
				  - ### Content
				  collapsed:: true
				  - Internet-Draft                                                 M. Toomim
				  Expires: Mar 8, 2020                                   Invisible College
				  Intended status: Proposed Standard                            B. Bellomy
				                                                     Invisible College
				                                                           Aug 2, 2020
				  
				                               Linked JSON
				                   draft-toomim-httpbis-linked-json-00
				  
				  Abstract
				  
				  Linked JSON is an extension to JSON that defines a new datatype: a
				  "Link".  A Link has a URI, and may have metadata.
				  
				  Links allow JSON objects to compose across websites, supporting
				  transclusion.  Optional metadata on links can be used to specify a
				  specific version or sub-slice of data to link to.
				  
				  
				  Status of this Memo
				  
				  This Internet-Draft is submitted in full conformance with the
				  provisions of BCP 78 and BCP 79.
				  
				  Internet-Drafts are working documents of the Internet Engineering
				  Task Force (IETF), its areas, and its working groups.  Note that
				  other groups may also distribute working documents as
				  Internet-Drafts.  The list of current Internet-Drafts is at
				  http://datatracker.ietf.org/drafts/current/.
				  
				  Internet-Drafts are draft documents valid for a maximum of six months
				  and may be updated, replaced, or obsoleted by other documents at any
				  time.  It is inappropriate to use Internet-Drafts as reference
				  material or to cite them other than as "work in progress."
				  
				  The list of current Internet-Drafts can be accessed at
				  https://www.ietf.org/1id-abstracts.html
				  
				  The list of Internet-Draft Shadow Directories can be accessed at
				  https://www.ietf.org/shadow.html
				  
				  
				  Table of Contents
				  
				  1. Introduction ....................................................4
				  2. Related Work ....................................................4
				  3. Resolution ..........................................,...........4
				  4. Media Type ..........................................,...........5
				  5. IANA Considerations .............................................5
				  6. Security Considerations .........................................5
				  7. Conventions .....................................................6
				  8. Copyright Notice ................................................6
				  9. References ......................................................6
				    9.1. Normative References .......................................6
				    9.2. Informative References .....................................6
				  
				  
				  1.  Introduction
				  
				  JSON documents often include URIs, or references to other documents,
				  as strings.  It is often useful to distinguish which strings actually
				  represent URIs, for example, to fetch and include the contents of
				  links, or to translate relative to absolute URIs.
				  
				  Linked JSON is an extension of JSON that adds a Link datatype, so that
				  URIs can be distinguished from ordinary strings.
				  
				  1.1.  Syntax
				  
				  A Link is encoded as a JSON object with a field named "link":
				  
				   { "link": "/david-macaulay" }
				  
				  Any object with a field named "link" is interpreted as a Link.  The
				  value of the link field MUST be a string containing a URI [RFC3986].
				  
				  Links MAY be embedded within arbitrary JSON:
				  
				   {
				     "book-name": "The way things work",
				     "image"      { "link": "/the-way-things-work.jpg" },
				     "author":    { "link": "/david-macaulay" }
				   }
				  
				  1.2.  Escaping
				  
				  To encode a field named "link" *without* creating a Link, prefix the
				  field with an underscore:
				  
				   { "_link": "this is not a link" }
				  
				  To encode "_link", prefix it with two underscores: "__link".  To
				  encode "__link", use "___link", and so on.
				  
				  1.3.  Metadata
				  
				  Links MAY specify metadata on other fields of the JSON object.
				  
				  For instance, a "version" field could be used to specify a specific
				  version to link to:
				  
				   {
				     "message": "Hey guys! I just published a new draft!",
				     "attachment: {
				       "link": "/books/the-way-things-work",
				       "version": "4.0.5"
				     }
				   }
				  
				  The metadata in [RFC8288] could also be expressed.
				  
				  Or a GraphQL [GRAPHQL] query:
				  
				   { "link": "/foo", "range": "(bar:9)[3,4]" }
				  
				  
				  2.  Related Work
				  
				  See [MNOT] for a survey of related work.  Linked JSON is most similar
				  to [JSONREF], but has the following differences:
				  - [Finite Model Theory and Game Comonads: Part 1 | The n-Category Café](https://omnivore.app/me/finite-model-theory-and-game-comonads-part-1-the-n-category-cafe-18d32a7db59)
				  collapsed:: true
				  site:: [golem.ph.utexas.edu](https://golem.ph.utexas.edu/category/2023/09/finite_model_theory_and_game_c.html)
				  date-saved:: [[01/22/2024]]
				  date-published:: [[09/07/2023]]
				  - ### Content
				  collapsed:: true
				  - \#\#\# Finite Model Theory and Game Comonads: Part 1
				  
				  \#\#\#\# Posted by Emily Riehl
				  
				  [![MathML-enabled post (click for more details).](https://proxy-prod.omnivore-image-cache.app/0x0,sSU_IkLson5g4653tePzSOpvq4vcmGk0czFf6ZfImxHk/https://golem.ph.utexas.edu/~distler/blog/images/MathML.png "MathML-enabled post (click for details).")](http://golem.ph.utexas.edu/~distler/blog/mathml.html)
				  
				  _guest post by [Elena Dimitriadis](https://www.math.univ-toulouse.fr/~edimitri/index%5Fen), [Richie Yeung](https://y-richie-y.github.io/), [Tyler Hanks](https://gataslab.org/), and [Zhixuan Yang](https://yangzhixuan.github.io/)_
				  
				  _Finite model theory_ ([Libkin 2004](https://doi.org/10.1007/978-3-662-07003-1)) studies finite models of logics. Its main motivation comes from computer science: a _finite relational structure_, i.e. a finite set AA with a finite set of relations on AA, is essentially a database in the sense of good old SQL tables, and a logic formula φ\\varphi with nn free variables is understood as a _query_ to the database that selects all nn\-tuples of AA that satisfy the formula φ\\varphi.
				  
				  [![MathML-enabled post (click for more details).](https://proxy-prod.omnivore-image-cache.app/0x0,sSU_IkLson5g4653tePzSOpvq4vcmGk0czFf6ZfImxHk/https://golem.ph.utexas.edu/~distler/blog/images/MathML.png "MathML-enabled post (click for details).")](http://golem.ph.utexas.edu/~distler/blog/mathml.html)
				  
				  Finite model theory is naturally related to _complexity theory_, as we may ask questions like what’s the time complexity to query a finite relational structure with a formula from some logic, and also the converse question—what kind of logic is needed to describe the algorithmic problems in a complexity class. For example, given a finite graph GG, we may ask if it is possible to write a first-order logic formula φ(u,v)\\varphi(u,v) using a relation symbol E(x,y)E(x,y) saying there is an edge from xx to yy, such that φ(u,v)\\varphi(u,v) is satisfied precisely by vertices u,v∈Gu, v \\in G that are connected by a path.
				  
				  From a logical point of view, what makes finite model theory interesting is that some prominent techniques in model theory fail for finite models. In particular, [compactness](https://en.wikipedia.org/wiki/Compactness%5Ftheorem) fails for finite model theory: a theory TT may not have a _finite_ model even if all its finite sub-theories S⊆TS \\subseteq T have finite models—consider e.g. T\={φ n∣n∈ℕ}T = \\left\\{ \\varphi\_n \\mid n \\in \\mathbb{N}\\right\\} where φ n\\varphi\_n asserts there are at least nn distinct elements.
				  
				  Fortunately there are model-theoretic techniques remaining valid in the finite context. One of them is _model-comparison games_, which characterise _logical equivalence_ of models, i.e. when two models satisfy exactly the same formulas of a logic.
				  
				  Such logical equivalences are very useful for showing _(in)expressivity_ of logics. For example, if we are able to show that two models 𝒜\\mathcal{A} and ℬ\\mathcal{B} of a theory satisfy exactly the same formulas from first-order logic, but there is a semantic property PP (in the meta-theory) which 𝒜\\mathcal{A} satisfies but ℬ\\mathcal{B} does not. Then we know the property PP necessarily cannot be expressed in first-order logic. This proof technique works for both finite and infinite models. In fact, using this technique one can show that connectivity of finite graphs cannot be expressed as a first-order logic formula with only the relation symbol E(x,y)E(x,y) for edges.
				  
				  Of course the logic equivalences for different logics need to be characterised by different games: first-order logic is characterised by _Ehrenfeucht-Fraïssé games_; the kk\-variable fragment of first-order logic is characterised by _pebble games_; modal logic is characterised by _bisimulation games_, and so on. 
				  
				  Despite being different, these games are structurally so similar that they almost begged to be unified. Their unification is eventually done by Abramsky and Shah ([2018](https://drops.dagstuhl.de/opus/volltexte/2018/9669), [2021](https://arxiv.org/abs/2010.06496)) in the categorical framework of _game comonads_. This two-part blog post aims to give a brief introduction to finite model theory and game comonads. This post will cover only a tiny part of this active research programme, but we will aim to exposit in a self-contained way the basics with some intuition that is hidden in the papers.
				  
				  \#\# Basics of First-Order Logic
				  
				  Let’s begin with a quick recap on classical _first-order logic_ (FOL). A _purely relational vocabulary_ σ\\sigma is a set of relation symbols {P 1,…,P i,…}\\left\\{P\_1,\\dots,P\_i,\\dots\\right\\} where each relation symbol P iP\_i has an associated arity n i∈ℕn\_i \\in \\mathbb{N}. In this post we do not consider vocabularies with function symbols or constants since they can be alternatively modelled as relations with axioms asserting their functionality, which slightly makes our life easier.
				  
				  The formulas φ\\varphi of FOL in a (purely relational) vocabulary σ\\sigma is inductively generated by the following grammar, where the meta-variable xx ranges over a countably infinite set of variables:
				  
				  φ ::\=x 1\=x 2|P i(x 1,…,x n i)|⊤|φ 1∧φ 2|⊥|φ 1∨φ 2|¬φ|∃x.φ|∀x.φ\\begin{array}{rl} \\varphi &::= x\_1 = x\_2 \\ |\\ P\_i(x\_1,\\dots,x\_{n\_i}) \\ |\\ \\top \\ |\\ \\varphi\_1 \\wedge \\varphi\_2 \\ |\\ \\bot \\ | \\ \\varphi\_1\\vee \\varphi\_2 \\ |\\ \\neg \\varphi \\ |\\ \\exists x. \\varphi \\ |\\ \\forall x. \\varphi \\end{array}
				  
				  A set TT of closed formulas (i.e. formulas with no free variables) is called a _theory_.
				  
				  We will only consider the classical semantics of FOL in the category of sets in this post. A _σ\\sigma\-relational structure_, or simply a _σ\\sigma\-structure_, 𝒜\=⟨A,⟨P i A⟩ P i∈σ⟩\\mathcal{A} = \\langle A, \\langle P\_i^A\\rangle\_{P\_i \\in \\sigma}\\rangle consists of a set AA and an n in\_i\-ary relation P i A⊆A n iP^A\_i \\subseteq A^{n\_i} on the set AA for each relation symbol P i∈σP\_i \\in \\sigma of arity n in\_i.
				  
				  A _homomorphism_ of σ\\sigma\-structures from 𝒜\\mathcal{A} to ℬ\\mathcal{B} is a function h:A→Bh\\colon A\\to B on the underlying sets such that for each relation symbol P iP\_i in σ\\sigma, we have P i A(a 1,…,a n i)P\_i^A(a\_1,\\dots,a\_{n\_i}) implies P i B(h(a 1),…,h(a n i))P\_i^B(h(a\_1),\\dots,h(a\_{n\_i})) for all a 1,…,a n i∈Aa\_1,\\dots,a\_{n\_i}\\in A. Each vocabulary σ\\sigma thus yields a category ℛ(σ)\\mathcal{R}(\\sigma) of σ\\sigma\-structures and homomorphisms.
				  
				  Let φ\\varphi be a FOL formula (in the vocabulary σ\\sigma) with nn free variables. The semantics of φ\\varphi in a σ\\sigma\-structure 𝒜\\mathcal{A} is an nn\-ary relation \[\[φ\]\]⊆A n\[\\!\[\\varphi\]\\!\] \\subseteq A^n on the set AA. We will write 𝒜⊧φ(a→)\\mathcal{A} \\models \\varphi (\\vec{a}) when some a→∈A n\\vec{a} \\in A^n is contained in \[\[φ\]\]\[\\!\[\\varphi\]\\!\] and also write 𝒜⊧φ\\mathcal{A} \\models \\varphi when φ\\varphi is a closed formula and ⟨⟩\\langle \\rangle is in \[\[φ\]\]⊆A 0\[\\!\[\\varphi\]\\!\] \\subseteq A^0. The relation \[\[φ\]\]\[\\!\[\\varphi\]\\!\] is inductively defined on φ\\varphi as follows (we implicity treats a→∈A n\\vec{a} \\in A^n as a function from the set of the nn free variables to the set AA):
				  
				  𝒜⊧(x i\=x j)(a→) ⇔ a→(x i)\=a→(x j) 𝒜⊧P i(x 1,…,x n i)(a→) ⇔ ⟨a→(x 1),…,a→(x n i)⟩∈P i A 𝒜⊧⊤ ⇔ true 𝒜⊧(φ 1∧φ 2)(a→) ⇔ 𝒜⊧φ 1(a→)and𝒜⊧φ 2(a→) 𝒜⊧⊥ ⇔ false 𝒜⊧(φ 1∨φ 2)(a→) ⇔ 𝒜⊧φ 1(a→)or𝒜⊧φ 2(a→) 𝒜⊧¬φ(a→) ⇔ 𝒜⊧φ(a→)does not hold 𝒜⊧(∃y.φ)(a→) ⇔ 𝒜⊧φ(a→\[y↦a′\])for some a′∈A 𝒜⊧(∀y.φ)(a→) ⇔ 𝒜⊧φ(a→\[y↦a′\])for all a′∈A\\begin{matrix} \\mathcal{A}\\models (x\_i = x\_j)(\\vec{a}) &\\iff& \\vec{a}(x\_i) = \\vec{a}(x\_j)\\\\ \\mathcal{A}\\models P\_i(x\_1,\\dots,x\_{n\_i})(\\vec{a}) &\\iff& \\langle \\vec{a}(x\_1),\\dots,\\vec{a}(x\_{n\_i})\\rangle \\in P\_i^A\\\\ \\mathcal{A}\\models \\top &\\iff& true\\\\ \\mathcal{A}\\models (\\varphi\_1 \\wedge \\varphi\_2)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi\_1(\\vec{a}) \\ \\text{and}\\ \\mathcal{A}\\models\\varphi\_2(\\vec{a})\\\\ \\mathcal{A}\\models \\bot &\\iff& false\\\\ \\mathcal{A}\\models (\\varphi\_1 \\vee \\varphi\_2)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi\_1(\\vec{a}) \\ \\text{or}\\ \\mathcal{A}\\models\\varphi\_2(\\vec{a})\\\\ \\mathcal{A}\\models \\neg\\varphi(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi(\\vec{a}) \\ \\text{does not hold}\\\\ \\mathcal{A}\\models (\\exists y. \\varphi)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi(\\vec{a}\[y \\mapsto a'\])\\ \\text{for some }a'\\in A\\\\ \\mathcal{A}\\models (\\forall y. \\varphi)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi(\\vec{a}\[y \\mapsto a'\])\\ \\text{for all } a'\\in A \\end{matrix}
				  
				  where a→\[y↦a′\]\\vec{a}\[y \\mapsto a'\] is the function mapping yy to a′a' and anything else xx to a→(x)\\vec{a}(x).
				  
				  \#\# Logical Equivalences and Ehrenfeucht-Fraïssé Games
				  
				  As motivated earlier, we are interested in characterising when two models 𝒜\\mathcal{A} and ℬ\\mathcal{B} of a relational vocabulary σ\\sigma satisfy exactly the same formulas, more precisely, when 𝒜⊧ϕ⇔ℬ⊧ϕ\\mathcal{A} \\models \\phi \\iff \\mathcal{B} \\models \\phi for all closed FOL formulas ϕ\\phi in the vocabulary σ\\sigma. When it is the case, 𝒜\\mathcal{A} and ℬ\\mathcal{B} are sometimes called _elementarily equivalent_.
				  
				  \#\#\# An Example of Logical Equivalence
				  
				  Let’s build up our intuition with a concrete example. Let σ\\sigma be the vocabulary with just one binary relation symbol ≤\\leq, and let 𝒜\\mathcal{A} be the model {0,1,…,1000}\\left\\{0, 1, \\dots, 1000\\right\\} with ≤\\leq being the usual linear order of natural numbers and similarly ℬ\\mathcal{B} be the model {0,1,…,1001}\\left\\{0, 1, \\dots, 1001\\right\\} with the same order. These two models are clearly different, and indeed they can be differentiated by a first-order logic formula in the vocabulary σ\\sigma—the formula 
				  
				  φ\=∃x 0∃x 1⋯∃x 1001.¬(x 0\=x 1)∧…¬(x i\=x j)…¬(x 1000\=x 1001)\\varphi = \\exists x\_0\\exists x\_1\\cdots\\exists x\_{1001}.\\ \\neg (x\_0 = x\_1) \\wedge \\dots \\neg(x\_i = x\_j) \\dots \\neg (x\_{1000} = x\_{1001})
				  
				  saying that there exist different elements is satisfied by ℬ\\mathcal{B} but not by 𝒜\\mathcal{A}.
				  
				  However, the formula φ\\varphi is a pretty big formula—it has 1002 quantifiers and 501501 clauses, so it is possible that small enough formulas cannot differentiate 𝒜\\mathcal{A} and ℬ\\mathcal{B} since they are pretty similar (they are both linear orders). Let’s consider some formulas with just a few quantifiers:
				  
				  * The vocabulary σ\={≤}\\sigma = \\left\\{ \\leq \\right\\} doesn’t have a constant, so there are no closed terms and thus no closed formulas other than ⊥\\bot and ⊤\\top. Thus 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all formulas with 0 quantifiers.
				  * Consider formulas of the form ∃x.φ(x)\\exists x.\\varphi(x) where φ\\varphi doesn’t have any quantifiers. We argue that 𝒜⊧∃x.φ(x)\\mathcal{A} \\models \\exists x.\\varphi(x) iff ℬ⊧∃x.φ(x)\\mathcal{B} \\models \\exists x.\\varphi(x) as follows: supposing 𝒜⊧∃x.φ(x)\\mathcal{A} \\models \\exists x.\\varphi(x) holds, this means that there is some a∈𝒜a \\in \\mathcal{A} such that 𝒜⊧φ(a)\\mathcal{A} \\models \\varphi(a), then we may choose any b∈ℬb \\in \\mathcal{B}, say b\=0∈ℬb = 0 \\in \\mathcal{B}, making ℬ⊧φ(b)\\mathcal{B} \\models \\varphi(b) because the formula φ\\varphi is inductively built from the variable xx, relations ≤\\leq, \=\=, and propositional connectives; both a≤aa \\leq a and b≤bb \\leq b are true, so we can inductively show that φ(a)\\varphi(a) and φ(b)\\varphi(b) agree for all φ\\varphi. Conversely, if ℬ⊧∃x.φ(x)\\mathcal{B} \\models \\exists x. \\varphi(x) is witnessed by some bb, we can always pick an arbitrary a∈𝒜a \\in \\mathcal{A} witnessing 𝒜⊧∃x.φ(x)\\mathcal{A} \\models \\exists x. \\varphi(x).  
				  Moreover, since the semantics of propositional connectives are defined compositionally, we can inductively show that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all closed formulas built out of exactly one ∃\\exists and \=\=, ¬\\neg, ∧\\wedge, and ∨\\vee. Since we are considering classical logic, universal quantification ∀x.φ(x)\\forall x. \\varphi(x) can be reduced to ¬∃x.φ(x)\\neg \\exists x. \\varphi(x), so 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on it so we conclude that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all FOL formulas with exactly one quanfier.
				  * This example gets interesting when we consider two nested quantifiers. Supposing ψ\=∃x.∃y.φ\\psi = \\exists x. \\exists y. \\varphi where φ\\varphi is quanfier-free, if 𝒜⊧ψ\\mathcal{A} \\models \\psi, there exist aa and a′∈𝒜a' \\in \\mathcal{A} such that 𝒜⊧φ(⟨a,a′⟩)\\mathcal{A} \\models \\varphi (\\langle a, a'\\rangle). Then we can also choose any two elements b,b′∈ℬb, b' \\in \\mathcal{B} such that, importantly, (i) b≤b′b \\leq b' iff a≤a′a \\leq a', and (ii) b\=b′b = b' iff a\=a′a = a'. This ensures ℬ⊧φ(⟨b,b′⟩)\\mathcal{B} \\models \\varphi(\\langle b, b'\\rangle) since the atomic formulas in φ\\varphi are built from ≤\\leq, \=\=, xx and yy, on which 𝒜\\mathcal{A} with the variable assignment ⟨x↦a,y↦a′⟩\\langle x\\mapsto a, y \\mapsto a'\\rangle and ℬ\\mathcal{B} with ⟨x↦b,y↦b′⟩\\langle x\\mapsto b, y \\mapsto b'\\rangle agree. Conversely, if ℬ⊧ψ\\mathcal{B} \\models \\psi witnessed by b,b′∈ℬb, b' \\in \\mathcal{B} we can also choose a matching pair a,a′∈𝒜a, a' \\in \\mathcal{A} making 𝒜⊧ψ\\mathcal{A} \\models \\psi.  
				  Now suppose ψ\=∃x.∀y.φ\\psi = \\exists x. \\forall y. \\varphi. Whenever 𝒜⊧ψ\\mathcal{A} \\models \\psi, there is some aa such that 𝒜⊧∀y.φ⟨x↦a⟩\\mathcal{A} \\models \\forall y.\\varphi \\langle x \\mapsto a\\rangle. In this case we can also choose an element b∈ℬb \\in \\mathcal{B} that mimics a∈𝒜a \\in \\mathcal{A}: if aa is the bottom element 00 in the structure 𝒜\\mathcal{A}, we let b\=0b = 0 as well; if aa is the top 10001000 in 𝒜\\mathcal{A}, we let bb be the top in ℬ\\mathcal{B}; otherwise we can choose an arbitrary 0<b<10010 \\lt b \\lt 1001. We then claim ℬ⊧∀y.φ⟨x↦b⟩\\mathcal{B} \\models \\forall y. \\varphi \\langle x \\mapsto b\\rangle as well, because if there is some b′b' making ℬ⊧φ⟨x↦b,y↦b′⟩\\mathcal{B} \\models \\varphi \\langle x\\mapsto b, y \\mapsto b'\\rangle not hold, we can also find an a′a' that is to aa in 𝒜\\mathcal{A} as b′b' is to bb in ℬ\\mathcal{B}: precisely, if b′\=bb' = b, we let a′\=aa' = a; if b′<bb' \\lt b, we let a′a' be any element in 𝒜\\mathcal{A} such that a′<aa' \\lt a, and similarly for the case b′\>bb' \\gt b (such a choice is always possible because earlier aa and bb are chosen to be at the same relative position). This choice of a′a' entails 𝒜⊧φ⟨x↦a,y↦a′⟩\\mathcal{A} \\models \\varphi\\langle x \\mapsto a, y \\mapsto a'\\rangle not true, leading to a contradiction. Therefore ℬ⊧ψ\\mathcal{B} \\models \\psi.  
				  By a symmetric argument, ℬ⊧∃x.∀y.φ\\mathcal{B} \\models \\exists x. \\forall y. \\varphi implies 𝒜⊧∃x.∀y.φ\\mathcal{A} \\models \\exists x. \\forall y. \\varphi as well. Moreover, by the compositionality of the semantics of propositional connectives, the two paragraphs above imply that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all FOL formulas with quantifiers are nested at most once.
				  
				  Hopefully working through the example above reveals the intuition behind logical equivalence for FOL: two σ\\sigma\-structures 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on a FOL formula φ\\varphi whenever a quantifier in φ\\varphi picks an element xx in AA or BB, there is always an element yy in the other structure that “simulates the behaviour” of xx in the model.
				  
				  \#\#\# The Ehrenfeucht-Fraïssé Game
				  
				  The intuition is precisely formulated as _Ehrenfeucht-Fraïssé (EF) games_. Given two σ\\sigma\-structures 𝒜\\mathcal{A} and ℬ\\mathcal{B} for any vocabulary σ\\sigma and a natural number kk, the kk\-round EF game for 𝒜\\mathcal{A} and ℬ\\mathcal{B} is a turn-based game between two players, called _the spoiler_ and _the duplicator_. Roughly speaking, the goal of the spoiler is to point out the difference between 𝒜\\mathcal{A} and ℬ\\mathcal{B} while the goal of the duplicator is to advocate that 𝒜\\mathcal{A} and ℬ\\mathcal{B} are the same. The rules are very simple: 
				  
				  1. **Movement**: at each round, the spoiler picks an element from one of the structures and the duplicator must respond with an element from the other structure. For example, if the spoiler picks an element from the structure 𝒜\\mathcal{A}, then the duplicator must pick an element b∈ℬb\\in\\mathcal{B}.
				  2. **Winning Condition**: After kk rounds, the game state consists of a→\=(a 1,…,a k)\\vec{a} = (a\_1,\\dots,a\_k) and b→\=(b 1,…,b i)\\vec{b} = (b\_1,\\dots,b\_i) representing the elements chosen from each structure at each round. The duplicator wins this play if the mapping a i↦b ia\_i \\mapsto b\_i defines a partial isomorphism between 𝒜\\mathcal{A} and ℬ\\mathcal{B}, i.e., if the substructures of 𝒜\\mathcal{A} and ℬ\\mathcal{B} generated by a→\\vec{a} and b→\\vec{b} are isomorphic. Otherwise, the spoiler has succeeded in showing the structures are different and wins.
				  
				  If the duplicator can guarantee a win after kk rounds no matter how the spoiler plays, we say the duplicator has a kk\-round winning strategy.
				  
				  The _quantifier rank_ qr(φ)\\mathop{qr}(\\varphi) of a FOL formula φ\\varphi is the depth of nesting of the quantifiers in φ\\varphi:
				  
				  qr(φ) \= 0 for atomicφ qr(o(φ 1,…,φ n)) \= max(qr(φ 1),…,qr(φ n)) for propositional connectiveso qr(Qx.φ) \= 1+qr(φ) for quantifiersQ\=∀,∃\\begin{array}{rcll} \\mathop{qr}(\\varphi) &=& 0 &\\text{for atomic}\\ \\varphi\\\\ \\mathop{qr}(o(\\varphi\_1, \\dots, \\varphi\_n)) &=& \\max(\\mathop{qr}(\\varphi\_1), \\dots, \\mathop{qr}(\\varphi\_n)) &\\text{for propositional connectives}\\ o\\\\ \\mathop{qr}(Q x. \\varphi) &=& 1 + \\mathop{qr}(\\varphi) &\\text{for quantifiers}\\ Q = \\forall, \\exists \\end{array}
				  
				  **Theorem** (Ehrenfeucht-Fraïssé). _If the duplicator has a winning strategy for the kk\-round EF game for 𝒜\\mathcal{A} and ℬ\\mathcal{B}, 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all closed FOL formulas of quantifier-rank kk. When the vocabulary σ\\sigma is finite, the converse is also true._
				  
				  _Proof sketch_: Assuming a winning strategy for the duplicator, and let ψ\\psi be any formula of quantifier rank kk. Without loss of generality, we can assume ψ\=Q 1x 1.⋯Q kx k.φ\\psi = Q\_{1} x\_{1}.\\cdots Q\_k x\_k.\\varphi where Q i∈{∀,∃}Q\_i \\in \\left\\{\\forall, \\exists\\right\\} are quantifiers and φ\\varphi is quantifier-free. We need to show 𝒜⊧ψ⇔ℬ⊧ψ\\mathcal{A} \\models \\psi \\iff \\mathcal{B} \\models \\psi. As we demonstrated in the example above, we consider how −⊧ψ\- \\models \\psi is defined inductively: if a quantifier Q i\=∃Q\_i = \\exists and one of 𝒜\\mathcal{A} and ℬ\\mathcal{B} satisfies the formula, we use the winning strategy for the duplicator to pick a matching element in the other structure; if a quantifier Q i\=∀Q\_i = \\forall and one of 𝒜\\mathcal{A} and ℬ\\mathcal{B} satisfies the formula, every counter-witness in the other structure leads to a counter-witness in this structure by the winning strategy of the duplicator, thus a contradiction.
				  
				  The converse direction is also interesting and is not demonstrated in the example in the last section. We only give a rough sketch here and refer interested readers to [Libkin (2004, section 3)](https://doi.org/10.1007/978-3-662-07003-1) for details. 
				  
				  Assuming that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all FOL formulas of rank kk, the duplicator’s strategy is that whenever the spoiler picks an element a i∈𝒜a\_i \\in \\mathcal{A} (or symmetrically b i∈ℬb\_i \\in \\mathcal{B}), the duplicator first constructs a FOL formula ϕ i\\phi\_{i} that “maximally describes the current board on 𝒜\\mathcal{A}”. Roughly speaking, ϕ i\\phi\_{i} is the conjunction of all formulas in the following set
				  
				  {ψ∣qr(ψ)\=i−1and𝒜⊧ψ⟨a 1,…,a i⟩}\\left\\{\\psi \\mid \\mathop{qr}(\\psi) = i - 1 \\ \\text{and}\\ \\mathcal{A} \\models \\psi\\, \\langle a\_1, \\dots, a\_i\\rangle\\right\\}
				  
				  A priori, this set may have infinitely many formulas, but since there are only finitely many atomic formulas when σ\\sigma is finite, ϕ i\\phi\_{i} can be reduced to a finite formula using the rules of propositional connectives. With the formula ϕ i\\phi\_{i} in hand, the duplicator can use the fact that 
				  
				  𝒜⊧∃x i.ϕ i⟨x 1↦a 1,…,x i−1⟩↦a i−1,\\mathcal{A} \\models \\exists x\_i. \\phi\_{i}\\langle x\_1 \\mapsto a\_1, \\dots, x\_{i-1}\\rangle \\mapsto a\_{i-1},
				  
				  witnessed by x i↦a ix\_i \\mapsto a\_i. Now using the assumption that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on FOL up to rank kk and the fact that ∃x i.ϕ i\\exists x\_i.\\phi\_i is of rank ii, ℬ\\mathcal{B} has an element witnessing the truth of this formula as well, which is going to be the duplicator’s response. □\\square
				  
				  EF games are very useful for showing inexpressivity results of FOL. Suppose we are interested in a property PP (in the meta-theory) on a class MM of σ\\sigma\-structures. If for every natural number kk, we can find two models 𝒜 k,ℬ k∈M\\mathcal{A}\_k, \\mathcal{B}\_k \\in M such that 
				  
				  1. the duplicator has a winning strategy for the kk\-round EF game on 𝒜 k\\mathcal{A}\_k and ℬ k\\mathcal{B}\_k, but
				  2. only one of 𝒜 k\\mathcal{A}\_k and ℬ k\\mathcal{B}\_k satisfy the property PP,
				  
				  Then by the EF theorem, the property PP cannot be expressed by a FOL formula, whatever the quantifier it has. 
				  
				  For example, using this technique, we can show that the evenness of finite linear orders is not expressible—there isn’t a FOL formula φ\\varphi in the vocabulary {≤}\\left\\{\\leq\\right\\} such that for every finite linear order 𝒜\=⟨A,≤ A⟩\\mathcal{A} = \\langle A, \\leq^A\\rangle, 𝒜⊧φ\\mathcal{A} \\models \\varphi exactly when aa has an even number of elements. (Hint: we pick 𝒜 k\\mathcal{A}\_k and ℬ k\\mathcal{B}\_k to be the linear order of 2 k2^k and 2 k+12^k + 1 elements respectively and play the EF games.)
				  
				  Posted at September 8, 2023 2:19 PM UTC 
				  
				  TrackBack URL for this Entry: https://golem.ph.utexas.edu/cgi-bin/MT-3.0/dxy-tb.fcgi/3498
				  
				  Read the post [Finite Model Theory and Game Comonads: Part 2](https://golem.ph.utexas.edu/category/2023/09/finite%5Fmodel%5Ftheory%5Fand%5Fgame%5Fc%5F1.html)  
				  **Weblog:** The n-Category Café  
				  **Excerpt:** In the Part 1 of this post, we saw how logical equivalences of first-order logic (FOL) can be characterised by a combinatory game. But there are still a few unsatisfactory aspects, which we'll clear up now.  
				  **Tracked:** September 11, 2023 4:09 PM
				  - [History of Actors (eighty-twenty news)](https://omnivore.app/me/https-eighty-twenty-org-2016-10-18-actors-hopl-ref-blog-danfinla-18d02554ccc)
				  collapsed:: true
				  site:: [eighty-twenty.org](https://eighty-twenty.org/2016/10/18/actors-hopl?ref=blog.danfinlay.com)
				  author:: Tony Garnock-Jones
				  labels:: [[actors]] [[distributed systems]]
				  date-saved:: [[01/13/2024]]
				  date-published:: [[10/17/2016]]
				  - ### Content
				  collapsed:: true
				  - Yesterday, I presented the history of actors to Christos Dimoulas’[History of Programming Languages](http://www.seas.harvard.edu/courses/cs252/2016fa/)class.
				  
				  Here are the written-out talk notes I prepared beforehand. There is an annotated bibliography at the end.
				  
				  ---
				  
				  \#\# Introduction
				  
				  Today I’m going to talk about the actor model. I’ll first put the model in context, and then show three different styles of actor language, including two that aim to be realistic programming systems.
				  
				  I’m going to draw on a few papers:
				  
				  * 2016 survey by Joeri De Koster, Tom Van Cutsem, and Wolfgang De Meuter, for taxonomy and common terminology (SPLASH AGERE 2016)
				  * 1990 overview by Gul Agha, who has contributed hugely to the study of actors (Comm. ACM 1990)
				  * 1997 operational semantics for actors by Gul Agha, Ian Mason, Scott Smith, and Carolyn Talcott.
				  * brief mention of some of the content of the original 1973 paper by Carl Hewitt, Peter Bishop, and Richard Steiger. (IJCAI 1973)
				  * Joe Armstrong’s 2003 dissertation on Erlang.
				  * 2005 paper on the language E by Mark Miller, Dean Tribble, and Jonathan Shapiro. (TGC 2005)
				  
				  I’ve put an annotated bibliography at the end of this file.
				  
				  \#\# Actors in context: Approaches to concurrent programming
				  
				  One way of classifying approaches is along a spectrum of private vs. shared state.
				  
				  * Shared memory, threads, and locking: very limited private state, almost all shared
				  * Tuple spaces and my own research, Syndicate: some shared, some private and isolated
				  * Actors: almost all private and isolated, just enough shared to do routing
				  
				  Pure functional “data parallelism” doesn’t fit on this chart - it lacks shared mutable state entirely.
				  
				  \#\# The actor model
				  
				  The actor model, then is
				  
				  * asynchronous message passing between entities (actors)  
				   * with guaranteed delivery  
				   * addressing of messages by actor identity
				  * a thing called the “ISOLATED TURN PRINCIPLE”  
				   * no shared mutable state; strong encapsulation; no global mutable state  
				   * no interleaving; process one message at a time; serializes state access  
				   * liveness; no blocking
				  
				  In the model, each actor performs “turns” one after the other: a turn is taking a waiting message, interpreting it, and deciding on both a state update for the actor and a collection of actions to take in response, perhaps sending messages or creating new actors.
				  
				  A turn is a sequential, atomic block of computation that happens in response to an incoming message.
				  
				  What the actor model buys you:
				  
				  * modular reasoning about state in the overall system, and modular reasoning about local state change within an actor, because state is private, and access is serialized; only have to consider interleavings of messages
				  * no “deadlocks” or data races (though you can get “datalock” and other global non-progress in some circumstances, from logical inconsistency and otherwise)
				  * flexible, powerful capability-based security
				  * failure isolation and fault tolerance
				  
				  It’s worth remarking that actor-like concepts have sprung up several times independently. Hewitt and many others invented and developed actors in the 1970s, but there are two occasions where actors seem to have been independently reinvented, as far as I know.
				  
				  One is work on a capability-based operating system, KeyKOS, in the 1980s which involved a design very much like Hewitt’s actors, feeding into research which led ultimately to the language E.
				  
				  The other is work on highly fault-tolerant designs for telephone switches, also in the 1980s, which culminated in the language Erlang.
				  
				  Both languages are clearly actor languages, and in both cases, apparently the people involved were unaware of Hewitt’s actor model at the time.
				  
				  \#\# Terminology
				  
				  There are two kinds of things in the actor model: _messages_, which are data sent across some medium of communication, and _actors_, which are stateful entities that can only affect each other by sending messages back and forth.
				  
				  Messages are completely immutable data, passed by copy, which may contain references to other actors.
				  
				  Each actor has
				  
				  * private state; analogous to instance variables
				  * an interface; which messages it can respond to
				  
				  Together, the private state and the interface make up the actor’s_behaviour_, a key term in the actor literature.
				  
				  In addition, each actor has
				  
				  * a mailbox; inbox, message queue
				  * an address; denotes the mailbox
				  
				  Within this framework, there has been quite a bit of variety in how the model appears as a concrete programming language.
				  
				  De Koster et al. classify actor languages into
				  
				  * The Classic Actor Model (create, send, become)
				  * Active Objects (OO with a thread per object; copying of passive data between objects)
				  * Processes (raw Erlang; receive, spawn, send)
				  * Communicating Event-Loops (no passive data; E; near/far refs; eventual refs; batching)
				  
				  We see instances of the actor model all around us. The internet’s IP network is one example: hosts are the actors, and datagrams the messages. The web can be seen as another: a URL denotes an actor, and an HTTP request a message. Seen in a certain light, even Unix processes are actor-like (when they’re single threaded, if they only use fds and not shm). It can be used as a structuring principle for a system architecture even in languages like C or C++ that have no hope of enforcing any of the invariants of the model.
				  
				  \#\# Rest of the talk
				  
				  For the rest of the talk, I’m going to cover the classic actor model using Agha’s presentations as a guide; then I’ll compare it to E, a communicating event-loop actor language, and to Erlang, a process actor language.
				  
				  \#\# Classic Actor Model
				  
				  The original 1973 actor paper by Hewitt, Bishop and Steiger in the International Joint Conference on Artificial Intelligence, is incredibly far out!
				  
				  It’s a position paper that lays out a broad and colourful research vision. It’s packed with amazing ideas.
				  
				  The heart of it is that Actors are proposed as a universal programming language formalism ideally suited to building artificial intelligence.
				  
				  The goal really was A.I., and actors and programming languages were a means to that end.
				  
				  It makes these claims that actors bring great benefits in a huge range of areas:
				  
				  * foundations of semantics
				  * logic
				  * knowledge-based programming
				  * intentions (software contracts)
				  * study of expressiveness of programming languages
				  * teaching of computation
				  * extensible, modular programming
				  * privacy and protection
				  * synchronization constructs
				  * resource management
				  * structured programming
				  * computer architecture
				  
				  (It’s amazingly “full-stack”: computer architecture!?)
				  
				  In each of these areas, you can see what they were going for. In some, actors have definitely been useful; in others, the results have been much more modest.
				  
				  In the mid-to-late 70s, Hewitt and his students Irene Greif, Henry Baker, and Will Clinger developed a lot of the basic theory of the actor model, inspired originally by SIMULA and Smalltalk-71\. Irene Greif developed the first operational semantics for it as her dissertation work and Will Clinger developed a denotational semantics for actors.
				  
				  In the late 70s through the 80s and beyond, Gul Agha made huge contributions to the actor theory. His dissertation was published as a book on actors in 1986 and has been very influential. He separated the actor model from its A.I. roots and started treating it as a more general programming model. In particular, in his 1990 Comm. ACM paper, he describes it as a foundation for CONCURRENT OBJECT-ORIENTED PROGRAMMING.
				  
				  Agha’s formulation is based around the three core operations of the classic actor model:
				  
				  * `create`: constructs a new actor from a template and some parameters
				  * `send`: delivers a message, asynchronously, to a named actor
				  * `become`: within an actor, replaces its behaviour (its state & interface)
				  
				  The classic actor model is a UNIFORM ACTOR MODEL, that is, everything is an actor. Compare to uniform object models, where everything is an object. By the mid-90s, that very strict uniformity had fallen out of favour and people often worked with _two-layer_ languages, where you might have a functional core language, or an object-oriented core language, or an imperative core language with the actor model part being added in to the base language.
				  
				  I’m going to give a simplified, somewhat informal semantics based on his 1997 work with Mason, Smith and Talcott. I’m going to drop a lot of details that aren’t relevant here so this really will be simplified.
				  
				  ```coq
				  e  :=  λx.e  |  e e  |  x  | (e,e) | l | ... atoms, if, bool, primitive operations ...
				    |  create e
				    |  send e e
				    |  become e
				  
				  l  labels, PIDs; we'll use them like symbols here
				  
				  ```
				  
				  and we imagine convenience syntax
				  
				  ```ocaml
				  e1;e2              to stand for   (λdummy.e2) e1
				  let x = e1 in e2   to stand for   (λx.e2) e1
				  
				  match e with p -> e, p -> e, ...
				    to stand for matching implemented with if, predicates, etc.
				  
				  ```
				  
				  We forbid programs from containing literal process IDs.
				  
				  ```coq
				  v  :=  λx.e  |  (v,v)  |  l
				  
				  R  :=  []  |  R e  |  v R  |  (R,e)  |  (v,R)
				    |  create R
				    |  send R e  |  send v R
				    |  become R
				  
				  ```
				  
				  Configurations are a pair of a set of actors and a multiset of messages:
				  
				  ```makefile
				  m  :=  l <-- v
				  
				  A  :=  (v)l  |  [e]l
				  
				  C  :=  < A ... | m ... >
				  
				  ```
				  
				  The normal lambda-calculus-like reductions apply, like beta:
				  
				  ```jboss-cli
				    < ... [R[(λx.e) v]]l ... | ... >
				  --> < ... [R[e{v/x}  ]]l ... | ... >
				  
				  ```
				  
				  Plus some new interesting ones that are actor specific:
				  
				  ```jboss-cli
				    < ... (v)l ...    | ... l <-- v' ... >
				  --> < ... [v v']l ... | ...          ... >
				  
				    < ... [R[create v]]l       ... | ... >
				  --> < ... [R[l'      ]]l (v)l' ... | ... >   where l' fresh
				  
				    < ... [R[send l' v]]l ... | ... >
				  --> < ... [R[l'       ]]l ... | ... l' <-- v >
				  
				    < ... [R[become v]]l       ... | ... >
				  --> < ... [R[nil     ]]l' (v)l ... | ... >   where l' fresh
				  
				  ```
				  
				  Whole programs `e` are started with
				  
				  where `a` is an arbitrary label.
				  
				  Here’s an example - a mutable cell.
				  
				  ```xl
				  Cell = λcontents . λmessage . match message with
				                                (get, k) --> become (Cell contents);
				                                             send k contents
				                                (put, v) --> become (Cell v)
				  
				  ```
				  
				  Notice that when it gets a `get` message, it first performs a `become`in order to quickly return to `ready` state to handle more messages. The remainder of the code then runs alongside the ready actor. Actions after a `become` can’t directly affect the state of the actor anymore, so even though we have what looks like multiple concurrent executions of the actor, there’s no sharing, and so access to the state is still serialized, as needed for the isolated turn principle.
				  
				  ```livecodeserver
				  < [let c = create (Cell 0) in
				   send c (get, create (λv . send c (put, v + 1)))]a | >
				  
				  < [let c = l1 in
				   send c (get, create (λv . send c (put, v + 1)))]a
				  (Cell 0)l1 | >
				  
				  < [send l1 (get, create (λv . send l1 (put, v + 1)))]a
				  (Cell 0)l1 | >
				  
				  < [send l1 (get, l2)]a
				  (Cell 0)l1
				  (λv . send l1 (put, v + 1))l2 | >
				  
				  < [l1]a
				  (Cell 0)l1
				  (λv . send l1 (put, v + 1))l2 | l1 <-- (get, l2) >
				  
				  < [l1]a
				  [Cell 0 (get, l2)]l1
				  (λv . send l1 (put, v + 1))l2 | >
				  
				  < [l1]a
				  [become (Cell 0); send l2 0]l1
				  (λv . send l1 (put, v + 1))l2 | >
				  
				  < [l1]a
				  [send l2 0]l3
				  (Cell 0)l1
				  (λv . send l1 (put, v + 1))l2 | >
				  
				  < [l1]a
				  [l2]l3
				  (Cell 0)l1
				  (λv . send l1 (put, v + 1))l2 | l2 <-- 0 >
				  
				  < [l1]a
				  [l2]l3
				  (Cell 0)l1
				  [send l1 (put, 0 + 1)]l2 | >
				  
				  < [l1]a
				  [l2]l3
				  (Cell 0)l1
				  [l1]l2 | l1 <-- (put, 1) >
				  
				  < [l1]a
				  [l2]l3
				  [Cell 0 (put, 1)]l1
				  [l1]l2 | >
				  
				  < [l1]a
				  [l2]l3
				  [become (Cell 1)]l1
				  [l1]l2 | >
				  
				  < [l1]a
				  [l2]l3
				  [nil]l4
				  (Cell 1)l1
				  [l1]l2 | >
				  
				  ```
				  
				  (You could consider adding a garbage collection rule like
				  
				  ```jboss-cli
				    < ... [v]l ... | ... >
				  --> < ...      ... | ... >
				  
				  ```
				  
				  to discard the final value at the end of an activation.)
				  
				  Because at this level all the continuations are explicit, you can encode patterns other than sequential control flow, such as fork-join.
				  
				  For example, to start two long-running computations in parallel, and collect the answers in either order, multiplying them and sending the result to some actor k’, you could write
				  
				  ```smali
				  let k = create (λv1 . become (λv2 . send k' (v1 * v2))) in
				  send task1 (req1, k);
				  send task2 (req2, k)
				  
				  ```
				  
				  Practically speaking, both Hewitt’s original actor language, PLASMA, and the language Agha uses for his examples in the 1990 paper, Rosette, have special syntax for ordinary RPC so the programmer needn’t manipulate continuations themselves.
				  
				  So that covers the classic actor model. Create, send and become. Explicit use of actor addresses, and lots and lots of temporary actors for inter-actor RPC continuations.
				  
				  Before I move on to Erlang: remember right at the beginning I told you the actor model was
				  
				  * asynchronous message passing, and
				  * the isolated turn principle
				  
				  ?
				  
				  The isolated turn principle requires _liveness_ \- you’re not allowed to block indefinitely while responding to a message!
				  
				  But here, we can:
				  
				  ```swift
				  let d = create (λc . send c c) in
				  send d d
				  
				  ```
				  
				  Compare this with
				  
				  ```armasm
				  letrec beh = (λc . become beh; send c c) in
				  let d = create beh in
				  send d d
				  
				  ```
				  
				  These are both degenerate cases, but in different ways: the first becomes inert very quickly and the actor `d` is never returned to an idle/ready state, while the second spins uselessly forever.
				  
				  Other errors we could make would be to fail to send an expected reply to a continuation.
				  
				  One thing the semantics here rules out is interaction with other actors before doing a `become`; there’s no way to have a waiting continuation perform the `become`, because by that time you’re in a different actor. In this way, it sticks to the very letter of the Isolated Turn Principle, by forbidding “blocking”, but there are other kinds of things that can go wrong to destroy progress.
				  
				  Even if we require our behaviour functions to be total, we can still get _global_ nontermination.
				  
				  So saying that we “don’t have deadlock” with the actor model is very much oversimplified, even at the level of simple formal models, let alone when it comes to realistic programming systems.
				  
				  In practice, programmers often need “blocking” calls out to other actors before making a state update; with the classic actor model, this can be done, but it is done with a complicated encoding.
				  
				  \#\# Erlang: Actors for fault-tolerant systems
				  
				  Erlang is an example of what De Koster et al. call a Process-based actor language.
				  
				  It has its origins in telephony, where it has been used to build telephone switches with fabled “nine nines” of uptime. The research process that led to Erlang concentrated on high-availability, fault-tolerant software. The reasoning that led to such an actor-like system was, in a nutshell:
				  
				  * Programs have bugs. If part of the program crashes, it shouldn’t corrupt other parts. Hence strong isolation and shared-nothing message-passing.
				  * If part of the program crashes, another part has to take up the slack, one way or another. So we need crash signalling so we can detect failures and take some action.
				  * We can’t have all our eggs in one basket! If one machine fails at the hardware level, we need to have a nearby neighbour that can smoothly continue running. For that redundancy, we need distribution, which makes the shared-nothing message passing extra attractive as a unifying mechanism.
				  
				  Erlang is a two-level system, with a functional language core equipped with imperative actions for asynchronous message send and spawning new processes, like Agha’s system.
				  
				  The difference is that it lacks `become`, and instead has a construct called `receive`.
				  
				  Erlang actors, called processes, are ultra lightweight threads that run sequentially from beginning to end as little functional programs. As it runs, no explicit temporary continuation actors are created: any time it uses receive, it simply blocks until a matching message appears.
				  
				  After some initialization steps, these programs typically enter a message loop. For example, here’s a mutable cell:
				  
				  ```erlang
				  mainloop(Contents) ->
				  receive
				    {get, K} -> K ! Contents,
				                mainloop(Contents);
				    {put, V} -> mainloop(V)
				  end.
				  
				  ```
				  
				  A client program might be
				  
				  ```crystal
				  Cell = spawn(fun mainloop(0) end),
				  Cell ! {get, self()},
				  receive
				  V -> ...
				  end.
				  
				  ```
				  
				  Instead of using `become`, the program performs a tail call which returns to the `receive` statement as the last thing it does.
				  
				  Because `receive` is a statement like any other, Erlang processes can use it to enter substates:
				  
				  ```routeros
				  mainloop(Service) ->
				  receive
				    {req, K} -> Service ! {subreq, self()},
				                receive
				                  {subreply, Answer} -> K ! {reply, Answer},
				                                        mainloop(Service)
				                end
				  end.
				  
				  ```
				  
				  While the process is blocked on the inner `receive`, it only processes messages matching the patterns in that inner receive, and it isn’t until it does the tail call back to `mainloop` that it starts waiting for `req` messages again. In the meantime, non-matched messages queue up waiting to be `receive`d later.
				  
				  This is called “selective receive” and it is difficult to reason about. It doesn’t _quite_ violate the letter of the Isolated Turn Principle, but it comes close. (`become` can be used in a similar way.)
				  
				  The goal underlying “selective receive”, namely changing the set of messages one responds to and temporarily ignoring others, is important for the way people think about actor systems, and a lot of research has been done on different ways of selectively enabling message handlers. See Agha’s 1990 paper for pointers toward this research.
				  
				  One unique feature that Erlang brings to the table is crash signalling. The jargon is “links” and “monitors”. Processes can ask the system to make sure to send them a message if a monitored process exits. That way, they can perform RPC by
				  
				  * monitoring the server process
				  * sending the request
				  * if a reply message arrives, unmonitor the server process and continue
				  * if an exit message arrives, the service has crashed; take some corrective action.
				  
				  This general idea of being able to monitor the status of some other process was one of the seeds of my own research and my language Syndicate.
				  
				  So while the classic actor model had create/send/become as primitives, Erlang has spawn/send/receive, and actors are processes rather than event-handler functions. The programmer still manipulates references to actors/processes directly, but there are far fewer explicit temporary actors created compared to the “classic model”; the ordinary continuations of Erlang’s functional fragment take on those duties.
				  
				  \#\# E: actors for secure cooperation
				  
				  The last language I want to show you, E, is an example of what De Koster et al. call a Communicating Event-Loop language.
				  
				  E looks and feels much more like a traditional object-oriented language to the programmer than either of the variations we’ve seen so far.
				  
				  ```applescript
				  def makeCell (var contents) {
				    def getter {
				        to get() { return contents }
				    }
				    def setter {
				        to set(newContents) {
				            contents := newContents
				        }
				    }
				    return [getter, setter]
				  }
				  
				  ```
				  
				  The mutable cell in E is interestingly different: It yields _two_values. One specifically for setting the cell, and one for getting it. E focuses on security and securability, and encourages the programmer to hand out objects that have the minimum possible authority needed to get the job done. Here, we can safely pass around references to`getter` without risking unauthorized update of the cell.
				  
				  E uses the term “vat” to describe the concept closest to the traditional actor. A vat has not only a mailbox, like an actor, but also a call stack, and a heap of local objects. As we’ll see, E programmers don’t manipulate references to vats directly. Instead, they pass around references to objects in a vat’s heap.
				  
				  This is interesting because in the actor model messages are addressed to a particular actor, but here we’re seemingly handing out references to something finer grained: to individual objects _within_ an actor or vat.
				  
				  This is the first sign that E, while it uses the basic create/send/become-style model at its core, doesn’t expose that model directly to the programmer. It layers a special E-specific protocol on top, and only lets the programmer use the features of that upper layer protocol.
				  
				  There are two kinds of references available: near refs, which definitely point to objects in the local vat, and far refs, which may point to objects in a different vat, perhaps on another machine.
				  
				  To go with these two kinds of refs, there are two kinds of method calls: _immediate_ calls, and _eventual_ calls.
				  
				  receiver.method(arg, …)
				  
				  receiver <- method(arg, …)
				  
				  It is an error to use an immediate call on a ref to an object in a different vat, because it blocks during the current turn while the answer is computed. It’s OK to use eventual calls on any ref at all, though: it causes a message to be queued in the target vat (which might be our own), and a _promise_ is immediately returned to the caller.
				  
				  The promise starts off unresolved. Later, when the target vat has computed and sent a reply, the promise will become resolved. A nifty trick is that even an unresolved promise is useful: you can _pipeline_them. For example,
				  
				  ```ruby
				  def r1 := x <- a()
				  def r2 := y <- b()
				  def r3 := r1 <- c(r2)
				  
				  ```
				  
				  would block and perform multiple network round trips in a traditional simple RPC system; in E, there is a protocol layered on top of raw message sending that discusses promise creation, resolution and use. This protocol allows the system to send messages like
				  
				  ```smalltalk
				  "Send a() to x, and name the resulting promise r1"
				  "Send b() to y, and name the resulting promise r2"
				  "When r1 is known, send c(r2) to it"
				  
				  ```
				  
				  Crucial here is that the protocol, the language of discourse between actors, allows the expression of concepts including the notion of a send to happen at a future time, to a currently-unknown recipient.
				  
				  The protocol and the E vat runtimes work together to make sure that messages get to where they need to go efficiently, even in the face of multiple layers of forwarding.
				  
				  Each turn of an E vat involves taking one message off the message queue, and dispatching it to the local object it denotes. Immediate calls push stack frames on the stack as usual for object-oriented programming languages; eventual calls push messages onto the message queue. Execution continues until the stack is empty again, at which point the turn concludes and the next turn starts.
				  
				  One interesting problem with using promises to represent cross-vat interactions is how to do control flow. Say we had
				  
				  ```armasm
				  def r3 := r1 <- c(r2)  // from earlier
				  if (r3) {
				  myvar := a();
				  } else {
				  myvar := b();
				  }
				  
				  ```
				  
				  By the time we need to make a decision which way to go, the promise r3 may not yet be resolved. E handles this by making it an error to depend immediately on the value of a promise for control flow; instead, the programmer uses a `when` expression to install a handler for the event of the resolution of the promise:
				  
				  ```livescript
				  when (r3) -> {
				  if (r3) {
				    myvar := a();
				  } else {
				    myvar := b();
				  }
				  }
				  
				  ```
				  
				  The test of r3 and the call to a() or b() and the update to myvar are_delayed_ until r3 has resolved to some value.
				  
				  This looks like it violates the Isolated Turn Principle! It seems like we now have some kind of interleaving. But what’s going on under the covers is that promise resolution is done with an incoming message, queued as usual, and when that message’s turn comes round, the when clause will run.
				  
				  Just like with classic actors and with Erlang, managing these multiple-stage, stateful interactions in response to some incoming message is generally difficult to do. It’s a question of finding a balance between the Isolated Turn Principle, and its commitment to availability, and encoding the necessary state transitions without risking inconsistency or deadlock.
				  
				  Turning to failure signalling, E’s vats are not just the units of concurrency but also the units of partial failure. An uncaught exception within a vat destroys the whole vat and invalidates all remote references to objects within it.
				  
				  While in Erlang, processes are directly notified of failures of other processes as a whole, E can be more fine-grained. In E, the programmer has a convenient value that represents the outcome of a transaction: the promise. When a vat fails, or a network problem arises, any promises depending on the vat or the network link are put into a special state: they become _broken promises_. Interacting with a broken promise causes the contained exception to be signalled; in this way, broken promises propagate failure along causal chains.
				  
				  If we look under the covers, E seems to have a “classic style” model using create/send/become to manage and communicate between whole vats, but these operations aren’t exposed to the programmer. The programmer instead manipulates two-part “far references” which denote a vat along with an object local to that vat. Local objects are created frequently, like in regular object-oriented languages, but vats are created much less frequently; and each vat’s stack takes on duties performed in “classic” actor models by temporary actors.
				  
				  \#\# Conclusion
				  
				  I’ve presented three different types of actor language: the classic actor model, roughly as formulated by Agha et al.; the process actor model, represented by Erlang; and the communicating event-loop model, represented by E.
				  
				  The three models take different approaches to reconciling the need to have structured local data within each actor in addition to the more coarse-grained structure relating actors to each other.
				  
				  The classic model makes everything an actor, with local data largely deemphasised; Erlang offers a traditional functional programming model for handling local data; and E offers a smooth integration between an imperative local OO model and an asynchronous, promise-based remote OO model.
				  
				  ---
				  
				  \#\# Annotated Bibliography
				  
				  \#\#\# Early work on actors
				  
				  * **1973\. Carl Hewitt, Peter Bishop, and Richard Steiger, “A universal modular ACTOR formalism for artificial intelligence,” in Proc. International Joint Conference on Artificial Intelligence**  
				  This paper is a position paper from which we can understand the motivation and intentions of the research into actors; it lays out a very broad and colourful research vision that touches on a huge range of areas from computer architecture up through programming language design to teaching of computer science and approaches to artificial intelligence.  
				  The paper presents a _uniform actor model_; compare and contrast with uniform object models offered by (some) OO languages. The original application of the model is given as PLANNER-like AI languages.  
				  The paper claims benefits of using the actor model in a huge range of areas:  
				   * foundations of semantics  
				   * logic  
				   * knowledge-based programming  
				   * intentions (contracts)  
				   * study of expressiveness  
				   * teaching of computation  
				   * extensible, modular programming  
				   * privacy and protection  
				   * synchronization constructs  
				   * resource management  
				   * structured programming  
				   * computer architecture  
				  The paper sketches the idea of a contract (called an “intention”) for ensuring that invariants of actors (such as protocol conformance) are maintained; there seems to be a connection to modern work on “monitors” and Session Types. The authors write:  
				  > The intention is the CONTRACT that the actor has with the outside world.  
				  Everything is super meta! Actors can have intentions! Intentions are actors! Intentions can have intentions! The paper presents the beginnings of a reflective metamodel for actors. Every actor has a scheduler and an “intention”, and may have monitors, a first-class environment, and a “banker”.  
				  The paper draws an explicit connection to capabilities (in the security sense); Mark S. Miller at<http://erights.org/history/actors.html> says of the Actor work that it included “prescient” statements about Actor semantics being “a basis for confinement, years before Norm Hardy and the Key Logic guys”, and remarks that “the Key Logic guys were unaware of Actors and the locality laws at the time, \[but\] were working from the same intuitions.”  
				  There are some great slogans scattered throughout, such as “Control flow and data flow are inseparable”, and “Global state considered harmful”.  
				  The paper does eventually turn to a more nuts-and-bolts description of a predecessor language to PLASMA, which is more fully described in Hewitt 1976.  
				  When it comes to formal reasoning about actor systems, the authors here define a partial order - PRECEDES - that captures some notion of causal connection. Later, the paper makes an excursion into epistemic modal reasoning.  
				  Aside: the paper discusses continuations; Reynolds 1993 has the concept of continuations as firmly part of the discourse by 1973, having been rediscovered in a few different contexts in 1970-71 after van Wijngaarden’s 1964 initial description of the idea. See J. C. Reynolds, “The discoveries of continuations,” LISP Symb. Comput., vol. 6, no. 3–4, pp. 233–247, 1993.  
				  In the “acknowledgements” section, we see:  
				  > Alan Kay whose FLEX and SMALL TALK machines have influenced our work. Alan emphasized the crucial importance of using intentional definitions of data structures and of passing messages to them. This paper explores the consequences of generalizing the message mechanism of SMALL TALK and SIMULA-67; the port mechanism of Krutar, Balzer, and Mitchell; and the previous CALL statement of PLANNER-71 to a universal communications mechanism.
				  * **1975\. Irene Greif. PhD dissertation, MIT EECS.**  
				  Specification language for actors; per Baker an “operational semantics”. Discusses “continuations”.
				  * **1976\. C. Hewitt, “Viewing Control Structures as Patterns of Passing Messages,” AIM-410**  
				  AI focus; actors as “agents” in the AI sense; recursive decomposition: “each of the experts can be viewed as a society that can be further decomposed in the same way until the primitive actors of the system are reached.”  
				  > We are investigating the nature of the _communication mechanisms_\[…\] and the conventions of discourse  
				  More concretely, examines “how actor message passing can be used to understand control structures as patterns of passing messages”.  
				  > \[…\] there is no way to decompose an actor into its parts. An actor is defined by its behavior; not by its physical representation!  
				  Discusses PLASMA (“PLAnner-like System Modeled on Actors”), and gives a fairly detailed description of the language in the appendix. Develops “event diagrams”.  
				  Presents very Schemely factorial implementations in recursive and iterative (tail-recursive accumulator-passing) styles. During discussion of the iterative factorial implementation, Hewitt remarks that `n` is not closed over by the `loop` actor; it is “not an acquaintance” of `loop`. Is this the beginning of the line of thinking that led to Clinger’s “safe-for-space” work?  
				  Everything is an actor, but some of the actors are treated in an awfully structural way: the trees, for example, in section V.4 on Generators:  
				  ```clojure  
				  (non-terminal:  
				  (non-terminal: (terminal: A) (terminal: B))  
				  (terminal: C))  
				  ```  
				  Things written with this `keyword:` notation look like structures. Their _reflections_ are actors, but as structures, they are subject to pattern-matching; I am unsure how the duality here was thought of by the principals at the time, but see the remarks regarding “packages” in the appendix.
				  * **1977\. C. Hewitt and H. Baker, “Actors and Continuous Functionals,” MIT A.I. Memo 436A**  
				  Some “laws” for communicating processes; “plausible restrictions on the histories of computations that are physically realizable.” Inspired by physical intuition, discusses the history of a computation in terms of a partial order of events, rather than a sequence.  
				  > The actor model is a formalization of these ideas \[of Simula/Smalltalk/CLU-like active data processing messages\] that is independent of any particular programming language.  
				  > Instances of Simula and Smalltalk classes and CLU clusters are actors”, but they are _non-concurrent_. The actor model is broader, including concurrent message-passing behaviour.  
				  Laws about (essentially) lexical scope. Laws about histories (finitude of activation chains; total ordering of messages inbound at an actor; etc.), including four different ordering relations. “Laws of locality” are what Miller was referring to on that erights.org page I mentioned above; very close to the capability laws governing confinement.  
				  Steps toward denotational semantics of actors.
				  * **1977\. Russell Atkinson and Carl Hewitt, “Synchronization in actor systems,” Proc. 4th ACM SIGACT-SIGPLAN Symp. Princ. Program. Lang., pp. 267–280.**  
				  Introduces the concept of “serializers”, a “generalization and improvement of the monitor mechanism of Brinch-Hansen and Hoare”. Builds on Greif’s work.
				  * **1981\. Will Clinger’s PhD dissertation. MIT**
				  
				  \#\#\# Actors as Concurrent Object-Oriented Programming
				  
				  * **1986\. Gul Agha’s book/dissertation.**
				  * **1990\. G. Agha, “Concurrent Object-Oriented Programming,” Commun. ACM, vol. 33, no. 9, pp. 125–141**  
				  Agha’s work recast the early “classic actor model” work in terms of_concurrent object-oriented programming_. Here, he defines actors as “self-contained, interactive, independent components of a computing system that communicate by asynchronous message passing”, and gives the basic actor primitives as `create`, `send to`, and `become`. Examples are given in the actor language Rosette.  
				  This paper gives an overview and summary of many of the important facets of research on actors that had been done at the time, including brief discussion of: nondeterminism and fairness; patterns of coordination beyond simple request/reply such as transactions; visualization, monitoring and debugging; resource management in cases of extremely high levels of potential concurrency; and reflection.  
				  > The customer-passing style supported by actors is the concurrent generalization of continuation-passing style supported in sequential languages such as Scheme. In case of sequential systems, the object must have completed processing a communication before it can process another communication. By contrast, in concurrent systems it is possible to process the next communication as soon as the replacement behavior for an object is known.  
				  Note that the sequential-seeming portions of the language are defined in terms of asynchronous message passing and construction of explicit continuation actors.
				  * **1997\. G. A. Agha, I. A. Mason, S. F. Smith, and C. L. Talcott, “A Foundation for Actor Computation,” J. Funct. Program., vol. 7, no. 1, pp. 1–72**  
				  Long paper that carefully and fully develops an operational semantics for a concrete actor language based on lambda-calculus. Discusses various equivalences and laws. An excellent starting point if you’re looking to build on a modern approach to operational semantics for actors.
				  
				  \#\#\# Erlang: Actors from requirements for fault-tolerance / high-availability
				  
				  * **2003\. J. Armstrong, “Making reliable distributed systems in the presence of software errors,” Royal Institute of Technology, Stockholm**  
				  A good overview of Erlang: the language, its design intent, and the underlying philosophy. Includes an evaluation of the language design.
				  
				  \#\#\# E: Actors from requirements for secure interaction
				  
				  * **2005\. M. S. Miller, E. D. Tribble, and J. Shapiro, “Concurrency Among Strangers,” in Proc. Int. Symp. on Trustworthy Global Computing, pp. 195–229.**  
				  As I summarised this paper for a seminar class on distributed systems: “The authors present E, a language designed to help programmers manage_coordination_ of concurrent activities in a setting of distributed, mutually-suspicious objects. The design features of E allow programmers to take control over concerns relevant to distributed systems, without immediately losing the benefits of ordinary OO programming.”  
				  E is a canonical example of the “communicating event loops” approach to Actor languages, per the taxonomy of the survey paper listed below. It combines message-passing and isolation in an interesting way with ordinary object-oriented programming, giving a two-level language structure that has an OO flavour.  
				  The paper does a great job of explaining the difficulties that arise when writing concurrent programs in traditional models, thereby motivating the actor model in general and the features of E in particular as a way of making the programmer’s job easier.
				  
				  \#\#\# Taxonomy of actors
				  
				  * **2016\. J. De Koster, T. Van Cutsem, and W. De Meuter, “43 Years of Actors: A Taxonomy of Actor Models and Their Key Properties,” Software Languages Lab, Vrije Universiteit Brussel, VUB-SOFT-TR-16-11**  
				  A very recent survey paper that offers a taxonomy for classifying actor-style languages. At its broadest, actor languages are placed in one of four groups:  
				   * The Classic Actor Model (create, send, become)  
				   * Active Objects (OO with a thread per object; copying of passive data between objects)  
				   * Processes (raw Erlang; receive, spawn, send)  
				   * Communicating Event-Loops (E; near and far references; eventual references; batching)  
				  Different kinds of “futures” or “promises” also appear in many of these variations in order to integrate asynchronous message_reception_ with otherwise-sequential programming.
				  - [Scumbag pissed that his ex-wife stopped f**king him](https://omnivore.app/me/https-www-youtube-com-watch-v-c-4-wkn-3-n-cq-4-o-18e1a5037db)
				  collapsed:: true
				  site:: [YouTube](https://www.youtube.com/watch?v=c4WKN3NCq4o)
				  author:: Stavvy's World Clips
				  date-saved:: [[03/07/2024]]
				  - ### Content
				  collapsed:: true
				  - [Scumbag pissed that his ex-wife stopped f\*\*king him](https://www.youtube.com/watch?v=c4WKN3NCq4o)
				  
				  By [Stavvy's World Clips](https://www.youtube.com/@stavvysworldclips)
				  - [Stack Definition Language (SDL) | Akash Network - Your Guide to Decentralized Cloud](https://omnivore.app/me/stack-definition-language-sdl-akash-network-your-guide-to-decent-18df6dc5413)
				  collapsed:: true
				  site:: [akash.network](https://akash.network/docs/getting-started/stack-definition-language/)
				  date-saved:: [[02/29/2024]]
				  - ### Content
				  collapsed:: true
				  - Customers / tenants define the deployment services, datacenters, requirements, and pricing parameters, in a “manifest” file (deploy.yaml). The file is written in a declarative language called Software Definition Language (SDL). SDL is a human friendly data standard for declaring deployment attributes. The SDL file is a “form” to request resources from the Network. SDL is compatible with the [YAML](https://yaml.org/) standard and similar to Docker Compose files.
				  
				  Configuration files may end in `.yml` or `.yaml`.
				  
				  A complete deployment has the following sections:
				  
				  * [version](\#version)
				  * [services](\#services)
				  * [profiles](\#profiles)
				  * [deployment](\#deployment)
				  * [persistent storage](https://akash.network/docs/network-features/persistent-storage/)
				  * [gpu support](\#gpu-support)
				  * [stable payment](\#stable-payment)
				  
				  An example deployment configuration can be found [here](https://github.com/akash-network/docs/tree/62714bb13cfde51ce6210dba626d7248847ba8c1/sdl/deployment.yaml).
				  
				  \#\#\#\# Networking
				  
				  Networking - allowing connectivity to and between workloads - can be configured via the Stack Definition Language (SDL) file for a deployment. By default, workloads in a deployment group are isolated - nothing else is allowed to connect to them. This restriction can be relaxed.
				  
				  \#\# Version
				  
				  Indicates version of Akash configuration file. Currently only `"2.0"` is accepted.
				  
				  \#\# Services
				  
				  The top-level `services` entry contains a map of workloads to be ran on the Akash deployment. Each key is a service name; values are a map containing the following keys:
				  
				  | Name       | Required | Meaning                                                                                                    |
				  | ---------- | -------- | ---------------------------------------------------------------------------------------------------------- |
				  | image      | Yes      | Docker image of the containerBest practices:avoid using image tags as Akash Providers heavily cache images |
				  | depends-on | No       | _**NOTE - field is marked for future use and currently has no impact on deployments.**_                    |
				  | command    | No       | Custom command use when executing container                                                                |
				  | args       | No       | Arguments to custom command use when executing the container                                               |
				  | env        | No       | Environment variables to set in running container. See [services.env](\#servicesenv)                        |
				  | expose     | No       | Entities allowed to connect to the services. See [services.expose](\#servicesexpose)                        |
				  
				  \#\#\# services.env
				  
				  A list of environment variables to expose to the running container.
				  
				  ```routeros
				  
				  env:
				- API_KEY=0xcafebabe
				- CLIENT_ID=0xdeadbeef
				  
				  ```
				  
				  \#\#\# services.expose
				  
				  \#\#\#\# Notes Regarding Port Use in the Expose Stanza
				  
				  * HTTPS is possible in Akash deployments but only self signed certs are generated.
				  * To implement signed certs the deployment must be front ended via a solution such as Cloudflare. If interested in this path, we have created docs for [Cloudflare with Akash](https://akash.network/docs/guides/tls-termination-of-akash-deployment/).
				  * You can expose any other port besides 80 as the ingress port (HTTP, HTTPS) port using as: 80 directive if the app understands HTTP / HTTPS. Example of exposing a React web app using this method:
				  
				  ```yaml
				- port: 3000
				  
				  as: 80
				  
				  to:
					- global: true
					  
					  accept:
					- www.mysite.com
					  
					  ```
					  
					  * In the SDL it is only necessary to expose port 80 for web apps. With this specification both ports 80 and 443 are exposed.
					  
					  `expose` is a list describing what can connect to the service. Each entry is a map containing one or more of the following fields:
					  
					  | Name   | Required | Meaning                                                                          |
					  | ------ | -------- | -------------------------------------------------------------------------------- |
					  | port   | Yes      | Container port to expose                                                         |
					  | as     | No       | Port number to expose the container port as                                      |
					  | accept | No       | List of hosts to accept connections for                                          |
					  | proto  | No       | Protocol type. Valid values = tcp or udp                                         |
					  | to     | No       | List of entities allowed to connect. See [services.expose.to](\#servicesexposeto) |
					  
					  The `as` value governs the default `proto` value as follows:
					  
					  > _**NOTE**_ \- when as is not set, it will default to the value set by the port mandatory directive.
					  
					  > _**NOTE**_ \- when one exposes as: 80 (HTTP), the Kubernetes ingress controller makes the application available over HTTPS as well, though with the default self-signed ingress certs.
					  
					  | port       | proto default |
					  | ---------- | ------------- |
					  | 80         | http & https  |
					  | all others | tcp & udp     |
					  
					  \#\#\# services.expose.to
					  
					  `expose.to` is a list of clients to accept connections from. Each item is a map with one or more of the following entries:
					  
					  | Name    | Value                        | Default                            | Description                                               |
					  | ------- | ---------------------------- | ---------------------------------- | --------------------------------------------------------- |
					  | service | A service in this deployment | Allow the given service to connect |                                                           |
					  | global  | true or false                | false                              | If true, allow connections from outside of the datacenter |
					  
					  If no service is given and `global` is true, any client can connect from anywhere (web servers typically want this).
					  
					  If a service name is given and `global` is `false`, only the services in the current datacenter can connect. If a service name is given and `global` is `true`, services in other datacenters for this deployment can connect.
					  
					  If `global` is `false` then a service name must be given.
					  
					  \#\# profiles
					  
					  The `profiles` section contains named compute and placement profiles to be used in the [deployment](\#deployment).
					  
					  \#\#\# profiles.compute
					  
					  `profiles.compute` is map of named compute profiles. Each profile specifies compute resources to be leased for each service instance uses uses the profile.
					  
					  Example:
					  
					  This defines a profile named `web` having resource requirements of 2 vCPUs, 2 gigabytes of memory, and 5 gigabytes of storage space available.
					  
					  ```dts
					  
					  web:
					  
					  cpu: 2
					  
					  memory: "2Gi"
					  
					  storage: "5Gi"
					  
					  ```
					  
					  `cpu` units represent a vCPU share and can be fractional. When no suffix is present the value represents a fraction of a whole CPU share. With a `m` suffix, the value represnts the number of milli-CPU shares (1/1000 of a CPU share).
					  
					  Example:
					  
					  | Value  | CPU-Share |
					  | ------ | --------- |
					  | 1      | 1         |
					  | 0.5    | 1/2       |
					  | "100m" | 1/10      |
					  | "50m"  | 1/20      |
					  
					  `memory`, `storage` units are described in bytes. The following suffixes are allowed for simplification:
					  
					  | Suffix | Value  |
					  | ------ | ------ |
					  | k      | 1000   |
					  | Ki     | 1024   |
					  | M      | 1000^2 |
					  | Mi     | 1024^2 |
					  | G      | 1000^3 |
					  | Gi     | 1024^3 |
					  | T      | 1000^4 |
					  | Ti     | 1024^4 |
					  | P      | 1000^5 |
					  | Pi     | 1024^5 |
					  | E      | 1000^6 |
					  | Ei     | 1024^6 |
					  
					  \#\#\# profiles.placement
					  
					  `profiles.placement` is map of named datacenter profiles. Each profile specifies required datacenter attributes and pricing configuration for each [compute profile](\#profilescompute) that will be used within the datacenter. It also specifies optional list of signatures of which tenants expects audit of datacenter attributes.
					  
					  Example:
					  
					  ```dts
					  
					  westcoast:
					  
					  attributes:
					  
					  region: us-west
					  
					  signedBy:
					  
					  allOf:
				- "akash1vz375dkt0c60annyp6mkzeejfq0qpyevhseu05"
				  
				  anyOf:
				- "akash1vl3gun7p8y4ttzajrtyevdy5sa2tjz3a29zuah"
				  
				  pricing:
				  
				  web:
				  
				  denom: uakt
				  
				  amount: 8
				  
				  db:
				  
				  denom: uakt
				  
				  amount: 100
				  
				  ```
				  
				  This defines a profile named `westcoast` having required attributes `{region="us-west"}`, and with a max price for the `web` and `db` [compute profiles](\#profilescompute) of 8 and 15 `uakt` per block, respectively. It also requires that the provider’s attributes have been [signed by](\#profilesplacementsignedby) the accounts `akash1vz375dkt0c60annyp6mkzeejfq0qpyevhseu05` and `akash1vl3gun7p8y4ttzajrtyevdy5sa2tjz3a29zuah`.
				  
				  \#\#\# profiles.placement.signedBy
				  
				  **Optional**
				  
				  The `signedBy` section allows you to state attributes that must be signed by one or more accounts of your choosing. This allows for requiring a third-party certification of any provider that you deploy to.
				  
				  \#\# Deployment
				  
				  The `deployment` section defines how to deploy the services. It is a mapping of service name to deployment configuration.
				  
				  Each service to be deployed has an entry in the `deployment`. This entry is maps datacenter profiles to [compute profiles](\#profilescompute) to create a final desired configuration for the resources required for the service.
				  
				  Example:
				  
				  ```yaml
				  
				  web:
				  
				  westcoast:
				  
				  profile: web
				  
				  count: 20
				  
				  ```
				  
				  This says that the 20 instances of the `web` service should be deployed to a datacenter matching the `westcoast` datacenter profile. Each instance will have the resources defined in the `web` [compute profile](\#profilescompute) available to it.
				  
				  \#\# GPU Support
				  
				  GPUs can be added to your workload via inclusion the compute profile section. The placement of the GPU stanza can be viewed in the full compute profile example shown below.
				  
				  > _**NOTE**_ \- currently the only accepted vendor is `nvidia` but others will be added soon
				  
				  ```yaml
				  
				  profiles:
				  
				  compute:
				  
				  obtaingpu:
				  
				  resources:
				  
				  cpu:
				  
				    units: 1.0
				  
				  memory:
				  
				    size: 1Gi
				  
				  gpu:
				  
				    units: 1
				  
				    attributes:
				  
				      vendor:
				  
				        nvidia:
					- model: 4090
					  
					  storage:
					  
					  size: 1Gi
					  
					  ```
					  
					  \#\#\# Additional GPU Use Notes
					  
					  \#\#\#\# Full GPU SDL Example
					  
					  To view an example GPU enabled SDL in full for greater context, review this [example](https://github.com/akash-network/awesome-akash/blob/c24af5335be2bccb9d47c95bdd6ab68645fcd679/torchbench/torchbench%5Fgpu%5Fsdl%5Fcuda12%5F0.yaml\#L3) which utilized the declaration of several GPU models.
					  
					  \#\#\#\# Model Specification Optional
					  
					  The declaration of a GPU model is optional in the SDL. If your deployment does not require a specific GPU model, leave the model declaration blank as seen in the following example.
					  
					  ```yaml
					  
					  gpu:
					  
					  units: 1
					  
					  attributes:
					  
					  vendor:
					  
					  nvidia:
					  
					  ```
					  
					  \#\#\#\# Multiple Models Declared
					  
					  If your deployment is optimized to run on multiple GPU models, include the appropriate list of models as seen in the following example. In this usage, any Akash provider that has a model in the list will bid on the deployment.
					  
					  ```yaml
					  
					  gpu:
					  
					  units: 1
					  
					  attributes:
					  
					  vendor:
					  
					  nvidia:
					- model: 4090
					- model: t4
					  
					  ```
					  
					  \#\# Stable Payment
					  
					  Use of Stable Payments is supported in the Akash SDL and is declared in the placement section of the SDL as shown in the example below.
					  
					  > _**NOTE**_ \- currently only `Axelar USDC (usdc)` is supported and `denom` must be specified as the precise IBC channel name shown in the example.
					  
					  ```yaml
					  
					  placement:
					  
					  global:
					  
					  pricing:
					  
					  web:
					  
					  denom: ibc/170C677610AC31DF0904FFE09CD3B5C657492170E7E52372E48756B71E56F2F1
					  
					  amount: 100
					  
					  bew:
					  
					  denom: ibc/170C677610AC31DF0904FFE09CD3B5C657492170E7E52372E48756B71E56F2F1
					  
					  amount: 100
					  
					  ```
					  
					  \#\#\#\# Full GPU SDL Example 
					  
					  To view an example Stable Payment enabled SDL in full for greater context, review this [example](https://gist.github.com/chainzero/040d19bdb20d632009b8ae206fb548f5).
					  - [Wait, Don’t Touch That!. Mutual Exclusion Locks & JavaScript | by Dan Pupius | Medium Engineering](https://omnivore.app/me/wait-don-t-touch-that-mutual-exclusion-locks-java-script-by-dan--18df2bea623)
					  collapsed:: true
					  site:: [Medium Engineering](https://medium.engineering/wait-dont-touch-that-a211832adc3a)
					  author:: Dan Pupius
					  date-saved:: [[02/28/2024]]
					  date-published:: [[01/16/2015]]
					  - ### Content
					  collapsed:: true
					  - \#\# Mutual Exclusion Locks & JavaScript
					  
					  _Everyone_ knows that JavaScript runtimes are single-threaded.
					  
					  The runtime contains a message queue and each message has a function associated with it. Messages are taken off the queue and the associated function is run to completion. When the stack unrolls and the function completes, the next message is taken from the queue. While a function is executing no other messages can be processed, you can’t preempt execution, and no other work can be done.
					  
					  This makes JavaScript programming simple, right?
					  
					  You don’t need to worry about concurrency. No parallel code accessing the same resource. No lock contention. No synchronization issues. Right?
					  
					  Mostly. But there are a couple of exceptional cases.
					  
					  Different browser tabs can run in different processes and [web workers](http://www.html5rocks.com/en/tutorials/workers/basics/) have introduced threading to the browser. So while it is certainly an edge case, situations do exist where shared resources — such as LocalStorage — may be accessed concurrently.
					  
					  At Medium we use LocalStorage for tracking read time. It increases the chance of making sure reports make it back to the server even in cases where the user navigates away or closes the tab. But we need to avoid multiple tabs sending the data multiple times.
					  
					  For a solution we look to a [1985 paper](http://research.microsoft.com/en-us/um/people/lamport/pubs/fast-mutex.pdf) by [Leslie Lamport](http://en.wikipedia.org/wiki/Leslie%5FLamport), then at DEC, now at Microsoft Research. It describes a [mutual exclusion locking algorithm](http://www.cs.rochester.edu/research/synchronization/pseudocode/fastlock.html) for use in situations where there are no atomic test-and-set operations. While the paper is in the context of multiprocessor environments, this is exactly the problem we have with our concurrent access to LocalStorage.
					  
					  The algorithm looks like this:
					  
					  1.  Set X = i  
					  2.  If Y != 0: Restart  
					  3.  Set Y = i  
					  4.  If X != i:  
					  5.    Delay  
					  6.    If Y != i: Restart  
					  7.  [Has lock, do work]  
					  8.  Set Y = 0Where **X**and **Y**are shared memory slots and **i**is a unique identifier for the client.
					  
					  In English:
					  
					  1. Always set X to the current client’s unique identifier.
					  2. If Y is not zero then another client has the lock, so restart.
					  3. If Y was zero, then set Y to the client ID.
					  4. If X has changed, there’s a possibility of contention. So…
					  5. Delay for long enough for another client to have seen Y as zero and tried to write it. (We added a random jitter here to minimize the chance of a client being starved.)
					  6. If the client didn’t win Y, then restart the whole process.
					  7. The lock was won, or there was no sign of contention, so now we can do our work.
					  8. Clear Y to allow another client to take the lock.
					  
					  And in JavaScript:
					  
					  The algorithm is fast if there is no contention, but in practice a JS implementation will use setTimeout for the restart (\#2, \#6) and delay (\#5) steps, which imposes latency. That means you want to design your application to minimize the likelihood of contention. In our reporting example, we do frequent, lock-free writes to probabilistically unique keys, then a client claims a lock while it reads, sends, and deletes the reports.
					  
					  An additional consideration is clients dying while they hold the lock, which can happen in the case of error or a closed tab. To solve this, at step \#2 you can add a timeout to the claim for Y, in _\_isLockAvailable()_.
					  
					  This is definitely an edge case, and while it’s not a problem most will stumble upon, I’d wager that as the web platform progresses the need will become more common. If I’m right, the Fast Mutex will be a useful tool to have in your arsenal.
					  
					  Further reading:
					  
					  * [A Fast Mutual Exclusion Algorithm](http://research.microsoft.com/en-us/um/people/lamport/pubs/fast-mutex.pdf)
					  * [Fast Mutual Exclusion, Even With Contention](http://www.cs.rochester.edu/research/synchronization/pseudocode/fastlock.html)
					  * [Concurrency and the Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/EventLoop)
					  * [JavaScript concurrency and locking the HTML5 LocalStorage](http://balpha.de/2012/03/javascript-concurrency-and-locking-the-html5-localstorage/)
					  * [What is a mutex?](http://stackoverflow.com/questions/34524/what-is-a-mutex)
					  - [A Comprehensive Guide to the JavaScript Event Loop](https://omnivore.app/me/a-comprehensive-guide-to-the-java-script-event-loop-18dd432b2a3)
					  collapsed:: true
					  site:: [codedamn news](https://codedamn.com/news/programming/comprehensive-guide-javascript-event-loop)
					  date-saved:: [[02/22/2024]]
					  date-published:: [[03/20/2023]]
					  - ### Content
					  collapsed:: true
					  - JavaScript is an essential language for web development, enabling developers to create interactive and dynamic web applications. One key concept every JavaScript developer should understand is the event loop, which plays a crucial role in JavaScript's asynchronous behavior. In this comprehensive guide, we will explore the JavaScript event loop, its inner workings, and how it affects the execution of your code. This guide is designed to be beginner-friendly, with clear explanations and code examples to illustrate the concepts.
					  
					  \#\# Understanding JavaScript Concurrency Model
					  
					  Before diving into the event loop, it's important to understand the JavaScript concurrency model. JavaScript is single-threaded, which means it can only execute one task at a time. However, it also has asynchronous capabilities that allow it to perform multiple tasks seemingly at once. This is achieved through the event loop, which helps manage the execution of tasks in a non-blocking manner.
					  
					  \#\#\# The Call Stack
					  
					  The call stack is a data structure that JavaScript uses to keep track of function calls and their execution context. Whenever a function is called, its context is pushed onto the call stack. When the function finishes executing, its context is popped off the stack, and the execution returns to the calling function.
					  
					  Consider the following code example:
					  
					  `function firstFunction() {
					  console.log('First function executed');
					  }
					  
					  function secondFunction() {
					  firstFunction();
					  console.log('Second function executed');
					  }
					  
					  secondFunction();`
					  
					  The call stack's state changes as follows:
					  
					  1. Initially, the call stack is empty.
					  2. The `secondFunction()` call is pushed onto the stack.
					  3. The `firstFunction()` call is pushed onto the stack.
					  4. The `firstFunction()` call is popped off the stack after it finishes executing.
					  5. The `secondFunction()` call is popped off the stack after it finishes executing.
					  
					  \#\#\# Web APIs, Callback Queue, and Event Loop
					  
					  To handle asynchronous tasks, JavaScript relies on web APIs provided by the browser or the runtime environment (e.g., Node.js). Examples of such tasks include timers, XMLHttpRequests, and DOM events.
					  
					  When an asynchronous task is completed, its callback function is added to the callback queue (also known as the task queue or the message queue). The event loop constantly checks the call stack and the callback queue. If the call stack is empty and there's a callback in the queue, the event loop pushes the callback onto the call stack for execution.
					  
					  \#\# The Event Loop in Action
					  
					  Now that we have an understanding of the JavaScript concurrency model, let's see how the event loop works with a practical example.
					  
					  `console.log('Start');
					  
					  setTimeout(function timeoutCallback() {
					  console.log('Timeout');
					  }, 0);
					  
					  console.log('End');`
					  
					  Here's what happens in this example:
					  
					  1. The `console.log('Start')` call is pushed onto the call stack and executed, displaying "Start" in the console.
					  2. The `setTimeout()` function is called, and its callback (`timeoutCallback`) is sent to the web API with a delay of 0 milliseconds.
					  3. The `console.log('End')` call is pushed onto the call stack and executed, displaying "End" in the console.
					  4. After the delay, the `timeoutCallback` is moved from the web API to the callback queue.
					  5. The event loop checks the call stack and, finding it empty, pushes the `timeoutCallback` onto the call stack.
					  6. The `timeoutCallback` is executed, displaying "Timeout" in the console.
					  
					  Even though the delay in `setTimeout()` was set to 0 milliseconds, the "Timeout" message appears after "End" due to the event loop's operation.
					  
					  \#\# Promises and Microtasks
					  
					  Promises, a feature introduced in ECMAScript 2015 (ES6), are another wayto handle asynchronous operations in JavaScript. They provide a cleaner syntax and better error handling compared to traditional callback-based approaches. With promises, the event loop introduces another queue called the microtask queue.
					  
					  \#\#\# Microtask Queue
					  
					  The microtask queue is similar to the callback queue but has a higher priority. When a promise is resolved or rejected, its `.then()` or `.catch()` callbacks are added to the microtask queue. The event loop prioritizes executing tasks from the microtask queue over those in the callback queue.
					  
					  Consider the following example:
					  
					  `console.log('Start');
					  
					  setTimeout(function timeoutCallback() {
					  console.log('Timeout');
					  }, 0);
					  
					  Promise.resolve().then(function promiseCallback() {
					  console.log('Promise');
					  });
					  
					  console.log('End');`
					  
					  The output will be:
					  
					  ```sql
					  Start
					  End
					  Promise
					  Timeout
					  
					  ```
					  
					  Here's the step-by-step breakdown:
					  
					  1. The `console.log('Start')` call is pushed onto the call stack and executed, displaying "Start" in the console.
					  2. The `setTimeout()` function is called, and its callback (`timeoutCallback`) is sent to the web API with a delay of 0 milliseconds.
					  3. The `Promise.resolve()` call is executed, and its callback (`promiseCallback`) is added to the microtask queue.
					  4. The `console.log('End')` call is pushed onto the call stack and executed, displaying "End" in the console.
					  5. The event loop checks the microtask queue and pushes the `promiseCallback` onto the call stack for execution, displaying "Promise" in the console.
					  6. The `timeoutCallback` is moved from the web API to the callback queue after the delay.
					  7. The event loop checks the call stack and, finding it empty, pushes the `timeoutCallback` onto the call stack.
					  8. The `timeoutCallback` is executed, displaying "Timeout" in the console.
					  
					  \#\#\# async/await
					  
					  Another addition to ECMAScript 2017 (ES8) is the `async/await` syntax, which further simplifies asynchronous code. This syntax allows you to write asynchronous code in a synchronous manner using `async` functions and the `await` keyword. Internally, `async/await` relies on promises and the microtask queue.
					  
					  Here's an example using `async/await`:
					  
					  `console.log('Start');
					  
					  setTimeout(function timeoutCallback() {
					  console.log('Timeout');
					  }, 0);
					  
					  async function asyncFunction() {
					  console.log('Before await');
					  await Promise.resolve();
					  console.log('After await');
					  }
					  
					  asyncFunction();
					  
					  console.log('End');`
					  
					  The output will be:
					  
					  ```sql
					  Start
					  Before await
					  End
					  After await
					  Timeout
					  
					  ```
					  
					  The `await` keyword causes the execution of the `asyncFunction()` to pause until the promise is resolved, allowing other synchronous code to continue executing. The callback after the `await` is added to the microtask queue and executed before the `timeoutCallback` from the callback queue.
					  
					  \#\# Conclusion
					  
					  Understanding the JavaScript event loop is essential for mastering asynchronous programming in JavaScript. By grasping the concepts of the call stack, callback queue, microtask queue, and event loop, you can write more efficient and maintainable code. Moreover, modern features like promises and `async/await` provide cleaner and more readable syntax for handling asynchronous tasks.
					  
					  \#\#\# Sharing is caring
					  
					  Did you like what Mehul Mohan wrote? Thank them for their work by sharing it on social media.
					  
					  \#\# No comments so far
					  - [The Mystery of Order - by Robin Hanson - Overcoming Bias](https://omnivore.app/me/the-mystery-of-order-by-robin-hanson-overcoming-bias-18dc3aa93dc)
					  collapsed:: true
					  site:: [substack.com](https://substack.com/inbox/post/141774385)
					  author:: Substack
					  labels:: [[robin-hanson]] [[overcoming–bias]]
					  date-saved:: [[02/19/2024]]
					  - ### Content
					  collapsed:: true
					  - In the ancient world, people tended to see rival nations as ruled by illicit tyrants, while their nation was ruled justly by a king. 
					  
					  Two centuries ago when religion was first declining, many predicted that crime would greatly increase as a result. It was mainly fear of God, they said, that kept most from committing crime. But then crime came down.
					  
					  Centuries ago, there was widespread skepticism about joint stock companies, as it was presumed that firm managers would usually steal firm assets. It took centuries to convince skeptics. Many still presume that firms naturally enslave employees, unless prevented by good government. 
					  
					  Many in Europe predicted that the first U.S. president, George Washington, would not step down from office when voted out of office. Why give up such power? But step down he did. 
					  
					  Many say that nations naturally conquer their much weaker neighbors, at least if those do not have strong alliance partners. But all the nations near the U.S. are much weaker, none with protecting alliances, yet it has been a long time since the U.S. has even considered conquering them. 
					  
					  Disasters like the sinking of the Titanic are usually depicted as full of anarchy, violence, and every man for himself. But in fact, order usually prevails. 
					  
					  Many have long presumed that a king must be a better fighter than any who guard or advise him, as else such a guard would fight and kill him to replace him. 
					  
					  Many today presume that corrupt officials are endemic to most of the world, as it seems natural to them that police and regulators will usually demand bribes, that official treasurers will steal the money they oversee, and that militaries will take over governments via coups. 
					  
					  Yesterday in response to my [puzzling](https://www.overcomingbias.com/p/why-not-for-profit-govt) over the absence of for-profit governments, many said it was obvious that such regimes would enslave residents, and perhaps even eat them, and also refuse to pay off investors. 
					  
					  I guess their reasoning is like that in the above cases: in the usual human condition order only arises from selfish depravity, and our current world is a fragile rare exception, whose exceptionality would end with for-profit governments. But then why don’t for-profit firms today enslave their employees and customers? Good government stops them! 
					  
					  And why don’t government treasurers now steal our money, our militaries now take over our governments, or our elected leaders now refuse to step down at term end? Because they are good people, from our rare good people culture. And why wouldn’t those good people also be good under for-profit governments? Maybe because money is profane, and corrupts the sacred hearts of good sacred-government people. 
					  
					  And this is why we may need sacred money to enable for-sacred-profit governments. People seem to presume that order is naturally rare, only existing due to either ruthless power or rare good-sacred-hearted people like themselves. 
					  
					  **Added**: A related presumption of depravity is that AIs will exterminate humans, if given half a chance.
					  - [Fantas, Eel, and Specification 16: Extend · Tom Harding](https://omnivore.app/me/fantas-eel-and-specification-16-extend-tom-harding-18da59ced72)
					  collapsed:: true
					  site:: [tomharding.me](http://www.tomharding.me/2017/06/12/fantas-eel-and-specification-16/)
					  date-saved:: [[02/13/2024]]
					  date-published:: [[06/11/2017]]
					  - ### Content
					  collapsed:: true
					  - You’re **still** here? That means you survived `Monad`! See? Told you it isn’t that scary. It’s nothing we haven’t **already seen**. Well, _today_, we’re going to revisit `Chain` with one _slight_ difference. As we know, `==Chain==` ==takes an== `==m a==` ==to an== `==m b==` ==with some help from an== `==a== ==-====&gt;== ==m b==` ==function.== It **sequences** our ideas. However, _what if I told you_… we could go **backwards**? Let’s `Extend` your horizons.
					  
					  Don’t get too excited; the disappointment of `Contravariant` will come flooding back! We’re certainly not saying that we have a magical `undo` for any `Chain`. What we _are_ saying, though, is that there are some types for which we can get from `m a` to `m b` via `m a -> b`. Instead of **finishing** in the context, we **start** in it!
					  
					  Two questions probably come to mind:
					  
					  * **How** is this useful?
					  * … See question 1?
					  
					  Well, we’ll be answering **at least** one of those questions today! _Before_ that, though, let’s go back to our _old_ format and **start** with the **function**:
					  
					  ```livescript
					  extend :: Extend w => w a ~> (w a -> b) -> w b
					  
					  ```
					  
					  > Notice we’re using `w` here. I kid you _not_, we use `w` because it looks like an upside-down `m`, and `m` was what we used for `Chain`. It’s literally there to make the whole thing look **back-to-front** and **upside-down**. I’m told that mathematicians call this a **joke**, so do _try_ to laugh!
					  
					  _As we said_, it’s `Chain` going backwards. It even has the same laws backwards! Say hello, once again, to our old friend **associativity**:
					  
					  ```jboss-cli
					  // RHS: Apply to f THEN chain g
					  m.chain(f).chain(g)
					  === m.chain(x => f(x).chain(g))
					  
					  // RHS: extend f THEN apply to g
					  w.extend(f).extend(g)
					  === x.extend(w_ => g(w_.extend(f)))
					  
					  ```
					  
					  > Wait, so, if `extend` has **associativity**, and if it looks a bit like a `Semigroup`… (_all together now_) **where’s the `Monoid`**?!
					  
					  It’s just like `chain`, but backwards. **Everything is backwards**. It’s really the only thing you need to remember!
					  
					  ?thgir ,taen ytterP
					  
					  ---
					  
					  So, _aside_ from it being backwards, is there anything _useful_ about `Extend`? _Or_, are we just getting a bit tired at this point? **Both**! Hopefully more the former, though…
					  
					  Let’s start with an old friend, [the Pair (or Writer) type](http://www.tomharding.me/2017/04/27/pairs-as-functors/). When we `chain`, we have _total_ control over the **output** of our function: we say what we _append_ to the **left-hand side**, and what we want the right-hand value to be. There was, however, one thing we _can’t_ do: see what was already _in_ the left part!
					  
					  `Pair` really gave us a wonderful way to achieve **write-only** state, but we had no way of **reading** what we’d written! Wouldn’t it be **great** if we had a `map`\-like function that let us take a **sneaky peek** at the left-hand side? Something like this, perhaps:
					  
					  ```livescript
					  map :: (m, a) ~> (a  -> b) -> (m, b)
					  
					  -- Just like map, but we get a sneaky peek!
					  sneakyPeekMap :: (m, a)
					  ~> ((m, a) -> b)
					  -> (m, b)
					  
					  ```
					  
					  It’s really just like `map`, but we get some **context**! Now that we can, whenever we like, take a look at how the left-hand side is doing, we can feed our **hungry adventurer**:
					  
					  ```crmsh
					  //- Sum represents "hunger"
					  const Adventurer = Pair(Sum)
					  
					  //+ type User = { name :: String, isHungry :: Bool }
					  const exampleUser = {
					  name: 'Tom',
					  isHungry: false
					  }
					  
					  // You get the idea... WorkPair again!
					  
					  //+ slayDragon :: User -> Adventurer User
					  const slayDragon = user =>
					  Adventurer(Sum(100), user)
					  
					  //+ slayDragon :: User -> Adventurer User
					  const runFromDragon = user =>
					  Adventurer(Sum(50), user)
					  
					  //- Eat IF we're hungry
					  //+ eat :: User -> Adventurer User
					  const eat = user =>
					  user.isHungry
					  ? Adventurer(Sum(-100), {
					  ... user,
					  isHungry: false
					  })
					  
					  : Adventurer(Sum(0),   user)
					  
					  //- How could we know when we're hungry?
					  //- This function goes the other way...
					  //+ areWeHungry :: Adventurer User -> User
					  const areWeHungry =
					  ({ _1: { value: hunger }, _2: user }) =>
					  hunger > 200
					  ? { ... user, isHungry: true }
					  : user
					  
					  // Now, we do a thing, check our hunger,
					  // and eat if we need to!
					  
					  // WE ARE SELF-AWARE.
					  // SKYNET
					  
					  slayDragon(exampleUser)
					  .sneakyPeekMap(areWeHungry).chain(eat)
					  // Pair(Sum(100), not hungry)
					  
					  .chain(slayDragon)
					  .sneakyPeekMap(areWeHungry).chain(eat)
					  // Pair(Sum(200), not hungry)
					  
					  .chain(runFromDragon)
					  .sneakyPeekMap(areWeHungry).chain(eat)
					  // Pair(Sum(150), not hungry)!
					  
					  ```
					  
					  Just with this `sneakyPeekMap`, we can now inspect our character stats and feed that _back_ into our actions. This is _so_ neat: any time you want to update one piece of data depending on another, `sneakyPeekMap` is **exactly** what you need. Oh, and by the way, it has a much more common name: `extend`!
					  
					  ---
					  
					  _So, can I just think of `extend` as `sneakyPeekMap`?_ I mean, you basically can; that intuition will get you a **long** way. As an homage to [Hardy Jones’ functional pearl](https://joneshf.github.io/programming/2015/12/31/Comonads-Monoids-and-Trees.html), let’s build a [**Rose Tree**](https://en.wikipedia.org/wiki/Rose%5FTree):
					  
					  ```php
					  //- A root and a list of children!
					  //+ type RoseTree a = (a, [RoseTree a])
					  const RoseTree = daggy.tagged(
					  'RoseTree', ['root', 'forest']
					  )
					  
					  ```
					  
					  _Maybe_, you looked at that type and a little voice in the back of your mind said `Functor`. If so, **kudos**:
					  
					  ```javascript
					  RoseTree.prototype.map = function (f) {
					  return RoseTree(
					  f(this.root), // Transform the root...
					  this.forest.map( // and other trees.
					  rt => rt.map(f)))
					  }
					  
					  ```
					  
					  **No problem**; transform the root node, and then _recursively_ map over all the child trees. **Bam**. _Now_, imagine if we wanted to _colour_ this tree’s nodes depending on how many **children** they each have. Sounds like we’d need to take… a **sneaky peek**!
					  
					  ```javascript
					  //+ extend :: RoseTree a
					  //+        ~> (RoseTree a -> b)
					  //+        -> RoseTree b
					  RoseTree.prototype.extend =
					  function (f) {
					  return RoseTree(
					  f(this),
					  this.forest.map(rt =>
					  rt.extend(f))
					  )
					  }
					  
					  // Now, it's super easy to do this!
					  MyTree.extend(({ root, forest }) =>
					  forest.length < 1
					  ? { ... root, colour: 'RED' }
					  : forest.length < 5
					  ? { ... root, colour: 'ORANGE' }
					  : { ... root, colour: 'GREEN' })
					  
					  ```
					  
					  With our new-found **superpower**, each node gets to pretend to be the **one in charge**, and can watch over their own forests. The trees in those forests then do the same, and so on, until we `map`\-with-a-sneaky-peek **the entire forest**! Again, I linked to Hardy’s article above, which contains a much **deeper** dive into trees specifically; combining `RoseTree` with Hardy’s ideas is enough to make your own **billion-dollar React clone**!
					  
					  ---
					  
					  Let’s cast our minds _waaay_ back to [the Setoid post](http://www.tomharding.me/2017/03/09/fantas-eel-and-specification-3/), when we looked at `List`. List, it turns out, is a `Functor`:
					  
					  ```javascript
					  //- Usually, if you can write something for
					  //- Array, you can write it for List!
					  List.prototype.map = function (f) {
					  return this.cata({
					  // Do nothing, we're done!
					  Nil: () => Nil,
					  
					  // Transform head, recurse on tail
					  Cons: (head, tail) =>
					  Cons(f(head), tail.map(f))
					  })
					  }
					  
					  // e.g. Cons(1, Cons(2, Nil))
					  //      .map(x => x + 1)
					  // === Cons(2, Cons(3, Nil))
					  
					  ```
					  
					  Now, for this **convoluted example**, let’s imagine we have some weather data for the last few days:
					  
					  ```javascript
					  const arrayToList = xs => {
					  if (!xs.length) return Nil
					  
					  const [ head, ... tail ] = this
					  return Cons(head, arrayToList(tail))
					  }
					  
					  List.prototype.toArray = function () {
					  return this.cata({
					  Cons: (head, tail) => ([
					  head, ... tail.toArray()
					  ]),
					  
					  Nil: () => []
					  })
					  }
					  
					  // Starting from today... (in celsius!)
					  const temperatures = arrayToList(
					  [23, 19, 19, 18, 18, 20, 24])
					  
					  ```
					  
					  What we want to do is `map` over this list to determine whether the temperature has been warmer or cooler than the day before! To do that, we’ll probably need to do _something_ sneakily… any ideas?
					  
					  ```javascript
					  //+ List a ~> (List a -> b) -> List b
					  List.prototype.extend = function (f) {
					  return this.cata({
					  // Extend the list, repeat on tail.
					  Cons: (head, tail) => Cons(
					  f(this), tail.extend(f)
					  ),
					  
					  Nil: () => Nil // We're done!
					  })
					  }
					  
					  // [YAY, YAY, YAY, YAY, BOO, BOO, ???]
					  temperatures
					  .extend(({ head, tail }) =>
					  tail.length == 0 ? '???'
					   : head < tail.head
					     ? 'BOO' : 'YAY')
					  .toArray()
					  
					  ```
					  
					  We only used the _head_ of the tail this time, but we could use the whole thing if we wanted! We have the entire thing available to peek sneakily.
					  
					  > For example, we could use this technique for lap times to record whether they’re **faster or slower** than the **average** so far! **Have a go!**
					  
					  As we said, we can mimic the same behaviour for `Array` to save us all the to-and-fro with `List`:
					  
					  ```javascript
					  Array.prototype.extend = function (f) {
					  if (this.length === 0) return []
					  
					  const [ _, ... tail ] = this
					  return [ f(this), ... tail.extend(f) ]
					  }
					  
					  // Now, we can use array-destructuring :)
					  ;[23, 19, 19, 18, 18, 20, 24].extend(
					  ([head, ... tail]) =>
					  tail.length == 0
					  ? '???'
					  : head < tail[0]
					  ? 'BOO' : 'YAY')
					  
					  ```
					  
					  ---
					  
					  I brought these examples up for a reason, Fantasists. _Often_, `extend` isn’t just `f => W.of(f(this))` for some `W` type; that’s what `of` is for! `extend` is about being able to `map` while being aware of the surrounding **construct**.
					  
					  Think of it like this: when we used `Chain`, we had total **write** access to the **output** _constructor_ and _values_. We could turn `Just` into `Nothing`, we could fail a `Task`, and we could even change the length of an `Array`. **Truly, we were powerful**.
					  
					  With `Extend`, we get full **read** access to the **input** _constructor_ and _values_. It’s the **opposite idea**.
					  
					  Whereas `Chain` lets us **inform the future** of the computation, `Extend` lets us **be informed by the past**. _This is the kind of sentence that ends up on a mug, you know!_
					  
					  ```xl
					  -- chain: `map` with write-access to output
					  -- extend: `map` with read-access to input
					  
					  map    :: f a -> (  a ->   b) -> f b
					  chain  :: m a -> (  a -> m b) -> m b
					  extend :: w a -> (w a ->   b) -> w b
					  
					  ```
					  
					  ---
					  
					  There are lots of cool examples of `Extend`, but they are often overlooked and generally considered a more “advanced” topic. After all, with `Monad`, we’re free to build **anything**; why bother continuing? Well, I hope this gives you an idea of how they work and where you can find them! After all, these are all just **design patterns**: just use them when they’re **appropriate**!
					  
					  _So, `Semigroup` is to `Monoid` as `Chain` is to `Monad` as `Extend` is to…?_ We’ll find out next time! Before we go, though, here’s a very tricky challenge to keep you busy:
					  
					  > With `Writer`, we needed a `Semigroup` on the left to make a `Chain` instance, but we didn’t for `Extend`. `Reader` has an `Extend` instance; can you think of how we might write that?
					  
					  Until then, take a look through [this article’s code](https://gist.github.com/richdouglasevans/61eae2a787bd616f04e63a642a0dca5d), and take care!
					  
					  ♥
					  - ### Highlights
					  collapsed:: true
					  - > `Chain` takes an `m a` to an `m b` with some help from an `a -> m b` function. [⤴️](https://omnivore.app/me/fantas-eel-and-specification-16-extend-tom-harding-18da59ced72#16884eac-2eb6-4dd0-8e70-c264aaaeca65)
					  - [Fantas, Eel, and Specification 17: Comonad · Tom Harding](https://omnivore.app/me/fantas-eel-and-specification-17-comonad-tom-harding-18da59bd286)
					  collapsed:: true
					  site:: [tomharding.me](http://www.tomharding.me/2017/06/19/fantas-eel-and-specification-17/)
					  date-saved:: [[02/13/2024]]
					  date-published:: [[06/18/2017]]
					  - ### Content
					  collapsed:: true
					  - **‘Ello ‘ello**! Remember that `Monad` thing we used to be afraid of, and how it just boiled down to a way for us to **sequence** our ideas? How `Extend` was really just `Chain` backwards? Well, today, we’ll answer the question that I’m sure has plagued you _all_ week: **what _is_ a backwards `Monad`**?
					  
					  First up, we should talk about the **name**. _No_, not the `monad` bit - the `co` bit. When we talk about structures like `Monad`, we sometimes talk about the idea of the **dual structure**. Now, for our purposes, we can just think of this as, “_The same, but with all the arrows backwards_”.
					  
					  > This is a _surprisingly_ good intuition for dual structures. Seriously.
					  
					  Hey, that was our **first hint**! `Comonad` is `Monad` with “the arrows backwards”. When we **boil it down**, there are really only two **interesting** things that a `Monad` can do:
					  
					  ```xl
					  -- For any monad `m`:
					  of :: a -> m a
					  chain :: m a -> (a -> m b) -> m b
					  
					  ```
					  
					  From this, we can derive all the other fun stuff like `join`, `map`, `ap`, and whatnot. So, let’s write this all **backwards**, turning our entire types the **wrong way round**:
					  
					  ```elm
					  -- Turn the arrows around...
					  coOf :: a <- m a
					  coChain :: m a <- (a <- m b) <- m b
					  
					  -- Or, more familiarly...
					  -- For any Comonad `w`:
					  coOf :: w a -> a
					  coChain :: w a -> (w a -> b) -> w b
					  
					  ```
					  
					  Well, here’s the **good news**: we already know `coChain`, and we call it `extend`! That leaves us with that `coOf` function, which the [glorious Fantasy Land spec](https://github.com/fantasyland/fantasy-land) calls **`extract`**.
					  
					  When I first looked at `extract`, I got a bit confused. Couldn’t we do that with `Monad`? If not, what’s the _point_ in a `Monad` if we can’t get a value back _out_? What helped me was looking **a little closer** at `extract`:
					  
					  That function takes **any** `Comonad` and returns the value **inside**. We couldn’t do that for `Maybe`, because some of our values - `Nothing` \- don’t have a value to return! We couldn’t do it for `Array`; what if it’s **empty**? We couldn’t do it for `Promise`; we don’t know what the value _is_ yet! It turns out that, for a _lot_ of `Monad` types, this function **isn’t as easy** to write as we might think at first glance.
					  
					  Let’s think about `Maybe` for a second, though. Would it be a `Comonad` if we removed the `Nothing` option? Well, yes, but then it wouldn’t be a `Maybe` \- it would be `Identity` with a funny name!
					  
					  What about `Array`? What if we made a type like this:
					  
					  ```actionscript
					  //- An array with AT LEAST ONE element.
					  //+ data NonEmpty = NonEmpty a (Array a)
					  const NonEmpty = daggy.tagged(
					  'NonEmpty', ['head', 'tail']
					  )
					  
					  // Extend would function the same way as it
					  // did for Array in the last article...
					  
					  NonEmpty.prototype.extract = function () {
					  return this.head
					  }
					  
					  // e.g.
					  NonEmpty(1, [2, 3, 4]).extract() // 1
					  
					  ```
					  
					  Now we have a **type** for non-empty lists, we can **guarantee** a value to extract! This type, it transpires, forms a beautiful `Comonad`.
					  
					  > A piece of good advice is to **make illegal states unrepresentable**. If we need an array somewhere in our code that **must** have at least one element, using the `NonEmpty` type gives us an API with that **guarantee**!
					  
					  So, `chain` gave us sequencing with **write** access to the **output**, and `of` let us **lift** a value into the computation whenever we liked. `extend` gives us sequencing with **read** access to the **input**, and `extract` lets us **extract** a value out of the computation whenever we like!
					  
					  > If you’ve followed the blog series up until now, [the Comonad laws](https://github.com/fantasyland/fantasy-land\#comonad) are going to be what you’ve come to expect. **No new ideas**!
					  
					  ---
					  
					  Now, before we start to assume that all `Comonad` types are just **bastardised `Monad` types**, let’s look at something **very** comonadic: `Store`.
					  
					  ```actionscript
					  //+ data Store p s = Store (p -> s) p
					  const Store = daggy.tagged(
					  'Store', ['lookup', 'pointer']
					  )
					  
					  ```
					  
					  The intuition here is that `lookup` represents a function to get things _out_ of a “store” of `s`\-values, indexed by `p`\-values. So, if we pass a `p` to the `lookup` function, we’ll get out its corresponding `s`. The `pointer` value represents the “current” value. Think of this like the **read head** on an old **hard disk**.
					  
					  Now, to make this type more useful, we can stick a couple of functions onto this:
					  
					  ```actionscript
					  //- "Move" the current pointer.
					  //+ seek :: Store p s ~> p -> Store p s
					  Store.prototype.seek = function (p) {
					  return Store(this.lookup, p)
					  }
					  
					  //- Have a look at a particular cell.
					  //+ peek :: Store p s ~> p -> s
					  Store.prototype.peek = function (p) {
					  return this.lookup(p)
					  }
					  
					  ```
					  
					  And, wouldn’t you know it, we can also make this a functor by [mapping over the **function**](http://www.tomharding.me/2017/04/15/functions-as-functors/)!
					  
					  ```reasonml
					  //- Compose the functions! Yay!
					  Store.prototype.map = function (f) {
					  const { lookup, pointer } = this
					  
					  return Store(lookup.map(f), pointer)
					  // Store(p => f(lookup(p)), pointer)
					  }
					  
					  ```
					  
					  Now, if we’re going to make a `Comonad` of our `Store`, we first need to make it an `Extend` instance. Remember: `extend` should behave like `map`, but with **read-access to the input**. Here’s where `Store` gets _really_ sneaky.
					  
					  ```javascript
					  //+ extend :: Store p s ~> (Store p s -> t)
					  //+                     -> Store p t
					  Store.prototype.extend = function (f) {
					  return Store(
					  p => f(Store(this.lookup, p)),
					  this.pointer)
					  }
					  
					  ```
					  
					  Our `lookup` function now applies `f` to a `Store` **identical** to the original, but with the focus on the **given index**! Can you see the magic yet? Let’s **build something exciting**: [Conway’s **Game of Life**](https://en.wikipedia.org/wiki/Conway%27s%5FGame%5Fof%5FLife).
					  
					  ---
					  
					  For this, we’re going to use a “board” of `[[Bool]]` type to represent our “live” and “dead” cells. Something like this, perhaps:
					  
					  ```yaml
					  let start = [
					  [ true,  true,  false, false ],
					  [ true,  false, true,  false ],
					  [ true,  false, false, true  ],
					  [ true,  false, true,  false ]
					  ]
					  
					  ```
					  
					  If we want to look up a value in this store, we’re going to need an `x` and `y` coordinate. What better choice of structure to hold two numbers than a `Pair`?
					  
					  ```gml
					  const Game = Store(
					  ({ _1: x, _2: y }) =>
					  // Return the cell OR false.
					  y in start && x in start[y]
					  ? start[y][x]
					  : false,
					  
					  // We don't care about `pointer` yet.
					  Pair(0, 0))
					  
					  ```
					  
					  Now, we need to write out some logic! The rule for the Game of Life is that, if a `false` cell has **exactly three** `true` neighbours, make it true. If a `true` cell has **two or three** `true` neighbours, keep it as true. If **neither** apply, make it `false`. We can work this out for any cell with eight sneaky `peek`s!
					  
					  ```gml
					  // What will the current cell be next time?
					  //+ isSurvivor :: Store (Pair Int Int) Bool
					  //+            -> Bool
					  const isSurvivor = store => {
					  const { _1: x, _2: y } = store.pointer
					  
					  // The number of `true` neighbours.
					  const neighbours =
					  [ Pair(x - 1, y - 1) // NW
					  , Pair(x, y - 1)     // N
					  , Pair(x + 1, y - 1) // NE
					  
					  , Pair(x - 1, y)     // W
					  , Pair(x + 1, y)     // E
					  
					  , Pair(x - 1, y + 1) // SW
					  , Pair(x, y + 1)     // S
					  , Pair(x + 1, y + 1) // SE
					  ]
					  .map(x => store.peek(x)) // Look up!
					  .filter(x => x) // Ignore false cells
					  .length
					  
					  // Exercise: simplify this.
					  return store.extract() // Is it true?
					  ? neighbours === 2 || neighbours === 3
					  : neighbours === 2
					  }
					  
					  ```
					  
					  Now, _why_ did we go to all this trouble? Well, we now have a `Store (Int, Int) Bool` to `Bool` function, which is the exact shape that `extend` needs… and `extend` will (lazily!) apply this function to **every cell on the board!** By using `extend`, we now get to see the **entire board** one step into **the future**. Isn’t that _magical_?
					  
					  > I _strongly_ recommend you look at [the Gist for this article](https://gist.github.com/richdouglasevans/0f9a57e5a52b13e93c0c03630165ecd8) and be sure that this makes sense. `Store` is an **unfamiliar beast**.
					  
					  ---
					  
					  Now, there are plenty of other `Comonad` types, but they’re not quite as popular as `Monad` types, probably because their use isn’t so **obvious**. After all, we can write our applications just using `Monad` types, so this (_unfairly_) ends up in the _advanced_ box. How **rude**!
					  
					  For now, however, we’ll stop here. I will come back to `Comonad` in other posts - they’re my latest **obsession** \- but `Store` gives a really clear idea about why these are useful. Incidentally, if you want to play the Game of Life, [the article’s Gist](https://gist.github.com/richdouglasevans/0f9a57e5a52b13e93c0c03630165ecd8) has a working demo!
					  
					  Next time, we’ll be looking at `Bifunctor` and `Profunctor`: so simple, we’re going to do both at the same time! I promise: these last two are going to be a bit of a **cool-down session**. Until then!
					  
					  ♥
					  - [strawman:concurrency [ES Wiki]](https://omnivore.app/me/strawman-concurrency-es-wiki-18d971e3b8d)
					  collapsed:: true
					  site:: [web.archive.org](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman%253Aconcurrency)
					  date-saved:: [[02/11/2024]]
					  date-published:: [[06/16/2013]]
					  - ### Content
					  collapsed:: true
					  - The Wayback Machine - https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
					  
					  * [Communicating Event-Loop Concurrency and Distribution](\#communicating%5Fevent-loop%5Fconcurrency%5Fand%5Fdistribution)  
					  * [Concurrency Model and Promises](\#concurrency%5Fmodel%5Fand%5Fpromises)  
					  * [Vats](\#vats)  
					  * [Promises and Promise States](\#promises%5Fand%5Fpromise%5Fstates)  
					  * [Eventual Operations](\#eventual%5Foperations)  
					  * [Fundamental Static Q Methods](\#fundamental%5Fstatic%5Fq%5Fmethods)  
					  * [Promise methods](\#promise%5Fmethods)  
					  * [Syntactic Sugar](\#syntactic%5Fsugar)  
					  * [Non-fundamental Static Q Conveniences](\#non-fundamental%5Fstatic%5Fq%5Fconveniences)  
					  * [Q.delay](\#q.delay)  
					  * [Q.race](\#q.race)  
					  * [Q.all](\#q.all)  
					  * [Q.join](\#q.join)  
					  * [Q.memoize](\#q.memoize)  
					  * [Q.async](\#q.async)  
					  * [Q.defer()](\#q.defer)
					  * [Examples](\#examples)  
					  * [Infinite Queue](\#infinite%5Fqueue)  
					  * [Spawn](\#spawn)  
					  * [Vat.evalLater() as Async-PGAS](\#vat.evallater%5Fas%5Fasync-pgas)  
					  * [Open Vat](\#open%5Fvat)  
					  * [there](\#there)  
					  * [Map-Reduce Lite](\#map-reduce%5Flite)  
					  * [AMD Loader Lite](\#amd%5Floader%5Flite)
					  * [See](\#see)
					  
					  \#\# Communicating Event-Loop Concurrency and Distribution
					  
					  On both the browser and the server, JavaScript’s de-facto concurrency model is increasingly “shared nothing” communicating event loops. JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage. JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets. And server-side JavaScript has a rapidly growing role as the counterparty of these protocols, and increasingly uses event loops to service them. 
					  
					  This strawman consists of several major parts, not all of which need be accepted together.
					  
					  1. **Reality:** Codifying and formalizing JavaScript’s de-facto concurrency model as a de-jure model.
					  2. **Promises:** A way to  
					  * (**Q(p).post(), Q(p).get()**) Make asynchronous requests of objects that may not be synchronously reachable, such as remote objects.  
					  * (**Q(p).then()**) Ease the burden of local event loop programming, by reifying the ability to register a callback as a first class value.  
					  * (**[Q.async, yield](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:async%5Ffunctions "strawman:async_functions"):**) for implicit registration of shallow continuations on promises.
					  3. **Syntactic sugar**  
					  * **The infix “`!`” operator:** An eventual analog of “`.`“, for making eventual requests look more like immediate requests.
					  4. (**Q.makeFar()** and **Q.makeRemote()**) A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.  
					  * **Transport independence:** Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports.
					  5. (**Vat()**) An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.  
					  * **Worker independence:** Using `Vat`  
					  API  
					  as an abstraction layer around worker spawning on the browser or process spawning on the server.
					  6. (**Vat.evalLater(), there()**) Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops  
					  * **Symmetric Mobile Code:** Generalizes from the current use of JavaScript as mobile code sent only from server and only to browsers.
					  
					  \#\# Concurrency Model and Promises
					  
					  Aggregate objects into process-like units called _vats_. Objects in one vat can only send asynchronous messages to objects in other vats. _Promises_ represent such references to potentially remote objects. _Eventual message sends_ queue _eventual-deliveries_ in the work queue of the vat hosting the target object. A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a _turn_. A _then expression_ registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s _resolution_. The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
					  
					  This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking. The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like `alert`). Without blocking, conventional deadlock is impossible. Of course, less conventional forms of race condition and deadlock bugs [remain](https://web.archive.org/web/20160404122250/http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html "http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html").
					  
					  \#\# Vats
					  
					  Partition the JavaScript reference graph into separate units, corresponding to prior concepts variously called vats, workers, processes, tanks, or grains. We adopt the “_vat_” terminology here for expository purposes. Vats are only asynchronously coupled to each other, and represent the minimal possible unit of concurrency, transparent distribution, orthogonal persistence, migration, partial failure, resource control, preemptive termination/deallocation, and defense from denial of service. Each vat consists of 
					  
					  * a single sequential thread of control,
					  * a single call-return stack,
					  * a single fifo queue holding _eventual-deliveries_,
					  * an internal object heap,
					  * and incoming and outgoing _remote references_.
					  
					  A vat’s thread of control dequeues the next eventual-delivery from the queue and processes it to completion before proceeding to the next. When the queue is empty, the vat is idle. 
					  
					  const vat = Vat(); //makes a new vat, as an object local to the creating vat.
					  // A Vat has an ''evalLater'' method that evaluates a Program in a turn of that vat.
					  // The ''evalLater'' method returns a promise for the evaluation's completion value.
					  const funP = vat.evalLater('' + function fun(x, y) { return x + y; }); // see below
					  const sumP = funP ! (3, 5); // sumP will eventually resolve to 8, unless...
					  const doneP = vat.terminateAsap(new Error('die')); // that vat is terminated before ''sumP'' is resolved.
					  // If the vat is terminated first, then ''sumP'' resolves to a //rejected// problem, with
					  // (Error: die) as its alleged reason for rejection.
					  // Once the vat is terminated, ''doneP'' will eventually resolve to ''true''.
					  
					  The vat object that represents a new vat is local to the creating vat, so that a vat may be terminated without waiting for that vat’s eventual-delivery queue to drain. 
					  
					  The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole. We intend that WebWorkers can be implemented in terms of vats and vice versa. However, when vats are built on WebWorkers, in the absence of some kind of weak reference and gc notification mechanism, it is probably impossible to arrange for the collection of distributed garbage. Even with them, [much more](https://web.archive.org/web/20160404122250/http://erights.org/history/original-e/dgc/index.html "http://erights.org/history/original-e/dgc/index.html") is needed to enable collection of distributed cyclic garbage. On the other hand, when vats are provided more primitively, multiple vats within an address space can be jointly within the purview of a single concurrent garbage collector, enabling full gc among these co-resident vats. However, truly distributed vats would still be faced with these same distributed garbage collection worries.
					  
					  The “` '' + function... `” trick above depends on [function\_to\_string](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:function%5Fto%5Fstring "harmony:function_to_string") to actually pass a string which is the program source for the function, while nevertheless having the function itself appear in the spawning program as code rather than as a literal string. This helps IDEs, refactoring tools, etc. A vat’s `evalLater` method evaluates that string as a program in a safe scope – a scope containing only the standard global variables such as `Object`, `Array`, etc. Except for these, the source passed in should be _closed_ – should not contain free references to any other variables. If the function is closed but for these standard globals, and these standard globals are not shadowed or replaced in the spawning context, then an IDE’s scope analysis of the code remains accurate.
					  
					  \#\# Promises and Promise States
					  
					  We introduce a new opaque type of object, the _Promise_ to represent potentially remote references. A normal JavaScript direct reference may only designate an object within the same vat. Only promises may designate objects in other vats. A promise may be in one of several states:
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/0x0,s3MtXvPSlVBOuVroCwRCi7mcdELJGNC1uEMf9KMjOjhE/https://web.archive.org/web/20160404122250im_/http://wiki.ecmascript.org/lib/exe/fetch.php?w=&h=&cache=cache&media=strawman:refstates3.png)](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/lib/exe/detail.php?id=strawman%3Aconcurrency&cache=cache&media=strawman:refstates3.png "strawman:refstates3.png") (TODO: Revise diagram to replace “unresolved” with “pending” and “broken” with “rejected”.) 
					  
					  * _pending_ – when it is not yet determined what object the promise designates,  
					  * _pending-local_ – when the right to determine what the promise designates resides in the same vat,  
					  * _pending-remote_ – when that right is either in flight between vats or resides in a remote vat,
					  * _fulfilled_ – resolved to successfully designate some object,  
					  * _near_ – resolved to a direct reference to a local object,  
					  * _far_ – resolved to designate a remote object,
					  * _rejected_ – will never designate an object, for an alleged reason represented by an associated error.
					  
					  A promise may transition from _pending_ to any state. Additionally a promise can transition from _far_ to _rejected_. A _resolved_ promise can designate any non-promise value including primitives, `null`, and `undefined`. Primitives, `null`, `undefined`, and some objects are pass-by-copy. All other objects are pass-by-reference. A promise resolved to designate a pass-by-copy value is always near, i.e., it always designates a local copy of the value.
					  
					  If a function `foo` immediately returns either `X` or a promise which it later fulfills with `X`, we say that `foo` **_reveals_** `X`. Unless stated otherwise, we implicitly elide the error conditions from such statements. For the more explicit statement, append: _“or it throws, or it does not terminate, or it rejects the returned promise, or it never resolves the returned promise.”_ Put another way, such a function returns a **_reference_** to `X`, where by _reference_ we mean either `X` or a promise for `X`.
					  
					  \#\# Eventual Operations
					  
					  The existing JavaScript infix `.` (dot or _now_) operator enables synchronous interaction with the local object designated by a direct reference. We introduce a corresponding infix `!` (bang or _eventually_) operator for corresponding asynchronous interaction with objects eventually designated by either direct references or promises.
					  
					  Abstract Syntax: 
					  
					  Expression : ...
					  Expression ! [ Expression ] Arguments    // eventual send
					  Expression ! Arguments                   // eventual call
					  Expression ! [ Expression ]              // eventual get
					  Expression ! [ Expression ] = Expression // eventual put
					  delete Expression ! [ Expression ]       // eventual delete
					  
					  The `...` means “and all the normal right hand sides of this production. By “abstract” here I mean the distinction that must be preserved by parsing, i.e., in an [ast](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:ast "strawman:ast"), but without explaining the precedence and associativity which explains how this is unambiguously parsed. In all cases, the eventual form of an expression queues a eventual-delivery recording the need to perform the corresponding immediate form in the vat hosting the (eventually) designated object. The eventual form immediately evaluates to a promise for the result of eventually performing this eventual-delivery.
					  
					  function add(x, y) { return x + y; }
					  const sumP = add ! (3, 5); //sumP resolves in a later turn to 8.
					  
					  Attempted Concrete Syntax: 
					  
					  MemberExpression : ...
					  MemberExpression [nlth] ! [ Expression ]
					  MemberExpression [nlth] ! IdentifierName
					  CallExpression : ...
					  CallExpression [nlth] ! [ Expression ] Arguments
					  CallExpression [nlth] ! IdentifierName Arguments
					  MemberExpression [nlth] ! Arguments
					  CallExpression [nlth] ! Arguments
					  CallExpression [nlth] ! [ Expression ]
					  CallExpression [nlth] ! IdentifierName
					  UnaryExpression : ...
					  delete CallExpression [nlth] ! [ Expression ]
					  delete CallExpression [nlth] ! IdentifierName
					  LeftHandSideExpression :
					  Identifier
					  CallExpression [ Expression ]
					  CallExpression . IdentifierName
					  CallExpression [nlth] ! [ Expression ]
					  CallExpression [nlth] ! IdentifierName
					  
					  “`[nlth]`” above is short for “`[No LineTerminator here]`“, in order to unambiguously distinguish infix from prefix bang in the face of automatic semicolon insertion. 
					  
					  \#\# Fundamental Static Q Methods
					  
					  | Q(target) \-> targetP                         | Lifts the target argument into a promise designating the same object. If target is already a promise, then that promise is returned. (A promise for promise for T simplifies into a promise for T. Category theorists will be more pleased than Type theorists ;).)                                |
					  | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
					  | Q.reject(reason) \-> rejectedP                | Makes and returns a fresh _rejected_ promise recording (a sanitized form of) reason as the alleged reason for rejection. reason should generally be an immutable pass-by-copy Error object.                                                                                                        |
					  | Q.promise(f(resolve,reject)\->()) \-> promise | Makes a fresh promise, where the promise is initially _pending-local_, and the argument function f is called with resolve and reject functions for resolving this promise.                                                                                                                         |
					  | Q.isPromise(target) \-> boolean               | Is target a promise? If not, then using target as a target in the various promise operations is still equivalent to using Q(target), i.e., the promise operations will automatically lift all values to promises.                                                                                  |
					  | Q.makeFar(handler, nextSlotP) \-> promise     | Makes a resolved _far_ reference, which can only [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to _rejected_.                                                                          |
					  | Q.makeRemote(handler, nextSlotP) \-> promise  | Makes an _pending-remote_ promise, which can [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to anything.                                                                                |
					  | Q.ahorten(target1) \-> target2                | Returns the currently most resolved form of target1\. If target1 is a _fulfilled_ promise, return its resolution. If target1 is a promise that is following promise target2, then return target2. If target1 is a terminal _pending_ or _rejected_ promise, or a non-promise, then return target1. |
					  
					  Additional non-fundamental static Q convenience methods appear below.
					  
					  \#\# Promise methods
					  
					  Assuming `p` is a promise 
					  
					  | p.get(name) \-> valueP                    | Returns a promise for the result of eventually getting the value of the name property of target.                                                                                                                                               |
					  | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
					  | p.post(opt\_name, args) \-> resultP       | Eventually invoke the named method of target with these args. Returns a promise for what the result will be.                                                                                                                                   |
					  | p.send(opt\_name, ...args) \-> resultP    | p.send(m, a, b) is equivalent to p.post(m, \[a,b\])                                                                                                                                                                                            |
					  | p.fcall(...args) \-> resultP              | p.fcall(a, b) is equivalent to p.post(void 0, \[a,b\])                                                                                                                                                                                         |
					  | p.put(name, value) \-> voidP              | Eventually set the value of the name property of target to value. Return a promise-for-undefined, used for indicating completion.                                                                                                              |
					  | p.delete(name) \-> trueP                  | Eventually delete the name property of target. Returns a promise for the boolean result.                                                                                                                                                       |
					  | p.then(success, opt\_failure) \-> resultP | Registers functions success and opt\_failure to be called back in a later turn once target is _resolved_. If _fulfilled_, call success(resolution). Else if _rejected_, call opt\_failure(reason). Return a promise for the callback’s result. |
					  | p.end()                                   | If p resolves to _rejected_, log the reason to wherever uncaught exceptions go on this platform, e.g., onerror(reason).                                                                                                                        |
					  
					  
					  \#\# Syntactic Sugar
					  
					  | Abstract Syntax  | Expansion          | Simple Case  | Expansion            | JSON/RESTful equiv        |
					  | ---------------- | ------------------ | ------------ | -------------------- | ------------------------- |
					  | x ! \[i\](y, z)  | Q(x).send(i, y, z) | x ! p(y, z)  | Q(x).send(’p’, y, z) | POST https://...q=p {...} |
					  | x ! (y, z)       | Q(x).fcall(y, z)   | \-           | \-                   | POST https://... {...}    |
					  | x ! \[i\]        | Q(x).get(i)        | x ! p        | Q(x).get(’p’)        | GET https://...q=p        |
					  | x ! \[i\] = v    | Q(x).put(i, v)     | x ! p = v    | Q(x).put(’p’, v)     | PUT https://...q=p {...}  |
					  | delete x ! \[i\] | Q(x).delete(i)     | delete x ! p | Q(x).delete(’p’)     | DELETE https://...q=p     |
					  
					  
					  \#\# Non-fundamental Static Q Conveniences
					  
					  \#\#\# Q.delay
					  
					  Reveal the `answer` sometime after `millis` milliseconds have elapsed.
					  
					  Q.delay = function(millis, answer = undefined) {
					  return Q.promise(resolve => {
					  setTimeout(() => resolve(answer), millis);
					  });
					  };
					  
					  \#\#\# Q.race
					  
					  Given a list of promises, returns a promise for the resolution of whichever promise we notice has completed first.
					  
					  Q.race = function(answerPs) {
					  return Q.promise((resolve,reject) => {
					  for (answerP of answerPs) {
					  Q(answerP).then(resolve,reject);
					  };
					  });
					  };
					  
					  We can compose `Q.race`, `Q.delay`, and `Q.reject` to timeout eventual requests.
					  
					  var answer = Q.race([bob ! foo(carol), 
					       Q.delay(5000, Q.reject(new Error("timeout")))]);
					  
					  \#\#\# Q.all
					  
					  Often it’s useful to collect several promised answers, in order to react either when _all_ the answers are ready or when _any_ of the promises becomes _rejected_.
					  
					  Q.all = function(answerPs) {
					  let countDown = answerPs.length;
					  const answers = [];
					  if (countDown === 0) { return Q(answers); }
					  return Q.promise((resolve,reject) => {
					  answerPs.forEach((answerP, index) => {
					  Q(answerP).then(answer => {
					  answers[index] = answer;
					  if (--countDown === 0) { resolve(answers); }
					  }, reject);
					  });
					  });
					  };
					  
					  We can compose `Q.all`, `then`, and [destructuring](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:destructuring "harmony:destructuring") to delay until several operands are revealed
					  
					  var sumP = Q.all([xP, yP]).then(([x, y]) => (x + y);
					  
					  \#\#\# Q.join
					  
					  Join is our eventual equality operation. Any messages sent on the join of `xP` and `yP` are only delivered if `xP` and `yP` both reveal the same target, in which case these messages are eventually delivered to that target and this joined promise itself eventually becomes fulfilled to designate that target. Otherwise, all these messages are discarded with the usual rejected promise contagion.
					  
					  Q.join = function(xP, yP) {
					  return Q.all([xP, yP]).then(([x, y]) => {
					  if (Object.is(x, y)) {
					  return x;
					  } else {
					  throw new Error("not the same");
					  }
					  });
					  };
					  
					  \#\#\# Q.memoize
					  
					  `Q.memoize` of a one argument function returns a new similar one argument function, except that it (eventually) calls the original function no more than once for each such argument. The memo function immediately returns a promise for what the original function will eventually return. Equivalence of arguments is determined by the optional memoMap passed in, which defaults to a new WeakMap() if absent. (Passing a memoMap in also allows the caller to seed the mapping with some prior associations.)
					  
					  The difference from a traditional synchronous memoizer is that the original function is called _eventually_ after a promise for its result is _already_ memoized, enabling cycles to work. For example, if memoF === memoize(f) and f(x) calls memoF(x), then an outer call to memoF(x) schedules an eventual call to f(x) which makes an inner call to memoF(x). Both outer and inner calls to memoF(x) returns a promise for what f(x) will eventually return.
					  
					  Q.memoize = function(oneArgFuncP, memoMap = WeakMap()) {
					  
					  return function oneArgMemo(arg) {
					  if (memoMap.has(arg)) {
					  return memoMap.get(arg);
					  } else {
					  const resultP = oneArgFuncP ! (arg);
					  memoMap.set(arg, resultP);
					  return resultP;
					  }
					  }
					  };
					  
					  \#\#\# Q.async
					  
					  \#\#\# Q.defer()
					  
					  (Will likely be deprecated)
					  
					  Q.defer = function() {
					  const deferred = {};
					  deferred.promise = Q.promise((resolve,reject) => {
					  deferred.resolve = resolve;
					  deferred.reject = reject;
					  });
					  return deferred;
					  };
					  
					  \#\# Examples
					  
					  \#\# Infinite Queue
					  
					  function makeQueue() {
					  let rear;
					  let front = Q.promise(r => { rear = r; });
					  return Object.freeze({
					  enqueue: function(elem) {
					  let nextRear;
					  const nextTail = Q.promise(r => { nextRear = r; });
					  rear({head: elem, tail: nextTail});
					  rear = nextRear;
					  },
					  dequeue: function() {
					  const result = front ! head;
					  front = front ! tail;
					  return result;
					  }
					  });
					  }
					  
					  Calling `queue.dequeue()` will return a promise for the next element that has or will be enqueued.
					  
					  \#\# Spawn
					  
					  The following `spawn` function is a simple abstraction built on `Vat` and `then` that captures a simple common case:
					  
					  function spawn(src) {
					  const vat = Vat();
					  const resultP = vat.evalLater(src);
					  Q(resultP).then(function(_) {
					  vat.terminateAsap(new Error('done')); 
					  });
					  return resultP;
					  }
					  
					  const sumP = spawn('3+5'}); // sumP eventually resolves to 8.
					  
					  The argument string to `spawn` is evaluated in a new Vat spawned for that purpose. Spawn returns a promise for what that string will evaluate to. Once that promise resolves, the spawned vat is shut down.
					  
					  \#\# Vat.evalLater() as Async-PGAS
					  
					  In the Async-PGAS language X10, the “at” statement is defined as
					  
					  // x10 grammar, not javascript or proposed javascript
					  Statement :
					  at ( PlaceExpression ) Statement
					  
					  The “at” statement first evaluates the PlaceExpression to a place, which is analogous to a vat. It then evaluates the Statement at that place. The statement evaluates with the lexical scope containing the “at” statement, so the locality of the values bound to these identifiers is the locality they have at that place rather than at the location containing the “at” statement. Although the argument to `Vat.evalLater` must be a closed expression (modulo whitelisted globals), we can get the same effect, a bit more verbosely, by passing in these bindings as an explicit eventual call to a closed function.
					  
					  For example, the X10-ish program
					  
					  const x = 6;
					  let ultimateP;
					  at (place) { ultimateP = x*7; }
					  
					  can be expressed using `aVat.evalLater()` as
					  
					  const x = 6;
					  let ultimateP = place.evalLater(x => x*7) ! (x);
					  
					  \#\# Open Vat
					  
					  The `makeOpenVat` function makes an `OpenVat` function. Each `OpenVat` function is like the `Vat` function, in that both make an return a new vat instance. Each `OpenVat` function additionally maintains a side table mapping from all incoming remote promises to the evaluation function of the open vat made by this `OpenVat` function. The reason we call such vats _open_ is that, given a remote promise into such a vat and the `OpenVat` function that made that vat, one can thereby inject new code into that vat.
					  
					  TODO: explain the membrane variation used below.
					  
					  function makeOpenVat() {
					  const wm = WeakMap();
					  
					  function OpenVat() {
					  const vat = Vat();
					  const openVat = Object.freeze({
					  evalLater: makeMembraneX(
					  vat.evalLater,
					  { registerRemote: function(remote) {
					  wm.set(remote, openVat.evalLater); }}),
					  terminateAsap: vat.terminateAsap
					  });
					  return openVat;
					  }
					  OpenVat.evalAt = function(p, src) {
					  return wm.get(p)(src);
					  };
					  return OpenVat;
					  }
					  
					  \#\# there
					  
					  The `there(p, ...)` function is analogous to `Q(p).then(...)`, except instead of merely relocating the execution of the callback in time till after `p` is resolved, it further relocates it in spacetime, to where and when p is near. Like `Q(p).then(...)`, `there` immediately returns a promise for the eventual outcome. We do not likewise relocate the errback, so that we can still notify it and it can still react on the requesting side to a partition between the requestor and `p`‘s host.
					  
					  function there(p, callback, opt_errback) {
					  var callbackP = OpenVat.evalAt(p, '' + callback);
					  return (callbackP ! (p)).then(
					  v => v,
					  opt_errback);
					  }
					  
					  \#\# Map-Reduce Lite
					  
					  Given an initial result value, a list of potentially remote promises to elements, a closed mobile `mapper` function from elements to mapped result values, and an associative / commutative `reducer` function from pairs of references to result values to a new reference to a result value, `mapReduce` immediately returns a promise for the result of mapping all the elements where they live, and reducing all of these results together with the initial result value to a result.
					  
					  I call this “Map-Reduce Lite” because, unlike a production map-reduce infrastructure, the following `mapReduce` does all reductions on the spawning machine, which is therefore a scaling bottleneck, and has none of the fault-tolerance. Here, any failure causes the overall map-reduce to fail, i.e., the returned promise becomes a _rejected_ promise. The mapper and reduction functions are like the conventional functional programming variety, rather than the map-reduce variety which arranged for group-by keys.
					  
					  /**
					  * Type/Guard syntax below is only a placeholder, not a serious proposal.
					  * @param initValue ::T2
					  * @param elemPs    ::Array[Ref[T1]]  // i.e., Array[T1 | Promise[T1]]
					  * @param mapper    ::(T1 -> T2)      // closed mobile function
					  * @param reducer   ::(T2 x T2 -> T2)
					  * @reveals         ::T2              // i.e., @returns ::Ref[T2]
					  */
					  function mapReduce(initValue, elemPs, mapper, reducer) {
					  let countDown = elemPs.length;
					  if (countDown === 0) { return initValue; }
					  let result = initValue;
					  
					  return Q.promise((resolve, reject) => {
					  elemPs.forEach(elemP => {
					  const mappedP = there(elemP, mapper);
					  Q(mappedP).then(mapped => {
					  result = reducer(result, mapped);
					  if (--countDown === 0) { resolve(result); }
					  }, reject);
					  });
					  });
					  }
					  
					  \#\# AMD Loader Lite
					  
					  This is a minimal Asynchronous Module Definition (AMD) Loader for a subset of the [AMD API](https://web.archive.org/web/20160404122250/https://github.com/amdjs/amdjs-api/wiki/AMD "https://github.com/amdjs/amdjs-api/wiki/AMD") specification. In this subset, `define` is called with exactly two arguments, a `dependencies` list of module names, and a factory function with one parameter per dependency.
					  
					  function makeSimpleAMDLoader(fetch, moduleMap = Map()) {
					  var loader;
					  
					  function rawLoad(id) {
					  return Q(fetch(id)).then(src => {
					  var result = Q.reject(new Error('"define" not called by: ' + id));
					  function define(deps, factory) {
					  result = Q.all(deps.map(loader)).then(imports => {
					  return factory(...imports);
					  });
					  }
					  define.amd = {lite: true};
					  
					  Function('define', src)(define);
					  return result;
					  });
					  }
					  return loader = Q.memoize(rawLoad, moduleMap);
					  }
					  
					  If module “W” depends on modules “X”, “Y”, and “Z”, then only once the promises for the “X”, “Y”, and “Z” modules have all been fulfilled will the “W” factory function be called with these module instances. The result of calling this factory function will then become the “W” module instance.
					  
					  What it means to _be_ the source for the “W” module is that `fetch(”W”)` will eventually return that source string. For example, a given `fetch` function might fetch it from `https://example.com/prefix/W.js`.
					  
					  // At https://example.com/prefix/W.js
					  define(['X', 'Y', 'Z'], function(X, Y, Z) {
					  return X(Y, Z);
					  })
					  
					  Since the memo mapping we need is from module names, which are strings rather than objects, we need to provide an explicit memoMap argument to `Q.memoize`, which should be a map that accepts strings as keys.
					  
					  Although the cycle tolerance of `Q.memoize` is generally useful, here it hurts. Because `define` won’t call the factory function until all (`Q.all`) of the dependencies are fulfilled, any cyclic AMD dependencies cause an undetected deadlock. Still, in the naive absence of this cycle tolerance, such cyclic dependencies would have instead caused an infinite regress which is even worse. Better would be cycle detection and rejection, which we leave as an exercise for the reader. 
					  
					  \#\# See
					  - [Elite-Only Financial Markets - by Robin Hanson](https://omnivore.app/me/elite-only-financial-markets-by-robin-hanson-18d8327e902)
					  collapsed:: true
					  site:: [substack.com](https://substack.com/inbox/post/141238258)
					  author:: Substack
					  labels:: [[robin-hanson]] [[prediction-markets]]
					  date-saved:: [[02/07/2024]]
					  - ### Content
					  collapsed:: true
					  - Prediction markets are financial markets, but compared to typical financial markets they are intended more to aggregate info than to hedge risks. Thus we can use our general understanding of financial markets to understand prediction markets, and can also try to apply whatever we learn about prediction markets to financial markets more generally.
					  
					  With this in mind, consider the newly published paper _[Crowd prediction systems: Markets, polls, and elite forecasters](https://www.sciencedirect.com/science/article/abs/pii/S0169207023001413)_, by Atanasov, Witkowski, Mellers, and Tetlock.
					  
					  They use data from 1300 forecasters on 147 questions over four years from the Good Judgement Project’s entry in the IARPA ACE tournament, 2011-2015\. (I was part of another [team](https://www.overcomingbias.com/p/join-gmus-daggre-teamhtml) in that tournament.) They judge outcomes by averaging quadratic Brier-score accuracy over questions and time. 
					  
					  They find that:
					  
					  1. Participants who used my logarithmic market scoring rule (LMSR) mechanism did better than those using a continuous double auction market (mainly better when few traders), and did about as well as those using a complex poll aggregation mechanism, which did better than simpler polling aggregation methods.
					  2. One element of the complex polling mechanism, an “extremization” power-law transformation of probabilities, also makes market prices more accurate.
					  3. Participants who were put together into teams did better than those who were not.
					  4. Accuracy is _much_ (\~14-18%) better if you take the 2% of participants most accurate in a year, and then using only these “elites” in future years. It didn’t matter which mechanism was used when selecting that 2% elite.
					  
					  The authors see this last result as their most important:
					  
					  > The practical question we set to address focused on a manager who seeks to maximize forecasting performance in a crowdsourcing environment through her choices about forecasting systems and crowds. Our investigation points to specific recommendations. …Our results offer a clear recommendation for improving accuracy: employ smaller, elite crowds. These findings are relevant to corporate forecasting tournaments as well as to the growing research literature on public forecasting tournaments. Whether the prediction system is an LMSR market or prediction polls, managers could improve performance by selecting a smaller, elite crowd based on prior performance in the competition. Small, elite forecaster crowds may yield benefits beyond accuracy. For example, when forecasts use proprietary data or relate to confidential outcomes, employing a smaller group of forecasters may help minimize information leakage. 
					  
					  This makes sense for a manager who plans to ask \~>1300 participants \~>150 questions over \~>4 years, and who trusts some subordinate to judge how exactly to select this elite, and how to set the complex polling parameters, if they use a polling mechanism. But I’ve been mainly interested in using prediction markets as public institutions for cases where there’s a lot of distrust re motives and rationality. Such as in law, governance, policy, academia, and science. And in such contexts, I worry a lot more about the discretionary powers required to implement an elite selection system.
					  
					  To see my concern, consider stock markets, whose main social function is to channel investment into the most valuable opportunities. More accurate stock prices better achieve this function, and the above results suggest that we’d get much more accurate stock prices by greatly limiting who can speculate in stock markets. Hold some contests where applicants compete with fake trades to grow small initial trading budgets, and only let the top, say, 2% of such such contestants make speculative price-influencing trades in real stock markets. Maybe also force them to join teams, instead of trading individually. (Forcing extremization seems unnecessary, as specialists can profit by making those price adjustments.) (Note: these are my suggestions; study authors didn’t discuss this.) 
					  
					  Yes, stock market speculators today are already far from randomly selected from the general population, and are thus already “elite” in that sense. Even so, while the 1300 forecasters in the above study were far random samples of the public, only letting the top 2% of them participate was a win in the above study. Thus only letting the top 2% of wannabe stock speculators trade in real stock markets is plausibly also a win for stock market price accuracy. (Note: the study authors admit they choose the 2% figure somewhat arbitrarily, so a wide range of selectivity, maybe 0.5% to 10%, might work about as well.)
					  
					  To prevent others from speculating, we might force hedging trades to be made via long-delay call markets, which [pushes](https://www.overcomingbias.com/p/are-financial-markets-too-short-term) out speculators with shorter-fuse info, or via regulators verifying their legit hedging needs (e.g, regular paycheck deposit, withdraw for retirement, or big medical expense). And we might insist that hedgers focus on general index funds unless they can show reasons to trade more specific assets. 
					  
					  One problem with this is that most of profits made by winning speculators today come from the losses of less elite speculators, not from hedge traders. And if we could better segregate hedge traders into call markets, elite speculator profits would be even smaller. Thus unless we could subsidize elite-only stock markets, perhaps via automated market makers, elite speculators would be fighting over a much smaller pool of profits than today, which would likely cut into price accuracy. 
					  
					  Another problem is that it would be hard to prevent speculator contestants from privately buying winning trades, turning this elite selection process into more of a monetary auction. Other professional credentialing processes today use schools and tests, where it is harder to just buy success.
					  
					  But even if we can prevent contest cheating, and subsidize elite stock market speculation, I fear corruption of elite speculator certification. That is, the official organization in charge of deciding who qualifies as elite speculators may succumb to pressures to favor some groups, and to be overly restrictive to favor insiders. And once you imagine official consensus on legal, policy, or science questions being set by financial markets prices, you can imagine all the more possible pressures to control who is allowed to influence such prices.
					  
					  For now, I recommend that robust public institutions built using financial markets let as many parties as possible trade in them, even foreigners. But yes, I remain open to the possibility that we could eventually learn well enough how to usefully constrain participation, and to prevent special interests from capturing selection powers. And if I were running a large set of markets for some private owner, I’d be more open to constraining participation to speculative elites.
					  - [Why web3 matters: Harnessing DePIN and ReFi as a Force for Good | by Mark C. Ballandies | WiHi — Weather | Feb, 2024 | Medium](https://omnivore.app/me/why-web-3-matters-harnessing-de-pin-and-re-fi-as-a-force-for-goo-18d82457b06)
					  collapsed:: true
					  site:: [WiHi — Weather](https://medium.com/wihi-weather/why-web3-matters-harnessing-depin-and-refi-as-a-force-for-good-e693be80fff3)
					  author:: Mark C. Ballandies
					  date-saved:: [[02/07/2024]]
					  date-published:: [[02/02/2024]]
					  - ### Content
					  collapsed:: true
					  - ![](https://proxy-prod.omnivore-image-cache.app/700x400,sh5jiuabk8tjpqd6Zfu9A8hNj5IPxHd3r28RoEO8qK1o/https://miro.medium.com/v2/resize:fit:700/1*WVE12HnT6VNSHlVdBeMMsA.png)
					  
					  web3 has an image problem. It is frequently perceived by the external world as an unproductive endeavor, steeped in unproven concepts and devoid of tangible, real-world advantages. This skepticism is multi-faceted: Investors question whether web3 projects can achieve product-market fit, regulators express concerns over the sustainability and utility of these systems, such as energy consumption, and their safety, including scam risks. And, the public, [despite evidence suggesting effectiveness](https://medium.com/coinmonks/complex-systems-part-2-managing-complexity-with-bottom-up-solutions-9d6fadd88cc4), increasingly doubts whether web3’s foundational values — decentralization, autonomy, and ownership — are truly efficacious in tackling humanity’s challenges, or if they simply contribute to greater disorder.
					  
					  However, the rise of new web3 sectors like Decentralized Physical Infrastructure Networks (DePIN) and Regenerative Finance (ReFi) are providing the crypto space for the first time with compelling narratives and products for the public. These innovations not only make revolutionary ideas such as co-ownership more accessible to a wider audience but also offer persuasive reasons to dispel prevailing doubts on cryptos positive impact. By leveraging web3 tools such as smart contracts, DAOs, NFTs, and token incentives, these verticals are at the forefront of a global movement for the betterment of human society and nature. They tackle critical issues like sustainability and enhance quality of life for everyone, demonstrating the potential of blockchain technology and its concepts for a positive, lasting real-world impact.
					  
					  This article explores this emerging potential of web3 as a powerful force for positive change and illustrates how it can establish a significant mainstream presence. It suggests initiating a cohesive, global movement by emphasizing the importance of leadership, the art of being a first follower, and collective action in driving transformative trends. Happy reading.
					  
					  \#\# DePIN & ReFi — Two promising web3 verticals
					  
					  \#\# Decentralized Physical Infrastructure Networks (DePIN)
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/700x394,sxQ0sEqesfewJXP4rW2Pr8fJEikPtNyRFrQCWsZ-VXp8/https://miro.medium.com/v2/resize:fit:700/1*Az_TjU8pWDk40cFbwaqWmA.jpeg)
					  
					  Source IoTeX: <https://iotex.io/blog/depin-landscape-map/>
					  
					  Decentralized physical infrastructure networks (DePINs) build different types of infrastructure serving markets ranging from energy over mobility to weather. They are a new vertical in the web3 that drive real-world demand by entering and extending established markets, hence solving one of the web3’s main problems and in this way enabling a new wave of crypto adoption.
					  
					  In particular, [DePINs are three sided markets](https://medium.com/wihi-weather/depins-are-three-sided-markets-an-evaluation-guide-for-investors-to-access-decentralized-8f82273c8d02) which enable them to construct customer-facing products (e.g. mobile plans ([Helium](https://www.helium.com/5G)), GPS positioning improvements ([onocoy](https://www.onocoy.com/)), or weather & climate intelligence ([WiHi](http://www.wihi.cc/))) using methodologies and approaches from traditional business modeling (e.g. problem-solution fit, product-market fit, etc.) and thus enabling web3 projects to demonstrate investors that they have utility (resp. enable investors to access their market/ make a valuation).   
					  Also, regulators and governments are immediately getting the usefulness of having functioning and well-maintained public infrastructures such as in energy, GPS positioning, or [mapping](https://www.proto-geo.xyz/) ([proto](https://www.proto-geo.xyz/)) and explains their recent strategic investments in projects such as [srcful ](https://twitter.com/frahlg/status/1724846753105473761)or [onocoy](https://www.euspa.europa.eu/newsroom/news/eu-space-start-ups-claim-victory-myeuspace-competition). In addition, what increasingly convinces the general public of web3’s potential is seeing that its values such as co-ownership and decentralization can construct global infrastructures ([see Helium](https://explorer.helium.com/)). Finally, this realization is supported by research results that are slowly spreading in the public that such systems[ are actually the more resilient and efficient](https://medium.com/coinmonks/complex-systems-part-2-managing-complexity-with-bottom-up-solutions-9d6fadd88cc4) way to build global infrastructures compared to hierarchical top-down management..
					  
					  DePIN uses web3 concepts such as tokens to crowdsource investment cost in infrastructures and align diverse stakeholder behind a collective identity, and DAOs to coordinate the emerging complex and interdependent networks of action. This drives their capital and operational expenditures down when compared with traditional and hierarchical approaches (e.g. by governments or large corporations) and facilitates a much quicker infrastructure build out, and thus ultimately a better, cheaper and more resilient service to end-customers.
					  
					  \#\# **Regenerative Finance (ReFi)**
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/700x451,sIhOv1zgcK5ufNwGdKMJcOVhGPLSlsfRJtpXKcOdiT9Y/https://miro.medium.com/v2/resize:fit:700/0*YC086ZTapqodWP5y)
					  
					  Source: <https://mirror.xyz/%5Fnext/image?url=https%3A%2F%2Fimages.mirror-media.xyz%2Fpublication-images%2FwLsGIoGZtCu%5F2%5FNZtFpO4.png&w=3840&q=75>
					  
					  Regenerative finance is an umbrella term for web3 projects that have a positive impact on the environment and nature. In particular, they are addressing the sustainability crisis we are in.
					  
					  For instance, ReFi projects such as [sunrise stake](https://www.sunrisestake.com/) or [coral tribe](https://www.coraltribe.io/) use web3 to solve the [Climate Action Trilemma](https://akshay93aditya.substack.com/p/the-climate-action-trilemma-for-the) of low-income areas: In the past, using traditional non-web3 mechanisms, only two of the following three conditions could be fulfilled at the same time: i) Financing of a project, ii) Speed of (climate) Action, and iii) Socio-economic development of a region. Now with web3 we can provide the financing of e.g. climate-positive projects (via tokens), facilitate speed of action (via self-organization and collective intelligence, e.g. DAOs) and improve the socio-economic status of participating actors (via co-ownership). Moreover, by relying on the autonomy of its actors and the permissionless nature of these systems, web3 allows everyone to co-create and shape those systems, hence the approach is using local knowledge of environments and context to have a sustainable, lasting and efficient impact in a particular region, which is orchestrated on a global scale, ultimately resulting in a global movement towards a more holistic and sustainable life. This gives not only empowerment, autonomy and thus a sense of meaning and agency to people on the ground (e.g. web1 and non-digitals), [but is also increasingly recognized by governments and regulators as the way forward to address humanity’s global challenges](https://www.undp.org/blog/think-globally-act-locally).
					  
					  \#\# Building a movement with DePIN and ReFi
					  
					  So, DePIN and ReFi leverage web3 technologies (like token incentives and DAOs) to address real-world challenges, offering a ‘triple win’ for web3, individuals, and society by enhancing global living standards and aligning with sustainability goals.
					  
					  The task remaining is to educate the public about this beauty and improve web3’s image.
					  
					  For this, we think that we, web3, need to unite (again) under a single banner, focusing on _real-world impacts and products_, to transition from digital to real-world relevance. Historically, web3 has periodically converged into movements such as DAOs, DeFi, and NFTs, amplifying their mainstream appeal and diversifying participation. Each convergence sparking wider adoption and enriching the ecosystem.
					  
					  We think that DePIN and ReFi are poised to be the unifying themes for web3, promising to permeate all aspects of life, including non-digital communities, with sustainable solutions and robust infrastructures. This could lead to a significant and potentially final crypto bull run, fully integrating web3 into mainstream society, with future applications being more rational and problem-specific.
					  
					  > The challenge left is to initiate this final, transformative web3 movement.
					  
					  \#\# Leadership Lessons from the shirtless dancing guy
					  
					  There is a pivotal short video of the shirtless dancing guy (3 min) illustrating comprehensibly how to create a movement — please make sure to watch it (and re-watch it — its a classic! Kudos to [Sarah Rietze](https://www.linkedin.com/in/sarahrietze/) for showing us).
					  
					  Let’s summarize the creation of a movement.
					  
					  \#\# The starting point: A leader
					  
					  It needs a lone nut with a great idea/ vision who goes out and stands there for her or himself. In particular, the leader has to expose himself up to the extent that he or she can look ridiculous. The leader and the idea have to be public. Moreover, the leader has to perform simple and instructional things to manifest the idea, which is key— it has to be easy to follow. Finally, the leader has to nurture the first few followers as equals, encourage them to act! It is not about him, its about the idea.
					  
					  Applied to crypto, Satoshi has been this lone nut. In the beginning it might have looked ridiculous (hence maybe the anonymity), but the idea was public — [the whitepaper and software where released by Satoshi via SourceForge as open source](https://www.techtarget.com/whatis/feature/A-timeline-and-history-of-blockchain-technology). Bitcoin is now on GitHub..  
					  Hence, it was easy to follow (for developers) — and they all automatically became equal, because the only thing which defines membership in Bitcoin is hashing power of a miner which anyone can deploy.   
					  Finally, Satoshi stepped back in 2010 and let the first followers take over, demonstrating that the whole thing is not about him.
					  
					  \#\# The pivotal moment: The First Follower
					  
					  > If leader is the flint, the first follower is the spark that makes the fire
					  
					  The first follower plays a crucial role — it transforms the leader from being a lone nut into being one. In particular, they show publicly how to follow. For this, like the leader, they need to have the guts to be ridiculed.   
					  In order to make the movement successful, those followers need to be empowered to call their friends in to join from their own capacity.
					  
					  First followers in Bitcoin where amongs others the guy who bought the pizza, or [Gavin Andresen](https://en.wikipedia.org/wiki/Gavin%5FAndresen) who was standing behind Bitcoin with his name.
					  
					  \#\# The turning point: Second Follower(s)
					  
					  The second follower(s) are the turning point — They prove that the first have done well, or in the words of the video “Its not a lone nut, and not two nuts, there is a crowd and a crowd is news”.
					  
					  This is the point where the movement needs to get public (in contrast to the idea getting public). In general, a movement must be public. It also has to be clear that the public sees more than just the leader. In particular, everyone needs to see the followers, _because followers create new followers_, not the leader.
					  
					  In terms of Bitcoin, there were many second followers, but the most important one probably is Ethereum. They brought blockchain into the public awareness with [THE DAO](https://en.wikipedia.org/wiki/The%5FDAO).
					  
					  \#\# Tipping point: The crowd is following
					  
					  When everything has been put in place, there comes the moment, where it is no longer risky to jump in — new joiners won’t stand out, they won’t be ridiculed and if they hurry, they might even be part of the in-crowed.  
					  Eventually, the remaining people will need to join because otherwise they will start being ridiculed for not joining. This is the point where the movement turns into a collective action.
					  
					  The tipping point for web3 will be reached when non-crypto people join the space not because they want to speculate, but when they want to contribute to a better world, respectively they are simply receiving a better service and do not want to miss out.
					  
					  \#\# Don’t forget: The environment
					  
					  The most important thing that has not been mentioned in the previous video is that a movement actually requires the right milieu. The video takes clearly place in a festival, so people already have the basic prerequisites to join a dancing movement — namely the _intrinsic motivation_ to dance.
					  
					  In terms of Bitcoin, the right milieu has been the financial crisis and the previous work and discussion on creating digital money which created a group of IT-affine people among which the idea could spread in the early days.
					  
					  \#\# Collective action is now
					  
					  Our engagement in Decentralized Physical Infrastructure Networks (DePIN) and Regenerative Finance (ReFi) has proven us that crypto ideas and values are well-received by non-web3 audiences. From hardware enthusiasts maintaining stations over academics working on weather forecasting or AI research to governmental agencies and small and medium-sized companies — all of them are looking forward to greater co-operation, co-ownership and co-creation.
					  
					  This signals us that the time for real-world crypto utility and adoption is now. In particular,
					  
					  > The environment is ready to be pollinated by web3.
					  
					  Nevertheless, we need a strategic approach:
					  
					  1. The web3 community has to unite around DePIN and ReFi, either by adopting existing projects or initiating new ones.
					  2. Then, web2 individuals have to be involved in these projects, particularly those interested in DePIN and ReFi-related fields like hardware or sustainable finance, is crucial. Their feedback so far as ‘second followers’ has been very positive and encouraging.
					  3. Finally, the movement needs to become public for bridging the gap between web2 and web3, enhancing mainstream awareness and adoption.
					  
					  These steps will help transition web3 from a niche to a significant influence in addressing global challenges.
					  
					  > “The best way to make a movement, if you really care, is to courageously follow and show others how to follow”
					  
					  Be a first follower and join this movement as a force for good!
					  
					  No clue where to start? Check out this list of awesome projects bringing web3 to the real world:
					  
					  [DePIN and ReFi — Executive summaries on sustainable web3 projects with real world utility](https://medium.com/@bcmark/depin-and-refi-executive-summaries-on-sustainable-web3-projects-with-real-world-utility-d2c8bae7b88e)
					  - [How I take Notes: Putting Theory into Practice](https://omnivore.app/me/how-i-take-notes-putting-theory-into-practice-18d802025ec)
					  collapsed:: true
					  site:: [substack.com](https://substack.com/inbox/post/108663756)
					  author:: Substack
					  labels:: [[note-taking]]
					  date-saved:: [[02/06/2024]]
					  - ### Content
					  collapsed:: true
					  - In the [first part](https://hybridhacker.email/p/how-i-take-notes-mastering-the-basics) of the "**How I Take Notes**" series, we covered some basic theory and principles of note-taking. The goal wasn't to be comprehensive, but rather to introduce you to concepts that will help you understand the practical part I'll be sharing with you today.
					  
					  However, note-taking is a vast and constantly evolving field, with various methodologies. But remember, **there's no right or wrong way to take notes**. The most important thing is to **find a system that works best for you**, based on your personal needs, style, and habits.
					  
					  In this issue, I'll be sharing my own approach to note-taking, with the hope that it will provide you with some inspiration to enhance your own process. While my approach may not be the perfect fit for you, I believe that it's always useful to learn from others and discover new techniques that can improve our productivity and creativity. 
					  
					  So, without further ado, let's dive right in!
					  
					  We already talked about the importance of processing notes and how PARA and Zettelkasten methods can help with that. But what exactly is a note-taking workflow?
					  
					  A note-taking workflow is a structured process for capturing, organizing, processing, and reviewing information using a note-taking system. It involves multiple stages, such as:
					  
					  * 🎣 **Capturing notes**. This initial step involves using a specific tool like a notebook or a digital note-taking app to capture notes.
					  * 🗂️ **Organizing notes**. In this step, notes are organized by topic, category, or other criteria, making them easy to find and retrieve later.
					  * ⚙️ **Processing notes**. This step entails reviewing and analyzing the notes, extracting key information, and transforming them into a more permanent form, such as literature notes and evergreen/permanent notes.
					  * 👀 **Reviewing notes**. This step involves regularly reviewing notes to reinforce learning and identify new connections and insights.
					  * 🧹 **Clean notes**. This last step is crucial to be sure that you keep your system clean and efficient.
					  
					  A good note-taking workflow should be flexible, efficient, and effective, enabling the user to capture and retain important information, retrieve it easily, and use it to achieve their goals.
					  
					  \#\# 🗄️ Building a Note-taking System
					  
					  Good note-taking methodologies and efficient workflows are the most important parts of what is called a note-taking system. But there’s one last bit missing.
					  
					  While we've discussed effective methodologies and workflows for note-taking, **selecting the right tools is equally important** and part of the note-taking system itself. In theory, you can use pen and paper to implement what we've seen so far, but we are in 2023, and we of course want to take digital notes. 
					  
					  Several apps are available for note-taking, and while all of them are potentially suitable for handling complex processes, some make life more manageable. 
					  
					  Apple Notes, MS OneNote, Simple Notes, and others are all suitable for basic note-taking, but they don't offer tools to process, link, and review notes.
					  
					  Here's a list of tools that are much more suitable for our purpose:
					  
					  * [Obsidian](https://obsidian.md/) (my choice)
					  * [LogSeq](https://logseq.com/)
					  * [Notion](https://www.notion.so/)
					  * [Roam Research](https://roamresearch.com/)
					  * [Joplin](https://joplinapp.org/)
					  * [Reflect](https://reflect.app/)
					  
					  All of these tools have their unique features, but **I recommend Obsidian** for its **simplicity**, **active community**, and **wide range of plugins** that extend its functionality. It's an excellent tool that can help you capture and organize information in a way that is both useful and accessible.
					  
					  So if you haven't already, please [download Obsidian](https://obsidian.md/) and start exploring it. 
					  
					  \#\# 🚶‍♂️First Steps with Obsidian
					  
					  At first, Obsidian might seem daunting due to its extensive range of options and functionalities. However, as you familiarize yourself with the app, you'll grow to appreciate its powerful capabilities.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x793,s-W8kZgdDqDyo0EygrnVCBAGSloCkiX8-AJ35DmXmZk4/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb7d18f2d-63ad-4f6c-946a-9e139415bc51_1914x1043.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb7d18f2d-63ad-4f6c-946a-9e139415bc51%5F1914x1043.png)
					  
					  Obsidian is a Markdown editor that enables you to perform basic tasks such as:
					  
					  * **📝 Creating documents in markdown format**
					  * **📄 Utilizing templates to generate documents**
					  * **👀 Previewing formatted documents**
					  * **🗄️ Organizing documents within folders**
					  * **🔗 Linking documents to one another**
					  * **\#️⃣ Tagging documents**
					  * **📅 Creating daily notes**
					  
					  These features alone provide a solid foundation for implementing an effective note-taking system and applying the methodologies we've discussed earlier. However, Obsidian also offers additional functionalities that elevate note-taking to new heights:
					  
					  * **🔙 Backlinks, which show you where notes are connected**
					  * **📊 Graph view, providing a comprehensive, bird's-eye view of all your notes**
					  * **🔁 Sync (paid), allowing you to back up your notes in the cloud**
					  * **🎨 Themes, offering customization for the appearance of your interface**
					  * **🧩 Plugins (core and community), which expand existing functionalities**
					  
					  Numerous tutorials, videos, and communities are dedicated to Obsidian, and while this article isn't intended as an Obsidian tutorial, I highly recommend exploring the app on your own. 
					  
					  Although this workflow can potentially work with any note-taking app, from this point on, all examples presented will refer specifically to Obsidian.
					  
					  \#\# 🚀 My Note-taking System
					  
					  While I love the note-taking methodologies I've described (PARA, Zettelkasten), taking them literally wouldn't be entirely compatible with my needs. For example, PARA is based on the **actionability of notes and folder structure**, while Zettelkasten counts on links, and ideally, **folders shouldn't be used**.
					  
					  That's why I use an hybrid approach, taking the best parts of different methods:
					  
					  * **PARA to manage actionable things and generic knowledge about my interests**
					  * **Zettelkasten for complex and conceptual notes**
					  * **Additional folders for journaling, personal stuff, people, and system things**
					  * **Johnny Decimal system to glue everything together (more on that later)**
					  
					  So let's dive into my notes organization.
					  
					  \#\#\# 📅 Journaling
					  
					  This is where everything starts. Every morning, the first note that automatically opens in my Obsidian vault is the daily note in the Journal folder. For this daily note, I created a template to record recurring things like:
					  
					  * **🏃 Morning routine**
					  * **📝 Notes of the day**
					  * **⛏️ Discovered today**
					  * **🔗 Interesting links**
					  * **🤩 Mood**
					  
					  This note is my companion throughout the day. At the end of the day or week, I process it.
					  
					  \#\#\# 🗂️ PARA
					  
					  Together with the Journaling folder, I have the PARA structure with Projects, Areas, Resources, and Archive folders. This is where I store and keep the notes and notions that are more actionable.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x443,s2uXZ1MnAkXlMCLGhZR8zOSQt6Q03ugfGmlvWMbuJayI/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3e455e9a-316d-4534-9fc1-0a35cb10f4d9_1471x448.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3e455e9a-316d-4534-9fc1-0a35cb10f4d9%5F1471x448.png)
					  
					  * In the **Projects** folder, I keep things related to specific projects, such as my newsletter, work projects, and side projects. Just one folder for each project, no nested folders.
					  * In the **Areas** folder, I collect things that are still actionable but have no fixed deadlines. Subfolders for work and essays are examples.
					  * In the **Resources** folder, I keep mainly notions about my interests, such as Technology, Cooking, and Outdoor folders.
					  * Lastly, I have the **Archive** folder where I put completed elements from the above folders.
					  
					  \#\#\# 🧠 Neurons
					  
					  The Neurons folder is where I implement the Zettelkasten method. This method involves creating small, individual notes or "neurons" that can be linked together to form a network of knowledge. Within the Neuron folder, I have several subfolders to categorize my notes.
					  
					  * **Fleeting Notes**: These are quick, informal notes that capture a passing thought, idea, or observation. They are not necessarily meant to be kept long-term, but can be useful for sparking creativity or brainstorming.
					  * **Literature Notes**: When reading a book, article, or a tweet, I take notes on important concepts, ideas, or quotes that I want to remember. These notes can be used to summarize the material, draw connections to other concepts, or provide support for future writing or research.
					  * **Evergreen Notes**: These are more comprehensive notes that capture deeper insights, analyses, or ideas. They are meant to be more permanent and can be used as building blocks for future writing, essays, or thinking.
					  
					  Using the Zettelkasten method and these subfolders within the Neurons folder, I can easily link related notes together and create a network of knowledge that can be easily navigated and expanded upon.
					  
					  \#\#\# 👥 Humans
					  
					  The "Humans" folder is where I keep track of people and companies that I find interesting or relevant to my work and personal interests. Within this folder, I use templates to record information about:
					  
					  * **Contacts**: This includes people I know personally or have worked with, as well as people I've met at events or through online communities.
					  * **Influencers**: I follow a number of thought leaders, bloggers, and other influencers in various fields. In this section, I keep notes on their work and ideas, as well as links to their websites, blogs, and social media profiles.
					  * **Companies**: I'm always on the lookout for interesting startups and businesses that are doing innovative things. Here, I keep notes on companies I'm researching or following, including their mission, products or services, and any news or updates I come across.
					  
					  Using this system, I can quickly access information about the people and companies that are important to me, and stay up-to-date on their latest activities and ideas.
					  
					  \#\#\# ⚙️ System
					  
					  This section typically includes templates, attachments, drawings, and other system-level elements that support your note-taking workflow.
					  
					  Templates can be particularly useful in this section. You can use templates to standardize the structure and content of your notes, making it easier to find and retrieve information later. Templates can be as simple as a blank document with a consistent header or footer, or they can be more complex, with multiple sections and pre-filled information. For example, you could create a template for meeting notes that includes a section for attendees, agenda items, and action items.
					  
					  \#\#\# 👨‍💻 Johnny.Decimal
					  
					  While I agree with the "less folders, the better" approach, I find both PARA and Zettelkasten too strict. For this reason, I use [Johnny.Decimal](https://johnnydecimal.com/).
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/800x400,sPAZPkm0qlJUSJjN6zc_-TAchn8SRywKHyyJIBjzVJm4/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F695bed65-1061-4e80-93cd-7431659f4eef_800x400.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F695bed65-1061-4e80-93cd-7431659f4eef%5F800x400.png)
					  
					  Johnny.Decimal is a system for organizing digital files and information. It uses a decimal numbering system to create a hierarchy of categories and subcategories. Each level of the hierarchy is separated by a decimal point. For example, a category might be assigned the number 100, and subcategories within that category might be assigned numbers such as 110, 120, and so on.
					  
					  As you may see from the screenshot above, I use it in a bit different way, but the concept is pretty similar.
					  
					  This system makes it easy to find and organize information quickly and efficiently. 
					  
					  \#\# 👉 Conclusion
					  
					  Today we covered:
					  
					  * **Note-taking workflows**
					  * **Note-taking systems**
					  * **Obsidian** (the software I use to take notes)
					  * **A quick look at how I process my daily notes**
					  * **A quick look at how I organize all my notes** (folders and Johnny Decimal system)
					  
					  What's next? Here's my suggestion:
					  
					  * 👀 Carefully read both issues ([part 1](https://hybridhacker.email/p/how-i-take-notes-mastering-the-basics) and this one) of How I take notes and be comfortable with the basic concepts
					  * 💾 [Download Obsidian](https://obsidian.md/) and become familiar with it
					  * 🧭 Explore all the options Obsidian provides and become familiar with concepts like Core Plugins and Community Plugins
					  * 💭 Think about the information that is relevant in your life and based on their actionability, decide where it could fit in
					  * 🏗️ Start building your personal knowledge system!
					  
					  Be aware, I know it can initially seem daunting and like something that requires a lot of effort. But the more you get into note-taking, the more you will discover how it can make you more productive, organized, and effective in what you do. 
					  
					  Don’t be afraid to try it out and don’t aim to get everything done immediately. Building a note-taking system is something that can take months if not years, and it's a continuous process of evolution, so it's a never-ending effort. 
					  
					  🚀 **The important thing is to start!**
					  
					  \#\# 🎁 Download My Obsidian Base Vault
					  
					  As a Paid Subscriber, you can download my Obsidian Base Valut from [here](https://drive.google.com/file/d/1ZWBsuXrmbaYTXaw1%5FKEQMoZr9pH9zNGz/view?usp=drive%5Flink).
					  
					  \#\# That’s all folks
					  
					  That's all for today! As always, I would love to hear from my readers (and if you've read this far, you're definitely one of the good ones). Please don't hesitate to add me on [LinkedIn](https://www.linkedin.com/in/nicolaballotta) or [Twitter](https://twitter.com/nicolaballotta) and send me a message; I always respond to everyone.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/80x0,saMWw-royHVUzsypndZKZsdJbmA_ktOhBYR1uySVC0hY/https://substackcdn.com/image/fetch/w_80,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F54103c8e-f322-4beb-ae21-11a448db6190_400x400.jpeg)](https://substack.com/profile/189714-domitilla-ferrari?utm%5Fsource=post-reactions-face)
					  - [How I take Notes: Mastering the Basics - by Nicola Ballotta](https://omnivore.app/me/how-i-take-notes-mastering-the-basics-by-nicola-ballotta-18d801feb14)
					  collapsed:: true
					  site:: [substack.com](https://substack.com/inbox/post/106938195)
					  author:: Substack
					  date-saved:: [[02/06/2024]]
					  - ### Content
					  collapsed:: true
					  - I'm honest, if I had to start this essay from a blank page, I'd probably spend hours looking at the blinking cursor without writing a single character. But luckily enough, I've been thinking about this post for weeks now, and guess what? I have all my notes about it, which made starting the piece much more straightforward.
					  
					  When I was younger, precisely in school, **I literally hated taking notes**. I know people who still love to show their school notebooks and diaries, full of notes, schemas, drawings, schedules, and writings. I also still have mine at my parents' house, and you could probably sell them as new.
					  
					  I was so arrogant as to pretend to record everything in my head. Well, I did that for a long time, and I was also good enough at it. But unfortunately, **our brain's capacity is limited** (approximately **2.5 Petabytes of storage** according [Scientific American](https://www.scientificamerican.com/article/what-is-the-memory-capacity/)), and so at one point, my brain started overflowing, losing information. This is when I started taking notes, and this is also where **my life changed**.
					  
					  Someone might think note-taking is just about recording informations, but it's not. I said note-taking literally changed my life, and it was not just a catchy phrase. If you ask yourself why you take notes, the first answers that will come to your mind would probably be "_**to not forget important things**_" or "_**to organize my knowledge**_". And for sure, **these are good reasons to take notes**, but there are also other aspects that probably are not so evident at first glance.
					  
					  Some of the benefits of note-taking, especially if done the right way, include:
					  
					  * 🧠 **Increased memory efficiency**. Writing is a very good way to impress things in your brain and exercise it.
					  * ✍️ **Become better at writing**. Taking notes means writing more, and the more you write, the better you become at it.
					  * 📖 **Become better at reading**. As for writing, reading is a huge part of note-taking.
					  * 🤓 **Improved understanding**. Writing notes means processing information (we will see after how) and processing information helps better understand concepts.
					  * 👮‍♂️ **Increased discipline**. As you may imagine, to be good at all the points I mentioned before, you need to be disciplined, and writing notes is a good way to become it!
					  
					  Another **common misconception is that** **taking structured notes is just for researchers or students**. Nothing could be further from the truth.
					  
					  Taking notes is simply for everyone. It's just a process, **a habit that you establish in your everyday routine** that helps process and keep track of any kind of information. It doesn't matter if you are tracking your research in neuroscience, your fitness progress, your mood, or your cooking recipes; it works the same way.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1019x457,sDHjRo1Jy-zMfe6NryEmMkxpQiu5a441sU0nuQaQYknI/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F73d2d758-3490-40f8-a556-e5435efa4302_1019x457.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F73d2d758-3490-40f8-a556-e5435efa4302%5F1019x457.png)
					  
					  There's **an input**, you decide if this input **is relevant** for you, if it is **you process it** and then you store it in a note. Easy peasy.
					  
					  \#\# 📚 A Bit of Theory
					  
					  When you start delving deep into note-taking, **you will discover an entirely new world**.
					  
					  There are entire books about note-taking, methodologies to organize your notes, to grow them, and at one point, to make them yours. It would probably take many issues of this newsletter to go through them all. For the scope of this essay, which is **to share the way I take notes** and **introduce you to this world**, I will just go through some concepts that you will see used in the more practical part.
					  
					  \#\#\# PKM vs PKD
					  
					  Note-taking refers to the action of recording information in a place of your choice, such as a notebook, text file, dedicated software, database, etc. However, it is often put in relation to **Personal Knowledge Management** (PKM).
					  
					  Personal Knowledge Management refers to the **process of** **collecting, organizing, and managing one's own personal knowledge** and information in a way that is useful and accessible.
					  
					  Some people also make a further distinction between Personal Knowledge Management and **Personal Knowledge Development** (PKD). While the former involves collecting and organizing notes, the latter refers to the **process of acquiring, organizing, and integrating new information and experiences into one's existing knowledge base**. It involves actively seeking out new information, reflecting on one's experiences, and making connections between new and old knowledge.
					  
					  \#\#\# P.A.R.A.
					  
					  The P.A.R.A. method is a note-taking and personal knowledge management system created by [Tiago Forte](https://fortelabs.com/), a productivity and organization expert.
					  
					  P.A.R.A. stands for **Projects**, **Areas**, **Resources**, and **Archives**, and the main idea is to organize your notes **based on their actionability**.
					  
					  * 📂 **Projects** \- This section is designated for your ongoing projects. A project refers to a series of tasks that are linked to a specific objective and must be completed within a **defined timeframe**. These tasks represent the **tangible actions** you intend or must undertake.
					  * 📍**Areas** \- This section contains **ongoing activities** that are **still actionable** but **do not have a fixed deadline**, such as work notes, writing, personal finance, and maintaining one's health. These are tasks that demand regular maintenance and attention. As these activities are consistently monitored, new projects may emerge from this section.
					  * ⛏️ **Resources** \- This section serves as a r**epository for information on topics that pique your interest** or prove valuable to you. As the title implies, these are nuggets of knowledge that you wish to retain and consult at a later time.
					  * 🗄️ **Archives** \- This section is designated for **storing items from the other three categories that have already been finished** or are no longer relevant. This includes items such as completed projects, inactive areas, or resources that you no longer wish to actively manage.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1019x457,s2ypoZipZ0YGspNkXQD6U3Zvh4roiqo1_orv4pwksWRY/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67fa7065-82f3-47ee-adfc-3de16d299332_1019x457.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67fa7065-82f3-47ee-adfc-3de16d299332%5F1019x457.png)
					  
					  According to Tiago Forte, **these are the four folders you should have** to organize your knowledge and where to put your notes.
					  
					  As you may have noticed, while the Projects and Archives sections are pretty clear sections, the **Area and Resources sections are less defined and could overlap**. Don't worry too much about this; these are just guidelines and inspirations. You can decide how to manage your knowledge!
					  
					  Personally, I use the Area folder for work-related notes, essays, etc. - things that regularly change and are more actionable. While I put most of the things I'm interested in (e.g. Technology, Cooking, Outdoor) in the Resources folder. You will find people who choose where to put things based on how personal they are and others who put just files, schemes, drawings in the Resource folder. **Do whatever you feel is right for you**.
					  
					  \#\#\# Zettelkasten
					  
					  Zettelkasten is **something in between a knowledge management and development system** that was ideated by German sociologist [Niklas Luhmann](https://en.wikipedia.org/wiki/Niklas%5FLuhmann). The name "Zettelkasten" means "slip box" or "index card system" in German, and it refers to a physical or digital collection of notes that are stored and organized in a specific way to aid in idea generation and knowledge organization.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1019x457,s8SBE167FS-8exOGcVTi22tNLHeZwEmUOrXh8TdqPe4U/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4e0dded0-9b30-4220-8f03-44f8f68ad820_1019x457.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4e0dded0-9b30-4220-8f03-44f8f68ad820%5F1019x457.png)
					  
					  In a Zettelkasten system, **each note is created on a separate card or digital file**, with a unique identifier, date, and title. The notes are then organized by topic or theme and **connected to other relevant notes through a system of links or tags**. The goal of this system is to allow for flexible and creative idea generation and organization, as well as to enable effective retrieval and synthesis of information.
					  
					  The Zettelkasten methodology has three fundamental concepts:
					  
					  * 📥 **Fleeting Notes** \- These are notes in their most primitive stage. Whenever something sparks your interest, an idea, or something you want to explore further, you should quickly take note of it. For example, "I see a relation between Availability and Accountability in remote working." This should be a **brief but meaningful note**. It's not important how you write it or if it contains typos. You can see it as a reminder that needs to be processed later.
					  * 📚 **Literature Notes** \- These notes are useful when you want to reference something that already exists, such as a theory, idea, or notion. Ideally, even if it's coming from an external source, **you should write this note in your own words**. However, some notes can be copy-pasted.
					  * 🍀 **Permanent Notes** \- This is one of the most important pieces of the Zettelkasten method. While Literature Notes are thoughts coming from third parties, **permanent notes** (also known as Evergreen Notes) **reflect your thoughts**. They could be influenced by Literature Notes, but they should always include your personal thoughts on a particular topic.
					  
					  For Zettelkasten, you should only have these three folders. However, finding notes could become difficult, so how can you find them effectively? In Zettelkasten, there are two more concepts that are crucial:
					  
					  * 🔗 **Links** \- As we saw before, the main idea is to take atomic notes that you can process and elaborate on. But these notes are not very useful alone. That's why linking is one of the core principles of Zettelkasten. Linking multiple notes together can make them much more powerful.
					  * 🏷️ **Tags** \- All your ideas and notes, following the Zettelkasten approach, are scattered in these three folders. That's why you should always use tags. Tags make it easy to find things.
					  
					  I know these are a lot of concepts all together. Please consider that there are entire books discussing Zettelkasten (see the resources section), so it's challenging to condense everything in this article. But let me provide a real example to help you understand better.
					  
					  \#\#\#\# 🪚 A Practical Example
					  
					  Let's take the idea I mentioned earlier. Let’s pretend that while thinking or reading something, I came up with the idea that _Availability and Accountability are related when we talk about remote working_.
					  
					  This is what I would do (I actually did that before writing “[Balancing Availability and Accountability in Remote Working](https://hybridhacker.email/p/balancing-availability-and-accountability)”):
					  
					  * 📥 I take a quick fleeting note about my idea. It should **be meaningful** so I can come back to it later and understand what I was thinking about.
					  * 📚 Meanwhile, I want to have a clearer idea of what Availability and Accountability are, so I start gathering information from the web (or reading a book, or watching videos), to better understand these concepts. When I have a better understanding, I **create a literature note for Accountability and one for Availability**, storing the information I got from third parties there. I also put meaningful tags so I can easily find these notes when needed.
					  * 🍀 After thinking more about my idea, I decide it's time to transform it into something permanent. So I create a permanent/evergreen note. This note **is written in my own words and reflects my idea**, even though I can link the literature notes I created earlier to help contextualize. I also tag this note so it's easier to find when I need it.
					  
					  You see the power of this approach? I hope so, because we are just scratching the surface!
					  
					  \#\# Resources
					  
					  The concepts I explained earlier should be sufficient to understand the basics and approach the more practical guide that we will explore in the next part of this essay. If you wish to dive deeper, here are some interesting resources you can read.
					  
					  📚 **Books**
					  
					  * [How to Take Smart Notes](https://www.amazon.com/How-Take-Smart-Notes-Technique-dp-3982438802/dp/3982438802/) (Sönke Ahrens) - Probably the best and most famous book about Zettelkasten method
					  * [Building a Second Brain](https://www.amazon.com/Building-Second-Brain-Organize-Potential/dp/1982167386) (Tiago Forte) - A must read for notes taking (includes the P.A.R.A method)
					  
					  🔗 **Links**
					  
					  * [Progressive Summarization](https://fortelabs.com/blog/progressive-summarization-a-practical-technique-for-designing-discoverable-notes/) \- How to write better and easily discoverable notes
					  * [Writing Atomic Notes](https://zettelkasten.de/posts/create-zettel-from-reading-notes/) \- Good and practical examples on how to take atomic notes and process them
					  
					  \#\#\#\# 🗣️ Other
					  
					  * <https://www.reddit.com/r/NoteTaking/> \- Subreddit where to exchange informations on how to take notes
					  * <https://www.reddit.com/r/Zettelkasten/> \- Subreddit dedicated to Zettelkasten methodology
					  
					  \#\# ⏩ What to Expect Next
					  
					  I know that all these concepts together, could seem intimidating, but they will be needed to understand the second and more practical part of my note-taking approach. In the next part of “How I take Notes” I will tell you:
					  
					  * How I process notes and transform them into meaningful knowledge
					  * How my PKM workflow works
					  * How I structured my PKM folders
					  * How I use Obsidian to manage my PKM applying the principles I described
					  
					  \#\# ✌️ That’s all folks
					  
					  That's all for today! As always, I would love to hear from my readers (and if you've made it this far, you're definitely one of the bravest). Please don't hesitate to connect with me on [LinkedIn](https://www.linkedin.com/in/nicolaballotta) and send a message. I always respond to everyone!
					  - [Re: UMP / CORS: Implementor Interest from Mark S. Miller on 2010-04-22 (public-webapps@w3.org from April to June 2010)](https://omnivore.app/me/re-ump-cors-implementor-interest-from-mark-s-miller-on-2010-04-2-18d7afa5cb5)
					  collapsed:: true
					  site:: [lists.w3.org](https://lists.w3.org/Archives/Public/public-webapps/2010AprJun/0293.html)
					  author:: Mark S. Miller (erights@google.com)
					  date-saved:: [[02/05/2024]]
					  date-published:: [[04/21/2010]]
					  - ### Content
					  collapsed:: true
					  - On Mon, Apr 19, 2010 at 12:43 AM, Anne van Kesteren <[annevk@opera.com](mailto:annevk@opera.com?Subject=Re%3A%20UMP%20%2F%20CORS%3A%20Implementor%20Interest&In-Reply-To=%3Cj2x4d2fac901004221633h6cd611dide6041ee5e6f8e2d%40mail.gmail.com%3E&References=%3Cj2x4d2fac901004221633h6cd611dide6041ee5e6f8e2d%40mail.gmail.com%3E)>wrote:
					  
					  > Hopefully it helps calling out attention to this in a separate thread.
					  >
					  > In <http://lists.w3.org/Archives/Public/public-webapps/2010AprJun/0043.htmlMaciej> states Apple has no interest in implementing UMP from the UMP
					  > specification. (I believe this means that a CORS defined subset that roughly
					  > matches UMP is fine.) They want to retain their CORS support.
					  >
					  > For Opera I can say we are planning on supporting on CORS in due course and
					  > have no plans on implementing UMP from the UMP specification.
					  >
					  > It would be nice if the three other major implementors (i.e. Google,
					  > Mozilla, and Microsoft) also stated their interest for both specifications,
					  > especially including whether removing their current level of CORS support is
					  > considered an option.
					  >
					  
					  Caja does plan to implement UMP and not CORS. Caja is a user agent built as
					  a virtual browser-in-browser, translating from the subset of future web
					  standards it accepts (e.g., a subset of ES5/strict) into the subset of
					  future web standards supported by current browsers (e.g., a subset of ES3).
					  Caja accepts not just JavaScript of course -- Caja parses a sanitized subset
					  of HTML HTML5's tag soup algorithm. Since Caja helps protect the Yahoo! home
					  page, in some quantitative sense it is a larger user agent than Safari,
					  Opera, or Chrome.
					  
					  Caja intermediates the dereferencing of all URLs through a
					  container-supplied URL translation policy. Say cajoled code inlined on a
					  page running on site X makes a cross-origin request to a server addressed as
					  site Y. Caja does support all Yahoo! A-grade browsers including IE6. To
					  emulate the cross-origin request on IE6, obviously, our only choice is to
					  relay the request through the X server. Since the X server has no access to
					  the browser's cookies for site Y, obviously, we cannot emulate the CSRF
					  vulnerabilities of full CORS even if we wanted to. UniformRequests can be
					  faithfully relayed through intermediaries. Full CORS cannot. Thus,
					  UniformRequests have a better incremental transition story for software that
					  can be deployed today.
					  
					  
					  >
					  >
					  > --
					  > Anne van Kesteren
					  > <http://annevankesteren.nl/>
					  >
					  >
					  
					  
					  -- 
					  Cheers,
					  --MarkM
					  - [Anxiety, Mood Swings and Sleepless Nights: Life Near a Bitcoin Mine - The New York Times](https://omnivore.app/me/anxiety-mood-swings-and-sleepless-nights-life-near-a-bitcoin-min-18d7336016a)
					  collapsed:: true
					  site:: [The New York Times](https://www.nytimes.com/2024/02/03/us/bitcoin-arkansas-noise-pollution.html?referringSource=articleShare&smid=nytcore-ios-share)
					  author:: Gabriel J.X. Dance
					  date-saved:: [[02/04/2024]]
					  date-published:: [[02/03/2024]]
					  - ### Content
					  collapsed:: true
					  - ![](https://proxy-prod.omnivore-image-cache.app/0x0,swwjTFKyeD8OaMmOXma6Oif7wuSZejWucV3h5DSTpavw/https://static01.nyt.com/images/2024/01/17/autossell/bitcoin-loop/bitcoin-loop-threeByTwoMediumAt2X.jpg)
					  
					  A Bitcoin operation that opened last May near Greenbrier, Ark.Credit...Video by Rory Doyle for The New York Times
					  
					  Pushed by an advocacy group, Arkansas became the first state to shield noisy cryptocurrency operators from unhappy neighbors. A furious backlash has some lawmakers considering a statewide ban.
					  
					  A Bitcoin operation that opened last May near Greenbrier, Ark.Credit...Video by Rory Doyle for The New York Times
					  
					  [Gabriel J.X. Dance](https://www.nytimes.com/by/gabriel-dance)
					  
					  Gabriel J.X. Dance visited several Bitcoin operations in Arkansas, including one near Greenbrier.
					  
					  On a sweltering July evening, the din from thousands of computers mining for Bitcoins pierced the night. Nearby, Matt Brown, a member of the Arkansas legislature, monitored the noise alongside a local magistrate.
					  
					  As the two men investigated complaints about the operation, Mr. Brown said, a security guard for the mine loaded rounds into an AR-15-style assault rifle that had been stored in a car.
					  
					  “He wanted to make sure that we knew he had his gun — that we knew it was loaded,” Mr. Brown, a Republican, said in an interview.
					  
					  The Bitcoin outfit here, 45 minutes north of Little Rock, is one of three sites in Arkansas owned by a network of companies embroiled in tense disputes with residents, who say the noise generated by computers performing trillions of calculations per second ruins lives, lowers property values and drives away wildlife.
					  
					  Scores of the operations have popped up in recent years across the United States. When a mining computer lands on numbers that Bitcoin’s algorithm accepts, the payout is currently worth about a quarter-million dollars. The more computers an operation has, the better chance of earning the payout.
					  
					  The industry is often criticized for its vast energy use — [often a boon for the fossil-fuel industry](https://www.nytimes.com/2023/04/09/business/bitcoin-mining-electricity-pollution.html) — and noise is a common complaint. Though some elected officials like Mr. Brown and other Bitcoin operators in Arkansas have voiced support for the beleaguered residents, a new state law has given the companies a significant leg up.
					  
					  Image
					  
					  ![A fenced compound surrounded by trees and power lines. ](https://proxy-prod.omnivore-image-cache.app/600x399,sP4nXzlBNQ-SBSHE0IXVLjmG33B3TViLYM3Z3JaWx9Kw/https://static01.nyt.com/images/2024/01/17/multimedia/00bitcoin-arkansas-wjgf/00bitcoin-arkansas-wjgf-articleLarge.jpg?quality=75&auto=webp&disable=upscale)
					  
					  The site north of Little Rock is one of three in the state owned by a network of companies embroiled in tense disputes with residents.Credit...Rory Doyle for The New York Times
					  
					  The Arkansas Data Centers Act, popularly called the Right to Mine law, offers Bitcoin miners legal protections from communities that may not want them operating nearby. Passed just eight days after it was introduced, the law was written in part by the Satoshi Action Fund, a nonprofit advocacy group based in Mississippi whose co-founder worked in the Trump administration rolling back Obama-era climate policies.
					  
					  “The state of Arkansas has pulled off a surprise victory and become the first in the nation to pass the ‘Right to Mine’ \#Bitcoin bill in both the House and Senate,” Dennis Porter, the fund’s chief executive, [posted](https://twitter.com/Dennis%5FPorter%5F/status/1644544978251808768) on social media when the law passed last April.
					  
					  A similar bill passed in Montana last May, and the group has said it hopes to enact its successful formula in more than a dozen other states. Bills written in collaboration with the group were introduced last month in several states including [Indiana](https://iga.in.gov/pdf-documents/123/2024/house/bills/HB1388/HB1388.01.INTR.pdfhttps://iga.in.gov/pdf-documents/123/2024/house/bills/HB1388/HB1388.01.INTR.pdf), [Missouri](https://documents.house.mo.gov/billtracking/bills241/hlrbillspdf/3683H.01I.pdf), [Nebraska](https://nebraskalegislature.gov/bills/view%5Fbill.php?DocumentID=54648) and [Virginia](https://lis.virginia.gov/cgi-bin/legp604.exe?241+ful+SB339+pdf).
					  
					  Founded five years ago as the Energy 45 Fund, the group sought to tout Mr. Trump’s energy and environmental agenda and “[defend the greatest president in modern history](https://www.eenews.net/articles/ex-epa-official-on-rollbacks-snowballs-and-selling-trump/).” Its founder, Mandy Gunasekara, had spent the previous two years at the Environmental Protection Agency, where she played a key role in the decision to pull the United States out of the Paris climate accord and helped repeal the Clean Power Plan, which aimed to reduce emissions from coal-burning power plants.
					  
					  The group is widely lionized by the Bitcoin community, both for its legislative work and for its combative stance toward critics of the industry. But the fund’s aggressive approach has riled others in the Bitcoin community who say they prefer to build consensus around cryptocurrency operations.
					  
					  Arry Yu, executive director of the U.S. Blockchain Coalition, an industry group, said Arkansas residents were “taken advantage” of.
					  
					  “We need to take a humble approach, work with the communities, don’t hijack their journeys and their lives,” Ms. Yu said. “And if they move slowly, too slow for you, too bad.”
					  
					  The strife in Arkansas reflects disagreements across the United States as Bitcoin mining has grown by leaps and bounds. Environmental activists, troubled by the industry’s electricity consumption and resulting pollution, have called for federal regulation, while backers of the operations say the mines often help stabilize vulnerable electrical grids and provide jobs in rural areas.
					  
					  Concerns about the Arkansas mines have expanded beyond the initial noise complaints to include their connections to Chinese nationals. The operations are connected to a larger influx of Chinese ownership across the United States, The New York Times [reported](https://www.nytimes.com/2023/10/13/us/bitcoin-mines-china-united-states.html) in October, some of which has drawn national security scrutiny.
					  
					  A web of shell companies connects the Arkansas operators to a multibillion-dollar business partially owned by the Chinese government, according to public records obtained by residents opposed to the operations. In November, the Arkansas attorney general’s office opened an [investigation](https://arkansasadvocate.com/wp-content/uploads/2023/12/12.11.23-Act-636-Letters.pdf) into them for potentially violating a state law barring businesses controlled by Chinese nationals from owning land.
					  
					  A lawyer representing the operations said an independent security contractor was responsible for the incident near Greenbrier and the company never authorized any guard to “brandish a firearm.” They also said that the attorney general’s investigation was based on a “misunderstanding” and that they are legally allowed to conduct business.
					  
					  Despite efforts to build bipartisan support, the Satoshi fund has succeeded predominantly in red states. But in Arkansas, where the state legislature is dominated by Republicans, it is conservatives who have led calls to repeal the law, including Senator Bryan King, a poultry farmer whose district includes a property purchased by one of the companies tied to the Chinese government. He said it was not fair that the Bitcoin operators received special protections under the law, which shields them from “[discriminatory industry specific regulations and taxes,”](https://www.arkleg.state.ar.us/Home/FTPDocument?path=%2FACTS%2F2023R%2FPublic%2FACT851.pdf) including noise ordinances and zoning restrictions.
					  
					  “They’re in a protected class more than any other business out there,” Mr. King said.
					  
					  As restrictions introduced in Congress have failed to gain traction, states and cities have stepped in to fill the void. But as Arkansas has demonstrated, unsatisfying results can leave residents feeling betrayed.
					  
					  Image
					  
					  ![A portrait of a woman in a pink shirt.](https://proxy-prod.omnivore-image-cache.app/0x0,sQupyK5gHaw_ahA2-AdWzz_wzoPXYGsS8guLsRHrZwAw/https://static01.nyt.com/images/2024/02/05/multimedia/00bitcoin-arkansas-print-2/00bitcoin-arkansas-04-jbvc-articleLarge.jpg?quality=75&auto=webp&disable=upscale)
					  
					  Gladys Anderson and nearly two dozen neighbors filed a lawsuit against the owners of a Bitcoin operation near Greenbrier, Ark., blaming the operation for various health problems.Credit...Rory Doyle for The New York Times
					  
					  \#\# ‘It’s Exhausting’
					  
					  “Hell” is how Gladys Anderson describes life since the Bitcoin operation near Greenbrier opened last May less than 100 yards from her home.
					  
					  Computers have been running mostly around the clock, she said, creating so much noise — they require constant cooling by loud fans — that her son no longer goes outside. “The reason we moved out here was to get away from people, get away from noise,” she said.
					  
					  Her son, who requires full-time care for autism, has also grown more agitated and aggressive, she said. “It’s exhausting mentally, emotionally, physically,” Ms. Anderson said.
					  
					  In July, she and nearly two dozen neighbors filed a lawsuit against the owners, NewRays One, blaming the operation for various health problems, including increased blood pressure, anxiety, difficulty sleeping and mood swings.
					  
					  The lawsuit also suggests the mine has depressed property values.
					  
					  “Who would want to purchase property near the noisy site?” one of the residents, Rebecca Edwards, wrote in an affidavit. “Short answer: No one.”
					  
					  Lawyers representing NewRays are seeking to have the case thrown out, citing the Right to Mine law, among other arguments. Recently, the same judge overseeing the lawsuit ruled in a separate case that a local ordinance restricting noise at a related operation was likely to be discriminatory, violating the state law.
					  
					  A lawyer for NewRays disputed the allegations made by Ms. Anderson and the other residents, telling The Times that the company looked forward to defending itself in court. As for the lawsuit at the related operation, in which NewRays is a partner, the lawyer said the mine would be a “responsible neighbor” and hoped to find additional ways “to give back to the community.”
					  
					  Image
					  
					  ![A house and a computing compound separated by a smattering of trees. ](https://proxy-prod.omnivore-image-cache.app/0x0,sk7rNpInS-g_HrFUxkrEXo5UHYE_ndJ7O3t5G8pesLQg/https://static01.nyt.com/images/2024/02/05/multimedia/00bitcoin-arkansas-print-1/00bitcoin-arkansas-02-jbvc-articleLarge.jpg?quality=75&auto=webp&disable=upscale)
					  
					  The Bitcoin mine, left, is less than 100 yards from Ms. Anderson’s home, top right.Credit...Rory Doyle for The New York Times
					  
					  After the law was signed by Gov. Sarah Huckabee Sanders in April, 49 of the state’s 76 counties enacted ordinances limiting noise levels at data centers, including cryptocurrency mining operations, before it took effect in August, according to the Association of Arkansas Counties. The legality of those ordinances, and local governments’ inability to regulate the industry, is now central to the struggle between residents and the Bitcoin operators, with some elected officials who voted for the state law now opposing it.
					  
					  “What wasn’t explained was the nature of these crypto mines and how they can cause an intolerable noise with no regard for neighbors or wildlife,” Representative Jeremiah Moore, a Republican whose district includes a Bitcoin operation, said in an email.
					  
					  Mr. Moore said the mining bill had been disingenuously promoted to lawmakers as protecting an industry that would create jobs and benefit nearby communities. He recently joined several other lawmakers in drafting a proposed statewide ban on industrial-level cryptocurrency mining.
					  
					  Senator Joshua Bryant, a Republican co-sponsor of the pro-mining legislation, said in an interview that the law was meant to protect the property rights of Bitcoin miners and that he believed a significant amount of pushback was a result of misdirected anti-Chinese sentiment.
					  
					  Mr. Bryant said he that was exploring the possibility of a statewide noise ordinance “to address potential health and safety harms to citizens of the state,” and that “ultimately we have to continue to figure out how to live with our neighbors.”
					  
					  The primary sponsor of the law, Representative Rick McClure, also a Republican, did not respond to requests for comment from The Times.
					  
					  A primary backer of the legislation was Cryptic Farms, a Bitcoin-mining company run by Cameron Baker, an Arkansas native. Mr. Baker said his company did not anticipate that “bad actors” might exploit the law.
					  
					  “It wasn’t really on our radar that somebody was going to come in right behind the passage of this bill and present themselves as this perfect villain that does everything wrong,” he said in an interview.
					  
					  Tom Harford, an executive at Cryptic Farms who leads the Arkansas Blockchain Council, an industry group, said that he regretted putting residents in a position “where they have no recourse” and that “no law is perfect.”
					  
					  Mr. Harford said he “helped tweak” the law, but it was primarily written by Eric Peterson, head of policy at the Satoshi Action Fund. Mr. Peterson declined to comment.
					  
					  “It’s a Satoshi bill,” Mr. Harford said.
					  
					  Image
					  
					  ![A woman speaks into a microphone.](https://proxy-prod.omnivore-image-cache.app/0x0,sICeoTCsFVaYnAg9TJ6ePeAFxZ9-bdR0I2IsjcABReug/https://static01.nyt.com/images/2024/01/17/multimedia/00bitcoin-arkansas-pqzv/00bitcoin-arkansas-pqzv-articleLarge.jpg?quality=75&auto=webp&disable=upscale)
					  
					  Mandy Gunasekara, a founder of the Satoshi Action Fund, which helped write the Bitcoin law.Credit...Rogelio V. Solis/Associated Press
					  
					  \#\# From Trump to Bitcoin
					  
					  The history of the Satoshi Action Fund is unconventional, to say the least.
					  
					  Ms. Gunasekara, its co-founder, gained notoriety in 2015 while working for Senator Jim Inhofe, Republican of Oklahoma, bringing him a snowball on the Senate floor while he argued that climate change was a hoax.
					  
					  She is married to a lobbyist who for years represented the oil industry (and who is also a co-founder of the fund), and has railed against what she calls the left’s “[woke](https://dailycaller.com/2022/06/08/gunasekara-republican-climate-plan-will-stop-bidens-assault-on-american-energy-and-american-pocketbooks/)” climate agenda. Last year, the Mississippi Supreme Court disqualified her in an election for a utilities regulatory board because she did not meet residency requirements.
					  
					  Before launching the fund in 2019, Ms. Gunasekara worked as a senior adviser to Scott Pruitt, the first E.P.A. administrator under Mr. Trump. After she returned to the E.P.A. in 2020 as chief of staff to Mr. Pruitt’s successor, Andrew Wheeler, the fund appeared to languish, changing its name from the Energy 45 Fund to Energy Moms and then to Alliance for Energy Workers.
					  
					  Legal experts who reviewed the group’s tax filings during those years described them as slapdash and containing obvious contradictions. The group reported to the I.R.S., for instance, that its board of directors had zero members — but then, on the same form, reported that it had documented every meeting the board held.
					  
					  As of 2021, it appeared to be an empty vessel waiting for a purpose. That purpose appears to have arrived by way of a phone call from Mr. Pruitt. On a [podcast](https://www.youtube.com/watch?v=N%5FuL7Cnu3qc), Ms. Gunasekara said Mr. Pruitt suggested that two of them start a business selling electricity contracts to Bitcoin miners.
					  
					  It is unclear if Ms. Gunasekara and her old E.P.A. boss went into business; neither she nor Mr. Pruitt responded to requests for comment. But nearly a year later, Ms. Gunasekara, her husband and Mr. Porter repurposed the nonprofit as the Satoshi Action Fund, focused on Bitcoin and mining operations in particular. (Satoshi is the pseudonym associated with the unknown inventor of Bitcoin.)
					  
					  One of the fund’s purposes, Ms. Gunasekara said during a [speech announcing the organization,](https://www.youtube.com/watch?v=6YHHpn22drU) is to tell the “very good stories” that Bitcoin mining has to offer, including the “role of rural revitalization.”
					  
					  The group has held events in multiple states and in Washington, including handing out books on Bitcoin to lawmakers, and recently started a second nonprofit to publish scientific papers about the benefits of Bitcoin mining.
					  
					  It has also tried to grow Bitcoin’s base of support beyond conservative Republicans like Senator Ted Cruz of Texas and Senator Cynthia Lummis of Wyoming, both of whom have publicly championed it.
					  
					  In November, at the North American Blockchain Summit in Fort Worth, Mr. Porter interviewed Senator Ron Wyden, Democrat of Oregon, about the benefits of blockchain technologies.
					  
					  Mr. Wyden spoke of the promise of a “digital dollar” and putting medical records on a blockchain, a digital ledger that records cryptocurrency transfers. But later, in an interview with The Times, Mr. Wyden said he opposed the state bills pushed by the Satoshi Fund, including the one in Arkansas, and the energy-intensive process required for Bitcoin mining.
					  
					  “It’s pretty clear that I’m not a big supporter,” he said. “Quite the opposite.”
					  
					  In Greenbrier, as the lawsuit wears on, Ms. Anderson said she and her neighbors have struggled to pay their lawyer. A fund-raiser in October brought the community together, but the proceeds barely put a dent in their debt. Still, she says, as long as they can afford it, they will fight the mine.
					  
					  “I don’t want to be run out of my home,” she said.
					  
					  David A. Fahrenthold contributed reporting from Washington and Michael Forsythe from New York.
					  - [Applications of Category Theory to Web Programming](https://omnivore.app/me/u-e-7-fd-5-f-38-c-1-b-2-11-ee-be-47-ffeef-2843-cdc-master-vilja--18d6949d824)
					  collapsed:: true
					  site:: [omnivore.app](https://omnivore.app/attachments/u/e7fd5f38-c1b2-11ee-be47-ffeef2843cdc/master_Vilja_Peter_2018copy.pdf)
					  date-saved:: [[02/02/2024]]
					  - ### Content
					  collapsed:: true
					  -
					  - [https://aaltodoc.aalto.fi/server/api/core/bitstreams/e0f5aefc-556f-4254-bf9d-03c22f930357/content](https://omnivore.app/me/https-aaltodoc-aalto-fi-server-api-core-bitstreams-e-0-f-5-aefc--18d69462845)
					  collapsed:: true
					  site:: [aaltodoc.aalto.fi](https://aaltodoc.aalto.fi/server/api/core/bitstreams/e0f5aefc-556f-4254-bf9d-03c22f930357/content)
					  date-saved:: [[02/02/2024]]
					  - ### Content
					  collapsed:: true
					  - Failed to fetch content
					  - [(9) X](https://omnivore.app/me/9-x-18d68dfedd3)
					  collapsed:: true
					  site:: [X (formerly Twitter)](https://twitter.com/kankodu/thread/1752581744803680680)
					  date-saved:: [[02/02/2024]]
					  - ### Content
					  collapsed:: true
					  - \#\# Thread
					  
					  A ![🧵](https://proxy-prod.omnivore-image-cache.app/0x0,sE-f4Z_wHe4DVfiSB-_nLZDelx3Gh2nYvtTxjkOHZBKU/https://abs-0.twimg.com/emoji/v2/svg/1f9f5.svg "Thread") on how yesterday's 
					  
					  attack worked. The protocol did everything right. They rounded in the protocol's favour whenever they should but one additional function, meant to only reduce the user's funds, ended up enabling the attack. How?
					  
					  [![Image](https://proxy-prod.omnivore-image-cache.app/0x0,sqhlbsrSfFT1gu-poPlYBYdk6rglXGjcdqsGF2x4zzrU/https://pbs.twimg.com/media/GFJrzsnbkAAaBMy?format=png&name=medium)](https://twitter.com/kankodu/status/1752581744803680680/photo/1)
					  
					  The protocol used shares mechanism to calculate the current debt of a user. Note: In the codebase, borrow shares are referred to as base/part and borrow assets amounts are referred to as elastic/amount.
					  
					  When a user borrows certain funds, they get minted borrow shares based on the current totalBorrowAssets and totalBorrowShares ratio. As interest is owed from the user, totalBorrowAssets increases without totalBorrowShares increasing and in turn it increases...
					  
					  ...the proportional amount that the user has to repay as well. The culprit function was the repayForAll function. This allowed anyone to repay everyone's debt. To accomplish this, the protocol reduced totalBorrowAssets without totalBorrowShares in repayForAll function.
					  
					  The attacker borrowed funds using a fllashloan and use the repayForAll function first.
					  
					  [![Image](https://proxy-prod.omnivore-image-cache.app/0x0,sbLiErWMRJQNcmwbFFDIGqk69E3hDLOjr_sIHKQRR76Y/https://pbs.twimg.com/media/GFJr0z3awAAAruT?format=jpg&name=large)](https://twitter.com/kankodu/status/1752581764021948430/photo/1)
					  
					  The attacker couldn't repay all of the amount as there was a check that totalBorrowAssets needed to be greater than 1000 ether. Regardless, the attacker repaid as much as they could, which reduced totalBorrowAssets such that totalBorrowAssets:totalBorrowShares was \~ 1:26.
					  
					  Now, the attacker repaid all the existing loans for all the borrowers. For the last remaining borrower, the user repaid all but 100 wei of shares. At that point, totalBorrowedAssets were 100, and totalBorrowShares were 3.
					  
					  [![Image](https://proxy-prod.omnivore-image-cache.app/0x0,svCZ3iVYOA1-5GvaT0IymUODm3EoxOMnd9JzPrePm7qQ/https://pbs.twimg.com/media/GFJr1kaasAEn6PL?format=jpg&name=medium)](https://twitter.com/kankodu/status/1752581778140012657/photo/1)
					  
					  The attacker repaid 1 wei of share 3 times, which meant totalAssets reduced to 0, and totalBorrowShares were 97\. Now the attacker started borrowing 1 wei from their account. At this point, TotalBorrowAssets is 1, and totalBorrowShares is 98.
					  
					  Attacker put up a very low amount of collateral (100 wei) and in a loop, started borrowing 1 wei of assets and repaying 1 wei of borrow shares.
					  
					  [![Image](https://proxy-prod.omnivore-image-cache.app/0x0,smkzT8O4qtmOJj3yAmO59iF2aI-49Xkz-IuZfDizq-38/https://pbs.twimg.com/media/GFJr2WAbMAA6JkN?format=jpg&name=medium)](https://twitter.com/kankodu/status/1752581791402373577/photo/1)
					  - [1.2 - Pulled Away from Blogs](https://omnivore.app/me/pulled-away-from-blogs-indie-microblogging-18d67ed194d)
					  collapsed:: true
					  site:: [book.micro.blog](https://book.micro.blog/pulled-away-from/)
					  labels:: [[indie microblogging]]
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - _“You were the captain of a ship, sailing aimlessly through the wilds of the Web. Occasionally you would drop anchor and stop to peruse all the great content that netizens were putting out into the world.” — [The Web Is Fucked](https://thewebisfucked.com/)_
					  
					  If you wanted to publish anything on the web in the early 2000s, you created a blog. Blogs had personality. People commented on each others blog, helping build loose communities. They met in person at events like the SXSW Interactive conference.
					  
					  Slowly, the rise of larger platforms pulled attention away from blogs. More and more former bloggers posted their content on social networks first.
					  
					  Anil Dash was interviewed by Matt Mullenweg [on the Distributed podcast](https://distributed.blog/2019/12/12/transcript-episode-16-anil-dash/), talking about blogging less often because there were other venues to post to like Twitter:
					  
					  > the biggest thing chipping away at it is having other venues and other platforms
					  
					  When I talked to Tantek Çelik for the interview in Part 3, he acknowledged this period as Twitter was taking off where even he stopped blogging:
					  
					  > Just even personally, the last blog post I wrote on my old blog was in August of 2008\. I did not have anything on my own site in 2009\. 2009 was a really weird transitional period, because I both saw that happening and I saw it happening to myself.
					  
					  By 2010, you could see this pattern across the web. Many of the pioneers of blogging had either completely stopped posting, or cut back their posts to longer essays a few times a year.
					  
					  What happened? Social networks were simply easier to post to, and the feeling of engagement in getting likes or replies was more compelling than publishing into the void of the blogosphere, wondering if anyone was listening.
					  
					  At the same time, blog comments were getting harder to manage. There was more comment spam. Bloggers started pointing their readers to social networks if they wanted to reply to a post, effectively offloading user registration and moderation to other centralized platforms.
					  
					  It wasn’t a stretch to embrace social networks because bloggers were already actively using some centralized platforms, like Flickr. If a blog was already leaning on Flickr for photo storage, it was a small step to go to other platforms for short text posts.
					  
					  Before Twitter was large enough and stable enough to dominate centralized microblogging, several competing social networks were launched with a focus on microblogging.
					  
					  ---
					  
					  Twitter now has over 300 million monthly active users. Centralized platforms have become a winner-take-all game because you can’t move your followers. Leaving Twitter or Facebook means starting over.
					  
					  Earlier it wasn’t clear Twitter would dominate. In 2007, Twitter was still small enough that you and all your friends could try a new service without feeling like you were leaving everything behind. Twitter was often flaky, with the “fail whale” as a reminder that maybe a better, more stable network existed elsewhere.
					  
					  There was Pownce with private posts and more sharing options. Ello and App.net as reactions to Twitter’s developer-hostile API. LiveJournal, MySpace, Jaiku, and Diaspora.
					  
					  Every one of these competitors had their own unique take on microblogging. How long should a post be? Should the friends model by asymmetrical, so anyone can follow anyone, or require approval of friend requests?
					  
					  In 2009, Facebook overtook MySpace [in unique visitors in the US](https://www.pcworld.com/article/523618/facebook%5Fovertakes%5Fmyspace%5Fin%5Fus.html). No other social network would reach the same scale until Instagram.
					  
					  ---
					  
					  Google Reader had become the most popular platform for subscribing to RSS feeds. It was free and easy to use, but aspects of its centralized nature such as comments and favorites were stuck in a silo, difficult to migrate away from.
					  
					  Paid services that are as popular as Google Reader aren’t usually discontinued as Google Reader was, but Google Reader wasn’t a paid service. It’s because Google Reader was free and ad-supported — but just a small part of Google’s business — that they were able to drop it.
					  
					  When Google Reader shut down, there was no migration plan to other RSS readers. Marco Arment, early Tumblr developer and creator of the podcast app Overcast, [blogged in 2013](https://marco.org/2013/03/14/baby-steps-replacing-google-reader) that developers needed to move quickly to fill the void left by Google, standardizing on a Reader-compatible API that could work with most apps without major API changes:
					  
					  > We need to start simple. We don’t have much time. And if we _don’t_ do it this way, the likely alternative is that a few major clients will make their own custom sync solutions that won’t work with any other company’s clients, which won’t bring them nearly as much value as it will remove from their users.
					  
					  A common API didn’t happen. Instead, we do have a few popular feed reader platforms like Feedbin and Feedly that tried to fill the void. Apps like NetNewsWire and Reeder have been updated to support multiple APIs.
					  
					  This modern feed sync ecosystem might be healthier than one dominated by a single player, but it created friction and doubt in what people should move to. It was easier to just use Twitter and convince yourself you weren’t missing anything from RSS feeds.
					  
					  Blogs survived in the background because they were still a better fit for people who wanted to own their content, carving out a little space for themselves on the web. The death of Google Reader was a reminder to indie-minded bloggers that ad-based platforms had advertisers as customers, not users.
					  
					  [Next: Leaving Twitter →](https://book.micro.blog/leaving-twitter/)
					  - [1.1 - Penn Station
					  ](https://omnivore.app/me/penn-station-indie-microblogging-18d67ec9e8a)
					  collapsed:: true
					  site:: [book.micro.blog](https://book.micro.blog/penn-station/)
					  labels:: [[indie microblogging]]
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - _“The station structure, designed after the Qual d’Orsay, Paris, but twice as large, will be 1,500 ft. in length by 500 ft. in width, three decked, inclose 25 tracks at tunnel level, which will be approached by gradual carriage drive and walkway.” — The Brooklyn Daily Eagle Almanac, 1906_
					  
					  When my family was visiting New York City a few years ago, we took a train out of Pennsylvania Station on the way up to Montreal for the second half of our vacation. It was raining a little as we walked from the hotel, but I thought we’d still have no trouble finding the station. After a few minutes we gave up and had to ask someone where the entrance was.
					  
					  We couldn’t find it because it looked like every other street corner in Manhattan. But it wasn’t always like that. It used to look like this:
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sknyiPoYnXNiwV32s1U3AOk00WI7YGYFa5RQ4IvVsw_4/https://book.micro.blog/uploads/2019/30b89f1d7d.jpg "Pennsylvania Station in the 1910s")
					  
					  The New York Times [recently described the old Penn Station this way](https://www.nytimes.com/2019/04/24/nyregion/old-penn-station-pictures-new-york.html):
					  
					  > With its swarming crowds and dust motes dancing in shafts of smoky light, the station was catnip to midcentury photographers, filmmakers, artists and architects. It was the architectural embodiment of New York’s vaulted ambition and open arms.
					  
					  In the 1960s, facing declining train usage and financial problems, the Pennsylvania Railroad sold the rights to everything above ground and the incredible station pictured above was demolished. It was only afterwards, when it actually happened, that everyone fully realized what they had lost. Determined to not let other beautiful architectural landmarks get destroyed, the city passed a law to restrict similar demolition. Grand Central Terminal was preserved because of the lesson learned from letting Pennsylvania Station go.
					  
					  I was thinking about this story — failing to do the right thing, but applying that knowledge to the _next_ thing — while re-reading [Marco Arment’s excellent post on the future of podcasting](https://marco.org/2016/05/07/apple-role-in-podcasting). In it, he lays out the technical details for how podcasting works today, and makes the case for leaving it alone. I especially like this part, on his determination to keep Overcast a sort of pure MP3 client:
					  
					  > By the way, while I often get pitched on garbage podcast-listening-behavioral-data integrations, **I’m never adding such tracking to Overcast.** Never. The biggest reason I made a free, mass-market podcast app was so I could take stands like this.
					  
					  I should have realized it earlier, but I don’t think I really connected all of Marco’s goals with Overcast until Daniel Jalkut and I had him on our podcast Core Intuition, [episode 200](http://coreint.org/200). We talked about many of these same themes as Marco was finishing up Overcast 2.0\. Marco talked about larger podcast software teams that are trying to lock down podcasting:
					  
					  > They’re trying to lock it down for themselves. Usually it’s: let’s build a closed platform, and our technology can power your podcasting. And it’s like all these buzzwords that really just mean we are trying to lock down this open medium to a centralized close thing that we control. And then we are the gatekeepers, and we have all the power, and all the money flows through us.
					  
					  Everywhere I looked there seemed to be a debate about the role of podcast aggregators like Spotify and Apple. There was a great discussion [on Upgrade](https://www.relay.fm/upgrade/88) with Myke Hurley and Jason Snell about this. It started [about halfway through](https://overcast.fm/+DeCngCgj8/43:07), with Myke describing the potential for Apple to lock down podcasting:
					  
					  > For there to be more data about listeners — like to know stuff about where you are in the world, demographic information, how old you are, to know if you’ve listened to the whole show, what parts you’ve skipped — for Apple to know this information, they would have to kind of lock down a lot of the way that podcasts work. They would probably need to be hosting the files and reserving them on their own. They would need to do more tracking.
					  
					  And [in a response to Marco on MacStories](https://www.macstories.net/stories/a-podcasting-divergence/), Federico Viticci wrote about the parallel trend in the web industry toward centralized services like Facebook and Medium that allow “content professionals” to monetize their writing. In doing so, those writers give up many of the benefits of the open web:
					  
					  > But the great thing about the free and decentralized web is that the aforementioned web platforms are optional and they’re alternatives to an existing open field where independent makers can do whatever they want. I can own my content, offer my RSS feed to anyone, and resist the temptation of slowing down my website with 10 different JavaScript plugins to monitor what my users do. No one is forcing me to agree to the terms of a platform.
					  
					  While the open web still exists, we dropped the ball protecting and strengthening it. Fewer people’s first choice for publishing is to start a web site hosted at their own domain. Like the destruction of Pennsylvania Station, sometimes you only know in hindsight that you’ve made a mistake. We were so caught up in Twitter and Facebook that we let the open web crumble. I’m not giving up — I think we can get people excited about blogging and owning their own content again — but it would have been easier if we had realized [what we lost](http://anildash.com/2012/12/the-web-we-lost.html) earlier.
					  
					  Reading posts like Marco’s and Federico’s, and listening to Jason and Myke on Upgrade, I’m convinced that podcasting will remain open because as a community _we know better now_. We can see the dominance of YouTube controlling nearly all web video. We can learn from the mistakes with the web and the threats of closed platforms, making sure that podcasting is preserved as a simple technology that no one controls.
					  
					  And we can take a fresh look at all the blogging platforms that have come before, learn from what worked well with previous social networks and what failed, and recommit to bringing the best parts back to today’s web. This time, prepared for the future.
					  
					  [Next: Pulled away from blogs →](https://book.micro.blog/pulled-away-from/)
					  - [0.4 -, The way forward · Indie Microblogging.](https://omnivore.app/me/the-way-forward-indie-microblogging-18d67e8ee5e)
					  collapsed:: true
					  site:: [book.micro.blog](https://book.micro.blog/the-way-forward/)
					  labels:: [[indie microblogging]]
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - _“Most important things in life are a hassle. If life’s hassles disappeared, you’d want them back.” — Hayao Miyazaki_
					  
					  The blog AltPlatform had published a number of articles about the open web and indie blogging. Brian Hendrickson [wrote about emerging protocols from the IndieWeb and Mastodon](http://web.archive.org/web/20170913190455/http://altplatform.org/2017/06/22/flip-the-iceberg/), and how these standards could eventually reach a scale that would “flip the iceberg” to become the more dominant way we communicate online:
					  
					  > Open source tools like WordPress, [1999.io](http://1999.io/) and Mastodon.social are creating many small networks of publishers, and popular tools like Twitter and Micro.blog could peer with them. If all of the social networks outside of Facebook interoperated at some level, they might eventually “flip the iceberg” and become the dominant form of social networking.
					  
					  Compatibility between new blog-focused platforms could eventually become bigger than any one social network. This compatibility comes from open standards. (I’ll talk more about the IndieWeb and Mastodon in parts 3 and 5.)
					  
					  It is daunting to create a new microblog platform — to compete with Twitter and Facebook, to go up against more established companies with better funding — and creating a new social network from scratch usually does not work. The huge platforms are [super-aggregators](https://stratechery.com/2017/defining-aggregators/), ad-based with little cost to acquire content or scale, and difficult to compete with network effects.
					  
					  For years many developers have wanted an alternative but have not been able to get mainstream traction. Developers and entrepreneurs with the best intentions, great talent, and a larger team than we have for Micro.blog.
					  
					  So to flip the iceberg, we must start with a simpler goal: encourage more people to blog. We must play the long game, building deliberately so that the foundation will last for years.
					  
					  It’s not about leaving Twitter and moving to the next platform. It’s about redistributing microblog posts across the web, with a diverse set of platforms.
					  
					  This redistribution is already happening. [Indie Map](https://indiemap.org/) took thousands of independent blogs and started mapping the relationships as people use their blog to link and reply across sites. Instead of a concentration of users on a single site like Twitter or Facebook, users are spread out, posting at sites they control.
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,scxLUkF4Wp72QH1EbCE1A66RgBFgovqNQNjiHTt1pTpE/https://book.micro.blog/uploads/2020/458ccd0c59.png)
					  
					  Blogging at a personal domain name is a kind of investment in the future of the web. It’s a statement that you value your own writing and are ready to contribute to making the web better.
					  
					  This book is divided into 6 major parts:
					  
					  * **Part 1: Rewind** takes us back to early social networks and blogging platforms. It also includes interview questions with Marco Arment about the early days of Tumblr, and a conversation with Leah Culver about Pownce.
					  * **Part 2: Foundation** outlines the basics of indie microblogging, with a quick-start guide for WordPress and how Micro.blog fits into the ecosystem of indie microblogs. There’s an overview of JSON Feed and an interview with NetNewsWire developer Brent Simmons.
					  * **Part 3: IndieWeb** is all about the IndieWeb movement. I’ll cover why we care about owning our content and the building block API standards of the IndieWeb. It’s capped off with an interview with IndieWeb co-founders Tantek Çelik and Aaron Parecki.
					  * **Part 4: Hypertext** covers the unique nature of web formats, focusing on photography, UI, and HTML.
					  * **Part 5: Decentralization** takes us to Mastodon, WebSub, and real-time notifications between blogs.
					  * **Part 6: Community** is about the intersection between blogs and platforms. The impact of harassment, misinformation, and politics on healthy communities. We end with a conversation with Micro.blog community manager Jean MacDonald.
					  
					  Getting millions of new bloggers to post to their own site won’t be easy. Nothing worthwhile ever is. It seems like a never-ending hassle to convince people to blog when Facebook onboards new users so effortlessly. But we’ve seen in the years since Micro.blog launched that this will work. There’s no question that more people are blogging today because of Micro.blog and the larger IndieWeb movement, and we’ve only scratched the surface.
					  
					  [Next: Part 1: Rewind →](https://book.micro.blog/rewind/)
					  - [https://book.micro.blog/the-way-forward/](https://omnivore.app/me/https-book-micro-blog-the-way-forward-18d67e8e8aa)
					  collapsed:: true
					  site:: [book.micro.blog](https://book.micro.blog/the-way-forward/)
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - Your link is being saved...
					  - [0.3 - mission to movement](https://omnivore.app/me/mission-to-movement-indie-microblogging-18d67e7f8aa)
					  collapsed:: true
					  site:: [book.micro.blog](https://book.micro.blog/mission-to-movement/)
					  labels:: [[indie microblogging]]
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - _“Progress depends on our changing the world to fit us. Not the other way around.” — Halt and Catch Fire_
					  
					  Basecamp started as the Chicago-based 37signals, a web design company known for pushing back against accepted conventions. They used to say that [copywriting is a form of user interface design](https://gettingreal.37signals.com/ch09%5FCopywriting%5Fis%5FInterface%5FDesign.php):
					  
					  > Great interfaces are written. If you think every pixel, every icon, every typeface matters, then you also need to believe every letter matters.
					  
					  The best products don’t just have marketing copy; they have a mission statement. They don’t just sell a tool; they sell a movement.
					  
					  Sometimes our products are confusing to new users — a UI that is too different, or trying to do too many things. These failures are an opportunity to improve beyond just bug fixes. Instead of only explaining what the product does, how can we pitch it in a way that strengthens a community around the idea, spreading through members in a more meaningful way than we can by ourselves.
					  
					  And unlike a one-way press release, a community is inherently two-way. Every mention of the idea is both marketing and feedback. Someone blogs about how they’re excited for the product, but also how they wish it had a certain missing feature. Someone in the press writes a review, but also with a list of pros and cons.
					  
					  This cycle means the product gets better. And if we’re thoughtful in that first approach to marketing copy, then every blog post, review, and tweet that follows is laced with a little part of our mission statement.
					  
					  Who doesn’t want to build products that resonate so well, that go from nice utilities or productivity apps to something our customers fall in love with?
					  
					  [Kyle Neath echoed this in a blog post](https://warpspire.com/posts/idea-businesses), writing that it’s about ideas, not products:
					  
					  > People want to be part of ideas. Being part of a company who builds a successful product is cool… but being part of an idea is a lot more attractive. If you can build a business where both your employees and your customers think they’re part of an idea, you’ve created something special.
					  
					  The venture capitalist (and blogger) Fred Wilson [wrote about](https://avc.com/2020/01/what-to-work-on/) focusing on work that is inspiring and that can have an impact:
					  
					  > You must work on something that inspires you and others, you must work on something with a significant impact, and you must do it in a way that makes getting where you want to go as easy as possible and keeps you there as long as possible.
					  
					  People are increasingly disillusioned with larger companies. When a company gets too big, that company inevitably forgets why they exist. Not even the greatest, mission-driven companies seem to be able to escape this fate. The only way to stay true to your roots is to stay small. And smaller companies will inherently discourage centralization.
					  
					  It might seem that short and often ephemeral posts have trained us with short-attention spans. To see the movement we must look over a longer period at the collection of all those posts — not just our own posts, but the potential for microblog posts all across the web.
					  
					  This book you’re reading is longer than I had intended, especially ironic given that its subject matter is short posts. But the goal is big. It’s not about any one new social network. It’s about a new way of thinking about publishing on the web.
					  
					  It’s not enough to have temporary, viral movements like `\#DeleteFacebook` or outrage over Elon Musk’s latest meme tweets. We need something sustainable that permanently changes the narrative.
					  
					  What is the mission for indie microblogging? There are four guiding themes in this book that we will keep returning to:
					  
					  * **Better features.** Learning from the user interface innovations of social networks — both the good choices and what we can do better.
					  * **Open standards.** How the work of the IndieWeb and even older blogging APIs can improve interoperability and freedom on the web.
					  * **Content ownership.** Why nearly everything starts with personal domain names.
					  * **Smaller social networks.** The technical overview of Micro.blog, Mastodon, and pushback against massive social networks.
					  
					  It’s the combination of all four themes that will move the web forward.
					  
					  [Next: The way forward →](https://book.micro.blog/the-way-forward/)
					  - [0.2 - Uses for a microblog · Indie Microblogging.](https://omnivore.app/me/uses-for-a-microblog-indie-microblogging-18d67e6779f)
					  collapsed:: true
					  site:: [book.micro.blog](https://book.micro.blog/uses-for-microblog/)
					  labels:: [[indie microblogging]]
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - _“You donʼt know if your idea is any good the moment itʼs created. Neither does anyone else. The most you can hope for is a strong gut feeling that it is. And trusting your feelings is not as easy as the optimists say it is. Thereʼs a reason why feelings scare us.” — Hugh MacLeod, [Ignore Everybody](https://hughcards.net/book/)_
					  
					  Sometimes it’s hard to put words down. Staring at a large, empty text box from blog software that wants you to give your post a title, as if every thought is fully formed before you know what to say. Doubt and imposter’s syndrome creep in.
					  
					  The solution is easier, quick posting. Just start writing without the pressure of getting it all right.
					  
					  People who haven’t been posting regularly to their own blog also often struggle with deciding what content should be on a social network and what should be posted to a microblog. My default is to post everything on my own site, and cross-post to other networks, which takes the guesswork out of where to post. But there are several types of content that are naturally well-suited to an indie microblog that you control.
					  
					  **Photos**
					  
					  Sharing photos is an important part of Micro.blog. I put a custom photo picker and filters in the original Micro.blog iOS app to encourage everyone to post photos to their blog, so that you end up with a great collection of your best photos at your own domain name. Because photos are square by default, they look great in the Micro.blog timeline, and with cross-posting Micro.blog can attach photos to your tweets or send posts to other social networks.
					  
					  Some people on Micro.blog focus almost exclusively on posting photos, like [Robert Brook](https://micro.blog/robertbrook). Some people have even created separate blogs just for their photos at their own domain name hosted on Micro.blog, like [burk.photos](http://burk.photos/). There’s [a special section of our curated Discover section](https://micro.blog/discover/photos) that features photos from more Micro.blog users.
					  
					  We’ll cover photos extensively in Part 4.
					  
					  **Linkblogging**
					  
					  Essentially two types of link blogs have evolved since the early days of blogging. The most traditional link blog can be seen in Dave Winer’s posts. These are links with a very short commentary. Many tweets are like this. In a way, this format is the purest form of microblogging.
					  
					  The second type of link blog starts to fall outside the limits of microblogging. Instead of just including a URL, authors use a quote from the linked material as the foundation for the post. The majority of Daring Fireball posts adopt this format. While author John Gruber is known for his full essays, those longer posts are infrequent today. He keeps his site active by linking to other interesting essays and tacking on his own brief opinion.
					  
					  **Books**
					  
					  Micro.blog has a Goodreads-inspired feature called Bookshelves that helps you track what books you’re reading or want to read. When you’re done reading a book, a quick click on the “New Post” button will prepare a microblog post with the book name, author, and link to the book details. It’s an easy way to share your favorite books and keep everything on your own blog.
					  
					  There’s also a books API and companion app for iOS and Android called Epilogue, plus third-party IndieWeb apps like IndieBookClub. If we put more of our content on our own blogs, services like Micro.blog can aggregate book data together, forming a more distributed version of centralized sites like Goodreads.
					  
					  **Travel blogs**
					  
					  Travel blogs tend to combine text about the trip and photos, with individual blog posts for days or major sections of the trip. This can be seen in microblogs like [this one from Mary Hatfield](https://marydhat.micro.blog/) as they travel and adventure across the world, but a microblog is also great for remembering short trips and vacations.
					  
					  We designed our Micro.blog companion app Sunlit for these kind of travel posts. The main interface on iOS allows grouping text and photos from multiple days into a blog post.
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sx_kVrlAFy3Vaoj9YoKfQwWo4fUd0e5TTUyeuYf_gFDQ/https://book.micro.blog/uploads/2020/51a9bd6263.png)
					  
					  Travel blogs can be longer posts, or short check-in posts with just the venue name. This can often be automated. Many people use the service [OwnYourSwarm](https://ownyourswarm.p3k.io/) to copy Swarm check-ins to a microblog.
					  
					  **Business blogs**
					  
					  A microblog can be used for a company’s news updates or product release notes. For Micro.blog, we have [news.micro.blog](https://news.micro.blog/) where we post about new features or server downtime. These posts are short and also easy to copy over to a company’s account on social media.
					  
					  One advantage for businesses is that unlike social media, a microblog can be completely branded with the company’s web site design and domain name. Posts can also be included automatically in a company’s main web site using Micro.blog’s [Sidebar.js](https://help.micro.blog/2016/sidebar-js/) script.
					  
					  **Conversations**
					  
					  Blog posts are a great way to get feedback about an idea. Conversations around a blog post can take the form of replies on Micro.blog, or comments on external blog posts like WordPress.
					  
					  See Part 6 for more about replies and building communities.
					  
					  **Microcasting**
					  
					  Micro.blog is about making short-form content you own as simple to post as a tweet because we believe blogging should be easier. Podcasting should be easier too.
					  
					  Everyone had a story to tell, but for some people it’s too daunting to even think about needing to talk for half an hour. Microcasts are short-form podcasts. Creating a microcast is fun because it’s much easier to record and edit than the longer podcasts we’re all used to.
					  
					  Chet Collins has [published a microcast](https://chetjcollins.com/chetcast) where he shares bits from his day with his kids. He wrote [on his blog](https://www.chetjcollins.com/2020/04/20/why-i-microcast.html) about why a podcast is such a nice format for capturing these memories:
					  
					  > Audio recordings, in an open format, are about as future-proof as you can get. Even more than that, these recordings deliver the actual sound of my children’s voices, their laughter, and their unfiltered thoughts. They are the perfect time capsule of my children, recorded and preserved for the future.
					  
					  Using a podcast essentially provides some structure, transforming audio snippets from everyday life into a format that can be easily reviewed later. It’s more organized than a digital junk drawer of random video clips, which for most people are unlikely to ever be edited. Podcasts inherently have a narrator to give context.
					  
					  **…and everything else**
					  
					  What you had for lunch. The movie you just watched. If it’s something that could go on Twitter or Facebook, it can be a microblog post at your own site.
					  
					  [Next: Mission to movement →](https://book.micro.blog/mission-to-movement/)
					  - [Demystifying Project Estimation](https://omnivore.app/me/demystifying-project-estimation-18d66150ce8)
					  collapsed:: true
					  site:: [substack.com](https://substack.com/inbox/post/140700152)
					  author:: Substack
					  date-saved:: [[02/01/2024]]
					  - ### Content
					  collapsed:: true
					  - As a former System Engineer, I often found project estimations frustrating and seemingly useless.
					  
					  However, as I progressed into management and leadership roles, I came to understand that the issue was not with the estimates themselves; it was more about how my managers were handling them.
					  
					  Estimating a project or the latest feature you aim to deliver holds incredible value, not only for the business your team serves but also for you and your team. In fact, estimations bring clarity and alignment, which are crucial for delivering quickly and with minimal stress.
					  
					  That's why today, I’d like to delve deeper into this topic and try to demystify project estimations in all their aspects.
					  
					  To achieve this, I wanted to go beyond my own experience as an Engineering Leader and I invited my friend Jordan Cutler, an expert from the development side, to co-author this article and provide a well-rounded perspective.
					  
					  So before everything, I'll let him introduce himself.
					  
					  Hey there! I’m Jordan, Senior Software Engineer and author of the [High Growth Engineer](https://careercutler.substack.com/?r=1to968) newsletter that reached 40k subscribers in less than 1 year. I also teach a course on Maven called Mid-level to Senior Engineer. I love providing **actionable** advice to help you achieve your goals based on real-world experience.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x971,syh61I1A-jAa0X3NuDxr1IpH_A1MZl2qP2rUYfnweE7Y/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc70b94da-cde6-4a67-8ffc-3835de9168e9_4608x3072.jpeg)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc70b94da-cde6-4a67-8ffc-3835de9168e9%5F4608x3072.jpeg)
					  
					  \#\# ❓What is an Estimate?
					  
					  Estimates are **informed predictions** or assessments about a project, a feature, or a task. Let me reiterate: predictions. Very rarely will an estimate be 100% accurate, and it's crucial that all teams working together (stakeholders included) are aligned on this principle.
					  
					  But why are estimates so frustrating?
					  
					  These are just a few reasons:
					  
					  * 🤝 **Lack of collaboration**: often, teams work in silos instead of collaborating to achieve a common objective.
					  * 🧠 **Lack of knowledge**: there's often a lack of mutual understanding in each other's fields.
					  * 🌐 **More teams involved**: it's already challenging to estimate projects when your team is the only one involved; imagine the complexity when external teams join the party.
					  * 🎯 **Estimations are just guesses**: estimating projects is a time-consuming process, and even when all factors are known, they are, at best, educated guesses.
					  
					  \#\#\# What Affects Estimates Confidence
					  
					  Estimates' reliability can be influenced by multiple factors. Based on my experience, these are the most frequent:
					  
					  * 🏞️ **Nature of the project**: do you have domain knowledge of the project? Is this an iteration or something completely new? Is the project involving R&D? These are all factors that highly affect estimates.
					  * 👩‍🔬 **Experience**: the more experience your team has in one field, the more they will be able to do accurate estimates.
					  * ⏱️ **Invested time**: building precise requirements, conducting tech discoveries, brainstorming together, and defining T-Shirt sizes of tasks involved in a project to prioritize them, are all activities that take time. The more time you invest, the more accurate your estimates will become.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x973,sXQuBHLtsKwT7CVXBIzriRiPiXwF96R8MZlynEjF2JNc/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc9ad01ca-7488-4bc1-a3d3-a560b55ef670_3000x2004.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc9ad01ca-7488-4bc1-a3d3-a560b55ef670%5F3000x2004.png)
					  
					  I find this last point (Invested Time) particularly important, as it should also determine the confidence level of your estimates. 
					  
					  How much time you should invest in estimating a project depends on the level of confidence you want to achieve within a certain project. 
					  
					  This will be influenced by how critical it is for your stakeholders to have an accurate timeline from the beginning.
					  
					  \#\# ⏱️ Should you Estimate Projects?
					  
					  The reality is that it depends on company culture and other factors. Personally (and this is how I approach it within my teams), I believe estimates are required in the following cases:
					  
					  * 🎯 **Prioritization**: when your stakeholders have to decide where to allocate resources.
					  * 🤝 **More parties are involved**: if you have to coordinate with many functions that are external to your team.
					  * ⏳ **Time-sensitive Projects**: there are projects that require being delivered in a precise amount of time due to other dependencies or external events you can’t control.
					  
					  If you look at these 3 cases, you’ll notice that they are essentially the norm when it comes to product development teams, leading me to say that you’ll need estimates in approximately 85% of cases. 
					  
					  However, there is one particular scenario where I often try to avoid any estimation: projects that are primarily focused or that involve a lot of R&D.
					  
					  R&D is inherently unknown, so attempting to estimate it becomes pointless and in some cases counterproductive. What I usually do with projects involving R&D is include them as part of the discovery process, effectively integrating them into the estimation itself.
					  
					  \#\#\# A Different Point of View ( )
					  
					  Believe it or not, estimates are in your benefit as an engineer.
					  
					  * If you’re leading a project with other engineers, having a “target date” **promotes accountability**.
					  * Having a “target date” makes it **clear to execs what you are working on** and the value of your work.  
					  * For example, I’ve been on “neverending” projects before with no clear end and senior leaders would assume nothing is getting done without a target date.
					  * Having a “target date” makes it easier to **decline scope creep**.  
					  * For example, you can say, “I’m ok with adding that into the scope, but we’ll need to push the estimate back a bit to make up for it. Is that ok with you?”  
					  * Or more aggressively, “If we add that in, we’d need to push our estimate back more than I’d like. I think we should hold off on adding it for now.”
					  
					  I refer to the estimate as a “target date” because that can be another way to view it.
					  
					  In an ideal world, you ship on your estimated date, but it might not always happen. Still, having that “target” is good for goal setting.
					  
					  \#\# 🔍 How to Estimate Projects?
					  
					  There are numerous ways and frameworks to estimate projects, and it would be challenging to cover them all. However, I have some principles that I believe are essential to follow:
					  
					  * **⏳ No Deadlines**: estimates should never be confused with deadlines. They are informed predictions that become more precise with increased time investment.
					  * **🔄 Update Your Estimates**: building on the first principle, don't hesitate to continuously update your estimates. Again, they are not rigid deadlines; they are there to guide teams towards successful project completion.
					  * **🔍 Split Projects/Features into Smaller Chunks**: this approach makes projects and features more manageable and enhances the accuracy of estimates.
					  
					  Regarding methodology, I have explored various approaches throughout my career, and what I can confidently say is that 'leaner is better.' No story-points, Fibonacci, endless sprint plannings, etc.
					  
					  Within my teams, this is what we do: 
					  
					  * 👕 We prefer to use **T-shirt sizes** to assess initiatives, along with **customer value** they bring, to prioritize them.
					  * 📊 We then create **rough time estimates** to visualize them on a Gantt chart (using Clickup), making dependencies and progress more visible.
					  * 📌 Then, we use **Kanban** approach to get things done.
					  * 🔄 We regularly **update our estimates** as we progress until the release.
					  
					  It's nothing too fancy; we keep it simple and stress-free. 
					  
					  I recently engaged in a [meaningful conversation](https://www.linkedin.com/feed/update/urn:li:activity:7153316045993238528?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A7153316045993238528%2C7153318844478959616%29&replyUrn=urn%3Ali%3Acomment%3A%28activity%3A7153316045993238528%2C7153361335198576642%29&dashCommentUrn=urn%3Ali%3Afsd%5Fcomment%3A%287153318844478959616%2Curn%3Ali%3Aactivity%3A7153316045993238528%29&dashReplyUrn=urn%3Ali%3Afsd%5Fcomment%3A%287153361335198576642%2Curn%3Ali%3Aactivity%3A7153316045993238528%29) on LinkedIn with [Tom Hill](https://www.linkedin.com/in/thedeskoftom/) (Head of Software at MHR) where he wrote about the concept of '**slices of value**', which aligns with the idea of breaking initiatives into smaller, more manageable components, based on their value. 
					  
					  Tom also explained how they use historical data to estimate how many slices of value they can accommodate within a fixed timeframe (cycle time), resulting in increasingly accurate estimates over time. 
					  
					  I suggest to give it a read!
					  
					  I'll now leave it to Jordan to explain how he tackles estimates in a practical walkthrough.
					  
					  \#\#\# A Practical Use Case ( )
					  
					  I’ll run you through my exact process for building an estimate using an example.
					  
					  Let’s say we want to build this row of filters on Pinterest: 
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x736,sQsrBHXkVl87UVZpbsUAFgYbVsNtoxxOGo5tz25TSxrw/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F03be93d1-1836-4d6b-9547-97a1eeb45caf_3000x1517.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F03be93d1-1836-4d6b-9547-97a1eeb45caf%5F3000x1517.png)
					  
					  When you click it, it “refines” your search to include what you clicked.
					  
					  For example, clicking “Outdoor” in the above image searches for “outdoor christmas decoration.”
					  
					  First, I write out each of the non-trivial behaviors to support, along with their prioritization using the [MoSCoW framework](https://www.productplan.com/glossary/moscow-prioritization/).
					  
					  1. **Pill UI component (Must have):** Create the UI for a Clickable Pill component
					  2. **Pill text (Must have):** Determine pill texts to use and link the search upon click
					  3. **Pill row (Must have):** Display pills in a responsive row
					  4. **Images in pills (Should have):** Display a preview image in each pill
					  5. **Pill colors (Could have):** Display pills using a rotating set of colors
					  
					  You could have broken it down differently. The general idea is to focus on the behaviors from the user’s perspective.
					  
					  Your breakdown will be a bit more technical than what your product manager creates. You can use what your product manager made as a starting point for these requirements you want to implement.
					  
					  The combination of your requirements should support the use cases your product manager defined in a “Product Spec” or “[Product Requirements Document](https://www.productplan.com/glossary/product-requirements-document/).”
					  
					  **Now, you need to determine how much time each of these requirements will take.**
					  
					  So you break down each of those requirements into their tasks.
					  
					  Here’s an example using the “Pill text” requirement:
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x819,srD31qBj5QVXn1s1MJ-BxvbS2diuiUbnhh4GvZ63PXxI/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3c2b02e5-9d18-4d76-809f-db3d7a5f1d89_3000x1688.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3c2b02e5-9d18-4d76-809f-db3d7a5f1d89%5F3000x1688.png)
					  
					  I create a table for each one, give a t-shirt size, and then give a rough estimate including tests and PR reviews.
					  
					  Finally, I add up all the times. In this case, we get 2 weeks.
					  
					  **After doing this breakdown, it gets added to the original “Requirements” table.**
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x863,sa9Q9kW1P-HSGMr26LdSuxFP2IiNajwZe-9CzRvPxG2Y/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F548d8f0f-f05e-4ea7-83fb-22a9d6d8d44f_3000x1778.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F548d8f0f-f05e-4ea7-83fb-22a9d6d8d44f%5F3000x1778.png)
					  
					  After doing the breakdown for each requirement, you’ll have a variety of ranges.
					  
					  Only doing “Must haves,” the project could take 4 weeks.
					  
					  But if you include everything everyone wants to do, it would be 8.5 weeks.
					  
					  Now, you’re able to look like a pro 😎 by prioritizing only the most valuable work and cutting scope where possible.
					  
					  As a final step, I put this estimate table in the technical design doc to showcase how long each part will take.
					  
					  Then I incorporate time for rolling out the feature, PTO, holidays, etc. in the “Milestones” section of that design doc.
					  
					  Here’s what that looks like:
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/1456x863,snc-asg4cUGiR25xKBgHvZHfLHDM_6EaJ-ICcaUzqMcU/https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6d7366b7-4663-46f8-88c2-bc03d3d7734e_3000x1778.png)](https://substackcdn.com/image/fetch/f%5Fauto,q%5Fauto:good,fl%5Fprogressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6d7366b7-4663-46f8-88c2-bc03d3d7734e%5F3000x1778.png)
					  
					  Now you’re able to feel confident about the milestones you set, know you’re not missing anything, and make effective prioritization decisions.
					  
					  > _If you liked this in-depth deep dive, you can get access to the full template, example, and plenty of other templates like this by [becoming a paid subscriber to High Growth Engineer here](https://careercutler.substack.com/?r=1to968). Jordan also teaches this live in my Maven course, [Mid-level to Senior Engineer](https://maven.com/jordan-cutler/mid-level-to-senior)._
					  
					  \#\# 📐 Project Estimation Tips
					  
					  In conclusion, I'd like to provide you with some additional tips for effectively estimating projects:
					  
					  * 📚 **Educate your team and stakeholders:** cultivate a culture where all parties work together, feel involved, and are aligned on what is an estimate.
					  * 📋 **Estimate the estimates:** as we've seen, estimating projects can be a time-consuming task. That's why I believe it's a good practice to include the time required for estimation within the estimation process itself.
					  * 🔗 **Consider dependencies:** one of the most common errors I see when people estimate projects is missing dependencies. They estimate their work without taking other parties into consideration, resulting in inaccurate estimations.
					  * 💡 **Consider people's availability:** another common error I have observed many times is not considering people's availability when creating project timelines. Your team and others involved may have holidays or could be working on other projects, so it's important to factor this in when making your estimates.
					  * 🚧 **Add a 15% buffer:** I mentioned earlier that estimates are not deadlines, but some people may still treat the estimated timeline as a fixed deadline. Adding a buffer helps to avoid unrealistic expectations.
					  * 🔄 **Review and adjust regularly:** as the project progresses, regularly review your estimates and adjust them based on actual progress and any new information that arises.
					  * 📊 **Use Historical Data:** look at similar past projects to understand how long tasks took and what resources were needed. This can provide a realistic basis for your estimates.
					  
					  \#\# ✌️ That’s all folks
					  
					  That's all for today! As always, I would love to hear from my readers (and if you've made it this far, you're definitely one of the bravest). Please don't hesitate to connect with me on [LinkedIn](https://www.linkedin.com/in/nicolaballotta) and send a message. I always respond to everyone!
					  - [The State of Dapps on IPFS: Trust vs. Verification | IPFS Blog & News](https://omnivore.app/me/the-state-of-dapps-on-ipfs-trust-vs-verification-ipfs-blog-news-18d568f8590)
					  collapsed:: true
					  site:: [IPFS Blog & News](https://blog.ipfs.tech/dapps-ipfs/)
					  author:: Daniel Norman
					  date-saved:: [[01/29/2024]]
					  date-published:: [[01/28/2024]]
					  - ### Content
					  collapsed:: true
					  - ![The State of Dapps on IPFS: Trust vs. Verification](https://proxy-prod.omnivore-image-cache.app/0x0,sytbrTzz0w2VWtWs9x1Jl401VTOD3juuuSX73ArmmFKo/https://blog.ipfs.tech/assets/img/header.33f34147.png)
					  
					  \#\# [\#](\#preface) Preface 
					  
					  This blog post provides a comprehensive overview of the current landscape of dapps on IPFS through the lens of trust and verifiability. Given the nuance and breadth of this topic, this blog post is rather long.
					  
					  For easier navigation, use the [table of contents](\#contents).
					  
					  \#\# [\#](\#contents) Contents 
					  
					  * [Trust vs. verification in dapps](\#trust-vs-verification-in-dapps)
					  * [The benefits of IPFS for (d)app developers and users](\#the-benefits-of-ipfs-for-dapp-developers-and-users)
					  * [Primer on web app architectures: SPAs, MPA, PWA and dapps](\#primer-on-web-app-architectures-spas-mpa-pwa-and-dapps)  
					  * [The client-server spectrum](\#the-client-server-spectrum)  
					  * [SPA and MPA can be easily published to IPFS](\#spa-and-mpa-can-be-easily-published-to-ipfs)  
					  * [SPA and MPA can also be PWA](\#spa-and-mpa-can-also-be-pwa)  
					  * [Dapps](\#dapps)  
					  * [How dapps get chain state](\#how-dapps-get-chain-state)
					  * [Publishing dapps: approaches and trade-offs](\#publishing-dapps-approaches-and-trade-offs)  
					  * [Without IPFS](\#without-ipfs)  
					  * [Publishing to IPFS](\#publishing-to-ipfs)
					  * [Loading dapps from IPFS: approaches and trade-offs](\#loading-dapps-from-ipfs-approaches-and-trade-offs)  
					  * [From a public gateway](\#from-a-public-gateway)  
					  * [With a local IPFS node](\#with-a-local-ipfs-node)  
					  * [With a local IPFS node & IPFS Companion browser extension](\#with-a-local-ipfs-node--ipfs-companion-browser-extension)  
					  * [With the Brave browser](\#with-the-brave-browser)
					  * [When running a Kubo node is not an option](\#when-running-a-kubo-node-is-not-an-option)
					  * [What if content addressing were native to the web?](\#what-if-content-addressing-were-native-to-the-web)
					  * [In-browser CID verification with JavaScript](\#in-browser-cid-verification-with-javascript)  
					  * [Browser constraints](\#browser-constraints)  
					  * [Approaches to IPFS in the browser](\#approaches-to-ipfs-in-the-browser)  
					  * [Helia and IPFS in the browser](\#helia-and-ipfs-in-the-browser)  
					  * [Verifying top-level pages, sub-resources, and async data](\#verifying-top-level-pages-sub-resources-and-async-data)  
					  * [Fetching and verifying async data with Helia](\#fetching-and-verifying-async-data-with-helia)  
					  * [Making Helia lighter and developer-friendly](\#making-helia-lighter-and-developer-friendly)  
					  * [Helia in a Service Worker](\#helia-in-a-service-worker)  
					  * [Local app installer](\#local-app-installer)
					  * [Most users don’t use CIDs directly](\#most-users-dont-use-cids-directly)
					  * [Naming systems and mutable pointer](\#naming-systems-and-mutable-pointer)  
					  * [DNSLink](\#dnslink)  
					  * [Ethereum Name System (ENS)](\#ethereum-name-system-ens)  
					  * [IPNS](\#ipns)
					  * [Conclusion](\#conclusion)
					  
					  If you are a decentralized web app (dapp) developer, there’s a good chance that you already publish the frontend of your dapp to IPFS. However, today, even if you do so, your users cannot benefit from the integrity IPFS provides without running their own IPFS node. If your users’ browser isn’t verifying that the frontend's source and resources match the CID you published, they are exposed to a wider attack surface, which can lead in the worst case to stolen funds.
					  
					  The root of the problem lies in the **difficulty users face verifying the integrity of dapps deployed to IPFS in a browser without running an IPFS node**. This hurdle means that many users are **trusting** —often unknowingly– a specific IPFS gateway. This goes against the IPFS principle that [**verification matters** (opens new window)](https://specs.ipfs.tech/architecture/principles/\#verification-matters) and puts users at risk.
					  
					  Over the last couple of months, the [IPFS Shipyard (opens new window)](https://ipfs-shipyard.org/) team has been working with several teams in the dapp ecosystem to understand the challenges they face and the broader problem space. With the formation of the [IPFS Dapps Working Group (opens new window)](https://github.com/ipfs/dapps-wg/) by the IPFS Shipyard team along with the [Liquity team (opens new window)](https://www.liquity.org/) and the IPFS community, we aim to address some of the immediate pain points faced by the dapp developers and users and provide better tooling. One of the main goals is to **establish verified retrieval as the norm for retrieving CIDs on the web**.
					  
					  This is not a new problem. Making CIDs native to the web platform has been a long-time goal of the IPFS project and remains a core goal of the [IPFS Ecosystem Working Group (opens new window)](https://blog.ipfs.tech/2023-introducing-the-ecosystem-working-group/). Making CIDs native to the web is an arduous road that involves wide-spanning collaboration with stakeholders including standard bodies, spec writers, browser vendors, and IPFS implementors.
					  
					  It’s worth noting that _trustlessness_ is an aspirational property of dapps, but a misleading term because trust cannot be completely eliminated. A better way to look at this is in terms of **verifiability** that content addressing enables, in addition to less reliance on authority, e.g. PKI, DNS and authoritative servers. Moreover, the web’s trust model is deeply ingrained and rooted in the [**Same-origin policy** (opens new window)](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin%5Fpolicy). One of the engineering challenges the working group faces is to meet the goal above within the boundaries and constraints of the same-origin policy.
					  
					  > **Note:** while this blog post is heavily focused on Dapps, almost all of it applies to any static web application that can be published to IPFS. That is, Progressive Web Apps (**PWA**), Single Page Applications (**SPA**) and any app that does not rely on server side rendering.
					  
					  \#\# [\#](\#the-benefits-of-ipfs-for-d-app-developers-and-users) The benefits of IPFS for (d)app developers and users
					  
					  IPFS is supposed to provide several benefits for web app developers and users:
					  
					  * **Resilience & censorship resistance:** by having multiple copies of the frontend’s CID on the IPFS network you ensure that even if multiple providers are unavailable or censored, the frontend is still retrievable and usable. In the most extreme case, it’s enough for there to be a single provider for content to be retrievable.
					  * **End-to-end integrity:** as long as a user of your Dapp has the CID you shared, they can be sure they are running the exact code that you published by **verifying** locally. Local verification is crucial since Dapps interact with a blockchain and malicious code can lead to loss of user funds. Integrity is adjacent to trustlessness — because verification eliminates the need to trust the source of data.
					  * **Legal and regulatory compliance:** as regulatory bodies adopt regulation, e.g. **[MiCA (opens new window)](https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/markets-crypto-assets-regulation-mica),** which applies to crypto assets and their Dapps, the degree to which services are decentralized comes under scrutiny. While the legal question cannot be answered by the blog post (this is not legal advice), IPFS, through the former two points, should provide the means to maximize decentralization and do so provably.
					  * **Data portability, no vendor lock-in, and [credible exit (opens new window)](https://subconscious.substack.com/p/credible-exit):** once you onboard data and make it content-addressed with CIDs, you are free to move it between implementations, services, and jurisdictions, theoretically, without paying the onboarding cost again.
					  
					  The reality, however, is more complex because there are various approaches to publishing and fetching dapps from IPFS that make subtle trade-offs between **trustlessness, resilience, UX, and performance.**
					  
					  In the next section, we’ll take a look at web app architectures, and what dapps are, and then dive deeper into the actual approaches you see in the wild.
					  
					  \#\# [\#](\#primer-on-web-app-architectures-spas-mpa-pwa-and-dapps) Primer on web app architectures: SPAs, MPA, PWA and dapps
					  
					  The rapidly evolving nature of web application architectures has given birth to many terms, abbreviations, and web development frameworks. This section will attempt to provide a high-level overview of some of the main web app architecture patterns, how dapps and how they relate to publishing to IPFS. If you are already familiar with these, feel free to skip ahead.
					  
					  \#\#\# [\#](\#the-client-server-spectrum) The client-server spectrum
					  
					  Today’s web applications can be seen as being positioned somewhere on a **server-client spectrum** regarding where the logic (rendering, authorization, processing user input) lives. On the server end of the spectrum, you have server-rendered apps where most logic is encapsulated in the server, e.g. WordPress, Laravel, and Ruby on Rail apps. On the client end, you have Single Page Applications (SPA), where all routing and rendering logic is client side. SPAs typically have a single entry index.html with a JavaScript bundle that routes all use. Once the JS is loaded, it takes over rendering, navigation, and network (asynchronously submitting user input) responsibilities. Another approach that sits somewhere in the middle is the multi-page application (MPA) with a pre-rendered HTML file per route that typically contains only the necessary JS for the given route.
					  
					  It’s worth noting that many modern web development frameworks support more than one architecture and even the blending of different approaches on a per-route basis. For example, a [Next.js supports both MPAs with Static Exports (opens new window)](https://nextjs.org/docs/pages/building-your-application/deploying/static-exports) and server-side rendering.
					  
					  [web.dev has a useful article that delves into this topic in more detail (opens new window)](https://web.dev/articles/rendering-on-the-web).
					  
					  Because SPA and MPA are statically generated, you can easily host them on any server that can serve static files (HTML, JS, CSS, etc.). That makes them a great fit for publishing on both traditional CDNs and IPFS.
					  
					  \#\#\# [\#](\#spa-and-mpa-can-also-be-pwa) SPA and MPA can also be PWA
					  
					  A progressive web app ([PWA (opens new window)](https://developer.mozilla.org/en-US/docs/Web/Progressive%5Fweb%5Fapps/Guides/What%5Fis%5Fa%5Fprogressive%5Fweb%5Fapp)), is a web app that runs in a browser while providing a user experience like that of a platform-specific native app, e.g. the ability to function offline, update in the background, and send notifications to the OS.
					  
					  The key thing to understand is that what makes an app a PWA (web app manifest and and service worker) is complementary to MPAs and SPAs. [In other words, both SPA and MPA architectures can be used to build a PWA. (opens new window)](https://web.dev/learn/pwa/architecture)
					  
					  \#\#\# [\#](\#dapps) Dapps
					  
					  Dapps, short for Decentralised Apps, is an umbrella term for applications deployed as a smart contract to a blockchain. Since interacting with smart contracts directly can be a clunky experience, dapps are typically comprised of two components:
					  
					  * Smart contracts deployed to a smart contract blockchain like Ethereum (and other EVM chains, e.g. Filecoin).
					  * A frontend to interact with those contracts from the web browser. Typically the frontend will be a static app (SPA/MPA) that is deployed to a CDN and/or published to IPFS.
					  
					  \#\#\# [\#](\#how-dapps-get-chain-state) How dapps get chain state
					  
					  In this architecture, the static site will need to fetch blockchain state, specifically the state associated with the Dapp’s smart contracts. This can be done using the following approaches:
					  
					  * The most naive is to use the [Ethereum JSON-RPC API (opens new window)](https://ethereum.org/en/developers/docs/apis/json-rpc/) which every Ethereum execution client/node exposes. The Ethereum execution client is software that keeps an up-to-date full state by synching with the rest of the network and updating the state tree every time a new block is produced. Dapps that rely on the JSON-RPC API will either use a hosted Ethereum node provider like Quicknode, Alchemy, and Infura, or run their own node.
					  * Since the JSON-RPC API is usually too low-level with unindexed data to provide rich frontend functionality, many Dapps will instead query an additional indexing layer like [The Graph (opens new window)](https://thegraph.com/). The Graph is a protocol for indexing and querying blockchain data and makes it possible to efficiently query chain state using GraphQL. For example, Uniswap uses [this approach (opens new window)](https://docs.uniswap.org/api/subgraph/overview) to fetch data from the Uniswap smart contracts.
					  
					  In both approaches, retrieval of chain state happens as async requests invoked by the frontend code.
					  
					  It’s also pretty common for the smart contracts and frontend of a dapp to be open source, which allows anyone to audit the code. For example, Uniswap publishes both the source of their [smart contracts (opens new window)](https://github.com/Uniswap/v3-core) and [interface (opens new window)](https://github.com/Uniswap/interface) on [GitHub (opens new window)](https://github.com/Uniswap).
					  
					  One thing to note is that the degree of decentralization of a Dapp can vary based on several factors that are beyond the scope of this post.
					  
					  **As a general rule of thumb, it’s only as decentralized as the least decentralized component.**
					  
					  This blog post is mostly concerned with the frontend component and the different ways that IPFS enables maximizing decentralization of its distribution and trustlessness. Throughout the post, we’ll be looking at Uniswap as an example, given its importance and the amount of money it secures. That being said, the insights apply to any Dapp of this structure.
					  
					  \#\# [\#](\#publishing-dapps-approaches-and-trade-offs) Publishing dapps: approaches and trade-offs
					  
					  \#\#\# [\#](\#without-ipfs) Without IPFS
					  
					  The most naive and common approach is to just deploy the dapp to a web server or CDN like Vercel, AWS, Netlify, and Cloudflare.
					  
					  For example, [the Uniswap team deploys (opens new window)](https://github.com/Uniswap/interface/actions/runs/7036990525/job/19150799879\#step:11:1) their frontend to Cloudflare Pages (as well as IPFS as we'll see in the section below) and makes the latest version available at https://app.uniswap.org.
					  
					  From the perspective of a user, this is arguably the most user-friendly and performant (with Cloudflare’s CDN), at the cost of being the least verifiable.
					  
					  Dapp users have no way to verify that the source of the frontend matches the original code published on GitHub. Moreover, the reliance on DNS comes with risks such as fat finger human errors and other DNS attacks, e.g. DNS takeovers — these are admittedly unlikely but important to consider.
					  
					  | Rating                           |   |
					  | -------------------------------- | - |
					  | Verifiable                       | ❌ |
					  | Resilience/Censorship resistance | ❌ |
					  
					  \#\#\#\# [\#](\#at-the-mercy-of-multiple-authorities) At the mercy of multiple authorities
					  
					  Another thing to consider about deploying without IPFS is that the app must comply with **the terms of service of multiple authorities**:
					  
					  1. “.org” TLD owner
					  2. “uniswap.org” DNS Registrar
					  3. “uniswap.org” DNS Nameserver (when different to the registrar)
					  4. Certificate Authority (CA) that provides TLS cert for https://app.uniswap.org
					  5. CDN/HTTP Hosting service serves the site traffic
					  6. ISP/[AS (opens new window)](https://en.wikipedia.org/wiki/Autonomous%5Fsystem%5F%28Internet%29) of the HTTP Hosting provider
					  
					  \#\#\# [\#](\#publishing-to-ipfs) Publishing to IPFS
					  
					  From the perspective of a Dapp developer, publishing to IPFS is pretty straightforward. You take your frontend build and add it to your IPFS node or to a pinning service. Publishing to IPFS results in a CID which represents that version of the frontend.
					  
					  Uniswap, for example, has automated [publishing to IPFS with Pinata (opens new window)](https://github.com/Uniswap/interface/actions/runs/7036990525/job/19150799879\#step:8:21) as part of their build process, and they publish the CID for each version in the release:
					  
					  ![Uniswap release on GitHub](https://proxy-prod.omnivore-image-cache.app/0x0,s-nwt9tqPCSq-06NIp0TCO0yC-MokSFVm4DA728fhgss/https://blog.ipfs.tech/assets/img/uniswap-release.ed6beeff.png)
					  
					  One thing to consider here is where the CID is generated. In the ideal case, this should happen in the build process, e.g. by packing the build outputs into a CAR file with a CID in the build process. If you upload the raw files to a pinning service, you are trusting the pinning service to generate the CID for the input data.
					  
					  To increase the resilience and censorship resistance of your deployed app, you can pin the CID to more than one pinning service or IPFS node.
					  
					  | Rating                           |    |
					  | -------------------------------- | -- |
					  | Verifiable                       | 👍 |
					  | Resilience/Censorship resistance | 👍 |
					  
					  \#\# [\#](\#loading-dapps-from-ipfs-approaches-and-trade-offs) Loading dapps from IPFS: approaches and trade-offs
					  
					  \#\#\# [\#](\#from-a-public-gateway) From a public gateway
					  
					  With the CID of a dapp at hand, you can load the frontend from any public IPFS gateway directly in your browser, e.g.:
					  
					  https://bafybeihwj3n7fgccypsiisijwuklg3souaoiqs7yosk5k5lc6ngnhnmnu4.ipfs.dweb.link/
					  
					  https://bafybeihwj3n7fgccypsiisijwuklg3souaoiqs7yosk5k5lc6ngnhnmnu4.ipfs.cf-ipfs.com/
					  
					  The problem with this approach is that you haven’t verified the response, so you don’t know if you the response **matches the CID.** In effect, you are **trusting the gateway** to return the correct response.
					  
					  Another minor challenge that arises is that each version you load and each gateway you load it from will have a different origin, so any local state the dapp relies on in localStorage or IndexedDB will be tied to that specific version of the dapp (CID) at that specific gateway, i.e., `bafy1.ipfs.cf-ipfs.com` is a different origin to `bafy1.ipfs.dweb.link` even though they are the same CID.
					  
					  | Rating                           |                     |
					  | -------------------------------- | ------------------- |
					  | Verifiable                       | ❌                   |
					  | Resilience/Censorship resistance | 👍 (other gateways) |
					  
					  > **Note:** Resilience depends on whether the content has been cached and the number of providers/copies on the network
					  
					  Note that some Dapp developers will run their own dedicated gateways either on their infrastructure or by using a dedicated gateway service, e.g. Pinata, Filebase. This can result in better performance. As for trust, it shifts it around, and without verification, the users are left to decide whether they trust the gateway operator.
					  
					  \#\#\# [\#](\#with-a-local-ipfs-node) With a local IPFS node
					  
					  If you have a local IPFS node installed, e.g. [Kubo (opens new window)](https://docs.ipfs.tech/install/command-line/) or [IPFS Desktop (opens new window)](https://docs.ipfs.io/install/ipfs-desktop/), then you can use the IPFS gateway exposed by your local node. It looks as follows: http://bafybeihwj3n7fgccypsiisijwuklg3souaoiqs7yosk5k5lc6ngnhnmnu4.ipfs.localhost:8080/
					  
					  Note that it will only work if you are running an IPFS node with the gateway listening on port 8080)
					  
					  When you open this URL, the local IPFS node will handle content routing (finding providers for the CID), fetching the content, and verification.
					  
					  The main hurdle with this approach is that it requires running an IPFS node in addition to typing a long URL. But you get the full benefits of **verifiability. The only thing you need to trust is the CID you received is indeed the one published by Uniswap.**
					  
					  From a performance perspective, it may be slow on the first load, but once fetched and cached locally, a given CID will essentially load instantly.
					  
					  | Rating                           |    |
					  | -------------------------------- | -- |
					  | Verifiable                       | 👍 |
					  | Resilience/Censorship resistance | 👍 |
					  
					  (Depends on whether the gateway has it cached and the number of providers/copies on the network)
					  
					  \#\#\# [\#](\#with-a-local-ipfs-node-ipfs-companion-browser-extension) With a local IPFS node & IPFS Companion browser extension
					  
					  [IPFS Companion (opens new window)](https://docs.ipfs.tech/install/ipfs-companion/) is a browser extension that simplifies access to IPFS resources and adds browser support for the IPFS protocol. It allows you to type IPFS protocol URLs, i.e., `ipfs://bafy...` directly in the browser, thereby improving the UX.
					  
					  Under the hood, IPFS companion handles IPFS URLs and redirects them to the gateway of the local IPFS node.
					  
					  | Rating                           |    |
					  | -------------------------------- | -- |
					  | Verifiable                       | 👍 |
					  | Resilience/Censorship resistance | 👍 |
					  
					  IPFS Companion also supports [DNSLink (opens new window)](https://dnslink.dev/) resolution (DNSLink is covered in more detail at the bottom of the article). When a user visits a URL, Companion will check for a [DNSLink (opens new window)](https://dnslink.dev/) DNS record for the hostname and, if found, will load the dapp from the local gateway instead of the remote origin. In this instance, trust is only delegated for the DNS resolution (hostname → CID).
					  
					  \#\#\# [\#](\#with-the-brave-browser) With the Brave browser
					  
					  [Brave Browser (opens new window)](https://brave.com/ipfs-support/) comes with native support for IPFS URLs that can be resolved by a public gateway or the built-in IPFS node. The latter is practically the same as the previous approach with a local IPFS node and the IPFS companion browser extension, though the user experience is better because it works out of the box.
					  
					  | Rating                           |    |
					  | -------------------------------- | -- |
					  | Verifiable                       | 👍 |
					  | Resilience/Censorship resistance | 👍 |
					  
					  \#\# [\#](\#when-running-a-kubo-node-is-not-an-option) When running a Kubo node is not an option
					  
					  All the previous examples that are verifiable depend on the user running an IPFS node, typically Kubo, a Go-based implementation of IPFS that runs as a separate process to the browser. Having a separate process frees you from the constraints imposed by browsers and affords more resources for the node to establish more connectivity to other IPFS nodes.
					  
					  ![Local kubo gateway](https://proxy-prod.omnivore-image-cache.app/0x0,sICI0KLHqcuKd6vqRsOGdJTAQjXnbj6Eq6Aq4LJBVA1o/https://blog.ipfs.tech/assets/img/local-kubo-node.91856c23.png)
					  
					  **However, running a Kubo node comes at the cost of a higher barrier to adoption, and in the case of mobile phones, is not an option.**
					  
					  \#\# [\#](\#what-if-content-addressing-were-native-to-the-web) What if content addressing were native to the web?
					  
					  In an ideal world, content addressing would be native to the web, but what could that look like?
					  
					  Content addressing is a paradigm shift to security on the web that is rooted in the same-origin policy. In many ways, this requires a reimagining of parts of the web which is beyond the scope of this post (though if you’re interested, check out Robin Berjon’s work on the [Web Tiles (opens new window)](https://berjon.com/web-tiles/).)
					  
					  Browser vendors tend to be defensive about adding new browser APIs and implementing specs for a myriad of reasons: maintenance burden, security risks, and lack of financial incentive.
					  
					  At a minimum, native IPFS support would involve the ability for the web browser itself to verify the integrity of content-addressed sites. A glimpse into that future is presented by `ipfs://` and `ipns://` in [Brave (opens new window)](https://brave.com/ipfs-support/) and [ipfs-chromium (opens new window)](https://github.com/little-bear-labs/ipfs-chromium/). It may arrive sooner in mainstream browsers if WebExtensions like [IPFS Companion (opens new window)](https://github.com/ipfs/ipfs-companion) can [register a protocol handler that is backed by a Service Worker (opens new window)](https://github.com/ipfs/in-web-browsers/issues/212).
					  
					  ![ipfs protocol handler backed by a service worker](https://proxy-prod.omnivore-image-cache.app/0x0,soMnyRWwQVaR8zgRuqRmgtSWPr9pV3wHqZcxYjPNdEM8/https://blog.ipfs.tech/assets/img/service-worker-gateway.1ca27e15.png)
					  
					  Since it will likely take time to come to fruition, the next section below will cover the pragmatic interim approaches to in-browser verified retrieval of CIDs.
					  
					  \#\# [\#](\#in-browser-verified-retrieval-of-cids) In-browser verified retrieval of CIDs
					  
					  To understand the emerging landscape of approaches to IPFS in the browser, it’s crucial to first understand some of the inherent constraints of the browser.
					  
					  \#\#\# [\#](\#browser-constraints) Browser constraints
					  
					  Browsers are sandboxed runtime environments that place critical constraints for using IPFS:
					  
					  * Limits on the type (WebSockets, WebRTC, WebTransport) and number of connections a browser tab can open and/or [fail to dial before being blocked or throttled (opens new window)](https://github.com/ipfs/in-web-browsers/issues/211). This can hinder content routing DHT traversals and content retrieval connections.
					  * If a website is in a [secure context (opens new window)](https://developer.mozilla.org/en-US/docs/Web/Security/Secure%5FContexts) when served over HTTPS, like most websites today, you are constrained to only opening connections to origins with a CA-signed TLS certificate, something that peers in the IPFS network rarely have. As you’ll see, there are two exceptions to this, namely WebTransport and WebRTC, that we’ll look into in the next section.
					  * Limits on the resources an inactive browser tab consumes, i.e., when you keep a tab open but it becomes inactive by moving to a different tab.
					  
					  \#\#\# [\#](\#approaches-to-ipfs-in-the-browser) Approaches to IPFS in the browser
					  
					  From a high level, several threads of work remove the need to run a Kubo node:
					  
					  * [**Trustless Gateway** (opens new window)](https://specs.ipfs.tech/http-gateways/trustless-gateway/): a _subset_ of the [path-gateway (opens new window)](https://specs.ipfs.tech/http-gateways/path-gateway/) that allows for light IPFS clients to retrieve both the CIDs bytes and verification metadata (the Merkle DAG), thereby allowing you to [verify its integrity without trusting the gateway (opens new window)](https://docs.ipfs.tech/reference/http/gateway/\#trustless-verifiable-retrieval).
					  * [Delegated routing (opens new window)](https://docs.ipfs.tech/concepts/how-ipfs-works/\#how-content-routing-works-in-ipfs): a mechanism for IPFS implementations to use for [offloading content routing, peer routing, and naming to another server over HTTP (opens new window)](https://specs.ipfs.tech/routing/http-routing-v1/). This allows browsers to skip traversing the DHT and opening many connections in the process.
					  * [WebTransport (opens new window)](https://connectivity.libp2p.io/\#webtransport): a new browser API to open persistent duplex connections from the browser in a similar fashion to WebSockets. But in contrast with WebSocket, [WebTransport supports self-signed certificates (opens new window)](https://connectivity.libp2p.io/\#webtransport?tab=certificate-hashes), allowing its use in a p2p setting without reliance on certificate authorities. WebTransport support was also added to Kubo over a year ago, which in theory means that browsers should be able to connect to any arbitrary Kubo node even in a [Secure Context (opens new window)](https://developer.mozilla.org/en-US/docs/Web/Security/Secure%5FContexts).
					  * WebRTC Direct: though WebRTC was designed for browser-to-browser, it can also be used for [browser-to-server connectivity (opens new window)](https://connectivity.libp2p.io/\#webrtc?tab=browser-to-standalone-node) without trusted TLS certificates (see [spec (opens new window)](https://github.com/libp2p/specs/blob/master/webrtc/webrtc-direct.md)). This is useful in browsers like Safari and Firefox where WebTransport might not be available (as of Q1 2024).
					  * [Service Worker API (opens new window)](https://developer.mozilla.org/en-US/docs/Web/API/Service%5FWorker%5FAPI): a browser API that allows, among other things, intercepting network requests in web applications for either caching or providing offline functionality. Service workers can be used to implement a caching and verification layer by intercepting HTTP requests to IPFS gateways in existing apps that already use IPFS gateways without verifying.
					  
					  [**Helia** (opens new window)](https://helia.io/) is where most of the active work is happening and implements many of these approaches for better IPFS support in the browser.
					  
					  \#\#\# [\#](\#helia-and-ipfs-in-the-browser) Helia and IPFS in the browser
					  
					  [Helia (opens new window)](https://github.com/ipfs/helia) is a lean, modular TypeScript implementation of IPFS that can run in server JS runtimes, e.g. Node.js and Deno, as well as in the browser. To explain browser-specific use-cases, this section will focus solely on **Helia in the browser.**
					  
					  From a high level, Helia can do two main things in the browser:
					  
					  * **Manage content-addressed data:** serializing user input and objects into content-addressable representation like dag-json or UnixFS (typically referred to as codecs in IPLD land), and packing CAR files.
					  * **Verified retrieval of CIDs**: e.g. given a CID, find providers for it, fetch it and verify the data for it. Helia can retrieve CIDs using both [Bitswap (opens new window)](https://specs.ipfs.tech/bitswap-protocol/) (over libp2p) and the [Trustless Gateway (opens new window)](https://specs.ipfs.tech/http-gateways/trustless-gateway/) (over HTTPS).
					  
					  > **Note:** the short-lived nature of a browser tab makes it **unsuitable for providing CIDs to the network**. Even though in theory, Helia is capable of this, it's not recommended. The most practical approach to publishing CIDs from the browser is to delegate that to a long-running IPFS node, either by uploading directly to a [pinning service (opens new window)](https://docs.ipfs.tech/concepts/persistence/\#pinning-services) or uploading CIDs to a self-hosted IPFS node.
					  
					  \#\#\# [\#](\#verifying-top-level-pages-sub-resources-and-async-data) Verifying top-level pages, sub-resources, and async data
					  
					  An important distinction to make in web applications is between top-level pages, sub-resources, and async resources and how they can be verified:
					  
					  * **Top-level pages** refers initial HTML payload that is returned to the first request by the browser to a given address and bootstraps the loading of an app. For example, the `index.html` file in a given version of the IPFS website: [bafybeidfqp36qutohidaaapir743mvjefv5ipkbrvqx3li3x6vm47vrdam (opens new window)](https://explore.ipld.io/\#/explore/bafybeidfqp36qutohidaaapir743mvjefv5ipkbrvqx3li3x6vm47vrdam/index.html).  
					  **Verification:** as discussed above, this is currently only possible with a local IPFS node that does top level verification when you load a CID via the local gateway, i.e. `cid.ipfs.localhost:8080`.
					  * **Sub-resources** refer to resources loaded after the initial HTML of the page was loaded, like a JS, CSS, and image files files that are included in script tags of the initial HTML. These resources may be from the same or other origins (unless explicitly prohibited by the [Content security policy (opens new window)](https://web.dev/articles/csp) set by the server).  
					  **Verification:** Either by loading the top level CID from a local gateway and ensuring that sub-resources are also loaded from the local node by using relative path.  
					  Another way relies on a feature called [Subresource Integrity (SRI) (opens new window)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource%5FIntegrity) that ensures the browser verifies the hash of `<script>` and `<link>` elements with the integrity attribute, however, this has limited utility for CIDs since the SHA256 hash matches the hash in the CID only if the resources were encoded with raw leaves and fit into a single IPFS Block; because [IPFS chunks files before hashing and may result in different hashes (opens new window)](https://docs.ipfs.tech/concepts/faq/\#why-doesn-t-my-sha-hash-match-my-cid).  
					  Another way, which is explored in more detail below, is to abstract much of the verification and fetching of CIDs into service workers, which allows you to intercept network requests and verify resources.
					  * **Async data** refers to data that is fetched asynchronously during the runtime of the app with the `fetch` API, e.g. JSON returned from an API.  
					  **Verification:** possible by using Helia or one of the abstractions on top of Helia to fetch CIDs. Like sub-resources, this can be abstracted into a service worker, so that the application code is just making fetch requests to relative path style gateways, e.g. `/ipfs/[CID]` in the app.
					  
					  ℹ️ **Today, Helia can fetch and verify async data and sub-resources. However, top-level verification without deeper browser integration remains an open engineering problem that the [IPFS Dapps working group (opens new window)](https://ipfs.fyi/dapps-wg) is working on.**
					  
					  \#\#\# [\#](\#verified-retrieval-data-with-helia) Verified retrieval data with Helia
					  
					  Let’s look at a real-world example, and how you could add Helia (or another library) to add verification. The Uniswap frontend makes a bunch of trusted async fetch requests to the Cloudflare IPFS gateway without verifying the response.
					  
					  One of them is to the following URL: `https://cloudflare-ipfs.com/ipns/tokens.uniswap.org` whose response is a JSON object of the tokens supported by Uniswap. This URL contains a [DNSlink](\#dnslink) (which is covered in more detail below) to resolve to a CID. For the sake of simplicity, let's assume that we already have the resolved CID: `bafybeia5ci747h54m2ybc4rf6yqdtm6nzdisxv57pk66fgubjsnnja6wq4`.
					  
					  The code for fetching this token list JSON from a trusted gateway looks along the lines of :
					  
					  With Helia, fetching and verifying the CID could look as follows:
					  
					  The example above is more convoluted than necessary because the JSON is encoded as UnixFS, which is the default encoding for files and directories in IPFS. When working with JSON, it's better to to encode the data with one of `json`, `dag-json`, or `dag-cbor` codecs which are more suitable and provide better ergonomics for working with JSON data.
					  
					  To demonstrate, here's an example with the same token list JSON encoded as `json` which has the CID `bagaaieracglt4ey6qsxtvzqsgwnsw3b6p2tb7nmx5wdgxur2zia7q6nnzh7q`
					  
					  See how these two compare below:
					  
					  This is more involved than the `fetch` API, but comes with all the benefits of IPFS: data is verified and can be fetched from more than one gateway, thereby increasing resilience.
					  
					  \#\#\# [\#](\#making-helia-lighter-and-developer-friendly) Making Helia lighter and developer-friendly
					  
					  To make it easier for developers to adopt Helia in dapps that lean heavily on gateways, we've been working on a couple of improvements:
					  
					  * [Configurable block brokers (opens new window)](https://github.com/ipfs/helia/pull/280): a generic interface for resolving CIDs to blocks. Allows developers to choose (and even implement their own) block fetching approach for their app, e.g. Trustless Gateways, Bitswap, or a combination of the two. [Released in Helia v2.1.0 (opens new window)](https://github.com/ipfs/helia/releases/tag/helia-v2.1.0)
					  * [@helia/http (opens new window)](https://github.com/ipfs/helia/issues/289): A browser optimised version of Helia that leans on trustless gateways and delegated routing to enable verified retrieval. This was the package that was used in the examples above.
					  * [@helia/verified-fetch (opens new window)](https://github.com/ipfs/helia/issues/348): A library that would provide a similar interface to the [Fetch API (opens new window)](https://developer.mozilla.org/en-US/docs/Web/API/Fetch%5FAPI) and accept native `ipfs://` and `ipns://` URIs and function like an IPFS gateway. We intend for it to serve as a drop-in replacement for `fetch` requests to trusted gateways.
					  
					  \#\#\# [\#](\#helia-in-a-service-worker) Helia in a Service Worker
					  
					  Another thread of work involves a [Service Worker (opens new window)](https://github.com/w3c/ServiceWorker/blob/main/explainer.md) registered by the app that intercepts CID requests to gateways (that are unverified) and uses Helia to fetch and verify. This works for sub-resources and async data and assumes that the app already fetches CIDs from a trusted IPFS gateway, e.g. `fetch('/ipfs/[CID]')...` , because they can be detected and handled by the service worker.
					  
					  From a technical perspective, the service worker is tied to the app’s origin and registered by the app’s code. Helia is imported and handles CID requests by fetching the raw blocks of the requested CID from trustless gateways (or directly from peers with supported transports), verifying, and caching.
					  
					  It’s worth noting that caching is one of the primary reasons that service workers allow intercepting HTTP requests. Since CIDs are immutable, they make for an easy cache.
					  
					  The benefit of this approach is that it can be adopted by apps that already rely on trusted gateways without significant architectural changes.
					  
					  Check out the [Helia service worker gateway repo (opens new window)](https://github.com/ipfs-shipyard/helia-service-worker-gateway) to learn more about this approach or try it out on https://helia-service-worker-gateway.on.fleek.co/.
					  
					  \#\#\# [\#](\#local-app-installer) Local app installer
					  
					  The local app installer approach was recently laid out in a [blog post (opens new window)](https://www.liquity.org//blog/decentralizing-defi-frontends-protecting-users-and-protocol-authors) by the Liquity team. The idea is that you have a static web app that serves as a local installer which facilitates the fetching and verifying of dapps directly in the browser. The local app installer consists of PWA and utilizes a service worker with the ENS client library and Helia to resolve ENS names, download and verify dapps and cache them locally.
					  
					  ![Local-app-installer](https://proxy-prod.omnivore-image-cache.app/0x0,sATBPHzxAndjgI29KJE98QreHCHgSvzQ5bF7W-LFi0dc/https://blog.ipfs.tech/assets/img/local-installer.93e590a6.png)
					  
					  Top level integrity remains a challenge for verifying the initial installer PWA code. To address this, the Liquity team is exploring packaging the installer as part of a browser extension.
					  
					  It’s worth pointing out that in this approach, each locally installed dapp must still be isolated into its own origin. The challenge here is that the initial payload (for the first load) for each origin, must still come from somewhere, i.e. a trusted server. Following initial payload, the frontend must only be fetched and verified once because it’s locally cached by the service worker.
					  
					  For this reason, along with the inherent challenges of the web security model laid out earlier in this post, it’s useful to think about trust as a spectrum. In this approach trust is minimised to the initial interaction. To delve deeper into this approach, check out Liquity’s blog [post (opens new window)](https://www.liquity.org/blog/decentralizing-defi-frontends-protecting-users-and-protocol-authors).
					  
					  \#\# [\#](\#most-users-don-t-use-cids-directly) Most users don’t use CIDs directly
					  
					  For the sake of simplicity, we assumed throughout this post that the starting point for a user is a CID, but in reality, this is rarely the case.
					  
					  CIDs are long and not very human-readable, so they tend to be abstracted from the user. Moreover, because CIDs represent an immutable version of the frontend, giving users the latest versions requires something like a persistent address that can be updated upon every release.
					  
					  \#\# [\#](\#naming-systems-and-mutable-pointer) Naming systems and mutable pointer
					  
					  There are three common approaches to this problem that provide a **stable identifier** that can change upon version releases. The following is a high level comparison:
					  
					  * **DNSLink**  
					  * **What are they:** A DNS TXT record points to a specific CID.  
					  * **Human friendly:** 👍  
					  * **Verifiable:** 👎  
					  * **Example name:** [blog.ipfs.tech (opens new window)](http://blog.ipfs.tech/) (technically `_dnslink.blog.ipfs.tech`)  
					  * **Integration with the IPFS:** through IPFS gateways under the `/ipns` namespace: [ipfs.io/ipns/blog.ipfs.tech/ (opens new window)](http://ipfs.io/ipns/DNS.NAME) or using subdomain resolution: [https://blog-ipfs-tech.ipns.cf\-ipfs.com/ (opens new window)](https://blog-ipfs-tech.ipns.cf-ipfs.com/)
					  * **Ethereum Name System** (**ENS):**  
					  * **What are they:** records for a `.ETH` name are stored on-chain and can point to any URL or CID, e.g. `ipfs://bafy...`  
					  * **Human friendly:** 👍  
					  * **Verifiable:** Potentially  
					  * **Example name:** `vitalik.eth`  
					  * **Integration with the IPFS:**  
					  * **IPFS path gateways:** under the `/ipns` namespace: [ipfs.io/ipns/vitalik.eth (opens new window)](http://ipfs.io/ipns/vitalik.eth)\`  
					  * **Subdomain gateways:** subdomain resolution (dots become dashes): [vitalik-eth.ipns.dweb.link (opens new window)](https://vitalik-eth.ipns.dweb.link/)  
					  * Using an ENS resolver like [eth.link (opens new window)](http://eth.link/) or eth.limo: [vitalik.eth.limo (opens new window)](https://vitalik.eth.limo/)
					  * **IPNS**  
					  * **What are they:** mutable pointers based on public keys and signed IPNS records pointing to a CID. Typically published to the DHT, though IPNS is transport agnostic and can be resolved and advertised using the delegated routing HTTP API.  
					  * **Human friendly:** 👎  
					  * **Verifiable:** 👍  
					  * **Example name:** `k51qzi5uqu5dhp48cti0590jyvwgxssrii0zdf19pyfsxwoqomqvfg6bg8qj3s`  
					  * **Integration with the IPFS:** through IPFS gateways  
					  * Path resolution: `https://cloudflare-ipfs.com/ipns/k51qzi5uqu5dhp48cti0590jyvwgxssrii0zdf19pyfsxwoqomqvfg6bg8qj3s`  
					  * Subdomain resolution : `https://k51qzi5uqu5dhp48cti0590jyvwgxssrii0zdf19pyfsxwoqomqvfg6bg8qj3s.ipns.dweb.link/`
					  
					  Some of these approaches can be combined, and there are some crucial security implications to each of the approaches and the way they are implemented.
					  
					  In the next paragraph, we’ll dive into the details and trade-offs of how each of these approaches.
					  
					  \#\#\# [\#](\#dnslink) DNSLink
					  
					  [DNSLink (opens new window)](https://dnslink.dev/) uses DNS [TXT (opens new window)](https://en.wikipedia.org/wiki/TXT%5Frecord) records in the `_dnslink` subdomain to map a DNS name, such as `blog.ipfs.tech` to an IPFS path, e.g. `/ipfs/bafy..`
					  
					  The main benefit of DNSLink is that it relies on all existing DNS infrastructure and tooling to provide stable human-friendly names that can be updated. The main drawback of DNSLink is that it comes with the same risks and attack surface associated with DNS records mentioned earlier in the post, most notably is the lack of verifiability. This can potentially be addressed by things like DNSSec and querying multiple DNS resolvers.
					  
					  For example, the Spark UI from the MakerDAO ecosystem is published to IPFS and uses DNSLink. Their DNSLink TXT record is `_dnslink.app.spark.fi` and has the value set to (at the time of writing):
					  
					  `dnslink=/ipfs/bafybeihxc3olye3k2z4ty6ete7qe6mvtplq52ixpqgwkaupqxwxsmduscm`
					  
					  DNSLinks can be resolved in a browser in ways:
					  
					  * Using an IPFS gateway, under the ipns namespace, e.g. [ipfs.io/ipns/blog.ipfs.tech/ (opens new window)](http://ipfs.io/ipns/DNS.NAME) or to ensure origin isolation, with the subdomain gateway would be [https://blog-ipfs-tech.ipns.dweb.link (opens new window)](https://blog-ipfs-tech.ipns.dweb.link/). (when using the subdomain gateway, dots are converted to dashes to avoid origin and TLS certificate complexity).
					  * Directly with the DNS name when its pointing to an IPFS Gateway. The IPFS gateway will resolve the DNSLink based on the `host:` header, e.g. https://app.spark.fi/.
					  
					  \#\#\# [\#](\#ethereum-name-system-ens) Ethereum Name System (ENS)
					  
					  ENS is a crypto native on-chain domain registry. Records for a `.ETH` namespace can be purchased and configured on-chain, by interacting with the ENS smart contracts.
					  
					  Each ENS name can have multiple records to link different profiles, e.g. GitHub, Twitter, and IPFS CIDs. The `contenthash` field can be used to point to a `ipfs://bafy...` URL, as specified [ENSIP-7 (opens new window)](https://docs.ens.domains/ens-improvement-proposals/ensip-7-contenthash-field).
					  
					  While ENS has a lot of similarities with DNS, like the dot-separated hierarchical structure, it is a fundamentally different system. Most notably, `.eth` is not a valid TLD in DNS, which means that it doesn’t natively resolve in most browsers.
					  
					  To address this challenge, several solutions have emerged to allow easily resolving `.eth` domains in the browser:
					  
					  * Cloudflare operates [eth.link (opens new window)](http://eth.link/), which allows resolving ENS names with a content hash by appending `.link` to the ENS name. For example, [vitalik.eth.link (opens new window)](http://vitalik.eth.link/) will load the content hash set on `vitalik.eth`.  
					  Under the hood, eth.link uses EthDNS to access information from ENS via DNS. In other words, it provides a DNS interface to the on-chain ENS registry. eth.link also provides a [DNS-over-HTTPS (opens new window)](https://en.wikipedia.org/wiki/DNS%5Fover%5FHTTPS) endpoint to perform DNS resolution of .eth records: `https://eth.link/dns-query`. For example, `curl -s -H "accept: application/dns-json" "https://eth.link/dns-query?name=vitalik.eth&type=TXT"` will return the ENS records of `vitalik.eth`.
					  * [Eth.limo (opens new window)](http://eth.limo/) is a similar service to [eth.link (opens new window)](http://eth.link/) that functions similarly, e.g. [vitalik.eth.limo (opens new window)](http://vitalik.eth.limo/).
					  * Using an IPFS gateway, under the `ipns` namespace, e.g. [ipfs.io/ipns/vitalik.eth (opens new window)](http://ipfs.io/ipns/vitalik.eth) (path resolution) or [vitalik-eth.ipns.dweb.link (opens new window)](http://vitalik-eth.ipns.dweb.link/) for subdomain resolution (when using the subdomain gateway, dots are converted to dashes to avoid origin and TLS certificate complexity).  
					  Under the hood, the IPFS gateway treats these the same way as DNSLink, but resolves `.eth` TLD via special ENS2DNS bridges (the default one is DoH at `resolver.cloudflare-eth.com`, [configurable in Kubo (opens new window)](https://github.com/ipfs/kubo/blob/master/docs/config.md\#dnsresolvers)).
					  * The Metamask browser plugin will automatically redirect .eth addresses to an IPFS gateway, as described above.
					  * The Brave browser supports `.eth` domains and resolves them using the Cloudflare EthDNS resolver.
					  
					  \#\#\#\# [\#](\#verifiability-of-ens) Verifiability of ENS
					  
					  The fact that ENS domains are registered is on-chain makes them verifiable in principle. However, in the solutions laid out above, trust is delegated to a trusted server which handles the resolution of the ENS name to the CID, e.g. [eth.limo (opens new window)](http://eth.limo/), or the DoH resolver at https://resolver.cloudflare-eth.com/dns-query.
					  
					  ENS names can be resolved in the browser using the Ethereum RPC API by retrieving the state from the chain, howerver, trust is just shifted to the Ethereum RPC API endpoint.
					  
					  A more verifiable approach would be to use an Ethereum light client, like [Helios (opens new window)](https://github.com/a16z/helios) or [eth-verifiable-rpc (opens new window)](https://github.com/dappnetbby/eth-verifiable-rpc), to verify ENS state using merkle proofs and the Ethereum state root hash, though this is still experimental and far from a common pattern in dapps.
					  
					  \#\#\# [\#](\#ipns) IPNS
					  
					  IPNS is a system for creating [cryptographically verifiable mutable pointers (opens new window)](https://specs.ipfs.tech/ipns/ipns-record/) to CIDs known as **IPNS names**, for example, [k51qzi5uqu5dhp48cti0590jyvwgxssrii0zdf19pyfsxwoqomqvfg6bg8qj3s (opens new window)](https://cid.ipfs.tech/\#k51qzi5uqu5dhp48cti0590jyvwgxssrii0zdf19pyfsxwoqomqvfg6bg8qj3s) is a base36-encoded IPNS name with its public key in-line. The public key can be used to verify corresponding IPNS records, which point to a CID and are signed by the private key. In other words, an IPNS name can be thought of as stable link that can be updated over time.
					  
					  IPNS names are key pairs that are not human-friendly (like DNS and ENS), so while they offer a stable pointer that can change over time, you still need to get the IPNS name from _somewhere_.
					  
					  A pretty common pattern is for ENS names to point to an IPNS name. Since updating ENS names requires paying gas for the on-chain transaction, this can be avoided by pointing the ENS name to an IPNS name, and updating the IPNS name to a new CID, upon new releases or updates.
					  
					  Like CIDs, IPNS names can be resolved using IPFS gateways, either in a [verifiable (opens new window)](https://specs.ipfs.tech/http-gateways/trustless-gateway/\#ipns-record-responses-application-vnd-ipfs-ipns-record) or trusted way. Trusted resolution is as simple as adding the name to the URL: https://cloudflare-ipfs.com/ipns/k51qzi5uqu5dhp48cti0590jyvwgxssrii0zdf19pyfsxwoqomqvfg6bg8qj3s. Verified IPNS resolution is a bit [more involved (opens new window)](https://specs.ipfs.tech/ipns/ipns-record/\#record-verification), but can be done end-to-end with Helia in the browser as follows:
					  
					  \#\# [\#](\#conclusion) Conclusion
					  
					  If you reached this far, congratulations. Hopefully, this blog post gave you an overview of the state of dapps on IPFS and the ongoing efforts to make verified retrieval of CIDs the norm.
					  
					  While trust remains central to the web, leaning on the verifiability of CIDs is a net win for both dapp developers and users.
					  
					  As we make more progress on the [@helia/verified\-fetch (opens new window)](https://github.com/ipfs/helia/pull/392) library, we will publish more guides and examples demonstrating its broad applicability in dapps.
					  
					  If you’re a dapp developer or user using IPFS, your input is valuable. We invite you to join the [IPFS Dapps Working Group (opens new window)](https://ipfs.fyi/dapps-wg) and help us shape the future of dapps on IPFS.
					  
					  \#\# Comments
					  - [Informal Blog - Reflecting on our CometBFT 2023 Journey: A Retrospective](https://omnivore.app/me/informal-blog-reflecting-on-our-comet-bft-2023-journey-a-retrosp-18d48d2a97a)
					  collapsed:: true
					  site:: [informal.systems](https://informal.systems/blog/reflecting-on-our-cometbft-2023-journey-a-retrospective)
					  labels:: [[cosmos-sdk]] [[cometbft]]
					  date-saved:: [[01/26/2024]]
					  date-published:: [[12/14/2023]]
					  - ### Content
					  collapsed:: true
					  - [![Informal Systems Logo](https://proxy-prod.omnivore-image-cache.app/0x0,slzZkKtvBdItujiIrMCGK8mL8cKFMRqxT6VPsMlhiS0Y/https://informal.systems/images/logo-informal.svg)](https://informal.systems/)
					  
					  \#\#\#\#\# Table of Contents
					  
					  The year 2023 has been an unforgettable year for the CometBFT team, and as the year ends, we would like to reflect on the most important events that took place during this year. 
					  
					  \#\#\# CometBFT debut
					  
					  Copy link to this section
					  
					  At the beginning of the year, we officially announced that Informal Systems would be responsible for [maintaining a fork of Tendermint Core ](https://x.com/informalinc/status/1613580954383040512) and [launching CometBFT ](https://informal.systems/blog/cosmos-meet-cometbft). 
					  
					  The team at Informal Systems assigned some of their most experienced engineers from Cosmos to work on this project. Although there were many unknowns and challenges, the team was eager and optimistic about being the project steward. 
					  
					  The Interchain Stack currently hosts almost 100 live blockchains with a combined market capitalization worth billions of dollars, all of which have widely embraced Tendermint consensus and CometBFT as their base layer. Despite the significant responsibility it entails, we are proud to have been chosen as stewards of CometBFT. 
					  
					  Our extensive contributions to the Cosmos Ecosystem have provided us with the knowledge and expertise to ensure that CometBFT continues to thrive under our care.
					  
					  \#\#\# First Official Release
					  
					  Copy link to this section
					  
					  In February, we announced the first official release, CometBFT [v0.34 ](https://github.com/cometbft/cometbft/blob/v0.34.27/CHANGELOG.md\#v03427). Our primary objective was to make it as easy as possible for users to switch from Tendermint Core v0.34 by making it fully compatible with the Tendermint Core v0.34 release series.
					  
					  In order to ensure that CometBFT works as expected, we started running more extensive QA with each new major release, starting with v0.34\. This allowed us to produce detailed [QA results ](https://github.com/cometbft/cometbft/tree/main/docs/qa) for each major release, giving developers insight into how those releases perform in 200-node networks, how they perform relative to previous releases, and reducing chances of regression. We also changed our major release process to [always include pre-releases ](https://github.com/cometbft/cometbft/blob/main/RELEASES.md\#pre-releases) such that it is clear to our users as to when a new release has been through QA.
					  
					  \#\#\# Building blocks of ABCI++ (ABCI 1.0)
					  
					  Copy link to this section
					  
					  In March, we achieved another significant milestone, a month after our previous one. We released CometBFT [v0.37 ](https://github.com/cometbft/cometbft/blob/v0.37.0/CHANGELOG.md\#v0370), which included ABCI 1.0\. This version introduced two new ABCI methods: PrepareProposal and ProcessProposal. With these new methods, we gave application developers more control over how blocks are built.
					  
					  \#\#\# Full ABCI++ (ABCI 2.0)
					  
					  Copy link to this section
					  
					  During the third quarter of this year, we launched ABCI 2.0 - the full implementation of ABCI++. The latest release (v0.38) introduced new ABCI methods which enable vote extensions \- a powerful new feature that allows application developers to address use cases such as validator-based price oracles and threshold-encrypted transactions. These methods, ExtendVote and VerifyVoteExtension,allow the application to add arbitrary data to pre-commit votes. These vote extensions are available to the proposer(s) of the next block height.
					  
					  Another significant change was the combination of the BeginBlock, DeliverTx, and EndBlock into a new method called FinalizeBlock. This release also included several other features, changes, bug fixes, and improvements.
					  
					  It marked the final release of ABCI++ - a significant evolution of ABCI. This project [began as a vision ](https://github.com/tendermint/spec/pull/254) several years ago, is the culmination of many teams’ work, and we are proud to have finally pushed it over the finish line.
					  
					  \#\#\# Beyond Releases: Highlighting Other Important Work
					  
					  Copy link to this section
					  
					  One of our significant accomplishments this year was the work carried out by our "optimization squads", with our primary goal being providing operators with ways to control, and ideally reduce, their costs. Since our team is not very big, but is too big for the whole team to work on a single task at a time, we’ve divided into two squads: one to work on bandwidth optimization and another squad to work on storage optimization. Allowing our core developers to concentrate on a particular aspect of the solution was very productive, which enabled us to make [significant progress on these two fronts, especially in Q3 ](https://informal.systems/blog/cometbft-engineering-update-q3-wrapped-up-and-q4-priorities), by releasing improvements and new features or conducting a series of experiments. 
					  
					  Some of the work here involved backwards-incompatible changes, and is therefore only available in the newest releases of CometBFT (such as the upcoming v1 release, with minor improvements in v0.34, v0.37 and v0.38).
					  
					  Another vital achievement to mention was our work in tackling security vulnerabilities. Ensuring [security ](https://github.com/cometbft/cometbft/blob/main/SECURITY.md) is our top priority, and we were pleased to work with the team responsible for the Interchain Bug Bounty program, successfully addressing a few [low-severity vulnerabilities ](https://github.com/cometbft/cometbft/security/advisories)that arose this year.
					  
					  \#\#\# The first CometBFT v1 pre-release
					  
					  Copy link to this section
					  
					  We are delighted to [announce ](https://informal.systems/blog/cometbft-v1-alpha-release) the release of the first pre-release version of the highly anticipated v1 release line. This version comes with many exciting features, including significant improvements in bandwidth consumption, new ways of optimizing storage usage, API versioning, enabling greater execution performance through parallelization in the interaction between CometBFT and the application, and a series of refactoring enhancements that aim to improve our team's efficiency. 
					  
					  \#\#\# Teamwork Makes the Dream Work: Acknowledging Our Partners and Collaborators
					  
					  Copy link to this section
					  
					  We are deeply grateful for our users' and partners' unwavering support and invaluable feedback. Your contributions have also been pivotal in developing the Interchain Stack and CometBFT. We understand that our progress is only possible because of your trust and collaboration, and we are truly thankful for that.
					  
					  We recognize that a resilient ecosystem is built on open communication and collective efforts. Your feedback has helped us to adapt and improve continuously, and we cannot thank you enough for your fantastic support and encouragement throughout this year. 
					  
					  Things to improve on a non-technical front: We want to forge closer collaborations with teams that are at the bleeding edge of CometBFT adoption, and which would therefore benefit from closer alignment. There is no magic formula for making this happen, it takes building trust through patience and deliberate, constructive communication.
					  
					  As we embark on a new year, we want to continue to work closely with you and strengthen our collaboration. We value your input and would be honored to receive your feedback on how we can continue to improve and grow together.
					  
					  \#\#\# Looking Ahead: Our Plans for 2024
					  
					  Copy link to this section
					  
					  In the short term, our top priority is to finalize the release for v1, and we are working diligently towards that objective. We also have several other improvements in the pipeline, such as Proposer-Based Timestamps (PBTS), improved testing infrastructure, better documentation, a more modular CometBFT, and many other enhancements that will benefit our users as outlined in the [Interchain Stack Roadmap 2024 ](https://docs.google.com/document/d/1j%5FVnHsM0Hsi2so9gf2oLFKhy3wVa7csCa3lTjlMYCsI/edit\#heading=h.aazbrhskhzvr)
					  
					  As the year draws to a close, we look forward to planning for the next year with enthusiasm and optimism. We are committed to providing our users exceptional value and fulfilling our promises. 
					  
					  We are honored to be part of the Interchain and are deeply grateful for the opportunity to play a role in its success.
					  
					  We extend our warmest wishes to all those involved.
					  
					  \#\# Subscribe to Blog Updates
					  
					  Stop obsessively reloading and let us email you when there are new posts. No spam. Promise.
					  
					  \#\# Start your career with Informal Systems
					  
					  Explore career opportunities at our cooperatively owned and governed organization, with world-class expertise in distributed systems, formal methods, and open-source ecosystem development.
					  
					  [Apply Now](https://informalsystems.bamboohr.com/careers)
					  - [Notes on Functional Programming III: Functor, Applicative & Monad || Chrysanthium](https://omnivore.app/me/notes-on-functional-programming-iii-functor-applicative-monad-ch-18d38448346)
					  collapsed:: true
					  site:: [chrysanthium.com](https://chrysanthium.com/notes-on-functional-programming-iii)
					  author:: akullpp
					  labels:: [[functional programming]] [[fp]] [[javascript]]
					  date-saved:: [[01/23/2024]]
					  - ### Content
					  collapsed:: true
					  - \#\#\#\#\#\# 2017/03
					  
					  Preliminary: [The Fantasy Land](https://github.com/fantasyland/fantasy-land) algebra is a specification which many good functional libraries implement and covers additional laws of algebraic structures which I won't cover.
					  
					  \#\# Functor
					  
					  A **functor** is just a container for values with a `map` method that applies a function to the values while consistently returning the new values in the same container.
					  
					  According to this definition an array is a functor:
					  
					  ```angelscript
					  const xs = [1, 2, 3]
					  // Array.isArray(xs) === true
					  const ys = xs.map((x) => x)
					  // Array.isArray(ys) === true
					  ```
					  
					  You have an array with values, apply a function via `map` to each of the values and get an array with values back.
					  
					  Unfortunately, not all data structures have a map function so you might need to write a wrapper which could be as simple as:
					  
					  ```pgsql
					  class Wrapper {
					  constructor(value) {
					  this.value = value
					  }
					  
					  map(f) {
					  return new Wrapper(f(this.value))
					  }
					  }
					  ```
					  
					  \#\#\# Advantages of a functor
					  
					  **A functor enables generalized behavior**
					  
					  It allows you to map over values without being tied to a specific structure.
					  
					  Furthermore, all advantages of `map` over `for` loops apply to which is mainly compactness and expressiveness.
					  
					  \#\# Applicative
					  
					  An **applicative** is an extension of a functor and is used to be able to apply functors to each other. In order to do this, it wraps a function around the value which is wrapped by the functor. The required `ap` method is used to automatically apply the already partially applied and wrapped function via `map` to the wrapped value of another functor:
					  
					  ```pgsql
					  Wrapper.prototype.ap = function (wrapped) {
					  return wrapped.map(this.value)
					  }
					  ```
					  
					  Additionally, an applicative must provide an `of` method which is used to create an instance with default minimal context:
					  
					  ```pgsql
					  Wrapper.of = (x) => new Wrapper(x)
					  ```
					  
					  > Functors which only have an `ap` method are called Apply. Functors which only have an `of` method are called Pointed Functors. If they have both they are called Applicatives.
					  
					  We will use `of` to write a new `map` method:
					  
					  ```pgsql
					  Wrapper.prototype.map = function (f) {
					  return Wrapper.of(f(this.value);
					  };
					  ```
					  
					  Let's assume we have the following setup:
					  
					  ```pgsql
					  const add = (x) => (y) => x + y
					  const wrapperOne = Wrapper.of(1)
					  const wrapperTwo = Wrapper.of(2)
					  const wrapperOneAdd = wrapperOne.map(add)
					  ```
					  
					  where `add` is a curried function and `wrapperOneAdd` is an object of type `Wrapper` with the wrapped value of `y => 1 + y`.
					  
					  To be explicit: The first box contains the integer `1`. The second box the result, i.e. `y => 1 + y`, of the function `add` which was applied to the boxed `1`. So we are two levels deep now.
					  
					  Using the `ap` method would result in a new wrapped value:
					  
					  ```pgsql
					  wrapperOneAdd.ap(wrapperTwo)
					  // Wrapper {value: 3}
					  ```
					  
					  The wrapped partial function `y => 1 + x` is applied to the unwrapped value of `2`, i.e. `y => 1 + 2` which is then returned as a wrapped value `3`.
					  
					  Keep in mind the box is always of the same type, the values not necessarily.
					  
					  \#\#\# Advantages of an Applicative
					  
					  An applicative is best used if you have several tasks which don't depend on each other, e.g. if you have several independent calls you could write the following interface:
					  
					  ```routeros
					  Task.of(renderPage)
					  .ap(Http.get('orders'))
					  .ap(Http.get('billing'));
					  .ap(Http.get('ads'));
					  .ap(Http.get('tracking'));
					  ```
					  
					  **An applicative promotes simplicity**
					  
					  Generally speaking it creates a simple interface for complex code. However the real advantages can only be grasped if a specific applicative is used. General applicatives like the one described above are rather rare. Further information will be provided in the monad section.
					  
					  \#\# Monads
					  
					  A **monad** applies a function which returns a wrapped value to a wrapped value. The main advantage over applicatives is that they run sequentially by providing the `chain` method. Here is an implementation of a general monad:
					  
					  ```cs
					  class Monad {
					  static of(value) {
					  return new Monad(value)
					  }
					  
					  constructor(value) {
					  this.value = value
					  }
					  
					  map(f) {
					  return Monad.of(f(this.value))
					  }
					  
					  ap(monad) {
					  return monad.map(this.value)
					  }
					  
					  chain(f) {
					  return this.map(f).value
					  }
					  }
					  ```
					  
					  Normally you would also implement the `join` method which is a straight forward helper method to return the value which helps us to flatten monads when chained:
					  
					  ```kotlin
					  join() {
					  return this.value;
					  }
					  
					  chain(f) {
					  return this.map(f).join();
					  }
					  ```
					  
					  However, just like general applicatives, a general implementation of a monad doesn't make much sense. One of the most common practical examples of a specific monad is the **Maybe monad**:
					  
					  ```scala
					  class Maybe extends Monad {
					  static of(value) {
					  return new Maybe(value)
					  }
					  
					  constructor(value) {
					  super(value)
					  }
					  
					  isNothing() {
					  return this.value === null || this.value === undefined
					  }
					  
					  map(f) {
					  return this.isNothing() ? Maybe.of(null) : Maybe.of(f(this.value))
					  }
					  }
					  ```
					  
					  It just checks whether the wrapped value is `null` or `undefined` and if it is, returns a wrapped `null`. Why is this interesting? Well, first of all it avoids pesky null checks. Additionally, it provides safety from runtime errors when chaining several methods where one may fail to return a value:
					  
					  ```javascript
					  const prop = (p) => (o) => o[p]
					  
					  const getUsername = (account) =>
					  Maybe.of(account).map(prop('personal')).map(prop('user')).map(prop('name'))
					  
					  // Might be retrieved async!
					  const user = {
					  personal: {
					  user: {
					  name: 'John Doe',
					  },
					  },
					  }
					  
					  getUsername(user)
					  // Maybe { value: 'John Doe' }
					  ```
					  
					  If one property in the path wouldn't exist, we wouldn't get an error but a `Maybe` with value `null`.
					  
					  Right now you probably think you have a déjà vu and yes you are correct _Promise_ is a monad.
					  
					  \#\#\# Advantages of a monad
					  
					  In general monads are concise and expressive with the ability to encapsulate side-effects.
					  
					  The advantages of specific monads are as numerous as their implementations ranging from the simple forms of _State_ ensuring the correct state flow, _Sequence_ where `;` can be a monad for control flow, _Maybe_ handling null checks, _Promise_ handling asynchronicity up to the complex ones of _Probability Distribution_ or _Transaction_ for databases.
					  
					  \#\# Links
					  
					  * [Notes on Functional Programming I: First-class, Pure, Curried Functions](https://chrysanthium.com/notes-on-functional-programming-i)
					  * [Notes on Functional Programming II: Composition & Point-free Style](https://chrysanthium.com/notes-on-functional-programming-ii)
					  - [strawman:concurrency [ES Wiki]](https://omnivore.app/me/strawman-concurrency-es-wiki-18d34d86412)
					  collapsed:: true
					  site:: [web.archive.org](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman%3Aconcurrency)
					  date-saved:: [[01/23/2024]]
					  date-published:: [[06/16/2013]]
					  - ### Content
					  collapsed:: true
					  - The Wayback Machine - https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
					  
					  * [Communicating Event-Loop Concurrency and Distribution](\#communicating%5Fevent-loop%5Fconcurrency%5Fand%5Fdistribution)  
					  * [Concurrency Model and Promises](\#concurrency%5Fmodel%5Fand%5Fpromises)  
					  * [Vats](\#vats)  
					  * [Promises and Promise States](\#promises%5Fand%5Fpromise%5Fstates)  
					  * [Eventual Operations](\#eventual%5Foperations)  
					  * [Fundamental Static Q Methods](\#fundamental%5Fstatic%5Fq%5Fmethods)  
					  * [Promise methods](\#promise%5Fmethods)  
					  * [Syntactic Sugar](\#syntactic%5Fsugar)  
					  * [Non-fundamental Static Q Conveniences](\#non-fundamental%5Fstatic%5Fq%5Fconveniences)  
					  * [Q.delay](\#q.delay)  
					  * [Q.race](\#q.race)  
					  * [Q.all](\#q.all)  
					  * [Q.join](\#q.join)  
					  * [Q.memoize](\#q.memoize)  
					  * [Q.async](\#q.async)  
					  * [Q.defer()](\#q.defer)
					  * [Examples](\#examples)  
					  * [Infinite Queue](\#infinite%5Fqueue)  
					  * [Spawn](\#spawn)  
					  * [Vat.evalLater() as Async-PGAS](\#vat.evallater%5Fas%5Fasync-pgas)  
					  * [Open Vat](\#open%5Fvat)  
					  * [there](\#there)  
					  * [Map-Reduce Lite](\#map-reduce%5Flite)  
					  * [AMD Loader Lite](\#amd%5Floader%5Flite)
					  * [See](\#see)
					  
					  \#\# Communicating Event-Loop Concurrency and Distribution
					  
					  On both the browser and the server, JavaScript’s de-facto concurrency model is increasingly “shared nothing” communicating event loops. JavaScript event loops within the browser (both frames and workers) send asynchronous messages to other JavaScript event loops via postMessage. JavaScript event loops in the browser send and receive asynchronous messages with servers using asynch XHR, and shortly, Server-Sent Events and WebSockets. And server-side JavaScript has a rapidly growing role as the counterparty of these protocols, and increasingly uses event loops to service them. 
					  
					  This strawman consists of several major parts, not all of which need be accepted together.
					  
					  1. **Reality:** Codifying and formalizing JavaScript’s de-facto concurrency model as a de-jure model.
					  2. **Promises:** A way to  
					  * (**Q(p).post(), Q(p).get()**) Make asynchronous requests of objects that may not be synchronously reachable, such as remote objects.  
					  * (**Q(p).then()**) Ease the burden of local event loop programming, by reifying the ability to register a callback as a first class value.  
					  * (**[Q.async, yield](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:async%5Ffunctions "strawman:async_functions"):**) for implicit registration of shallow continuations on promises.
					  3. **Syntactic sugar**  
					  * **The infix “`!`” operator:** An eventual analog of “`.`“, for making eventual requests look more like immediate requests.
					  4. (**Q.makeFar()** and **Q.makeRemote()**) A promise extension mechanism, so that promise handlers can turn local promise operations into remote messages.  
					  * **Transport independence:** Using remote object messaging as a symmetric abstraction layer, hiding the annoying differences among the various transports listed above as well as server-to-server TCP and UDP transports.
					  5. (**Vat()**) An event-loop spawning mechanism for spawning new event loops that run concurrently with the event loop which spawned it.  
					  * **Worker independence:** Using `Vat`  
					  API  
					  as an abstraction layer around worker spawning on the browser or process spawning on the server.
					  6. (**Vat.evalLater(), there()**) Using JavaScript itself as mobile code, so event loops can safely inject new behavior into other event loops  
					  * **Symmetric Mobile Code:** Generalizes from the current use of JavaScript as mobile code sent only from server and only to browsers.
					  
					  \#\# Concurrency Model and Promises
					  
					  Aggregate objects into process-like units called _vats_. Objects in one vat can only send asynchronous messages to objects in other vats. _Promises_ represent such references to potentially remote objects. _Eventual message sends_ queue _eventual-deliveries_ in the work queue of the vat hosting the target object. A vat’s thread processes each eventual-delivery to completion before proceeding to the next. Each such processing step is a _turn_. A _then expression_ registers a callback to happen in a separate turn once a promise is resolved, providing the callback with the promise’s _resolution_. The eventual send and then expressions immediately return a promise for the eventual outcome of the operation they register.
					  
					  This model is free of conventional race condition or deadlock bugs. While a turn is in progress, it has mutually exclusive access to all state to which it has synchronous access, i.e., all state within its vat, avoiding conventional race condition bugs without any explicit locking. The model presented here provides no locks or blocking constructs of any kind, although it does not forbid a host environment from providing blocking constructs (like `alert`). Without blocking, conventional deadlock is impossible. Of course, less conventional forms of race condition and deadlock bugs [remain](https://web.archive.org/web/20160404122250/http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html "http://www.hpl.hp.com/techreports/2009/HPL-2009-78.html").
					  
					  \#\# Vats
					  
					  Partition the JavaScript reference graph into separate units, corresponding to prior concepts variously called vats, workers, processes, tanks, or grains. We adopt the “_vat_” terminology here for expository purposes. Vats are only asynchronously coupled to each other, and represent the minimal possible unit of concurrency, transparent distribution, orthogonal persistence, migration, partial failure, resource control, preemptive termination/deallocation, and defense from denial of service. Each vat consists of 
					  
					  * a single sequential thread of control,
					  * a single call-return stack,
					  * a single fifo queue holding _eventual-deliveries_,
					  * an internal object heap,
					  * and incoming and outgoing _remote references_.
					  
					  A vat’s thread of control dequeues the next eventual-delivery from the queue and processes it to completion before proceeding to the next. When the queue is empty, the vat is idle. 
					  
					  const vat = Vat(); //makes a new vat, as an object local to the creating vat.
					  // A Vat has an ''evalLater'' method that evaluates a Program in a turn of that vat.
					  // The ''evalLater'' method returns a promise for the evaluation's completion value.
					  const funP = vat.evalLater('' + function fun(x, y) { return x + y; }); // see below
					  const sumP = funP ! (3, 5); // sumP will eventually resolve to 8, unless...
					  const doneP = vat.terminateAsap(new Error('die')); // that vat is terminated before ''sumP'' is resolved.
					  // If the vat is terminated first, then ''sumP'' resolves to a //rejected// problem, with
					  // (Error: die) as its alleged reason for rejection.
					  // Once the vat is terminated, ''doneP'' will eventually resolve to ''true''.
					  
					  The vat object that represents a new vat is local to the creating vat, so that a vat may be terminated without waiting for that vat’s eventual-delivery queue to drain. 
					  
					  The vat abstraction differs from the WebWorker abstraction, even though both are based on communicating event loops, since inter-vat messages are always directed at objects within a vat, not a vat as a whole. We intend that WebWorkers can be implemented in terms of vats and vice versa. However, when vats are built on WebWorkers, in the absence of some kind of weak reference and gc notification mechanism, it is probably impossible to arrange for the collection of distributed garbage. Even with them, [much more](https://web.archive.org/web/20160404122250/http://erights.org/history/original-e/dgc/index.html "http://erights.org/history/original-e/dgc/index.html") is needed to enable collection of distributed cyclic garbage. On the other hand, when vats are provided more primitively, multiple vats within an address space can be jointly within the purview of a single concurrent garbage collector, enabling full gc among these co-resident vats. However, truly distributed vats would still be faced with these same distributed garbage collection worries.
					  
					  The “` '' + function... `” trick above depends on [function\_to\_string](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:function%5Fto%5Fstring "harmony:function_to_string") to actually pass a string which is the program source for the function, while nevertheless having the function itself appear in the spawning program as code rather than as a literal string. This helps IDEs, refactoring tools, etc. A vat’s `evalLater` method evaluates that string as a program in a safe scope – a scope containing only the standard global variables such as `Object`, `Array`, etc. Except for these, the source passed in should be _closed_ – should not contain free references to any other variables. If the function is closed but for these standard globals, and these standard globals are not shadowed or replaced in the spawning context, then an IDE’s scope analysis of the code remains accurate.
					  
					  \#\# Promises and Promise States
					  
					  We introduce a new opaque type of object, the _Promise_ to represent potentially remote references. A normal JavaScript direct reference may only designate an object within the same vat. Only promises may designate objects in other vats. A promise may be in one of several states:
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/0x0,s3MtXvPSlVBOuVroCwRCi7mcdELJGNC1uEMf9KMjOjhE/https://web.archive.org/web/20160404122250im_/http://wiki.ecmascript.org/lib/exe/fetch.php?w=&h=&cache=cache&media=strawman:refstates3.png)](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/lib/exe/detail.php?id=strawman%3Aconcurrency&cache=cache&media=strawman:refstates3.png "strawman:refstates3.png") (TODO: Revise diagram to replace “unresolved” with “pending” and “broken” with “rejected”.) 
					  
					  * _pending_ – when it is not yet determined what object the promise designates,  
					  * _pending-local_ – when the right to determine what the promise designates resides in the same vat,  
					  * _pending-remote_ – when that right is either in flight between vats or resides in a remote vat,
					  * _fulfilled_ – resolved to successfully designate some object,  
					  * _near_ – resolved to a direct reference to a local object,  
					  * _far_ – resolved to designate a remote object,
					  * _rejected_ – will never designate an object, for an alleged reason represented by an associated error.
					  
					  A promise may transition from _pending_ to any state. Additionally a promise can transition from _far_ to _rejected_. A _resolved_ promise can designate any non-promise value including primitives, `null`, and `undefined`. Primitives, `null`, `undefined`, and some objects are pass-by-copy. All other objects are pass-by-reference. A promise resolved to designate a pass-by-copy value is always near, i.e., it always designates a local copy of the value.
					  
					  If a function `foo` immediately returns either `X` or a promise which it later fulfills with `X`, we say that `foo` **_reveals_** `X`. Unless stated otherwise, we implicitly elide the error conditions from such statements. For the more explicit statement, append: _“or it throws, or it does not terminate, or it rejects the returned promise, or it never resolves the returned promise.”_ Put another way, such a function returns a **_reference_** to `X`, where by _reference_ we mean either `X` or a promise for `X`.
					  
					  \#\# Eventual Operations
					  
					  The existing JavaScript infix `.` (dot or _now_) operator enables synchronous interaction with the local object designated by a direct reference. We introduce a corresponding infix `!` (bang or _eventually_) operator for corresponding asynchronous interaction with objects eventually designated by either direct references or promises.
					  
					  Abstract Syntax: 
					  
					  Expression : ...
					  Expression ! [ Expression ] Arguments    // eventual send
					  Expression ! Arguments                   // eventual call
					  Expression ! [ Expression ]              // eventual get
					  Expression ! [ Expression ] = Expression // eventual put
					  delete Expression ! [ Expression ]       // eventual delete
					  
					  The `...` means “and all the normal right hand sides of this production. By “abstract” here I mean the distinction that must be preserved by parsing, i.e., in an [ast](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:ast "strawman:ast"), but without explaining the precedence and associativity which explains how this is unambiguously parsed. In all cases, the eventual form of an expression queues a eventual-delivery recording the need to perform the corresponding immediate form in the vat hosting the (eventually) designated object. The eventual form immediately evaluates to a promise for the result of eventually performing this eventual-delivery.
					  
					  function add(x, y) { return x + y; }
					  const sumP = add ! (3, 5); //sumP resolves in a later turn to 8.
					  
					  Attempted Concrete Syntax: 
					  
					  MemberExpression : ...
					  MemberExpression [nlth] ! [ Expression ]
					  MemberExpression [nlth] ! IdentifierName
					  CallExpression : ...
					  CallExpression [nlth] ! [ Expression ] Arguments
					  CallExpression [nlth] ! IdentifierName Arguments
					  MemberExpression [nlth] ! Arguments
					  CallExpression [nlth] ! Arguments
					  CallExpression [nlth] ! [ Expression ]
					  CallExpression [nlth] ! IdentifierName
					  UnaryExpression : ...
					  delete CallExpression [nlth] ! [ Expression ]
					  delete CallExpression [nlth] ! IdentifierName
					  LeftHandSideExpression :
					  Identifier
					  CallExpression [ Expression ]
					  CallExpression . IdentifierName
					  CallExpression [nlth] ! [ Expression ]
					  CallExpression [nlth] ! IdentifierName
					  
					  “`[nlth]`” above is short for “`[No LineTerminator here]`“, in order to unambiguously distinguish infix from prefix bang in the face of automatic semicolon insertion. 
					  
					  \#\# Fundamental Static Q Methods
					  
					  | Q(target) \-> targetP                         | Lifts the target argument into a promise designating the same object. If target is already a promise, then that promise is returned. (A promise for promise for T simplifies into a promise for T. Category theorists will be more pleased than Type theorists ;).)                                |
					  | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
					  | Q.reject(reason) \-> rejectedP                | Makes and returns a fresh _rejected_ promise recording (a sanitized form of) reason as the alleged reason for rejection. reason should generally be an immutable pass-by-copy Error object.                                                                                                        |
					  | Q.promise(f(resolve,reject)\->()) \-> promise | Makes a fresh promise, where the promise is initially _pending-local_, and the argument function f is called with resolve and reject functions for resolving this promise.                                                                                                                         |
					  | Q.isPromise(target) \-> boolean               | Is target a promise? If not, then using target as a target in the various promise operations is still equivalent to using Q(target), i.e., the promise operations will automatically lift all values to promises.                                                                                  |
					  | Q.makeFar(handler, nextSlotP) \-> promise     | Makes a resolved _far_ reference, which can only [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to _rejected_.                                                                          |
					  | Q.makeRemote(handler, nextSlotP) \-> promise  | Makes an _pending-remote_ promise, which can [further resolve](https://web.archive.org/web/20160404122250/http://wiki.erights.org/wiki/Proxy\#makeProxy "http://wiki.erights.org/wiki/Proxy\#makeProxy") to anything.                                                                                |
					  | Q.ahorten(target1) \-> target2                | Returns the currently most resolved form of target1\. If target1 is a _fulfilled_ promise, return its resolution. If target1 is a promise that is following promise target2, then return target2. If target1 is a terminal _pending_ or _rejected_ promise, or a non-promise, then return target1. |
					  
					  Additional non-fundamental static Q convenience methods appear below.
					  
					  \#\# Promise methods
					  
					  Assuming `p` is a promise 
					  
					  | p.get(name) \-> valueP                    | Returns a promise for the result of eventually getting the value of the name property of target.                                                                                                                                               |
					  | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
					  | p.post(opt\_name, args) \-> resultP       | Eventually invoke the named method of target with these args. Returns a promise for what the result will be.                                                                                                                                   |
					  | p.send(opt\_name, ...args) \-> resultP    | p.send(m, a, b) is equivalent to p.post(m, \[a,b\])                                                                                                                                                                                            |
					  | p.fcall(...args) \-> resultP              | p.fcall(a, b) is equivalent to p.post(void 0, \[a,b\])                                                                                                                                                                                         |
					  | p.put(name, value) \-> voidP              | Eventually set the value of the name property of target to value. Return a promise-for-undefined, used for indicating completion.                                                                                                              |
					  | p.delete(name) \-> trueP                  | Eventually delete the name property of target. Returns a promise for the boolean result.                                                                                                                                                       |
					  | p.then(success, opt\_failure) \-> resultP | Registers functions success and opt\_failure to be called back in a later turn once target is _resolved_. If _fulfilled_, call success(resolution). Else if _rejected_, call opt\_failure(reason). Return a promise for the callback’s result. |
					  | p.end()                                   | If p resolves to _rejected_, log the reason to wherever uncaught exceptions go on this platform, e.g., onerror(reason).                                                                                                                        |
					  
					  
					  \#\# Syntactic Sugar
					  
					  | Abstract Syntax  | Expansion          | Simple Case  | Expansion            | JSON/RESTful equiv        |
					  | ---------------- | ------------------ | ------------ | -------------------- | ------------------------- |
					  | x ! \[i\](y, z)  | Q(x).send(i, y, z) | x ! p(y, z)  | Q(x).send(’p’, y, z) | POST https://...q=p {...} |
					  | x ! (y, z)       | Q(x).fcall(y, z)   | \-           | \-                   | POST https://... {...}    |
					  | x ! \[i\]        | Q(x).get(i)        | x ! p        | Q(x).get(’p’)        | GET https://...q=p        |
					  | x ! \[i\] = v    | Q(x).put(i, v)     | x ! p = v    | Q(x).put(’p’, v)     | PUT https://...q=p {...}  |
					  | delete x ! \[i\] | Q(x).delete(i)     | delete x ! p | Q(x).delete(’p’)     | DELETE https://...q=p     |
					  
					  
					  \#\# Non-fundamental Static Q Conveniences
					  
					  \#\#\# Q.delay
					  
					  Reveal the `answer` sometime after `millis` milliseconds have elapsed.
					  
					  Q.delay = function(millis, answer = undefined) {
					  return Q.promise(resolve => {
					  setTimeout(() => resolve(answer), millis);
					  });
					  };
					  
					  \#\#\# Q.race
					  
					  Given a list of promises, returns a promise for the resolution of whichever promise we notice has completed first.
					  
					  Q.race = function(answerPs) {
					  return Q.promise((resolve,reject) => {
					  for (answerP of answerPs) {
					  Q(answerP).then(resolve,reject);
					  };
					  });
					  };
					  
					  We can compose `Q.race`, `Q.delay`, and `Q.reject` to timeout eventual requests.
					  
					  var answer = Q.race([bob ! foo(carol), 
					       Q.delay(5000, Q.reject(new Error("timeout")))]);
					  
					  \#\#\# Q.all
					  
					  Often it’s useful to collect several promised answers, in order to react either when _all_ the answers are ready or when _any_ of the promises becomes _rejected_.
					  
					  Q.all = function(answerPs) {
					  let countDown = answerPs.length;
					  const answers = [];
					  if (countDown === 0) { return Q(answers); }
					  return Q.promise((resolve,reject) => {
					  answerPs.forEach((answerP, index) => {
					  Q(answerP).then(answer => {
					  answers[index] = answer;
					  if (--countDown === 0) { resolve(answers); }
					  }, reject);
					  });
					  });
					  };
					  
					  We can compose `Q.all`, `then`, and [destructuring](https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=harmony:destructuring "harmony:destructuring") to delay until several operands are revealed
					  
					  var sumP = Q.all([xP, yP]).then(([x, y]) => (x + y);
					  
					  \#\#\# Q.join
					  
					  Join is our eventual equality operation. Any messages sent on the join of `xP` and `yP` are only delivered if `xP` and `yP` both reveal the same target, in which case these messages are eventually delivered to that target and this joined promise itself eventually becomes fulfilled to designate that target. Otherwise, all these messages are discarded with the usual rejected promise contagion.
					  
					  Q.join = function(xP, yP) {
					  return Q.all([xP, yP]).then(([x, y]) => {
					  if (Object.is(x, y)) {
					  return x;
					  } else {
					  throw new Error("not the same");
					  }
					  });
					  };
					  
					  \#\#\# Q.memoize
					  
					  `Q.memoize` of a one argument function returns a new similar one argument function, except that it (eventually) calls the original function no more than once for each such argument. The memo function immediately returns a promise for what the original function will eventually return. Equivalence of arguments is determined by the optional memoMap passed in, which defaults to a new WeakMap() if absent. (Passing a memoMap in also allows the caller to seed the mapping with some prior associations.)
					  
					  The difference from a traditional synchronous memoizer is that the original function is called _eventually_ after a promise for its result is _already_ memoized, enabling cycles to work. For example, if memoF === memoize(f) and f(x) calls memoF(x), then an outer call to memoF(x) schedules an eventual call to f(x) which makes an inner call to memoF(x). Both outer and inner calls to memoF(x) returns a promise for what f(x) will eventually return.
					  
					  Q.memoize = function(oneArgFuncP, memoMap = WeakMap()) {
					  
					  return function oneArgMemo(arg) {
					  if (memoMap.has(arg)) {
					  return memoMap.get(arg);
					  } else {
					  const resultP = oneArgFuncP ! (arg);
					  memoMap.set(arg, resultP);
					  return resultP;
					  }
					  }
					  };
					  
					  \#\#\# Q.async
					  
					  \#\#\# Q.defer()
					  
					  (Will likely be deprecated)
					  
					  Q.defer = function() {
					  const deferred = {};
					  deferred.promise = Q.promise((resolve,reject) => {
					  deferred.resolve = resolve;
					  deferred.reject = reject;
					  });
					  return deferred;
					  };
					  
					  \#\# Examples
					  
					  \#\# Infinite Queue
					  
					  function makeQueue() {
					  let rear;
					  let front = Q.promise(r => { rear = r; });
					  return Object.freeze({
					  enqueue: function(elem) {
					  let nextRear;
					  const nextTail = Q.promise(r => { nextRear = r; });
					  rear({head: elem, tail: nextTail});
					  rear = nextRear;
					  },
					  dequeue: function() {
					  const result = front ! head;
					  front = front ! tail;
					  return result;
					  }
					  });
					  }
					  
					  Calling `queue.dequeue()` will return a promise for the next element that has or will be enqueued.
					  
					  \#\# Spawn
					  
					  The following `spawn` function is a simple abstraction built on `Vat` and `then` that captures a simple common case:
					  
					  function spawn(src) {
					  const vat = Vat();
					  const resultP = vat.evalLater(src);
					  Q(resultP).then(function(_) {
					  vat.terminateAsap(new Error('done')); 
					  });
					  return resultP;
					  }
					  
					  const sumP = spawn('3+5'}); // sumP eventually resolves to 8.
					  
					  The argument string to `spawn` is evaluated in a new Vat spawned for that purpose. Spawn returns a promise for what that string will evaluate to. Once that promise resolves, the spawned vat is shut down.
					  
					  \#\# Vat.evalLater() as Async-PGAS
					  
					  In the Async-PGAS language X10, the “at” statement is defined as
					  
					  // x10 grammar, not javascript or proposed javascript
					  Statement :
					  at ( PlaceExpression ) Statement
					  
					  The “at” statement first evaluates the PlaceExpression to a place, which is analogous to a vat. It then evaluates the Statement at that place. The statement evaluates with the lexical scope containing the “at” statement, so the locality of the values bound to these identifiers is the locality they have at that place rather than at the location containing the “at” statement. Although the argument to `Vat.evalLater` must be a closed expression (modulo whitelisted globals), we can get the same effect, a bit more verbosely, by passing in these bindings as an explicit eventual call to a closed function.
					  
					  For example, the X10-ish program
					  
					  const x = 6;
					  let ultimateP;
					  at (place) { ultimateP = x*7; }
					  
					  can be expressed using `aVat.evalLater()` as
					  
					  const x = 6;
					  let ultimateP = place.evalLater(x => x*7) ! (x);
					  
					  \#\# Open Vat
					  
					  The `makeOpenVat` function makes an `OpenVat` function. Each `OpenVat` function is like the `Vat` function, in that both make an return a new vat instance. Each `OpenVat` function additionally maintains a side table mapping from all incoming remote promises to the evaluation function of the open vat made by this `OpenVat` function. The reason we call such vats _open_ is that, given a remote promise into such a vat and the `OpenVat` function that made that vat, one can thereby inject new code into that vat.
					  
					  TODO: explain the membrane variation used below.
					  
					  function makeOpenVat() {
					  const wm = WeakMap();
					  
					  function OpenVat() {
					  const vat = Vat();
					  const openVat = Object.freeze({
					  evalLater: makeMembraneX(
					  vat.evalLater,
					  { registerRemote: function(remote) {
					  wm.set(remote, openVat.evalLater); }}),
					  terminateAsap: vat.terminateAsap
					  });
					  return openVat;
					  }
					  OpenVat.evalAt = function(p, src) {
					  return wm.get(p)(src);
					  };
					  return OpenVat;
					  }
					  
					  \#\# there
					  
					  The `there(p, ...)` function is analogous to `Q(p).then(...)`, except instead of merely relocating the execution of the callback in time till after `p` is resolved, it further relocates it in spacetime, to where and when p is near. Like `Q(p).then(...)`, `there` immediately returns a promise for the eventual outcome. We do not likewise relocate the errback, so that we can still notify it and it can still react on the requesting side to a partition between the requestor and `p`‘s host.
					  
					  function there(p, callback, opt_errback) {
					  var callbackP = OpenVat.evalAt(p, '' + callback);
					  return (callbackP ! (p)).then(
					  v => v,
					  opt_errback);
					  }
					  
					  \#\# Map-Reduce Lite
					  
					  Given an initial result value, a list of potentially remote promises to elements, a closed mobile `mapper` function from elements to mapped result values, and an associative / commutative `reducer` function from pairs of references to result values to a new reference to a result value, `mapReduce` immediately returns a promise for the result of mapping all the elements where they live, and reducing all of these results together with the initial result value to a result.
					  
					  I call this “Map-Reduce Lite” because, unlike a production map-reduce infrastructure, the following `mapReduce` does all reductions on the spawning machine, which is therefore a scaling bottleneck, and has none of the fault-tolerance. Here, any failure causes the overall map-reduce to fail, i.e., the returned promise becomes a _rejected_ promise. The mapper and reduction functions are like the conventional functional programming variety, rather than the map-reduce variety which arranged for group-by keys.
					  
					  /**
					  * Type/Guard syntax below is only a placeholder, not a serious proposal.
					  * @param initValue ::T2
					  * @param elemPs    ::Array[Ref[T1]]  // i.e., Array[T1 | Promise[T1]]
					  * @param mapper    ::(T1 -> T2)      // closed mobile function
					  * @param reducer   ::(T2 x T2 -> T2)
					  * @reveals         ::T2              // i.e., @returns ::Ref[T2]
					  */
					  function mapReduce(initValue, elemPs, mapper, reducer) {
					  let countDown = elemPs.length;
					  if (countDown === 0) { return initValue; }
					  let result = initValue;
					  
					  return Q.promise((resolve, reject) => {
					  elemPs.forEach(elemP => {
					  const mappedP = there(elemP, mapper);
					  Q(mappedP).then(mapped => {
					  result = reducer(result, mapped);
					  if (--countDown === 0) { resolve(result); }
					  }, reject);
					  });
					  });
					  }
					  
					  \#\# AMD Loader Lite
					  
					  This is a minimal Asynchronous Module Definition (AMD) Loader for a subset of the [AMD API](https://web.archive.org/web/20160404122250/https://github.com/amdjs/amdjs-api/wiki/AMD "https://github.com/amdjs/amdjs-api/wiki/AMD") specification. In this subset, `define` is called with exactly two arguments, a `dependencies` list of module names, and a factory function with one parameter per dependency.
					  
					  function makeSimpleAMDLoader(fetch, moduleMap = Map()) {
					  var loader;
					  
					  function rawLoad(id) {
					  return Q(fetch(id)).then(src => {
					  var result = Q.reject(new Error('"define" not called by: ' + id));
					  function define(deps, factory) {
					  result = Q.all(deps.map(loader)).then(imports => {
					  return factory(...imports);
					  });
					  }
					  define.amd = {lite: true};
					  
					  Function('define', src)(define);
					  return result;
					  });
					  }
					  return loader = Q.memoize(rawLoad, moduleMap);
					  }
					  
					  If module “W” depends on modules “X”, “Y”, and “Z”, then only once the promises for the “X”, “Y”, and “Z” modules have all been fulfilled will the “W” factory function be called with these module instances. The result of calling this factory function will then become the “W” module instance.
					  
					  What it means to _be_ the source for the “W” module is that `fetch(”W”)` will eventually return that source string. For example, a given `fetch` function might fetch it from `https://example.com/prefix/W.js`.
					  
					  // At https://example.com/prefix/W.js
					  define(['X', 'Y', 'Z'], function(X, Y, Z) {
					  return X(Y, Z);
					  })
					  
					  Since the memo mapping we need is from module names, which are strings rather than objects, we need to provide an explicit memoMap argument to `Q.memoize`, which should be a map that accepts strings as keys.
					  
					  Although the cycle tolerance of `Q.memoize` is generally useful, here it hurts. Because `define` won’t call the factory function until all (`Q.all`) of the dependencies are fulfilled, any cyclic AMD dependencies cause an undetected deadlock. Still, in the naive absence of this cycle tolerance, such cyclic dependencies would have instead caused an infinite regress which is even worse. Better would be cycle detection and rejection, which we leave as an exercise for the reader. 
					  
					  \#\# See
					  - [Finite Model Theory and Game Comonads: Part 1 | The n-Category Café](https://omnivore.app/me/finite-model-theory-and-game-comonads-part-1-the-n-category-cafe-18d32a7db59)
					  collapsed:: true
					  site:: [golem.ph.utexas.edu](https://golem.ph.utexas.edu/category/2023/09/finite_model_theory_and_game_c.html)
					  date-saved:: [[01/22/2024]]
					  date-published:: [[09/07/2023]]
					  - ### Content
					  collapsed:: true
					  - \#\#\# Finite Model Theory and Game Comonads: Part 1
					  
					  \#\#\#\# Posted by Emily Riehl
					  
					  [![MathML-enabled post (click for more details).](https://proxy-prod.omnivore-image-cache.app/0x0,sSU_IkLson5g4653tePzSOpvq4vcmGk0czFf6ZfImxHk/https://golem.ph.utexas.edu/~distler/blog/images/MathML.png "MathML-enabled post (click for details).")](http://golem.ph.utexas.edu/~distler/blog/mathml.html)
					  
					  _guest post by [Elena Dimitriadis](https://www.math.univ-toulouse.fr/~edimitri/index%5Fen), [Richie Yeung](https://y-richie-y.github.io/), [Tyler Hanks](https://gataslab.org/), and [Zhixuan Yang](https://yangzhixuan.github.io/)_
					  
					  _Finite model theory_ ([Libkin 2004](https://doi.org/10.1007/978-3-662-07003-1)) studies finite models of logics. Its main motivation comes from computer science: a _finite relational structure_, i.e. a finite set AA with a finite set of relations on AA, is essentially a database in the sense of good old SQL tables, and a logic formula φ\\varphi with nn free variables is understood as a _query_ to the database that selects all nn\-tuples of AA that satisfy the formula φ\\varphi.
					  
					  [![MathML-enabled post (click for more details).](https://proxy-prod.omnivore-image-cache.app/0x0,sSU_IkLson5g4653tePzSOpvq4vcmGk0czFf6ZfImxHk/https://golem.ph.utexas.edu/~distler/blog/images/MathML.png "MathML-enabled post (click for details).")](http://golem.ph.utexas.edu/~distler/blog/mathml.html)
					  
					  Finite model theory is naturally related to _complexity theory_, as we may ask questions like what’s the time complexity to query a finite relational structure with a formula from some logic, and also the converse question—what kind of logic is needed to describe the algorithmic problems in a complexity class. For example, given a finite graph GG, we may ask if it is possible to write a first-order logic formula φ(u,v)\\varphi(u,v) using a relation symbol E(x,y)E(x,y) saying there is an edge from xx to yy, such that φ(u,v)\\varphi(u,v) is satisfied precisely by vertices u,v∈Gu, v \\in G that are connected by a path.
					  
					  From a logical point of view, what makes finite model theory interesting is that some prominent techniques in model theory fail for finite models. In particular, [compactness](https://en.wikipedia.org/wiki/Compactness%5Ftheorem) fails for finite model theory: a theory TT may not have a _finite_ model even if all its finite sub-theories S⊆TS \\subseteq T have finite models—consider e.g. T\={φ n∣n∈ℕ}T = \\left\\{ \\varphi\_n \\mid n \\in \\mathbb{N}\\right\\} where φ n\\varphi\_n asserts there are at least nn distinct elements.
					  
					  Fortunately there are model-theoretic techniques remaining valid in the finite context. One of them is _model-comparison games_, which characterise _logical equivalence_ of models, i.e. when two models satisfy exactly the same formulas of a logic.
					  
					  Such logical equivalences are very useful for showing _(in)expressivity_ of logics. For example, if we are able to show that two models 𝒜\\mathcal{A} and ℬ\\mathcal{B} of a theory satisfy exactly the same formulas from first-order logic, but there is a semantic property PP (in the meta-theory) which 𝒜\\mathcal{A} satisfies but ℬ\\mathcal{B} does not. Then we know the property PP necessarily cannot be expressed in first-order logic. This proof technique works for both finite and infinite models. In fact, using this technique one can show that connectivity of finite graphs cannot be expressed as a first-order logic formula with only the relation symbol E(x,y)E(x,y) for edges.
					  
					  Of course the logic equivalences for different logics need to be characterised by different games: first-order logic is characterised by _Ehrenfeucht-Fraïssé games_; the kk\-variable fragment of first-order logic is characterised by _pebble games_; modal logic is characterised by _bisimulation games_, and so on. 
					  
					  Despite being different, these games are structurally so similar that they almost begged to be unified. Their unification is eventually done by Abramsky and Shah ([2018](https://drops.dagstuhl.de/opus/volltexte/2018/9669), [2021](https://arxiv.org/abs/2010.06496)) in the categorical framework of _game comonads_. This two-part blog post aims to give a brief introduction to finite model theory and game comonads. This post will cover only a tiny part of this active research programme, but we will aim to exposit in a self-contained way the basics with some intuition that is hidden in the papers.
					  
					  \#\# Basics of First-Order Logic
					  
					  Let’s begin with a quick recap on classical _first-order logic_ (FOL). A _purely relational vocabulary_ σ\\sigma is a set of relation symbols {P 1,…,P i,…}\\left\\{P\_1,\\dots,P\_i,\\dots\\right\\} where each relation symbol P iP\_i has an associated arity n i∈ℕn\_i \\in \\mathbb{N}. In this post we do not consider vocabularies with function symbols or constants since they can be alternatively modelled as relations with axioms asserting their functionality, which slightly makes our life easier.
					  
					  The formulas φ\\varphi of FOL in a (purely relational) vocabulary σ\\sigma is inductively generated by the following grammar, where the meta-variable xx ranges over a countably infinite set of variables:
					  
					  φ ::\=x 1\=x 2|P i(x 1,…,x n i)|⊤|φ 1∧φ 2|⊥|φ 1∨φ 2|¬φ|∃x.φ|∀x.φ\\begin{array}{rl} \\varphi &::= x\_1 = x\_2 \\ |\\ P\_i(x\_1,\\dots,x\_{n\_i}) \\ |\\ \\top \\ |\\ \\varphi\_1 \\wedge \\varphi\_2 \\ |\\ \\bot \\ | \\ \\varphi\_1\\vee \\varphi\_2 \\ |\\ \\neg \\varphi \\ |\\ \\exists x. \\varphi \\ |\\ \\forall x. \\varphi \\end{array}
					  
					  A set TT of closed formulas (i.e. formulas with no free variables) is called a _theory_.
					  
					  We will only consider the classical semantics of FOL in the category of sets in this post. A _σ\\sigma\-relational structure_, or simply a _σ\\sigma\-structure_, 𝒜\=⟨A,⟨P i A⟩ P i∈σ⟩\\mathcal{A} = \\langle A, \\langle P\_i^A\\rangle\_{P\_i \\in \\sigma}\\rangle consists of a set AA and an n in\_i\-ary relation P i A⊆A n iP^A\_i \\subseteq A^{n\_i} on the set AA for each relation symbol P i∈σP\_i \\in \\sigma of arity n in\_i.
					  
					  A _homomorphism_ of σ\\sigma\-structures from 𝒜\\mathcal{A} to ℬ\\mathcal{B} is a function h:A→Bh\\colon A\\to B on the underlying sets such that for each relation symbol P iP\_i in σ\\sigma, we have P i A(a 1,…,a n i)P\_i^A(a\_1,\\dots,a\_{n\_i}) implies P i B(h(a 1),…,h(a n i))P\_i^B(h(a\_1),\\dots,h(a\_{n\_i})) for all a 1,…,a n i∈Aa\_1,\\dots,a\_{n\_i}\\in A. Each vocabulary σ\\sigma thus yields a category ℛ(σ)\\mathcal{R}(\\sigma) of σ\\sigma\-structures and homomorphisms.
					  
					  Let φ\\varphi be a FOL formula (in the vocabulary σ\\sigma) with nn free variables. The semantics of φ\\varphi in a σ\\sigma\-structure 𝒜\\mathcal{A} is an nn\-ary relation \[\[φ\]\]⊆A n\[\\!\[\\varphi\]\\!\] \\subseteq A^n on the set AA. We will write 𝒜⊧φ(a→)\\mathcal{A} \\models \\varphi (\\vec{a}) when some a→∈A n\\vec{a} \\in A^n is contained in \[\[φ\]\]\[\\!\[\\varphi\]\\!\] and also write 𝒜⊧φ\\mathcal{A} \\models \\varphi when φ\\varphi is a closed formula and ⟨⟩\\langle \\rangle is in \[\[φ\]\]⊆A 0\[\\!\[\\varphi\]\\!\] \\subseteq A^0. The relation \[\[φ\]\]\[\\!\[\\varphi\]\\!\] is inductively defined on φ\\varphi as follows (we implicity treats a→∈A n\\vec{a} \\in A^n as a function from the set of the nn free variables to the set AA):
					  
					  𝒜⊧(x i\=x j)(a→) ⇔ a→(x i)\=a→(x j) 𝒜⊧P i(x 1,…,x n i)(a→) ⇔ ⟨a→(x 1),…,a→(x n i)⟩∈P i A 𝒜⊧⊤ ⇔ true 𝒜⊧(φ 1∧φ 2)(a→) ⇔ 𝒜⊧φ 1(a→)and𝒜⊧φ 2(a→) 𝒜⊧⊥ ⇔ false 𝒜⊧(φ 1∨φ 2)(a→) ⇔ 𝒜⊧φ 1(a→)or𝒜⊧φ 2(a→) 𝒜⊧¬φ(a→) ⇔ 𝒜⊧φ(a→)does not hold 𝒜⊧(∃y.φ)(a→) ⇔ 𝒜⊧φ(a→\[y↦a′\])for some a′∈A 𝒜⊧(∀y.φ)(a→) ⇔ 𝒜⊧φ(a→\[y↦a′\])for all a′∈A\\begin{matrix} \\mathcal{A}\\models (x\_i = x\_j)(\\vec{a}) &\\iff& \\vec{a}(x\_i) = \\vec{a}(x\_j)\\\\ \\mathcal{A}\\models P\_i(x\_1,\\dots,x\_{n\_i})(\\vec{a}) &\\iff& \\langle \\vec{a}(x\_1),\\dots,\\vec{a}(x\_{n\_i})\\rangle \\in P\_i^A\\\\ \\mathcal{A}\\models \\top &\\iff& true\\\\ \\mathcal{A}\\models (\\varphi\_1 \\wedge \\varphi\_2)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi\_1(\\vec{a}) \\ \\text{and}\\ \\mathcal{A}\\models\\varphi\_2(\\vec{a})\\\\ \\mathcal{A}\\models \\bot &\\iff& false\\\\ \\mathcal{A}\\models (\\varphi\_1 \\vee \\varphi\_2)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi\_1(\\vec{a}) \\ \\text{or}\\ \\mathcal{A}\\models\\varphi\_2(\\vec{a})\\\\ \\mathcal{A}\\models \\neg\\varphi(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi(\\vec{a}) \\ \\text{does not hold}\\\\ \\mathcal{A}\\models (\\exists y. \\varphi)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi(\\vec{a}\[y \\mapsto a'\])\\ \\text{for some }a'\\in A\\\\ \\mathcal{A}\\models (\\forall y. \\varphi)(\\vec{a}) &\\iff& \\mathcal{A}\\models \\varphi(\\vec{a}\[y \\mapsto a'\])\\ \\text{for all } a'\\in A \\end{matrix}
					  
					  where a→\[y↦a′\]\\vec{a}\[y \\mapsto a'\] is the function mapping yy to a′a' and anything else xx to a→(x)\\vec{a}(x).
					  
					  \#\# Logical Equivalences and Ehrenfeucht-Fraïssé Games
					  
					  As motivated earlier, we are interested in characterising when two models 𝒜\\mathcal{A} and ℬ\\mathcal{B} of a relational vocabulary σ\\sigma satisfy exactly the same formulas, more precisely, when 𝒜⊧ϕ⇔ℬ⊧ϕ\\mathcal{A} \\models \\phi \\iff \\mathcal{B} \\models \\phi for all closed FOL formulas ϕ\\phi in the vocabulary σ\\sigma. When it is the case, 𝒜\\mathcal{A} and ℬ\\mathcal{B} are sometimes called _elementarily equivalent_.
					  
					  \#\#\# An Example of Logical Equivalence
					  
					  Let’s build up our intuition with a concrete example. Let σ\\sigma be the vocabulary with just one binary relation symbol ≤\\leq, and let 𝒜\\mathcal{A} be the model {0,1,…,1000}\\left\\{0, 1, \\dots, 1000\\right\\} with ≤\\leq being the usual linear order of natural numbers and similarly ℬ\\mathcal{B} be the model {0,1,…,1001}\\left\\{0, 1, \\dots, 1001\\right\\} with the same order. These two models are clearly different, and indeed they can be differentiated by a first-order logic formula in the vocabulary σ\\sigma—the formula 
					  
					  φ\=∃x 0∃x 1⋯∃x 1001.¬(x 0\=x 1)∧…¬(x i\=x j)…¬(x 1000\=x 1001)\\varphi = \\exists x\_0\\exists x\_1\\cdots\\exists x\_{1001}.\\ \\neg (x\_0 = x\_1) \\wedge \\dots \\neg(x\_i = x\_j) \\dots \\neg (x\_{1000} = x\_{1001})
					  
					  saying that there exist different elements is satisfied by ℬ\\mathcal{B} but not by 𝒜\\mathcal{A}.
					  
					  However, the formula φ\\varphi is a pretty big formula—it has 1002 quantifiers and 501501 clauses, so it is possible that small enough formulas cannot differentiate 𝒜\\mathcal{A} and ℬ\\mathcal{B} since they are pretty similar (they are both linear orders). Let’s consider some formulas with just a few quantifiers:
					  
					  * The vocabulary σ\={≤}\\sigma = \\left\\{ \\leq \\right\\} doesn’t have a constant, so there are no closed terms and thus no closed formulas other than ⊥\\bot and ⊤\\top. Thus 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all formulas with 0 quantifiers.
					  * Consider formulas of the form ∃x.φ(x)\\exists x.\\varphi(x) where φ\\varphi doesn’t have any quantifiers. We argue that 𝒜⊧∃x.φ(x)\\mathcal{A} \\models \\exists x.\\varphi(x) iff ℬ⊧∃x.φ(x)\\mathcal{B} \\models \\exists x.\\varphi(x) as follows: supposing 𝒜⊧∃x.φ(x)\\mathcal{A} \\models \\exists x.\\varphi(x) holds, this means that there is some a∈𝒜a \\in \\mathcal{A} such that 𝒜⊧φ(a)\\mathcal{A} \\models \\varphi(a), then we may choose any b∈ℬb \\in \\mathcal{B}, say b\=0∈ℬb = 0 \\in \\mathcal{B}, making ℬ⊧φ(b)\\mathcal{B} \\models \\varphi(b) because the formula φ\\varphi is inductively built from the variable xx, relations ≤\\leq, \=\=, and propositional connectives; both a≤aa \\leq a and b≤bb \\leq b are true, so we can inductively show that φ(a)\\varphi(a) and φ(b)\\varphi(b) agree for all φ\\varphi. Conversely, if ℬ⊧∃x.φ(x)\\mathcal{B} \\models \\exists x. \\varphi(x) is witnessed by some bb, we can always pick an arbitrary a∈𝒜a \\in \\mathcal{A} witnessing 𝒜⊧∃x.φ(x)\\mathcal{A} \\models \\exists x. \\varphi(x).  
					  Moreover, since the semantics of propositional connectives are defined compositionally, we can inductively show that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all closed formulas built out of exactly one ∃\\exists and \=\=, ¬\\neg, ∧\\wedge, and ∨\\vee. Since we are considering classical logic, universal quantification ∀x.φ(x)\\forall x. \\varphi(x) can be reduced to ¬∃x.φ(x)\\neg \\exists x. \\varphi(x), so 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on it so we conclude that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all FOL formulas with exactly one quanfier.
					  * This example gets interesting when we consider two nested quantifiers. Supposing ψ\=∃x.∃y.φ\\psi = \\exists x. \\exists y. \\varphi where φ\\varphi is quanfier-free, if 𝒜⊧ψ\\mathcal{A} \\models \\psi, there exist aa and a′∈𝒜a' \\in \\mathcal{A} such that 𝒜⊧φ(⟨a,a′⟩)\\mathcal{A} \\models \\varphi (\\langle a, a'\\rangle). Then we can also choose any two elements b,b′∈ℬb, b' \\in \\mathcal{B} such that, importantly, (i) b≤b′b \\leq b' iff a≤a′a \\leq a', and (ii) b\=b′b = b' iff a\=a′a = a'. This ensures ℬ⊧φ(⟨b,b′⟩)\\mathcal{B} \\models \\varphi(\\langle b, b'\\rangle) since the atomic formulas in φ\\varphi are built from ≤\\leq, \=\=, xx and yy, on which 𝒜\\mathcal{A} with the variable assignment ⟨x↦a,y↦a′⟩\\langle x\\mapsto a, y \\mapsto a'\\rangle and ℬ\\mathcal{B} with ⟨x↦b,y↦b′⟩\\langle x\\mapsto b, y \\mapsto b'\\rangle agree. Conversely, if ℬ⊧ψ\\mathcal{B} \\models \\psi witnessed by b,b′∈ℬb, b' \\in \\mathcal{B} we can also choose a matching pair a,a′∈𝒜a, a' \\in \\mathcal{A} making 𝒜⊧ψ\\mathcal{A} \\models \\psi.  
					  Now suppose ψ\=∃x.∀y.φ\\psi = \\exists x. \\forall y. \\varphi. Whenever 𝒜⊧ψ\\mathcal{A} \\models \\psi, there is some aa such that 𝒜⊧∀y.φ⟨x↦a⟩\\mathcal{A} \\models \\forall y.\\varphi \\langle x \\mapsto a\\rangle. In this case we can also choose an element b∈ℬb \\in \\mathcal{B} that mimics a∈𝒜a \\in \\mathcal{A}: if aa is the bottom element 00 in the structure 𝒜\\mathcal{A}, we let b\=0b = 0 as well; if aa is the top 10001000 in 𝒜\\mathcal{A}, we let bb be the top in ℬ\\mathcal{B}; otherwise we can choose an arbitrary 0<b<10010 \\lt b \\lt 1001. We then claim ℬ⊧∀y.φ⟨x↦b⟩\\mathcal{B} \\models \\forall y. \\varphi \\langle x \\mapsto b\\rangle as well, because if there is some b′b' making ℬ⊧φ⟨x↦b,y↦b′⟩\\mathcal{B} \\models \\varphi \\langle x\\mapsto b, y \\mapsto b'\\rangle not hold, we can also find an a′a' that is to aa in 𝒜\\mathcal{A} as b′b' is to bb in ℬ\\mathcal{B}: precisely, if b′\=bb' = b, we let a′\=aa' = a; if b′<bb' \\lt b, we let a′a' be any element in 𝒜\\mathcal{A} such that a′<aa' \\lt a, and similarly for the case b′\>bb' \\gt b (such a choice is always possible because earlier aa and bb are chosen to be at the same relative position). This choice of a′a' entails 𝒜⊧φ⟨x↦a,y↦a′⟩\\mathcal{A} \\models \\varphi\\langle x \\mapsto a, y \\mapsto a'\\rangle not true, leading to a contradiction. Therefore ℬ⊧ψ\\mathcal{B} \\models \\psi.  
					  By a symmetric argument, ℬ⊧∃x.∀y.φ\\mathcal{B} \\models \\exists x. \\forall y. \\varphi implies 𝒜⊧∃x.∀y.φ\\mathcal{A} \\models \\exists x. \\forall y. \\varphi as well. Moreover, by the compositionality of the semantics of propositional connectives, the two paragraphs above imply that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all FOL formulas with quantifiers are nested at most once.
					  
					  Hopefully working through the example above reveals the intuition behind logical equivalence for FOL: two σ\\sigma\-structures 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on a FOL formula φ\\varphi whenever a quantifier in φ\\varphi picks an element xx in AA or BB, there is always an element yy in the other structure that “simulates the behaviour” of xx in the model.
					  
					  \#\#\# The Ehrenfeucht-Fraïssé Game
					  
					  The intuition is precisely formulated as _Ehrenfeucht-Fraïssé (EF) games_. Given two σ\\sigma\-structures 𝒜\\mathcal{A} and ℬ\\mathcal{B} for any vocabulary σ\\sigma and a natural number kk, the kk\-round EF game for 𝒜\\mathcal{A} and ℬ\\mathcal{B} is a turn-based game between two players, called _the spoiler_ and _the duplicator_. Roughly speaking, the goal of the spoiler is to point out the difference between 𝒜\\mathcal{A} and ℬ\\mathcal{B} while the goal of the duplicator is to advocate that 𝒜\\mathcal{A} and ℬ\\mathcal{B} are the same. The rules are very simple: 
					  
					  1. **Movement**: at each round, the spoiler picks an element from one of the structures and the duplicator must respond with an element from the other structure. For example, if the spoiler picks an element from the structure 𝒜\\mathcal{A}, then the duplicator must pick an element b∈ℬb\\in\\mathcal{B}.
					  2. **Winning Condition**: After kk rounds, the game state consists of a→\=(a 1,…,a k)\\vec{a} = (a\_1,\\dots,a\_k) and b→\=(b 1,…,b i)\\vec{b} = (b\_1,\\dots,b\_i) representing the elements chosen from each structure at each round. The duplicator wins this play if the mapping a i↦b ia\_i \\mapsto b\_i defines a partial isomorphism between 𝒜\\mathcal{A} and ℬ\\mathcal{B}, i.e., if the substructures of 𝒜\\mathcal{A} and ℬ\\mathcal{B} generated by a→\\vec{a} and b→\\vec{b} are isomorphic. Otherwise, the spoiler has succeeded in showing the structures are different and wins.
					  
					  If the duplicator can guarantee a win after kk rounds no matter how the spoiler plays, we say the duplicator has a kk\-round winning strategy.
					  
					  The _quantifier rank_ qr(φ)\\mathop{qr}(\\varphi) of a FOL formula φ\\varphi is the depth of nesting of the quantifiers in φ\\varphi:
					  
					  qr(φ) \= 0 for atomicφ qr(o(φ 1,…,φ n)) \= max(qr(φ 1),…,qr(φ n)) for propositional connectiveso qr(Qx.φ) \= 1+qr(φ) for quantifiersQ\=∀,∃\\begin{array}{rcll} \\mathop{qr}(\\varphi) &=& 0 &\\text{for atomic}\\ \\varphi\\\\ \\mathop{qr}(o(\\varphi\_1, \\dots, \\varphi\_n)) &=& \\max(\\mathop{qr}(\\varphi\_1), \\dots, \\mathop{qr}(\\varphi\_n)) &\\text{for propositional connectives}\\ o\\\\ \\mathop{qr}(Q x. \\varphi) &=& 1 + \\mathop{qr}(\\varphi) &\\text{for quantifiers}\\ Q = \\forall, \\exists \\end{array}
					  
					  **Theorem** (Ehrenfeucht-Fraïssé). _If the duplicator has a winning strategy for the kk\-round EF game for 𝒜\\mathcal{A} and ℬ\\mathcal{B}, 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all closed FOL formulas of quantifier-rank kk. When the vocabulary σ\\sigma is finite, the converse is also true._
					  
					  _Proof sketch_: Assuming a winning strategy for the duplicator, and let ψ\\psi be any formula of quantifier rank kk. Without loss of generality, we can assume ψ\=Q 1x 1.⋯Q kx k.φ\\psi = Q\_{1} x\_{1}.\\cdots Q\_k x\_k.\\varphi where Q i∈{∀,∃}Q\_i \\in \\left\\{\\forall, \\exists\\right\\} are quantifiers and φ\\varphi is quantifier-free. We need to show 𝒜⊧ψ⇔ℬ⊧ψ\\mathcal{A} \\models \\psi \\iff \\mathcal{B} \\models \\psi. As we demonstrated in the example above, we consider how −⊧ψ\- \\models \\psi is defined inductively: if a quantifier Q i\=∃Q\_i = \\exists and one of 𝒜\\mathcal{A} and ℬ\\mathcal{B} satisfies the formula, we use the winning strategy for the duplicator to pick a matching element in the other structure; if a quantifier Q i\=∀Q\_i = \\forall and one of 𝒜\\mathcal{A} and ℬ\\mathcal{B} satisfies the formula, every counter-witness in the other structure leads to a counter-witness in this structure by the winning strategy of the duplicator, thus a contradiction.
					  
					  The converse direction is also interesting and is not demonstrated in the example in the last section. We only give a rough sketch here and refer interested readers to [Libkin (2004, section 3)](https://doi.org/10.1007/978-3-662-07003-1) for details. 
					  
					  Assuming that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on all FOL formulas of rank kk, the duplicator’s strategy is that whenever the spoiler picks an element a i∈𝒜a\_i \\in \\mathcal{A} (or symmetrically b i∈ℬb\_i \\in \\mathcal{B}), the duplicator first constructs a FOL formula ϕ i\\phi\_{i} that “maximally describes the current board on 𝒜\\mathcal{A}”. Roughly speaking, ϕ i\\phi\_{i} is the conjunction of all formulas in the following set
					  
					  {ψ∣qr(ψ)\=i−1and𝒜⊧ψ⟨a 1,…,a i⟩}\\left\\{\\psi \\mid \\mathop{qr}(\\psi) = i - 1 \\ \\text{and}\\ \\mathcal{A} \\models \\psi\\, \\langle a\_1, \\dots, a\_i\\rangle\\right\\}
					  
					  A priori, this set may have infinitely many formulas, but since there are only finitely many atomic formulas when σ\\sigma is finite, ϕ i\\phi\_{i} can be reduced to a finite formula using the rules of propositional connectives. With the formula ϕ i\\phi\_{i} in hand, the duplicator can use the fact that 
					  
					  𝒜⊧∃x i.ϕ i⟨x 1↦a 1,…,x i−1⟩↦a i−1,\\mathcal{A} \\models \\exists x\_i. \\phi\_{i}\\langle x\_1 \\mapsto a\_1, \\dots, x\_{i-1}\\rangle \\mapsto a\_{i-1},
					  
					  witnessed by x i↦a ix\_i \\mapsto a\_i. Now using the assumption that 𝒜\\mathcal{A} and ℬ\\mathcal{B} agree on FOL up to rank kk and the fact that ∃x i.ϕ i\\exists x\_i.\\phi\_i is of rank ii, ℬ\\mathcal{B} has an element witnessing the truth of this formula as well, which is going to be the duplicator’s response. □\\square
					  
					  EF games are very useful for showing inexpressivity results of FOL. Suppose we are interested in a property PP (in the meta-theory) on a class MM of σ\\sigma\-structures. If for every natural number kk, we can find two models 𝒜 k,ℬ k∈M\\mathcal{A}\_k, \\mathcal{B}\_k \\in M such that 
					  
					  1. the duplicator has a winning strategy for the kk\-round EF game on 𝒜 k\\mathcal{A}\_k and ℬ k\\mathcal{B}\_k, but
					  2. only one of 𝒜 k\\mathcal{A}\_k and ℬ k\\mathcal{B}\_k satisfy the property PP,
					  
					  Then by the EF theorem, the property PP cannot be expressed by a FOL formula, whatever the quantifier it has. 
					  
					  For example, using this technique, we can show that the evenness of finite linear orders is not expressible—there isn’t a FOL formula φ\\varphi in the vocabulary {≤}\\left\\{\\leq\\right\\} such that for every finite linear order 𝒜\=⟨A,≤ A⟩\\mathcal{A} = \\langle A, \\leq^A\\rangle, 𝒜⊧φ\\mathcal{A} \\models \\varphi exactly when aa has an even number of elements. (Hint: we pick 𝒜 k\\mathcal{A}\_k and ℬ k\\mathcal{B}\_k to be the linear order of 2 k2^k and 2 k+12^k + 1 elements respectively and play the EF games.)
					  
					  Posted at September 8, 2023 2:19 PM UTC 
					  
					  TrackBack URL for this Entry: https://golem.ph.utexas.edu/cgi-bin/MT-3.0/dxy-tb.fcgi/3498
					  
					  Read the post [Finite Model Theory and Game Comonads: Part 2](https://golem.ph.utexas.edu/category/2023/09/finite%5Fmodel%5Ftheory%5Fand%5Fgame%5Fc%5F1.html)  
					  **Weblog:** The n-Category Café  
					  **Excerpt:** In the Part 1 of this post, we saw how logical equivalences of first-order logic (FOL) can be characterised by a combinatory game. But there are still a few unsatisfactory aspects, which we'll clear up now.  
					  **Tracked:** September 11, 2023 4:09 PM
					  - [An invitation to category theory - Chalkdust](https://omnivore.app/me/an-invitation-to-category-theory-chalkdust-18d329ae0fb)
					  collapsed:: true
					  site:: [Chalkdust](https://chalkdustmagazine.com/features/an-invitation-to-category-theory/)
					  author:: Tai-Danae Bradley
					  date-saved:: [[01/22/2024]]
					  date-published:: [[10/18/2018]]
					  - ### Content
					  collapsed:: true
					  - ![post](https://proxy-prod.omnivore-image-cache.app/0x0,s5Dp9ObKGeSthHi_JsRok2AznAlNM0ePIj_aihjfevq0/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/headerbridge.jpg?fit=1050%2C450&ssl=1)
					  
					  Image: Martin Damboldt
					  
					  Early in our mathematical education, we learn about a strong interplay between algebra and geometry—algebraic equations give rise to graphs and geometric figures, and geometric features can be encoded in algebraic expressions. It’s almost as if there’s a portal or bridge connecting these two realms in the grand landscape of mathematics: whatever occurs on one side of the bridge is mirrored on the other.
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/429x383,syz4KWcEEo-FfGiTFr_ZDtfmdzG5uilIAdBZBmTnDcr8/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/bridge-1024x915.jpg?resize=429%2C383)
					  
					  So although algebra and geometry are very different areas of mathematics, this connection suggests that they are intrinsically related. Incidentally, the \`bridge’ that spans them is a but a dim foreshadow of much deeper connections that exist between other branches of mathematics that also may, a priori, seem unrelated: set theory, group theory, linear algebra, topology, graph theory, differential geometry, and more. And what’s amazing is that these relationships—these bridges—are more than just a neat observation. They are _mathematics_, and that mathematics has a name: _category theory_.
					  
					  \#\# What is a category?
					  
					  Martin Kuppe once created a wonderful map of the mathematical landscape (see facing page) in which category theory hovers high above the ground, providing a sweeping vista of the terrain. It enables us to see relationships between various fields that are otherwise imperceptible at ground level, attesting that seemingly unrelated areas of mathematics aren’t so different after all. This becomes extraordinarily useful when you want to solve a problem in one realm (topology, say) but don’t have the right tools at your disposal. By ferrying the problem to a different realm (such as group theory), you’re able to see the problem in a different light and perhaps discover new tools that may help the solution become much easier. In fact, ==this is precisely how category theory came to be. It was birthed in the 1940s in an attempt to answer a difficult topological question by recasting it in an easier, algebraic light.==
					  
					  Thinking back to the mathematical landscape, you’ll notice that each realm consists of some objects (set theory has sets, group theory has groups, topology has topological spaces…) that relate to each other (sets relate via functions, groups relate via homomorphisms, topological spaces relate via continuous functions…) in sensible ways (such as composition and associativity).
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/654x520,sFkcmNx3sLB4lo__sWskVkmsXNEz6xFl_y8keFkW2Ym4/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/MathematistanHiresForChalkdust-1024x814.jpg?resize=654%2C520)](https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/MathematistanHiresForChalkdust.jpg)
					  
					  Martin Kuppe’s map of the mathematical landscape (used with permission)
					  
					  This common thread weaves throughout the landscape and unites the various fields. The mathematics of category theory formalises this unification. More concretely, a _category_ is a collection of _objects_ with relationships between them (called _morphisms_) that behave nicely in terms of _composition_ and _associativity_. This provides a template for mathematics, and depending on what you feed into that template, you’ll recover one of the mathematical realms: the category of sets consists of sets and relationships (ie functions) between them; the category of groups consists of groups and relationships (ie group homomorphisms) between them; the category of topological spaces consists of topological spaces and relationships (ie continuous functions) between them; and so on.
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/654x113,snKkSO_Po-7hyCXFRbEP6uaj1cTg31qOSXgp7GqO6mbQ/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/objects-1024x177.jpg?resize=654%2C113)](https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/objects.jpg)
					  
					  The analogy between a category and a template is due to Barry Mazur from his wonderfully written, non-technical introduction to category theory, [_When is one thing equal to some other thing?_](http://www.math.harvard.edu/~mazur/preprints/when%5Fis%5Fone.pdf) In it he writes, “This concept of category is an omni-purpose affair… There is hardly any species of mathematical object that doesn’t fit into this convenient, and often enlightening, template.” Indeed, as category theorist [Eugenia Cheng](http://chalkdustmagazine.com/interviews/in-conversation-with-eugenia-cheng/) so aptly put it in her treatise [_Higher-dimensional category theory_](http://cheng.staff.shef.ac.uk/misc/4000.pdf), ==“category theory is the mathematics of mathematics”.==
					  
					  \#\# It’s all about relationships
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/498x401,sLHqlP278FIyW4ZS5zQ3yWVmlVH-w23jS79ckpz8OvZ8/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/list-1024x826.jpg?resize=498%2C401)](https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/list.jpg)One of the main features of category theory is that it strips away a lot of detail: it’s not really concerned with the individual elements in your set, or whether or not your group is solvable, or if your topological space has a countable basis. So you might be thinking, “Eh, category theory sounds so abstract. Can any good come from this?” Happily, the answer is yes! ==An advantage of ignoring details is that our attention is diverted away from the individual objects and turned towards the relationships—the morphisms—that exist between them. And as any category theorist will tell you,== _==relationships are everything.==_
					  
					  Indeed, ==one of== ==the main maxims of category theory is that== _==a mathematical object is completely determined by its relationships to all other objects==_==.== To put it another way, two objects are essentially indistinguishable if and only if they relate to every object in the category in the same way. This theme (which is a consequence of a famous result called the Yoneda lemma) isn’t too different from what we observe in life. You can learn a lot about people by looking at their relationships—their Facebook friends, who they follow on Twitter, who they hang out with on Friday nights, for instance. And if you ever meet two people who have the _exact same_ set of friends, and whose interactions on social media are _exactly the same_, and who hang out with the _exact same_ people on Friday nights, then you might jokingly say, “You can’t even tell them apart”. Category theory informs us that, all jokes aside, this is actually true mathematically!
					  
					  [![](https://proxy-prod.omnivore-image-cache.app/360x183,s__a3_QkqRyPztZ6B6tMhOd-ya91GMapQviDMttvFJ-k/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/arrows-1024x521.jpg?resize=360%2C183)](https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/arrows.jpg)  
					  So you might wonder “Hmm, if mathematical relationships are _that_ important, then what about relationships between categories? Do _they_ exist?” Great question. The answer is: absolutely! In fact, these particular relationships have a name—they are called _functors_. But why stop there? What about the relationships between those relationships? They, too, have a name: _natural transformations._
					  
					  In fact, we can keep on going: “What about the relationships between the relationships between the relationships between the…?” Doing so will land us in higher-dimensional category theory, which is where much of Cheng’s research lies.
					  
					  As abstract as this may sound, these constructions—categories, functors, and natural transformations—comprise a treasure trove of theory that shows up almost everywhere, in mathematics _and_ in other disciplines! Since its inception, category theory has found natural applications in computer science, quantum physics, systems biology, chemistry, dynamical systems, and natural language processing, just to name a few. (The [website](http://appliedcategorytheory.org/workshops) \`Applied category theory’ contains a list of applications.) So even though category theory might sound a little abstract, it is highly applicable. And that’s no surprise. Category theory is all about relationships, and so is the world around us!
					  
					  \#\# Conclusion
					  
					  Categories are a little bit like anchovies: some folks love ’em, while for other folks they’re an acquired taste. So yes, it’s true that category theory may not help you find a delta for your epsilon, or determine if your group of order 520 is simple, or construct a solution to your PDE. For those endeavours, we do have to put our feet back on the ground. But thinking categorically can help serve as a beacon—it can strengthen your intuition and sharpen your insight—as you trek through the nooks and crannies of your favourite mathematical realms. And these days it’s especially hard to escape the pervasiveness of category theory throughout modern mathematics. So whatever your mathematical goals may be, learning a bit about categories will be well worth your time!
					  
					  _This article has been adapted from “What is Category Theory, Anyway?” first published on 17 January 2017 at [math3ma.com](http://math3ma.com/)_
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/150x150,sDTPntYsYNbbET28j84vez3lbvACMXIeN8egzsn91e5U/https://i0.wp.com/chalkdustmagazine.com/wp-content/uploads/2018/10/tai_danae_authorpic.jpg?resize=150%2C150)
					  
					  Tai-Danae Bradley is a PhD candidate in mathematics at the CUNY Graduate Center, where she spends much of her time thinking about category theory, topology, machine learning, and quantum physics. When not thinking on these things, she’s probably writing (and doodling) about them on her blog, Math3ma.
					  - ### Highlights
					  collapsed:: true
					  - > this is precisely how category theory came to be. It was birthed in the 1940s in an attempt to answer a difficult topological question by recasting it in an easier, algebraic light. [⤴️](https://omnivore.app/me/an-invitation-to-category-theory-chalkdust-18d329ae0fb#f1e7164c-8a40-4bb7-b8bf-52c0a0cb0411)
					  - > “category theory is the mathematics of mathematics”. [⤴️](https://omnivore.app/me/an-invitation-to-category-theory-chalkdust-18d329ae0fb#d302a5d2-6216-4426-9545-58ce5376f38b)
					  - > An advantage of ignoring details is that our attention is diverted away from the individual objects and turned towards the relationships—the morphisms—that exist between them. And as any category theorist will tell you, _relationships are everything._ [⤴️](https://omnivore.app/me/an-invitation-to-category-theory-chalkdust-18d329ae0fb#056e54f9-e34b-4ff2-ae64-7e54ad6a21bd)
					  - > one of the main maxims of category theory is that _a mathematical object is completely determined by its relationships to all other objects_. [⤴️](https://omnivore.app/me/an-invitation-to-category-theory-chalkdust-18d329ae0fb#89fc3040-eced-4e8e-9e74-3aaae59525c2)
					  - [Delaware Certificate of Amendment | Harvard Business Services, Inc.](https://omnivore.app/me/u-1-c-3-fb-806-b-784-11-ee-8-b-41-3-baa-386450-bb-delaware-certi-18d268df2c9)
					  collapsed:: true
					  site:: [omnivore.app](https://omnivore.app/attachments/u/1c3fb806-b784-11ee-8b41-3baa386450bb/DelawareCertificateofAmendmentHarvardBusinessServicesInc.3.pdf)
					  labels:: [[funding]] [[2024]] [[DeFi]]
					  date-saved:: [[01/20/2024]]
					  - ### Content
					  collapsed:: true
					  -
					  - [Delaware Certificate of Amendment | Harvard Business Services, Inc.](https://omnivore.app/me/delaware-certificate-of-amendment-harvard-business-services-inc-18d266cab7f)
					  collapsed:: true
					  site:: [Harvard Business Services, Inc.](https://www.delawareinc.com/blog/change-your-company-name-certificate-of-amendment/)
					  author:: Brett Melson
					  date-saved:: [[01/20/2024]]
					  date-published:: [[06/22/2021]]
					  - ### Content
					  collapsed:: true
					  - Is your company name outdated, or does it no longer reflect the nature of your business?
					  
					  You are allowed to officially change your company name. Often, people think it will be costly and time-consuming to change a company name, or that a new company must be filed with the new name.
					  
					  This is not true; your company name can be officially changed, quickly and easily.
					  
					  In some cases, the original name may have been too specific, such as Bob's Deck & Patio LLC, created before Bob began doing more general contracting and home building. In other cases, a company may be re-branding and want the centerpiece to be the new name.
					  
					  \#\# Famous company name changes
					  
					  Instead of filing a new company, you can simply call us. We can prepare and file a Delaware Certificate of Amendment to the Certificate of Formation (for LLCs) or Certificate of Incorporation (for corporations) with the Delaware Secretary of State’s office.
					  
					  Original Name Re-branded Name Quantum Computer Services AOL BackRub Google Brad’s Drink Pepsi-Cola Auction Web EBay Minnesota Mining and  
					  Manufacturing Co 3M Peter’s Super Submarines SUBWAY U-Tote’m 7-Eleven
					  
					  \#\# What is a Certificate of Amendment?
					  
					  A Certificate of Amendment is a legal document that is typically required when corporations and other business entities want to indicate any changes, like a change of company name.
					  
					  This filing officially changes the name, either immediately or on an effective date, whichever option is selected by the client. This is used to make changes to a Delaware LLC's Certificate of Formation or for other Delaware business entities.
					  
					  Filing a name amendment, as opposed to filing a new company, will allow you to maintain the history that is connected to your original filing. Assets don’t change hands, liabilities remain the same, all contracts remain in force and all accounting and tax records remain the same.
					  
					  The name changes but everything else remains intact. This saves you the hassle of opening a new bank account, [obtaining a new EIN](https://www.delawareinc.com/ourservices/obtain-tax-id-or-ein-for-delaware-company/) and creating entirely new internal documents. The formation date of the company remains the date of the original formation, which is an important consideration for many entrepreneurs.
					  
					  \#\#\# How To File a Certificate of Amendment
					  
					  The first step in the name change process is choosing the name. This can be a tough decision. For assistance in picking out the right name for your company, read [What to Consider When Naming a Company](https://www.delawareinc.com/blog/what-to-consider-when-naming-a-company/).
					  
					  To make sure your company name is available in Delaware, take advantage of our [free corporation search](https://www.delawareinc.com/name-check/ "Check Your Company Name's Availability!").
					  
					  After the name has been selected, a Certificate of Amendment will have to be prepared; it must be signed by an authorized officer of the company.
					  
					  Next, the Certificate of Amendment needs to be filed with the state, and the name will officially change as of the date and time the document is filed, or at some specified date known as the "effective date," after the file date.
					  
					  Company names cannot be effective before the filing date of the name amendment. The state of Delaware typically takes three to five business days to return [certified copies](https://www.delawareinc.com/ourservices/certified-copies-of-corporate-documents/) of the filing documents. We can file your Certificate with the state of Delaware, relieving you of the hassle and paperwork. 
					  
					  Once the amendment is filed, you should make sure everyone--your clients, your bank, et al.--is aware of the new name for the company.
					  
					  The easy way to inform everyone is to make a list and inform everyone on the list of your new company name. Be sure to include:
					  
					  * government agencies your company works with
					  * clients
					  * vendors
					  * banks
					  * the post office
					  * UPS
					  * Federal Express
					  * DHL
					  * any other businesses your company deals with
					  
					  Some of the bureaucracies involved may have a form to fill out, and they may ask you to return the form with a copy of the approved Certificate of Amendment.
					  
					  The IRS allows this change to be made rather easily. Simply [send a letter to the IRS](https://www.delawareinc.com/blog/change-your-business-name-with-the-irs/) stating the new name of the company, the old name, your EIN (Employer ID Number) and the signature of a corporate officer.
					  
					  \#\#\# Filing Your New Name in Other States where You Do Business
					  
					  Wait, there’s more!
					  
					  Traditionally, you’ll want the registered name to match in all states where you do business, which may include some states where you are registered as a foreign entity. Once the name is amended and approved in Delaware (or whatever your company's home state may be), you can file to change the name on your foreign registration with the Secretary of State in other states. 
					  
					  We can assist you in understanding what needs to be done and checking the name to make sure it’s also available in each applicable state.
					  
					  \#\#\# Steps to changing your company name:
					  
					  1. Choose your new name.
					  2. Do a corporation name search to make sure your new name is available with the state.
					  3. Prepare and sign a Certificate of Amendment.
					  4. Notify your employees, vendors, banks, and other parties about the new name.
					  5. Send a letter to the IRS, including your company EIN, notifying them of the name change.
					  6. Change your company names in any other states in which you are authorized to do business.
					  
					  Finally, go forward and prosper with the new company name that properly reflects the business you have worked so hard to build!
					  
					  Photo Credit: By Eviatar Bach (Own work) \[Public domain\], via Wikimedia Commons
					  
					  **\*Disclaimer\*:** Harvard Business Services, Inc. is neither a law firm nor an accounting firm and, even in cases where the author is an attorney, or a tax professional, nothing in this article constitutes legal or tax advice. This article provides general commentary on, and analysis of, the subject addressed. We strongly advise that you consult an attorney or tax professional to receive legal or tax guidance tailored to your specific circumstances. Any action taken or not taken based on this article is at your own risk. If an article cites or provides a link to third-party sources or websites, Harvard Business Services, Inc. is not responsible for and makes no representations regarding such source’s content or accuracy. Opinions expressed in this article do not necessarily reflect those of Harvard Business Services, Inc.
					  
					  More By [Brett Melson](https://www.delawareinc.com/blog/author/brett-melson/) 
					  
					  There are 7 comments left for **Change Your Company Name: Certificate of Amendment**
					  
					  Andrea Longenecker said: Monday, November 23, 2020
					  
					  my EIN name has changed from " Massage by Andrea" to "Hands to Wellness. I would like to amend my EIN. Thank you
					  
					  **Here's some information on changing your company name with the IRS. Please feel free to email us if you have additional questions.**
					  
					  **[https://www.delawareinc.com/blog/change-your-business-name-with-the-irs](https://www.delawareinc.com/blog/change-your-business-name-with-the-irs/)**
					  
					  How can we pay to Delaware Secretary of State for the certificate of amendment?
					  
					  **We would be happy to assist you with preparing and filing the Certificate of Amendment. You can also contact the Delaware Division of Corporations directly if you wish to prepare and file on your own. They can instruct you on how to make a direct payment.**
					  
					  I want to change my incorporation name. Can you let me know if it is possible ?
					  
					  **Pallav, we would be happy to assist you. You can reach us via LiveChat or call us 9-5 at 1-800-345-2677\. We'll guide you through the process.**
					  
					  Can I use the Certificate of Amendment to convert a California LLC into a Delaware Non Profit organization keeping the same name and the same operations? If not, please let me know the best way to go.
					  
					  **We can only assist with the Delaware side of a conversion by preparing and filing a Certificate of Conversion with the Delaware Secretary of State for approval. We will also prepare and file a Certificate of Incorporation with the Delaware Secretary of State for approval as well. Generally, clients will consult with an attorney convenient to them or their location or work with the California Division of Corporations directly to determine if the company will owe any fees in California prior to proceeding with the conversion. We can only assist with the Delaware part of this process.**
					  
					  Hi, I've stumbled upon your website looking to amend my Delaware company name. I would like to change the company name "Claimly Technologies, Inc." into "Cypher Technologies, Inc." Is it something you can help me with? Kind regards, Hyacinthe
					  
					  Yes, of course. You can reach us via LiveChat from our homepage or call us 9-5 at 1-800-345-2677\. We would be happy to assist you.
					  
					  1 | [2](https://www.delawareinc.com/blog/change-your-company-name-certificate-of-amendment/?startrow=6)
					  - [Chapter 8: Top 10 Trends in DeFi](https://omnivore.app/me/chapter-8-top-10-trends-in-de-fi-18d261b7bd1)
					  collapsed:: true
					  site:: [Snipd](https://share.snipd.com/episode/a4ddb7e5-8360-4286-87d0-a14a1b1534ff/summary)
					  date-saved:: [[01/20/2024]]
					  date-published:: [[01/07/2024]]
					  - ### Content
					  collapsed:: true
					  - Snipd AI 
					  
					  This podcast explores the top 10 trends in DeFi, including DEX platform updates, trading aggregators, payments, DeFi lending, oracles, and RWA diversification. It also discusses the importance of trustless bridges and interoperability protocols, challenges in the crypto market data landscape, and the increase in new transactions and the L2 side chain ecosystem for Bitcoin. 
					  
					  Read more 
					  
					  \#\# Podcast summary created with Snipd AI
					  
					  \#\#\# Quick takeaways
					  
					  * DeFi applications have the lowest marginal cost of capital due to decreased operational costs, indicating potential for increased growth.
					  * Liquid staking tokens (LSTs) have become the largest DeFi sector by TVL, offering yield-bearing IOUs and leveraging tokenized treasuries for safer lending alternatives.
					  
					  \#\#\# Deep dives
					  
					  \#\#\#\# DeFi's Growth and Potential
					  
					  DeFi represents a small fraction of global financial assets, indicating room for growth. On-chain financial rails offer safety and lower fees compared to off-chain rails. DeFi applications have the lowest marginal cost of capital due to decreased operational costs. The move towards regulation in DeFi might lead to changes that resemble reintermediation. However, Wall Street may also engage in DeFi storefronts, increasing volume over public blockchains in the long term.
					  
					  \#\# Get the Snipd  
					  podcast app
					  
					  Unlock the knowledge in podcasts with the podcast player of the future. 
					  
					  \#\# AI-powered  
					  podcast player
					  
					  Listen to all your favourite podcasts with AI-powered features 
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sJWcawIf7uOEm_lWCe9wa8eJIsB2ryEbHiRCapAyZRlM/https://share.snipd.com/assets/get-app/show-list-horizontal.png)
					  
					  \#\# Discover  
					  highlights
					  
					  Listen to the best highlights from the podcasts you love and dive into the full episode 
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sI-lM-LR1a7lc8ehIHZ3q657v4kTWSJN5POYdCseXAqM/https://share.snipd.com/episode/a4ddb7e5-8360-4286-87d0-a14a1b1534ff/assets/app-info/feed.png)
					  
					  \#\# Save any  
					  moment
					  
					  Hear something you like? Tap your headphones to save it with AI-generated key takeaways 
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sLpjSOgUmjWEDeUfebocG4JzbD150lZXLEfM-MB6JArA/https://share.snipd.com/episode/a4ddb7e5-8360-4286-87d0-a14a1b1534ff/assets/app-info/create_snip.png)
					  
					  \#\# Share  
					  & Export
					  
					  Send highlights to Twitter, WhatsApp or export them to Notion, Readwise & more 
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,szswhBrsJIZEX1tmq6X_0FodhXDrehd3JeOzhuOZEG7U/https://share.snipd.com/episode/a4ddb7e5-8360-4286-87d0-a14a1b1534ff/assets/app-info/share.png)
					  
					  \#\# AI-powered  
					  podcast player
					  
					  Listen to all your favourite podcasts with AI-powered features 
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sJWcawIf7uOEm_lWCe9wa8eJIsB2ryEbHiRCapAyZRlM/https://share.snipd.com/assets/get-app/show-list-horizontal.png)
					  
					  \#\# Discover  
					  highlights
					  
					  Listen to the best highlights from the podcasts you love and dive into the full episode 
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sI-lM-LR1a7lc8ehIHZ3q657v4kTWSJN5POYdCseXAqM/https://share.snipd.com/episode/a4ddb7e5-8360-4286-87d0-a14a1b1534ff/assets/app-info/feed.png)
					  - [Messari's Crypto Theses for 2024 | Chapter 11: Bonus - Whatever I Want](https://omnivore.app/me/messari-s-crypto-theses-for-2024-chapter-11-bonus-whatever-i-wan-18d26234e7b)
					  collapsed:: true
					  site:: [share.transistor.fm](https://share.transistor.fm/s/b1ea3126)
					  labels:: [[RSS]]
					  date-saved:: [[01/08/2024]]
					  date-published:: [[01/08/2024]]
					  - ### Content
					  collapsed:: true
					  - **Chapter 11: Bonus - Whatever I Want**11.1Why You Must Write  
					  11.2 No Idols  
					  11.3 Must Read  
					  11.4 Tips & Productivity Tricks  
					  11.5 Life Advice
					  
					  (_End: Disclaimers)_
					  
					  \#\#\#\# What is Messari's Crypto Theses for 2024?
					  
					  The 2024 edition of crypto’s most popular year-end report, now available in podcast format. Written by Messari CEO and Co-founder Ryan Selkis, this recording explores key trends, people, companies, and projects to watch across the crypto landscape, with predictions for 2024\. Learn more and view the report at <https://messari.io/crypto-theses-for-2024>
					  - [What is a Merkle Tree?](https://omnivore.app/me/what-is-a-merkle-tree-18c57202f64)
					  collapsed:: true
					  site:: [decentralizedthoughts.github.io](https://decentralizedthoughts.github.io/2020-12-22-what-is-a-merkle-tree/)
					  author:: Decentralized Thinkers
					  labels:: [[cryptography]] [[merkle-airdrop]]
					  date-saved:: [[12/10/2023]]
					  date-published:: [[12/21/2020]]
					  - ### Content
					  collapsed:: true
					  - In this post, we will demystify _Merkle trees_ using three examples of problems they solve:
					  
					  1. Maintaining integrity of files stored on Dropbox.com, or _file outsourcing_,
					  2. Proving a transaction between Alice and Bob occurred, or _set membership_,
					  3. Transmitting files over unreliable channels, or _anti-entropy_.
					  
					  (1)
					  
					  By the end of the post, you’ll be able to understand **key concepts** that, at this point, might intimidate you:
					  
					  1. A Merkle tree is[1](\#fn:consideredtobe) a **collision-resistant hash function**, denoted by MHT, that takes n inputs (x1,…,xn) and outputs a _Merkle root hash_ h\=MHT(x1,…,xn),
					  2. A verifier who only has the root hash h can be given an xi and an associated **Merkle proof** which convinces them that xi was the ith input used to compute h.  
					  * In other words, convinces them that there exist other xj’s such that h\=MHT(x1,…,xi−1,xi,xi+1,…,xn).  
					  * This ability to verify that an xi is the element at the ith position against the root hash h (i.e., without having to store all n inputs) is the fundamental power of Merkle trees which we explore in this post.
					  3. If a Merkle proof says that xi was the ith input used to computed h, no attacker can come up with another Merkle proof that says a different xi′≠xi was the ith input.  
					  * More formally, an attacker cannot come up with a Merkle root h and two values xi≠xi′ with proofs πi and πi′ that both verify for position i w.r.t. h.
					  
					  \#\# Prerequisites
					  
					  Merkle trees are built using an **underlying** _cryptographic hash functions_, which we assume you understand. If not, just read [our post on hash functions](https://decentralizedthoughts.github.io/2020-08-28-what-is-a-cryptographic-hash-function)!
					  
					  **Definition:** A hash function H is collision resistant if it is computationally-intractable[2](\#fn:handwave) for an adversary to find two different inputs x and x′ with the same hash (i.e., with H(x)\=H(x′)).
					  
					  **Notation:**
					  
					  * We often use \[k\]\={1,2,…,k} and \[i,j\]\={i,i+1,…,j−1,j}.
					  * Although a hash function H(x) is formalized as taking a single input x, we often invoke it with two inputs as H(x,y). Just think of this as H(z), where z\=(x | y) and | is a special delimiter character.
					  
					  Suppose you have n\=8 files, denoted by (f1,f2,…,f8), you have a [collision-resistant hash function](https://decentralizedthoughts.github.io/2020-08-28-what-is-a-cryptographic-hash-function) H and you want to have some fun.
					  
					  You start by hashing each file as hi\=H(fi):
					  
					  You could have a bit more fun by continuing to hash every two adjacent hashes:
					  
					  You could even go crazy and continue on these newly obtained hashes:
					  
					  In the end, you only live once, so you’d better hash these last two hashes as h1,8\=H(h1,4,h4,8):
					  
					  **Congratulations!**What you have done is computed a Merkle tree on n\=8 **leaves**, as depicted in the picture above.
					  
					  Note that every **node** in the tree stores a hash:
					  
					  * The ith **leaf** of the tree stores the hash hi of the file fi.
					  * Each **internal node** of the tree stores the hash of its two children.
					  * The h1,8 hash stored in the **root node** is called the **Merkle root hash**.
					  
					  You could easily generalize and compute a Merkle tree over any number n of files.
					  
					  It is useful to _think of computing the Merkle tree as computing a collision-resistant hash function_, denoted by MHT, which takes n inputs (e.g., the n files) and outputs the Merkle root hash. For example, we will often use the following notation:
					  
					  (2)h1,8\=MHT(f1,f2,…,f8)
					  
					  Importantly, note that computing MHT (i.e., computing the Merkle tree), involves many computations of its **underlying collision-resistant hash function** H.
					  
					  At the risk of overwhelming you with notation, it is useful to observe that h1,8 has a “recursive” structure:
					  
					  h1,8\=MHT(f1,…,f8)\=H(H(H(H(f1),H(f2)),H(H(f3),H(f4)),),H(H(H(f5),H(f6)),H(H(f7),H(f8)),))
					  
					  Really this is just a (verbose) mathematical representation of the Merkle tree that we visually depicted above!
					  
					  We often refer to a file fi as **“being in the Merkle tree”** to indicate it was one of the n input files used to compute the tree.
					  
					  \#\# This is a Merkle proof!
					  
					  Alright, you’ve computed your Merkle tree over your 8 files.
					  
					  Next, you upload your files **and** Merkle tree on Dropbox.com and delete everything from your computer except the Merkle root h1,8. Your goal is to later download any one of your files from Dropbox **and** make sure Dropbox did not accidentally corrupt it.
					  
					  This is the **file integrity** problem and Merkle trees will help you solve it.
					  
					  The **key idea** is that, after you download fi, you ask for a small part of the Merkle tree called a **Merkle proof**. This proof enables you to verify that the downloaded fi was not accidentally or maliciously modified.
					  
					  But how do you verify?
					  
					  Well, observe that the Merkle proof for fi is exactly the subset of hashes in the Merkle tree that, together with fi, allow you to recompute the root hash of the Merkle tree and check it matches the real hash h1,8, **without knowing any of the other hashed files**.
					  
					  So, to verify the proof, you simply “fill in the blanks” in the picture above by computing the missing hashes depicted with dotted boxes, in this order:
					  
					  h3′\=H(f3)h3,4′\=H(h3′,h4)h1,4′\=H(h1,2,h3,4′)h1,8′\=H(h1,4′,h5,8)
					  
					  Lastly, you check that the Merkle root h1,8′ you computed above is equal to the Merkle root h1,8 you kept locally! If that’s the case, then you can be sure you downloaded the correct fi (and we prove this later).
					  
					  But why does this check suffice as a proof that fi was downloaded correctly? Here’s some intuition. Since the Merkle proof verified, this means you were able to recompute the root hash h1,8 by using fi as the ith input and the Merkle proof as the remaining inputs. If the proof verification had yielded the same hash h1,8 but with a different file fi′≠fi as the ith input, then this would yield a collision in the underlying hash function H used to build the tree. This last observation is not trivial to see but we will help you see it later, when we argue security formally. 
					  
					  \#\#\# Why use large Merkle proofs anyway?
					  
					  Forget the Merkle tree! Since you only have eight files, you could check integrity by storing their hashes hi\=H(fi) rather than their Merkle root h1,8\=MHT(f1,…,f8). After all, the hi hashes are much smaller than the files themselves.
					  
					  Then, when you download fi, you hash it as yi\=H(fi) and check that hi\=yi. Since H is [collision-resistant](https://decentralizedthoughts.github.io/2020-08-28-what-is-a-cryptographic-hash-function), you can be certain that fi was not modified. (Indeed, we already discussed how [hash functions can be used for download file integrity](https://decentralizedthoughts.github.io/2020-08-28-what-is-a-cryptographic-hash-function) in our previous post.)
					  
					  One advantage of this approach is you no longer need to download Merkle proofs.
					  
					  Unfortunately, the problem with this approach is you have to store n hashes when you outsource n files. While this is fine when n\=8, it is no so great when n\=1,000,000,000!
					  
					  _“But that’s crazy! Who has one billion files?”_ you might protest.
					  
					  Well, just take a look at the [Certificate Transparency (CT)](https://en.wikipedia.org/wiki/Certificate%5FTransparency) project, which builds a Merkle tree over the set of all digital certificates of HTTPS websites. These Merkle trees easily have hundreds of millions of leaves and are designed to scale to billions.
					  
					  **Moral of the story:**To avoid the need for the verifier to store a hash for each one of the n outsourced file, we use Merkle trees. This way, you only need to store a Merkle root hash (rather than n hashes) and receive an O(log⁡n)\-sized Merkle proof with each downloaded file.
					  
					  If you are still concerned about large Merkle proof size, you should look at more _algebraic_ **vector commitments (VCs)**, such as recent ones based on [polynomial commitments](https://alinush.github.io/2020/05/06/aggregatable-subvector-commitments-for-stateless-cryptocurrencies.html) or on [RSA assumptions](https://alinush.github.io/2020/11/24/Catalano-Fiore-Vector-Commitments.html). However, be aware that VCs come with their own performance bottlenecks and other caveats.
					  
					  \#\# What else is a Merkle tree useful for?
					  
					  Now that we know how a Merkle tree is computed and how Merkle proofs work, let’s dig into a few more use-cases.
					  
					  \#\#\# Efficiently proving Bitcoin transactions were validated
					  
					  Recall that a Bitcoin block is just a _set of transactions_ that were validated by a miner.
					  
					  **Problem:**Sometimes it is useful for _Alice_, who is running Bitcoin on her mobile phone, to verify that she received a payment transaction from _Bob_.
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sUV1XU8FfrH4xk86Y5htdvP9vdGzFR7USLp-RQgpUwN0/https://decentralizedthoughts.github.io/uploads/merkle-bitcoin-1.png)
					  
					  **Inefficient, consensus-based solution:**Alice could simply download every newly mined Bitcoin block on her phone and inspect the block for a transaction from Bob that pays her. But this requires Alice to download a lot of data (1 MiB / block), which can be either slow or expensive, since mobile data costs money._(The unstated assumption here is that Alice relies on Bitcoin’s proof-of-work consensus protocol to decide whether a block is valid. These consensus-related details are covered in other posts[3](\#fn:post1),[4](\#fn:post2) on this blog.)_
					  
					  **Insecure, polling-based solution:**Note that Alice cannot just simply ask nodes on the Bitcoin network, _“Hey, was I paid by Bob?”_ since nodes can lie and say _“Yes, you were”_ showing her a transaction from Bob that is actually not yet incorporated into a block.
					  
					  **Efficient, Merkle-based solution:**If Alice was indeed paid by Bob, then Bitcoin can prove this to her via a **Merkle proof**. Specifically, Alice will ask a Bitcoin node if she was paid by Bob in the latest block, but instead of simply trusting the _“Yes, you were paid and here’s the transaction”_ answer, she will ask the node to _prove membership of the transaction_ in the block via a Merkle proof.
					  
					  Importantly, Alice never has to download the full block: she only needs to download a small part of the block called the **block header**, which contains the root of a Merkle tree built over all transactions in that block[5](\#fn:catena). This way, Alice can verify the Merkle proof leading to Bob’s transaction in this tree, which will assure her that Bob’s transaction is in the block without having to download the other transactions.
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sHfD6cW1O86pJFR4-xSzYoA-HtIB70_ofRJo2OXLtHEc/https://decentralizedthoughts.github.io/uploads/merkle-bitcoin-2.png)
					  
					  This Merkle-based solution still requires Alice to rely on Bitcoin’s proof-of-work consensus to validate block _headers_. However, since headers are 80 bytes, this is much more efficient than downloading full (1 MiB) blocks. Furthermore, some of you might note that, because of Bitcoin’s _chronologically-ordered_ Merkle tree, a node can still lie and say _“No, you weren’t paid by Bob”_ and Alice would have no way to tell the truth _efficiently_ without actually downloading every new block and inspecting it. One way this problem could be solved is by re-ordering Bitcoin’s Merkle tree, either by the payer or by the payee’s Bitcoin address. We leave the details of this to another post.
					  
					  **Moral of the story:**The beauty of Merkle trees is that a _prover_, who has a large set of data (e.g., thousands of transactions) can convince a _verifier_, who has access to the set’s Merkle root hash, that a piece of data (e.g., a single transaction) is in this large set by giving the verifier a Merkle proof.
					  
					  \#\#\# Downloading files over corrupted channels, or anti-entropy via Merkle trees
					  
					  In our previous example, we showed how you could outsource your files to Dropbox and **detect** malicious or accidental modifications of any of your files.
					  
					  Suppose the modifications were accidental due to an unreliable connection to Dropbox. It would be nice if you could not just _detect_ these modifications but actually **recover** from them and eventually download the original, unmodified file.
					  
					  **The naive solution** would be to simply restart the file download whenever you detect a modification via the Merkle-based technique we discussed before. However, this could be painfully slow. In fact, if the connection is sufficiently unreliable and if the file is sufficiently large, this might never terminate.
					  
					  A **better solution** is to split the file into **blocks** and use the Merkle tree to detect modifications _at the block level_ (i.e., at a finer granularity) rather than at the file level. This way, you only need to restart the download for incorrectly downloaded blocks, which helps you make steady progress.
					  
					  To keep things simple, we will focus on a simpler scenario where just one file is outsourced to Dropbox rather than n files. We will discuss later how this can be generalized to n files.
					  
					  \#\#\#\# Inefficient: Recovering corrupted file blocks with one Merkle proof per block
					  
					  As we said above, we will split the file f into, say, b\=8 **blocks**:
					  
					  (3)f\=(f1,f2,…,f8)
					  
					  Then, we will build a Merkle tree over these 8 blocks:
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sSg2VQFDipT7rrx6S8fUr4X3LBDKeSb6S3ZPl5W9Bpqg/https://decentralizedthoughts.github.io/uploads/merkle-anti-entropy-1.png)
					  
					  As before, you send f and the Merkle tree to Dropbox, and you store h1,8 locally.
					  
					  Next, to download f reliably, you will now download each block of f with its Merkle proof, which you verify as discussed in the beginning of the post. And if a block’s proof does not verify, you ask for that block and its proof again until it does. Ultimately, the unreliable channel will become reliable and the proof will verify.
					  
					  This approach is better than the previous one because, when the channel is unreliable, you do not need to restart the download of f from scratch. Instead, you only restart the download for the specific block of f that failed downloading.
					  
					  Note that there’s an interesting choice of block size to be made, as a function of the unreliability of the channel. However, this is beyond the purpose of this post. Also note that there are other ways to deal with unreliable channels, such as error-correcting codes, which again are beyond the purpose of this post.
					  
					  Unfortunately, in this **first solution**, you are re-downloading the full Merkle tree, in addition to the blocks themselves. This is because Dropbox sends a Merkle proof for _every_ block[6](\#fn:dedup), **even if that block is correct**.
					  
					  We will fix this next.
					  
					  \#\#\#\# Recovering corrupted file blocks with one Merkle proof per _corrupted_ block
					  
					  A **better solution** is to observe that you can first _optimistically_ download all b\=8 blocks, and rebuild the Merkle tree over them. If you get the same root hash h1,8, you have downloaded the file f correctly and you are done!
					  
					  Otherwise, let’s go through an example to see how you would identify the corrupted blocks. Assume, for simplicity, that only block 5 (denoted by f5) is corrupted. Then, the Merkle tree you re-compute during the download would differ _only slightly_ from the one you originally computed above.
					  
					  Specifically, you would re-compute the tree below, with the difference highlighted in red:
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,sczRugRyAofNXa7BZhN3Ck68TBVQYQUwSu2qILK8axIs/https://decentralizedthoughts.github.io/uploads/merkle-anti-entropy-2.png)
					  
					  If all blocks were correct, you expect the root hash of the Merkle tree above to be h1,8. However, the blocks you downloaded yielded a different root hash h1,8′. This tells you some of the blocks are corrupted!
					  
					  Even better, you realize that either:
					  
					  1. Some of the first b/2 blocks were corrupted,
					  2. Some of the last b/2 blocks were corrupted,
					  3. Or both!
					  
					  And because you have the actual root hash h1,8, you can actually tell which one of these cases you are in! How? You simply ask for the children (h1,4,h5,8) of the root h1,8 until you receive the correct ones that verify: i.e., the ones such that h1,8\=H(h1,4,h5,8).
					  
					  Once you have the correct children, you immediately notice that you computed the correct h1,4 but computed a different h5,8′ instead of h5,8. This tells you the first 4 blocks were correct, but the last 4 were not. As a result, you can now ignore the first (correct) half of the Merkle tree for blocks f1,…,f4 and focus on the second (corrupted) half for blocks f5,…,f8.
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,ssIxKdfscjG0LPF3PfofCJHnCo19LkfFPL-Gvuxd-b3A/https://decentralizedthoughts.github.io/uploads/merkle-anti-entropy-3.png)
					  
					  In other words, you have now reduced your initial problem of to a smaller subproblem! Specifically, you must now identify the corrupted block amongst blocks f5,…,f8. And, since you now know that their real Merkle root hash is h5,8, you just need to recursively apply the same technique!
					  
					  Importantly, you should convince yourself that this approach works even if there is more than one corrupted block: you will just have more sub-problems. Furthermore, note that if all blocks are corrupted, then this approach effectively downloads all Merkle hashes in the tree. However, if just one block is corrupted, this approach will only download the hashes along the path to that block (i.e., the ones in red in the figure above) and the Merkle proof for that block (i.e., the sibling nodes of the red nodes).
					  
					  In general, this approach only downloads (roughly) a Merkle proof _per corrupted block_, without re-downloading common hashes across different proofs. In contrast, the first solution downloaded one Merkle proof _per downloaded block_, even if the block was correct!
					  
					  \#\#\#\# Generalizing to more than one file
					  
					  We can generalize the approach above to multiple files. The key idea is to build a Merkle tree over each file’s blocks as already described. If we have n files, we get n Merkle trees with root hashes r1,r2,…,rn−1 and rn, respectively. Next, we build another Merkle tree over these root hashes. Lastly, denote this tree’s root hash by z1,n.
					  
					  For example, here’s what this would look like when n\=8:
					  
					  ![](https://proxy-prod.omnivore-image-cache.app/0x0,swmooD9jfOkXeuzY58biFCA8_G99-7fxLwkHjNWZFmHk/https://decentralizedthoughts.github.io/uploads/merkle-anti-entropy-4.png)
					  
					  Now, when downloading, say, the 2nd file f2 over an unreliable channel, you first ask for the 2nd leaf of the Merkle tree with root z1,8, which is r2, together with a Merkle proof. Once again, because the channel is unreliable, you might have to ask multiple times until the proof verifies. Finally, once you have r2, you can run the protocol described above, since you have the root hash of f2’s Merkle tree!
					  
					  \#\# Want more?
					  
					  Well, I hope you found all of this fascinating and want to learn more. You could start by going to the bonus section below and reading our formal security proof for Merkle trees.
					  
					  Then, you could read my three favorite Merkle tree papers, which I think are highly approachable even for beginners.
					  
					  First, you should read the paper on **history trees** by Crosby and Wallach[7](\#fn:CW09). History trees are Merkle trees that “grow” from left to right, in the sense that one is only allowed to append new leafs to the right of the tree.
					  
					  Despite their simplicity, history trees are incredibly powerful since they support _append-only proofs_: given an older tree of t leaves and a new tree of t′\=t+Δ leaves, one can prove (using a succinct O(log⁡t′)\-sized proof) that the new tree includes all the leaves from the old tree. This makes history trees very useful for building append-only logs such as [Certificate Transparency (CT)](https://en.wikipedia.org/wiki/Certificate%5FTransparency), which is at the core of securing HTTPS.
					  
					  _“Append-only logs? Don’t you mean blockchains?”_ you ask. Nope, I do not. These logs have a different mechanism to detect (rather than prevent) forks. However, each fork is always provably extended in append-only fashion using the proofs described above.
					  
					  Second, you should read the paper on CONIKS by Melara et al[8](\#fn:MBBplus15). CONIKS is also a transparency log, but geared more towards securing instant messaging apps such as Signal, rather than HTTPS. One interesting thing you’ll learn from this paper is how to lexicographically-order your Merkle trees so you can prove something is **not** in the tree, as we briefly touched upon in the Bitcoin section. In fact, I believe this paper takes the most sane, straightforward approach to doing so. Specifically, CONIKS builds a **Merkle prefix tree**, which is much simpler to implement than any binary search tree or treap (at least in my own experience). It also has the advantage of having expected O(log⁡n) height if the data being Merkle-ized is not adversarially-produced.
					  
					  A related paper is would be the Revocation Transparency (RT) manuscript[9](\#fn:LK15), which CONIKS can be regarded as improving upon in terms of proof size and other dimensions.
					  
					  Third, you should read the Verifiable Data Structures[10](\#fn:ELC16) manuscript by the Certificate Transparency (CT) team, which combines a history tree with a lexicographically-ordered tree (such as CONIKS) into a single system with its own advantages.
					  
					  At the end of the day, I think what I’m trying to say is _“why don’t you go read about transparency logs and come write a blog post on Decentralized Thoughts so I don’t have to do it!”_ :)
					  
					  \#\# Bonus: A Merkle tree is a collision-resistant hash function
					  
					  A very simple way to think of a Merkle hash tree with n _leaves_ is as a collision-resistant hash function MHT that takes n inputs[11](\#fn:contrast) and ouputs a 2λ\-bit hash, a.k.a. the _Merkle root hash_. More formally, the _Merkle hash function_ is defined as:
					  
					  (4)MHT:({0,1}∗)n→{0,1}2λ
					  
					  And the Merkle root of some input x→\=(x1,…,xn) is denoted by:
					  
					  (5)h\=MHT(x1,…,xn)
					  
					  Here, λ is a _security parameter_ typically set to 128, which implies Merkle root hashes are 256 bits.
					  
					  **Theorem (Merkle trees are collision-resistant)**: It is unfeasible to find two sets of leaves x→\=(x1,…,xn) and x→′\=(x1′,…,xn′) such that x→≠x→′ but MHT(x→)\=MHT(x′→).
					  
					  Instead of proving this theorem, we’ll prove an even stronger one below, which implies this one. However, if you want to prove this theorem, all you have to do is show that the existence of these two “inconsistent” sets of leaves implies a collision in the underlying hash function H. In fact, in the proof, you will “search” for this collision much like the anti-entropy protocol described above searches for corrupted blocks!
					  
					  **Theorem (Merkle proof consistency):** It is unfeasible to output a Merkle root h and two “inconsistent” proofs πi and πi′ for two different inputs xi and xi′ at the ith leaf in the tree of size n.
					  
					  **Proof:**The Merkle proof for xi is πi\=((h1,b1),…,(hk,bk)), where the hi’s are **sibling hashes** and the bi’s are **direction bits**. Specifically, if bi\=0, then hi is a left child of its parent node in the tree and if bi\=1, then it’s a right child.
					  
					  In our previous discussion, we never had to bring up these _direction bits_ because we always visually depicted a specific Merkle proof. However, here, we need to reason about _any_ Merkle proof for _any_ arbitrary leaf. Since such a proof can “take arbitrary left and right turns” as it’s going down the tree, we use these direction bits as “guidance” for the verifier. (A careful reader might notice that the direction bits can actually be derived from the leaf index i being proved and don’t actually need to be sent with the proof: the direction bits are obtained by flipping i’s binary representation.)
					  
					  Roughly speaking, to verify πi, the verifier uses xi together with the sibling hashes and the direction bits to compute the hashes (z1,…,zk+1) along the path from xi to the root. More precisely, the verifier:
					  
					  1. Sets z1\=xi.
					  2. For each j∈\[2,k\], computes zj as follows:  
					  * If bj−1\=1, then zj\=H(zj−1,hj−1)  
					  * If bj−1\=0, then zj\=H(hj−1,zj−1)
					  
					  Lastly, the verifier checks if zk+1 equals the Merkle root hash h. If it does, then the verification succeeds. Otherwise, it fails.
					  
					  Do not be intimidated by all the math above: we are merely generalizing the Merkle proof verification that we visually depicted in the Dropbox file outsourcing example at the beginning of this post.
					  
					  Similarly, the proof for xi′ is πi′\=((h1′,b1),…,(hk′,bk)). Note that the hi′ hashes could differ from the hi hashes, but the direction bits are the same, since both proofs are for the ith leaf in the tree.
					  
					  Since both proofs verify, the verification of the second proof πi′ for xi′ will yield hashes {z1′,z2′,…,zk+1′} along the same path from xi′ to the root such that zk+1′\=h.
					  
					  But recall from the verification of the first proof πi that we also have zk+1\=h. Thus, zk+1\=zk+1′. This is merely saying that, since both proofs verify, they yield the same root hash h\=zk+1\=zk+1′.
					  
					  The next step is to reason about how the two different xi≠xi′ could have possibly “hashed up” to the same root hash h. (Spoiler alert: only by having a collision in the underlying hash function H.) I think this is best explained by considering a few extreme cases and then generalizing. (Note that these are not the _only_ two cases; just two particularly enlightening ones.)
					  
					  **Extreme case \#1**: One way would have been for the proof verification to yield zj≠zj′ for all j∈\[k\] (but not for j\=k+1 since that’s the level of the Merkle root). In this case, without loss of generality, assume bk\=1. Then, we would have h\=zk+1\=H(zk,hk) and h\=zk+1′\=H(zk′,hk′). But since zk≠zk′, this gives a collision in H! (Alternatively, if bk\=0, just switch the inputs to the hash function.)
					  
					  **Extreme case \#2**: Another way would have been for the proof verification to yield zj\=zj′ for all j∈\[2,k+1\] (but not for j\=1 since that’s the level of xi and xi′ and they are not equal). In this case, without loss of generality, assume b1\=1. Then, we would have z2\=H(x1,h1) and z2′\=H(x1′,h1′). (Again, if bk\=0, just switch H’s inputs.) But since z2\=z2′ and xi≠xi′, this gives a collision in H!
					  
					  The point here is to see that, no matter what the two inconsistent proofs are, one can always work their way back to a collision in H, whether that collision is at the top of the tree (extreme case \#1), at the bottom of the tree (extreme case \#2) or anywhere in between, which we discuss next.
					  
					  You should now be able to see more easily that, as long as xi≠xi′ but the computed root hashes are the same (i.e., zk+1\=zk+1′\=h), then there must exist some level j∈\[k\] where there is a collision:
					  
					  ∃ level j∈\[k\] s.t. {H(zj−1,hj−1)\=H(zj−1′,hj−1′), if bj−1\=1H(hj−1,zj−1)\=H(hj−1′,zj−1′), if bj−1\=0but with zj−1≠zj−1′ or hj−1≠hj−1′
					  
					  (Again, recall that z1\=xi and z1′\=xi′.) But such a collision at level j implies a break in the collision-resistance of H, which is a contradiction. QED.
					  
					  The claim about the existence of such a level j might not be easy to understand at first glance. It is best to draw yourself a Merkle proof together with the hashes computed during its verification and run through the following mental exercise: 
					  
					  Start at the root of the tree, at level k+1! Since both proofs verify and yield the same root hash, it could be that we either have a collision in H at this level or we don’t. If we do have a collision, we are done. If we do not, then we know that zk\=zk′ and hk\=hk′.
					  
					  Next, work your way down and continue on the subtree with root hash zk\=zk′. Again, it must be that either there was a collision or that zk−1\=zk−1′ and hk−1\=hk−1′. If there was a collision, we are done. Otherwise, we continue recursively.
					  
					  In the end, we will get to the bottom level which is guaranteed to have z1≠z1′ (because xi≠xi′) while z2\=z2′ from the previous level, which yields a collision. This is actually the _extreme case \#2_ that we’ve handled above! No matter what, there will always be a collision!
					  
					  \#\# Acknowledgments
					  
					  Special thanks to [Ittai Abraham](https://decentralizedthoughts.github.io/about-ittai/) for reviewing a draft of this post and sending insightful comments.
					  
					  \#\# References
					  
					  1. To be more specific, a Merkle tree **can be viewed as** a hash function on n inputs, but can be so much more than that. For example, when Merkle hashing a _dictionary_ with a large key space, a Merkle tree can be viewed as a hash function on 2256 inputs, where most of them are not set (i.e., “null”), which makes computing it (in a careful manner) feasible. Importantly, these kinds of Merkle trees allow for **non-membership** proofs of inputs that are set to null. [↩](\#fnref:consideredtobe)
					  2. Computational intractability would deserve its own post. For now, just think of it as _“no algorithm we can conceive of can break collision resistance **and** finish executing before the heat death of the Universe.”_ [↩](\#fnref:handwave)
					  3. [The First Blockchain or How to Time-Stamp a Digital Document](https://decentralizedthoughts.github.io/2020-07-05-the-first-blockchain-or-how-to-time-stamp-a-digital-document/) [↩](\#fnref:post1)
					  4. [Security proof for Nakamoto Consensus](https://decentralizedthoughts.github.io/2019-11-29-Analysis-Nakamoto/) [↩](\#fnref:post2)
					  5. For a simple explanation of Bitcoin’s block structure, see the author’s [presentation on Catena](https://alinush.github.io/talks.html\#catena-efficient-non-equivocation-via-bitcoin), \#shamelessplug. [↩](\#fnref:catena)
					  6. I’m assuming Dropbox is smart and doesn’t send a hash twice when it’s shared by two proofs. This is why the overhead is only b−1. <a href="\#fnref:dedup" class="reversefootnote" role="doc-backlink">↩</a></p> </li> <li id="fn:CW09" role="doc-endnote"> <p><strong>Efficient Data Structures for Tamper-Evident Logging</strong>, by Crosby, Scott A. and Wallach, Dan S., <em>in Proceedings of the 18th Conference on USENIX Security Symposium</em>, 2009&nbsp;<a href="\#fnref:CW09" class="reversefootnote" role="doc-backlink">↩</a></p> </li> <li id="fn:MBBplus15" role="doc-endnote"> <p><strong>CONIKS: Bringing Key Transparency to End Users</strong>, by Marcela S. Melara and Aaron Blankstein and Joseph Bonneau and Edward W. Felten and Michael J. Freedman, <em>in 24th USENIX Security Symposium (USENIX Security 15)</em>, 2015, <a href="https://www.usenix.org/conference/usenixsecurity15/technical-sessions/presentation/melara">\[URL\]</a>&nbsp;<a href="\#fnref:MBBplus15" class="reversefootnote" role="doc-backlink">↩</a></p> </li> <li id="fn:LK15" role="doc-endnote"> <p><strong>Revocation Transparency</strong>, by Ben Laurie and Emilia Kasper, 2015, <a href="https://www.links.org/files/RevocationTransparency.pdf">\[URL\]</a>&nbsp;<a href="\#fnref:LK15" class="reversefootnote" role="doc-backlink">↩</a></p> </li> <li id="fn:ELC16" role="doc-endnote"> <p><strong>Verifiable Data Structures</strong>, by Eijdenberg, Adam and Laurie, Ben and Cutter, Al, 2016, <a href="https://github.com/google/trillian/blob/master/docs/papers/VerifiableDataStructures.pdf">\[URL\]</a>&nbsp;<a href="\#fnref:ELC16" class="reversefootnote" role="doc-backlink">↩</a></p> </li> <li id="fn:contrast" role="doc-endnote"> <p>In contrast, the collision-resistant functions <span class="MathJax\_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-245-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>H</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2513" style="width: 0.912em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.766em; height: 0px; font-size: 116%;"><span style="position: absolute; clip: rect(1.79em, 1000.77em, 2.73em, -1000em); top: -2.586em; left: 0em;"><span class="mrow" id="MathJax-Span-2514"><span class="mi" id="MathJax-Span-2515" style="font-family: STIXGeneral; font-style: italic;">H<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.047em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.586em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.056em; border-left: 0px solid; width: 0px; height: 0.869em;"></span></span></nobr><span class="MJX\_Assistive\_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>H</mi></math></span></span><script type="math/tex" id="MathJax-Element-245">H we discussed in our [previous post](https://decentralizedthoughts.github.io/2020-08-28-what-is-a-cryptographic-hash-function) take just one input x and hash it as h\=H(x). <a href="\#fnref:contrast" class="reversefootnote" role="doc-backlink">↩</a></p> </li> </ol> </div> </article> <div class="blog-tags"> Tags: <a href="/tags\#cryptography">cryptography</a> <a href="/tags\#merkle-hash-tree">merkle-hash-tree</a> <a href="/tags\#collision-resistant-hash-function">collision-resistant-hash-function</a> <a href="/tags\#integrity">integrity</a> </div> <!-- Check if any share-links are active --> <section id="social-share-section"> <span class="sr-only">Share: </span> <!--- Leave a comment on Twitter --> <a href="https://twitter.com/intent/tweet?text=What+is+a+Merkle+Tree%3F+https://decentralizedthoughts.github.io/2020-12-22-what-is-a-merkle-tree/" class="btn btn-social-icon btn-twitter" title="Leave a comment on Twitter"> <span class="fa fa-fw fa-twitter" aria-hidden="true"></span> <span class="sr-only">Twitter</span> </a> </section> <ul class="pager blog-pager"> <li class="previous"> <a href="/2020-12-12-raft-liveness-full-omission/" data-toggle="tooltip" data-placement="top" title="Raft does not Guarantee Liveness in the face of Network Faults">← Previous Post</a> </li> <li class="next"> <a href="/2021-02-28-good-case-latency-of-byzantine-broadcast-a-complete-categorization/" data-toggle="tooltip" data-placement="top" title="Good-case Latency of Byzantine Broadcast: a Complete Categorization">Next Post →</a> </li> </ul> <div class="disqus-comments"> </div> <div class="staticman-comments"> </div> <div class="justcomments-comments"> </div> </div> </div> </div> <footer> <div class="container beautiful-jekyll-footer"> <div class="row"> <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1"> <ul class="list-inline text-center footer-links"><li><a href="https://twitter.com/ittaia" title="Twitter"><span class="fa-stack fa-lg" aria-hidden="true"> <i class="fa fa-circle fa-stack-2x"></i> <i class="fa fa-twitter fa-stack-1x fa-inverse"></i> </span> <span class="sr-only">Twitter</span> </a> </li></ul> <p class="copyright text-muted"> Decentralized Thinkers &nbsp;•&nbsp; 2023 </p> <!-- Please don't remove this, keep my open source work credited :) --> <p class="theme-by text-muted"> Theme by <a href="https://deanattali.com/beautiful-jekyll/">beautiful-jekyll</a> </p> </div> </div> </div> </footer> <!-- doing something a bit funky here because I want to be careful not to include JQuery twice! --> <script> if (typeof jQuery == 'undefined') { document.write('<script src="/js/jquery-1.11.2.min.js"></scr' + 'ipt>'); }
					  - [Merkle Trees](https://omnivore.app/me/merkle-trees-18cfc251d0e)
					  collapsed:: true
					  site:: [asselinpaul.com](https://asselinpaul.com/merkle.html)
					  labels:: [[logseq]] [[cryptography]] [[merkle-airdrop]]
					  date-saved:: [[01/12/2024]]
					  - ### Content
					  collapsed:: true
					  - An efficient and verifiable data structure
					  
					  Merkle trees have become increasingly prevalent due to their key role in many decentralised systems. Although we are rarely exposed to them directly, they are worth studying for their elegance alone.
					  
					  \#\# Hash Functions
					  
					  A hash function is a mathematical function which maps anything (represented as a finite string of characters) into an output of fixed length. 
					  
					  This is tremendously useful. Any data has a signature that is significantly smaller than itself.
					  
					  ![hashing a string and an image example](https://proxy-prod.omnivore-image-cache.app/0x0,spSgTngAejCld5PFwb8k28su6seXV0GAKsPoAaDSeMFo/https://asselinpaul.com/img/merkle/hashing.png) 
					  
					  This signature can be used to verify the integrity of data downloaded from potentially dubious sources. Some sites will show the hash for files so that users can check whether the file has been tempered with.
					  
					  ![hashing an original versus a tempered version](https://proxy-prod.omnivore-image-cache.app/0x0,szpHB_wt6In68aZfnkVYnykhljgPg9fB0xQ_3XiUzLVc/https://asselinpaul.com/img/merkle/hashing2.png) 
					  
					  This highlights a key characteristic of hash functions. It must be _nearly_ impossible to modify a file to obtain a different file with the same hash (a hash collision).
					  
					  \#\#\# Properties of hash functions:
					  
					  * their input can be of any size
					  * their output is of fixed-size _e.g:_ 256-bit
					  * they are efficiently computable for an input of size _n_, its hash can be computed in _O(n)_ time
					  
					  \#\# Cryptographic Hash Functions
					  
					  Since Merkle trees are primarily used in settings where cryptography matters, our hash functions will have to abide by the following (stronger) properties:
					  
					  \#\#\# Collision Resistance
					  
					  A collision occurs when two distinct inputs produce the same output:
					  
					  ![example of a hash collision](https://proxy-prod.omnivore-image-cache.app/0x0,sYxeENI99j-1DR8jlwhjj4dZIyJPp-vPYLOVZPG-Vh7U/https://asselinpaul.com/img/merkle/collision.png) 
					  
					  A hash function is collision resistant if nobody can find a collision.
					  
					  It is worth noting that _"nobody can find"_ does not equate to _"does not exist"_. In fact, collisions do exist because the input space is larger than the output space. the input space is infinite whereas the output space is finite 
					  
					  This can be shown using the [pigeon-hole principle](https://en.wikipedia.org/wiki/Pigeonhole%5Fprinciple) , which states that if _n+1_ pigeons must inhabit _n_ pigeon-holes, at least one pigeon-hole must have more than one pigeon inhabitant.
					  
					  ![example of a hash collision](https://proxy-prod.omnivore-image-cache.app/0x0,scLtc4ma9jY61l1VpMmrzSirvrV-BfCZs1KYckKqCJnU/https://asselinpaul.com/img/merkle/io_space.png) 
					  
					  Using the pigeon-hole principle, we can formulate a process which is guaranteed to find a collision:
					  
					  Consider a hash function with a 256-bit output size. Pick 2256+1 distinct values and calculate their respective hashes. Since we picked more values than possible outputs, we are guaranteed that some pair collides when you apply the hash function.
					  
					  This method guarantees that we will cause a collision. We can reduce the number of values to examine while still being fairly likely to observe a collision by making use of the [birthday paradox](https://en.wikipedia.org/wiki/Birthday%5Fproblem) in a cryptographic setting (known as a [birthday attack](https://en.wikipedia.org/wiki/Birthday%5Fattack)). By examining the hashes of 2130+1 randomly chosen inputs, there is a 99.8 percent chance of finding a collision.
					  
					  This method of uncovering hash collisions works for every hash function but takes an impractical amount of time. It would likely take far longer than the age of the universe to find a collision using all the computing power available on earth. _Bitcoin and Cryptocurrency Technologies._ Princeton University Press, p.4
					  
					  For some hash functions, it it easy to produce collisions. Consider the following hash function:
					  
					  H(_x_) = _x_ mod 2256
					  
					  This function returns the last 256 bits of the input (a fixed-size output). It is trivial to find two values that have the same 256 bit ending. **5** and **5 + 2256** would cause a collision for example.
					  
					  We don't know whether such methods exist for other hash methods (_e.g_ [SHA512](https://en.wikipedia.org/wiki/SHA-2)). The general consensus is that these functions are suspected to be collision resistant but no formal proof has ever been made. The hash functions we rely on today have been tested thoroughly by mathematicians and no collision has been found. Occasionally, collisions are found after years of work and particular functions are phased out of cryptographic systems. such was the fate of [MD5](https://en.wikipedia.org/wiki/MD5)
					  
					  \#\#\# Hiding
					  
					  The second property our cryptographic hashes must abide by: given a hash output, it should be impossible to deduce its input.
					  
					  With **y = H(_x_)**, **_x_** can't be deduced from **y**.
					  
					  When the set of potential inputs is fairly restricted, this can be hard. Imagine a problem where you output _true_ or _false_. You then hash the result of the problem. An attacker can deduce whether the output was _true_ or _false_ by computing the hashes of those inputs on his own.
					  
					  In a sense, we want the hash input, **_x_** to be taken from a set that is very spread out. It turns out we can hide even an input that is not spread out by concatenating it with another input that is spread out.
					  
					  A hash function _H_ is said to be hiding if when a secret value _r_ is chosen from a probability distribution that has [_high min-entropy_](https://www.reddit.com/r/crypto/comments/4qwxdz/high%5Fminentropy/), then, given _H(r || x)_, it is infeasible to find x. where || denotes concatenation
					  
					  \#\#\# Puzzle Friendliness
					  
					  Lastly, we require hash functions to be puzzle friendly. This property is the least intuitive of the three.
					  
					  A hash function _H_ is said to be puzzle friendly if for ever every possible _n_\-bit output value, if _k_ is chosen from a distribution with high min-entropy, then it is infeasible to find _x_ such that _H(k || x)_ \= _y_ in time significantly less than 2n.
					  
					  In more concrete terms, if someone wants to have the hash function output **Z** and part of the input(**k**) has been chosen in a suitably random way, then it's very difficult to find the value **x** that makes the hash function output **Z**.
					  
					  \#\#\# An example: [SHA-256](https://en.wikipedia.org/wiki/SHA-2)
					  
					  Having examined the properties of cryptographic hash functions, let's take a look at a commonly used one: **[SHA-256](https://asselinpaul.com/SHA-256)**
					  
					  SHA-256 outputs 256-bit hashes and works by using the _Merkle-Damgård transform_ to convert the underlying fixed-length collision-resistant hash function the compression function into one that accepts arbitrary-length inputs.
					  
					  ![sha_256 diagram](https://proxy-prod.omnivore-image-cache.app/0x0,s9GPlXpfLn7hMwQYgluMR12gDL4y7hzLq-oOc9joSLpg/https://asselinpaul.com/img/merkle/sha_256.png) 
					  
					  The input is padded so that its length is a multiple of 512 bits. It can then be divided into the _n_ message block and processed as shown in the figure above.
					  
					  \#\#\# Recapitulative
					  
					  The hash serves as a fixed-length summary, or unambiguous signature, of a message. It gives us an efficient way to remember things we've seen before and to recognise them again. Whereas an entire file could have been hundreds-of-gigabytes in size, its hash is of fixed length (256 bits in our examples). This greatly reduces storage requirements and enables a great number of practical uses.
					  
					  It is worth noting that different applications of hash functions require varying properties. Although puzzle-friendliness might be required in cryptocurrency systems, it isn't for other applications.
					  
					  \#\# Hash Pointers
					  
					  ![hash pointer diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sFI2GpinpepXqC5RXy9Sfh8vaoPAIB7z2xjwSwCtYYQI/https://asselinpaul.com/img/merkle/hash_pointer.png) 
					  
					  A Hash Pointer is a data-structure that points to data along with a cryptographic hash of the data. In addition to enabling information retrieval, hash pointers can be used to verify the integrity of the data.
					  
					  \#\# Block Chains
					  
					  ![blockchain diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sfhTT81R7SiTcymb6FhD98Wri_vqmxKvWN06-v-EgbJc/https://asselinpaul.com/img/merkle/blockchain.png) 
					  
					  A block chain is a linked list that is built with hash pointers. It is a tamper-evident log because changes to the chain will cause detectable inconsistencies between a data block and the hash pointer to the data block (which cascades right through to the head pointer).
					  
					  ![blockchain cascading error diagram](https://proxy-prod.omnivore-image-cache.app/0x0,s5QiXMI3m1l0mdf4k6ZORhua0eXA47IZSR8iS1kby6DE/https://asselinpaul.com/img/merkle/blockchain2.png) 
					  
					  ![merkle tree diagram](https://proxy-prod.omnivore-image-cache.app/0x0,s1suFq6IGDmoMuCJNhzIa5aVXbSFYKjyyhtk-AtsIvLY/https://asselinpaul.com/img/merkle/merkle.png) 
					  
					  Ralph Merkle had the idea to combine binary trees and hash pointers to create the Merkle tree in 1979\. The leaves contain hash pointers (arranged in pairs) and the hash of each of these blocks is stored in the parent node.
					  
					  We only need to store the root of the tree to verify the integrity of the tree. As in the block chain example, a change at the leaf-level would induce a change to hashes that bubbles up all the way to the root node, effectively making the data-structure tamper-proof.
					  
					  The merkle tree improves over the block chain by enabling concise _proof of membership_. That is, it is efficient to figure out whether a data block is a member of the Merkle Tree.![](https://proxy-prod.omnivore-image-cache.app/0x0,sZbP1Yib1mky0uR-0Qc-nPy5_H6Xtuym1bsUKm0P1ViQ/https://asselinpaul.com/img/merkle/merkle_efficiency.png) Merkle tree efficiency in Bitcoin - [Source](http://chimera.labs.oreilly.com/books/1234000001802/ch07.html\#merkle%5Ftrees) 
					  
					  Proving that a data block is included in the tree only requires showing the blocks in the path from that data block to the root.
					  
					  ![merkle tree membership diagram](https://proxy-prod.omnivore-image-cache.app/0x0,sNSKnHUMCt-Ml8lxPAkXUA6hy3lX8lUt10P_M-7dlmOk/https://asselinpaul.com/img/merkle/membership.png) 
					  
					  If there are _n_ nodes in the tree, only about _log(n)_ items need to be shown. Even with large Merkle trees, we can prove membership in a relatively short time. The space and time requirements are of the order _log(n)_. whereas they are of the order _n_ for block chains 
					  
					  \#\# Uses of Merkle Trees
					  
					  \#\#\# Git
					  
					  The version control system uses Merkle Trees to see which files changed in the tracked directory.
					  
					  ![git merkle](https://proxy-prod.omnivore-image-cache.app/0x0,sOn6gOiiwV75CypwawtBWMIUNF4JyZwyoArSFDE-fxv4/https://asselinpaul.com/img/merkle/git.png) [Source](https://www.slideshare.net/japh44/code-is-not-text-how-graph-technologies-can-help-us-to-understand-our-code-better) 
					  
					  \#\#\# Bitcoin
					  
					  Merkle Trees are used to summarise all the transactions in a particular block, thus providing an efficient way to verify whether a transaction is included in a block.
					  
					  \#\#\# Databases
					  
					  Apache Cassandra, Dynamo and other NoSQL databases use Merkle trees to detect inconsistencies between replicas of databases. [Source](https://wiki.apache.org/cassandra/AntiEntropy) 
					  
					  Merkle trees are very commonly used to achieve consensus in distributed systems (_e.g:_ [IPFS](https://ipfs.io/) uses a Merkle-DAG , [BitTorrent](https://en.wikipedia.org/wiki/BitTorrent), [dat](https://datproject.org/)) or to counter data degradation (_e.g:_ [ZFS](https://en.wikipedia.org/wiki/ZFS)).
					  
					  \#\# Further Reading
					  
					  * Traditional Merkle trees cannot be applied to graphs because graphs are more complicated than trees. This paper from Ashish Kundu IBM T J Watson Research Center and Elisa Bertino Department of Computer Science and CERIAS, Purdue University expands on the subject: [On Hashing Graphs](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.294.8694)
					  * [Merkle Tree Traversal in Log Space and Time](https://link.springer.com/chapter/10.1007/978-3-540-24676-3%5F32) by Michael Szydlo RSA Laboratories presents a technique to efficiently traverse Merkle trees.
					  * [The Swirlds Hashgraph Consensus Algorithm: Fair, Fast, Byzantine Fault Tolerance](http://www.swirlds.com/whitepapers/) by Leemon Baird presents a different hash graph data-structure.
					  
					  \#\#\# Thank you for your time.
					  
					  \#\#\# Sources
					  
					  \#\#\# Colophon
					  - [History of Actors (eighty-twenty news)](https://omnivore.app/me/https-eighty-twenty-org-2016-10-18-actors-hopl-ref-blog-danfinla-18d02554ccc)
					  collapsed:: true
					  site:: [eighty-twenty.org](https://eighty-twenty.org/2016/10/18/actors-hopl?ref=blog.danfinlay.com)
					  author:: Tony Garnock-Jones
					  labels:: [[actors]] [[distributed systems]]
					  date-saved:: [[01/13/2024]]
					  date-published:: [[10/17/2016]]
					  - ### Content
					  collapsed:: true
					  - Yesterday, I presented the history of actors to Christos Dimoulas’[History of Programming Languages](http://www.seas.harvard.edu/courses/cs252/2016fa/)class.
					  
					  Here are the written-out talk notes I prepared beforehand. There is an annotated bibliography at the end.
					  
					  ---
					  
					  \#\# Introduction
					  
					  Today I’m going to talk about the actor model. I’ll first put the model in context, and then show three different styles of actor language, including two that aim to be realistic programming systems.
					  
					  I’m going to draw on a few papers:
					  
					  * 2016 survey by Joeri De Koster, Tom Van Cutsem, and Wolfgang De Meuter, for taxonomy and common terminology (SPLASH AGERE 2016)
					  * 1990 overview by Gul Agha, who has contributed hugely to the study of actors (Comm. ACM 1990)
					  * 1997 operational semantics for actors by Gul Agha, Ian Mason, Scott Smith, and Carolyn Talcott.
					  * brief mention of some of the content of the original 1973 paper by Carl Hewitt, Peter Bishop, and Richard Steiger. (IJCAI 1973)
					  * Joe Armstrong’s 2003 dissertation on Erlang.
					  * 2005 paper on the language E by Mark Miller, Dean Tribble, and Jonathan Shapiro. (TGC 2005)
					  
					  I’ve put an annotated bibliography at the end of this file.
					  
					  \#\# Actors in context: Approaches to concurrent programming
					  
					  One way of classifying approaches is along a spectrum of private vs. shared state.
					  
					  * Shared memory, threads, and locking: very limited private state, almost all shared
					  * Tuple spaces and my own research, Syndicate: some shared, some private and isolated
					  * Actors: almost all private and isolated, just enough shared to do routing
					  
					  Pure functional “data parallelism” doesn’t fit on this chart - it lacks shared mutable state entirely.
					  
					  \#\# The actor model
					  
					  The actor model, then is
					  
					  * asynchronous message passing between entities (actors)  
					  * with guaranteed delivery  
					  * addressing of messages by actor identity
					  * a thing called the “ISOLATED TURN PRINCIPLE”  
					  * no shared mutable state; strong encapsulation; no global mutable state  
					  * no interleaving; process one message at a time; serializes state access  
					  * liveness; no blocking
					  
					  In the model, each actor performs “turns” one after the other: a turn is taking a waiting message, interpreting it, and deciding on both a state update for the actor and a collection of actions to take in response, perhaps sending messages or creating new actors.
					  
					  A turn is a sequential, atomic block of computation that happens in response to an incoming message.
					  
					  What the actor model buys you:
					  
					  * modular reasoning about state in the overall system, and modular reasoning about local state change within an actor, because state is private, and access is serialized; only have to consider interleavings of messages
					  * no “deadlocks” or data races (though you can get “datalock” and other global non-progress in some circumstances, from logical inconsistency and otherwise)
					  * flexible, powerful capability-based security
					  * failure isolation and fault tolerance
					  
					  It’s worth remarking that actor-like concepts have sprung up several times independently. Hewitt and many others invented and developed actors in the 1970s, but there are two occasions where actors seem to have been independently reinvented, as far as I know.
					  
					  One is work on a capability-based operating system, KeyKOS, in the 1980s which involved a design very much like Hewitt’s actors, feeding into research which led ultimately to the language E.
					  
					  The other is work on highly fault-tolerant designs for telephone switches, also in the 1980s, which culminated in the language Erlang.
					  
					  Both languages are clearly actor languages, and in both cases, apparently the people involved were unaware of Hewitt’s actor model at the time.
					  
					  \#\# Terminology
					  
					  There are two kinds of things in the actor model: _messages_, which are data sent across some medium of communication, and _actors_, which are stateful entities that can only affect each other by sending messages back and forth.
					  
					  Messages are completely immutable data, passed by copy, which may contain references to other actors.
					  
					  Each actor has
					  
					  * private state; analogous to instance variables
					  * an interface; which messages it can respond to
					  
					  Together, the private state and the interface make up the actor’s_behaviour_, a key term in the actor literature.
					  
					  In addition, each actor has
					  
					  * a mailbox; inbox, message queue
					  * an address; denotes the mailbox
					  
					  Within this framework, there has been quite a bit of variety in how the model appears as a concrete programming language.
					  
					  De Koster et al. classify actor languages into
					  
					  * The Classic Actor Model (create, send, become)
					  * Active Objects (OO with a thread per object; copying of passive data between objects)
					  * Processes (raw Erlang; receive, spawn, send)
					  * Communicating Event-Loops (no passive data; E; near/far refs; eventual refs; batching)
					  
					  We see instances of the actor model all around us. The internet’s IP network is one example: hosts are the actors, and datagrams the messages. The web can be seen as another: a URL denotes an actor, and an HTTP request a message. Seen in a certain light, even Unix processes are actor-like (when they’re single threaded, if they only use fds and not shm). It can be used as a structuring principle for a system architecture even in languages like C or C++ that have no hope of enforcing any of the invariants of the model.
					  
					  \#\# Rest of the talk
					  
					  For the rest of the talk, I’m going to cover the classic actor model using Agha’s presentations as a guide; then I’ll compare it to E, a communicating event-loop actor language, and to Erlang, a process actor language.
					  
					  \#\# Classic Actor Model
					  
					  The original 1973 actor paper by Hewitt, Bishop and Steiger in the International Joint Conference on Artificial Intelligence, is incredibly far out!
					  
					  It’s a position paper that lays out a broad and colourful research vision. It’s packed with amazing ideas.
					  
					  The heart of it is that Actors are proposed as a universal programming language formalism ideally suited to building artificial intelligence.
					  
					  The goal really was A.I., and actors and programming languages were a means to that end.
					  
					  It makes these claims that actors bring great benefits in a huge range of areas:
					  
					  * foundations of semantics
					  * logic
					  * knowledge-based programming
					  * intentions (software contracts)
					  * study of expressiveness of programming languages
					  * teaching of computation
					  * extensible, modular programming
					  * privacy and protection
					  * synchronization constructs
					  * resource management
					  * structured programming
					  * computer architecture
					  
					  (It’s amazingly “full-stack”: computer architecture!?)
					  
					  In each of these areas, you can see what they were going for. In some, actors have definitely been useful; in others, the results have been much more modest.
					  
					  In the mid-to-late 70s, Hewitt and his students Irene Greif, Henry Baker, and Will Clinger developed a lot of the basic theory of the actor model, inspired originally by SIMULA and Smalltalk-71\. Irene Greif developed the first operational semantics for it as her dissertation work and Will Clinger developed a denotational semantics for actors.
					  
					  In the late 70s through the 80s and beyond, Gul Agha made huge contributions to the actor theory. His dissertation was published as a book on actors in 1986 and has been very influential. He separated the actor model from its A.I. roots and started treating it as a more general programming model. In particular, in his 1990 Comm. ACM paper, he describes it as a foundation for CONCURRENT OBJECT-ORIENTED PROGRAMMING.
					  
					  Agha’s formulation is based around the three core operations of the classic actor model:
					  
					  * `create`: constructs a new actor from a template and some parameters
					  * `send`: delivers a message, asynchronously, to a named actor
					  * `become`: within an actor, replaces its behaviour (its state & interface)
					  
					  The classic actor model is a UNIFORM ACTOR MODEL, that is, everything is an actor. Compare to uniform object models, where everything is an object. By the mid-90s, that very strict uniformity had fallen out of favour and people often worked with _two-layer_ languages, where you might have a functional core language, or an object-oriented core language, or an imperative core language with the actor model part being added in to the base language.
					  
					  I’m going to give a simplified, somewhat informal semantics based on his 1997 work with Mason, Smith and Talcott. I’m going to drop a lot of details that aren’t relevant here so this really will be simplified.
					  
					  ```coq
					  e  :=  λx.e  |  e e  |  x  | (e,e) | l | ... atoms, if, bool, primitive operations ...
					  |  create e
					  |  send e e
					  |  become e
					  
					  l  labels, PIDs; we'll use them like symbols here
					  
					  ```
					  
					  and we imagine convenience syntax
					  
					  ```ocaml
					  e1;e2              to stand for   (λdummy.e2) e1
					  let x = e1 in e2   to stand for   (λx.e2) e1
					  
					  match e with p -> e, p -> e, ...
					  to stand for matching implemented with if, predicates, etc.
					  
					  ```
					  
					  We forbid programs from containing literal process IDs.
					  
					  ```coq
					  v  :=  λx.e  |  (v,v)  |  l
					  
					  R  :=  []  |  R e  |  v R  |  (R,e)  |  (v,R)
					  |  create R
					  |  send R e  |  send v R
					  |  become R
					  
					  ```
					  
					  Configurations are a pair of a set of actors and a multiset of messages:
					  
					  ```makefile
					  m  :=  l <-- v
					  
					  A  :=  (v)l  |  [e]l
					  
					  C  :=  < A ... | m ... >
					  
					  ```
					  
					  The normal lambda-calculus-like reductions apply, like beta:
					  
					  ```jboss-cli
					  < ... [R[(λx.e) v]]l ... | ... >
					  --> < ... [R[e{v/x}  ]]l ... | ... >
					  
					  ```
					  
					  Plus some new interesting ones that are actor specific:
					  
					  ```jboss-cli
					  < ... (v)l ...    | ... l <-- v' ... >
					  --> < ... [v v']l ... | ...          ... >
					  
					  < ... [R[create v]]l       ... | ... >
					  --> < ... [R[l'      ]]l (v)l' ... | ... >   where l' fresh
					  
					  < ... [R[send l' v]]l ... | ... >
					  --> < ... [R[l'       ]]l ... | ... l' <-- v >
					  
					  < ... [R[become v]]l       ... | ... >
					  --> < ... [R[nil     ]]l' (v)l ... | ... >   where l' fresh
					  
					  ```
					  
					  Whole programs `e` are started with
					  
					  where `a` is an arbitrary label.
					  
					  Here’s an example - a mutable cell.
					  
					  ```xl
					  Cell = λcontents . λmessage . match message with
					                (get, k) --> become (Cell contents);
					                             send k contents
					                (put, v) --> become (Cell v)
					  
					  ```
					  
					  Notice that when it gets a `get` message, it first performs a `become`in order to quickly return to `ready` state to handle more messages. The remainder of the code then runs alongside the ready actor. Actions after a `become` can’t directly affect the state of the actor anymore, so even though we have what looks like multiple concurrent executions of the actor, there’s no sharing, and so access to the state is still serialized, as needed for the isolated turn principle.
					  
					  ```livecodeserver
					  < [let c = create (Cell 0) in
					  send c (get, create (λv . send c (put, v + 1)))]a | >
					  
					  < [let c = l1 in
					  send c (get, create (λv . send c (put, v + 1)))]a
					  (Cell 0)l1 | >
					  
					  < [send l1 (get, create (λv . send l1 (put, v + 1)))]a
					  (Cell 0)l1 | >
					  
					  < [send l1 (get, l2)]a
					  (Cell 0)l1
					  (λv . send l1 (put, v + 1))l2 | >
					  
					  < [l1]a
					  (Cell 0)l1
					  (λv . send l1 (put, v + 1))l2 | l1 <-- (get, l2) >
					  
					  < [l1]a
					  [Cell 0 (get, l2)]l1
					  (λv . send l1 (put, v + 1))l2 | >
					  
					  < [l1]a
					  [become (Cell 0); send l2 0]l1
					  (λv . send l1 (put, v + 1))l2 | >
					  
					  < [l1]a
					  [send l2 0]l3
					  (Cell 0)l1
					  (λv . send l1 (put, v + 1))l2 | >
					  
					  < [l1]a
					  [l2]l3
					  (Cell 0)l1
					  (λv . send l1 (put, v + 1))l2 | l2 <-- 0 >
					  
					  < [l1]a
					  [l2]l3
					  (Cell 0)l1
					  [send l1 (put, 0 + 1)]l2 | >
					  
					  < [l1]a
					  [l2]l3
					  (Cell 0)l1
					  [l1]l2 | l1 <-- (put, 1) >
					  
					  < [l1]a
					  [l2]l3
					  [Cell 0 (put, 1)]l1
					  [l1]l2 | >
					  
					  < [l1]a
					  [l2]l3
					  [become (Cell 1)]l1
					  [l1]l2 | >
					  
					  < [l1]a
					  [l2]l3
					  [nil]l4
					  (Cell 1)l1
					  [l1]l2 | >
					  
					  ```
					  
					  (You could consider adding a garbage collection rule like
					  
					  ```jboss-cli
					  < ... [v]l ... | ... >
					  --> < ...      ... | ... >
					  
					  ```
					  
					  to discard the final value at the end of an activation.)
					  
					  Because at this level all the continuations are explicit, you can encode patterns other than sequential control flow, such as fork-join.
					  
					  For example, to start two long-running computations in parallel, and collect the answers in either order, multiplying them and sending the result to some actor k’, you could write
					  
					  ```smali
					  let k = create (λv1 . become (λv2 . send k' (v1 * v2))) in
					  send task1 (req1, k);
					  send task2 (req2, k)
					  
					  ```
					  
					  Practically speaking, both Hewitt’s original actor language, PLASMA, and the language Agha uses for his examples in the 1990 paper, Rosette, have special syntax for ordinary RPC so the programmer needn’t manipulate continuations themselves.
					  
					  So that covers the classic actor model. Create, send and become. Explicit use of actor addresses, and lots and lots of temporary actors for inter-actor RPC continuations.
					  
					  Before I move on to Erlang: remember right at the beginning I told you the actor model was
					  
					  * asynchronous message passing, and
					  * the isolated turn principle
					  
					  ?
					  
					  The isolated turn principle requires _liveness_ \- you’re not allowed to block indefinitely while responding to a message!
					  
					  But here, we can:
					  
					  ```swift
					  let d = create (λc . send c c) in
					  send d d
					  
					  ```
					  
					  Compare this with
					  
					  ```armasm
					  letrec beh = (λc . become beh; send c c) in
					  let d = create beh in
					  send d d
					  
					  ```
					  
					  These are both degenerate cases, but in different ways: the first becomes inert very quickly and the actor `d` is never returned to an idle/ready state, while the second spins uselessly forever.
					  
					  Other errors we could make would be to fail to send an expected reply to a continuation.
					  
					  One thing the semantics here rules out is interaction with other actors before doing a `become`; there’s no way to have a waiting continuation perform the `become`, because by that time you’re in a different actor. In this way, it sticks to the very letter of the Isolated Turn Principle, by forbidding “blocking”, but there are other kinds of things that can go wrong to destroy progress.
					  
					  Even if we require our behaviour functions to be total, we can still get _global_ nontermination.
					  
					  So saying that we “don’t have deadlock” with the actor model is very much oversimplified, even at the level of simple formal models, let alone when it comes to realistic programming systems.
					  
					  In practice, programmers often need “blocking” calls out to other actors before making a state update; with the classic actor model, this can be done, but it is done with a complicated encoding.
					  
					  \#\# Erlang: Actors for fault-tolerant systems
					  
					  Erlang is an example of what De Koster et al. call a Process-based actor language.
					  
					  It has its origins in telephony, where it has been used to build telephone switches with fabled “nine nines” of uptime. The research process that led to Erlang concentrated on high-availability, fault-tolerant software. The reasoning that led to such an actor-like system was, in a nutshell:
					  
					  * Programs have bugs. If part of the program crashes, it shouldn’t corrupt other parts. Hence strong isolation and shared-nothing message-passing.
					  * If part of the program crashes, another part has to take up the slack, one way or another. So we need crash signalling so we can detect failures and take some action.
					  * We can’t have all our eggs in one basket! If one machine fails at the hardware level, we need to have a nearby neighbour that can smoothly continue running. For that redundancy, we need distribution, which makes the shared-nothing message passing extra attractive as a unifying mechanism.
					  
					  Erlang is a two-level system, with a functional language core equipped with imperative actions for asynchronous message send and spawning new processes, like Agha’s system.
					  
					  The difference is that it lacks `become`, and instead has a construct called `receive`.
					  
					  Erlang actors, called processes, are ultra lightweight threads that run sequentially from beginning to end as little functional programs. As it runs, no explicit temporary continuation actors are created: any time it uses receive, it simply blocks until a matching message appears.
					  
					  After some initialization steps, these programs typically enter a message loop. For example, here’s a mutable cell:
					  
					  ```erlang
					  mainloop(Contents) ->
					  receive
					  {get, K} -> K ! Contents,
					  mainloop(Contents);
					  {put, V} -> mainloop(V)
					  end.
					  
					  ```
					  
					  A client program might be
					  
					  ```crystal
					  Cell = spawn(fun mainloop(0) end),
					  Cell ! {get, self()},
					  receive
					  V -> ...
					  end.
					  
					  ```
					  
					  Instead of using `become`, the program performs a tail call which returns to the `receive` statement as the last thing it does.
					  
					  Because `receive` is a statement like any other, Erlang processes can use it to enter substates:
					  
					  ```routeros
					  mainloop(Service) ->
					  receive
					  {req, K} -> Service ! {subreq, self()},
					  receive
					  {subreply, Answer} -> K ! {reply, Answer},
					                        mainloop(Service)
					  end
					  end.
					  
					  ```
					  
					  While the process is blocked on the inner `receive`, it only processes messages matching the patterns in that inner receive, and it isn’t until it does the tail call back to `mainloop` that it starts waiting for `req` messages again. In the meantime, non-matched messages queue up waiting to be `receive`d later.
					  
					  This is called “selective receive” and it is difficult to reason about. It doesn’t _quite_ violate the letter of the Isolated Turn Principle, but it comes close. (`become` can be used in a similar way.)
					  
					  The goal underlying “selective receive”, namely changing the set of messages one responds to and temporarily ignoring others, is important for the way people think about actor systems, and a lot of research has been done on different ways of selectively enabling message handlers. See Agha’s 1990 paper for pointers toward this research.
					  
					  One unique feature that Erlang brings to the table is crash signalling. The jargon is “links” and “monitors”. Processes can ask the system to make sure to send them a message if a monitored process exits. That way, they can perform RPC by
					  
					  * monitoring the server process
					  * sending the request
					  * if a reply message arrives, unmonitor the server process and continue
					  * if an exit message arrives, the service has crashed; take some corrective action.
					  
					  This general idea of being able to monitor the status of some other process was one of the seeds of my own research and my language Syndicate.
					  
					  So while the classic actor model had create/send/become as primitives, Erlang has spawn/send/receive, and actors are processes rather than event-handler functions. The programmer still manipulates references to actors/processes directly, but there are far fewer explicit temporary actors created compared to the “classic model”; the ordinary continuations of Erlang’s functional fragment take on those duties.
					  
					  \#\# E: actors for secure cooperation
					  
					  The last language I want to show you, E, is an example of what De Koster et al. call a Communicating Event-Loop language.
					  
					  E looks and feels much more like a traditional object-oriented language to the programmer than either of the variations we’ve seen so far.
					  
					  ```applescript
					  def makeCell (var contents) {
					  def getter {
					  to get() { return contents }
					  }
					  def setter {
					  to set(newContents) {
					  contents := newContents
					  }
					  }
					  return [getter, setter]
					  }
					  
					  ```
					  
					  The mutable cell in E is interestingly different: It yields _two_values. One specifically for setting the cell, and one for getting it. E focuses on security and securability, and encourages the programmer to hand out objects that have the minimum possible authority needed to get the job done. Here, we can safely pass around references to`getter` without risking unauthorized update of the cell.
					  
					  E uses the term “vat” to describe the concept closest to the traditional actor. A vat has not only a mailbox, like an actor, but also a call stack, and a heap of local objects. As we’ll see, E programmers don’t manipulate references to vats directly. Instead, they pass around references to objects in a vat’s heap.
					  
					  This is interesting because in the actor model messages are addressed to a particular actor, but here we’re seemingly handing out references to something finer grained: to individual objects _within_ an actor or vat.
					  
					  This is the first sign that E, while it uses the basic create/send/become-style model at its core, doesn’t expose that model directly to the programmer. It layers a special E-specific protocol on top, and only lets the programmer use the features of that upper layer protocol.
					  
					  There are two kinds of references available: near refs, which definitely point to objects in the local vat, and far refs, which may point to objects in a different vat, perhaps on another machine.
					  
					  To go with these two kinds of refs, there are two kinds of method calls: _immediate_ calls, and _eventual_ calls.
					  
					  receiver.method(arg, …)
					  
					  receiver <- method(arg, …)
					  
					  It is an error to use an immediate call on a ref to an object in a different vat, because it blocks during the current turn while the answer is computed. It’s OK to use eventual calls on any ref at all, though: it causes a message to be queued in the target vat (which might be our own), and a _promise_ is immediately returned to the caller.
					  
					  The promise starts off unresolved. Later, when the target vat has computed and sent a reply, the promise will become resolved. A nifty trick is that even an unresolved promise is useful: you can _pipeline_them. For example,
					  
					  ```ruby
					  def r1 := x <- a()
					  def r2 := y <- b()
					  def r3 := r1 <- c(r2)
					  
					  ```
					  
					  would block and perform multiple network round trips in a traditional simple RPC system; in E, there is a protocol layered on top of raw message sending that discusses promise creation, resolution and use. This protocol allows the system to send messages like
					  
					  ```smalltalk
					  "Send a() to x, and name the resulting promise r1"
					  "Send b() to y, and name the resulting promise r2"
					  "When r1 is known, send c(r2) to it"
					  
					  ```
					  
					  Crucial here is that the protocol, the language of discourse between actors, allows the expression of concepts including the notion of a send to happen at a future time, to a currently-unknown recipient.
					  
					  The protocol and the E vat runtimes work together to make sure that messages get to where they need to go efficiently, even in the face of multiple layers of forwarding.
					  
					  Each turn of an E vat involves taking one message off the message queue, and dispatching it to the local object it denotes. Immediate calls push stack frames on the stack as usual for object-oriented programming languages; eventual calls push messages onto the message queue. Execution continues until the stack is empty again, at which point the turn concludes and the next turn starts.
					  
					  One interesting problem with using promises to represent cross-vat interactions is how to do control flow. Say we had
					  
					  ```armasm
					  def r3 := r1 <- c(r2)  // from earlier
					  if (r3) {
					  myvar := a();
					  } else {
					  myvar := b();
					  }
					  
					  ```
					  
					  By the time we need to make a decision which way to go, the promise r3 may not yet be resolved. E handles this by making it an error to depend immediately on the value of a promise for control flow; instead, the programmer uses a `when` expression to install a handler for the event of the resolution of the promise:
					  
					  ```livescript
					  when (r3) -> {
					  if (r3) {
					  myvar := a();
					  } else {
					  myvar := b();
					  }
					  }
					  
					  ```
					  
					  The test of r3 and the call to a() or b() and the update to myvar are_delayed_ until r3 has resolved to some value.
					  
					  This looks like it violates the Isolated Turn Principle! It seems like we now have some kind of interleaving. But what’s going on under the covers is that promise resolution is done with an incoming message, queued as usual, and when that message’s turn comes round, the when clause will run.
					  
					  Just like with classic actors and with Erlang, managing these multiple-stage, stateful interactions in response to some incoming message is generally difficult to do. It’s a question of finding a balance between the Isolated Turn Principle, and its commitment to availability, and encoding the necessary state transitions without risking inconsistency or deadlock.
					  
					  Turning to failure signalling, E’s vats are not just the units of concurrency but also the units of partial failure. An uncaught exception within a vat destroys the whole vat and invalidates all remote references to objects within it.
					  
					  While in Erlang, processes are directly notified of failures of other processes as a whole, E can be more fine-grained. In E, the programmer has a convenient value that represents the outcome of a transaction: the promise. When a vat fails, or a network problem arises, any promises depending on the vat or the network link are put into a special state: they become _broken promises_. Interacting with a broken promise causes the contained exception to be signalled; in this way, broken promises propagate failure along causal chains.
					  
					  If we look under the covers, E seems to have a “classic style” model using create/send/become to manage and communicate between whole vats, but these operations aren’t exposed to the programmer. The programmer instead manipulates two-part “far references” which denote a vat along with an object local to that vat. Local objects are created frequently, like in regular object-oriented languages, but vats are created much less frequently; and each vat’s stack takes on duties performed in “classic” actor models by temporary actors.
					  
					  \#\# Conclusion
					  
					  I’ve presented three different types of actor language: the classic actor model, roughly as formulated by Agha et al.; the process actor model, represented by Erlang; and the communicating event-loop model, represented by E.
					  
					  The three models take different approaches to reconciling the need to have structured local data within each actor in addition to the more coarse-grained structure relating actors to each other.
					  
					  The classic model makes everything an actor, with local data largely deemphasised; Erlang offers a traditional functional programming model for handling local data; and E offers a smooth integration between an imperative local OO model and an asynchronous, promise-based remote OO model.
					  
					  ---
					  
					  \#\# Annotated Bibliography
					  
					  \#\#\# Early work on actors
					  
					  * **1973\. Carl Hewitt, Peter Bishop, and Richard Steiger, “A universal modular ACTOR formalism for artificial intelligence,” in Proc. International Joint Conference on Artificial Intelligence**  
					  This paper is a position paper from which we can understand the motivation and intentions of the research into actors; it lays out a very broad and colourful research vision that touches on a huge range of areas from computer architecture up through programming language design to teaching of computer science and approaches to artificial intelligence.  
					  The paper presents a _uniform actor model_; compare and contrast with uniform object models offered by (some) OO languages. The original application of the model is given as PLANNER-like AI languages.  
					  The paper claims benefits of using the actor model in a huge range of areas:  
					  * foundations of semantics  
					  * logic  
					  * knowledge-based programming  
					  * intentions (contracts)  
					  * study of expressiveness  
					  * teaching of computation  
					  * extensible, modular programming  
					  * privacy and protection  
					  * synchronization constructs  
					  * resource management  
					  * structured programming  
					  * computer architecture  
					  The paper sketches the idea of a contract (called an “intention”) for ensuring that invariants of actors (such as protocol conformance) are maintained; there seems to be a connection to modern work on “monitors” and Session Types. The authors write:  
					  > The intention is the CONTRACT that the actor has with the outside world.  
					  Everything is super meta! Actors can have intentions! Intentions are actors! Intentions can have intentions! The paper presents the beginnings of a reflective metamodel for actors. Every actor has a scheduler and an “intention”, and may have monitors, a first-class environment, and a “banker”.  
					  The paper draws an explicit connection to capabilities (in the security sense); Mark S. Miller at<http://erights.org/history/actors.html> says of the Actor work that it included “prescient” statements about Actor semantics being “a basis for confinement, years before Norm Hardy and the Key Logic guys”, and remarks that “the Key Logic guys were unaware of Actors and the locality laws at the time, \[but\] were working from the same intuitions.”  
					  There are some great slogans scattered throughout, such as “Control flow and data flow are inseparable”, and “Global state considered harmful”.  
					  The paper does eventually turn to a more nuts-and-bolts description of a predecessor language to PLASMA, which is more fully described in Hewitt 1976.  
					  When it comes to formal reasoning about actor systems, the authors here define a partial order - PRECEDES - that captures some notion of causal connection. Later, the paper makes an excursion into epistemic modal reasoning.  
					  Aside: the paper discusses continuations; Reynolds 1993 has the concept of continuations as firmly part of the discourse by 1973, having been rediscovered in a few different contexts in 1970-71 after van Wijngaarden’s 1964 initial description of the idea. See J. C. Reynolds, “The discoveries of continuations,” LISP Symb. Comput., vol. 6, no. 3–4, pp. 233–247, 1993.  
					  In the “acknowledgements” section, we see:  
					  > Alan Kay whose FLEX and SMALL TALK machines have influenced our work. Alan emphasized the crucial importance of using intentional definitions of data structures and of passing messages to them. This paper explores the consequences of generalizing the message mechanism of SMALL TALK and SIMULA-67; the port mechanism of Krutar, Balzer, and Mitchell; and the previous CALL statement of PLANNER-71 to a universal communications mechanism.
					  * **1975\. Irene Greif. PhD dissertation, MIT EECS.**  
					  Specification language for actors; per Baker an “operational semantics”. Discusses “continuations”.
					  * **1976\. C. Hewitt, “Viewing Control Structures as Patterns of Passing Messages,” AIM-410**  
					  AI focus; actors as “agents” in the AI sense; recursive decomposition: “each of the experts can be viewed as a society that can be further decomposed in the same way until the primitive actors of the system are reached.”  
					  > We are investigating the nature of the _communication mechanisms_\[…\] and the conventions of discourse  
					  More concretely, examines “how actor message passing can be used to understand control structures as patterns of passing messages”.  
					  > \[…\] there is no way to decompose an actor into its parts. An actor is defined by its behavior; not by its physical representation!  
					  Discusses PLASMA (“PLAnner-like System Modeled on Actors”), and gives a fairly detailed description of the language in the appendix. Develops “event diagrams”.  
					  Presents very Schemely factorial implementations in recursive and iterative (tail-recursive accumulator-passing) styles. During discussion of the iterative factorial implementation, Hewitt remarks that `n` is not closed over by the `loop` actor; it is “not an acquaintance” of `loop`. Is this the beginning of the line of thinking that led to Clinger’s “safe-for-space” work?  
					  Everything is an actor, but some of the actors are treated in an awfully structural way: the trees, for example, in section V.4 on Generators:  
					  ```clojure  
					  (non-terminal:  
					  (non-terminal: (terminal: A) (terminal: B))  
					  (terminal: C))  
					  ```  
					  Things written with this `keyword:` notation look like structures. Their _reflections_ are actors, but as structures, they are subject to pattern-matching; I am unsure how the duality here was thought of by the principals at the time, but see the remarks regarding “packages” in the appendix.
					  * **1977\. C. Hewitt and H. Baker, “Actors and Continuous Functionals,” MIT A.I. Memo 436A**  
					  Some “laws” for communicating processes; “plausible restrictions on the histories of computations that are physically realizable.” Inspired by physical intuition, discusses the history of a computation in terms of a partial order of events, rather than a sequence.  
					  > The actor model is a formalization of these ideas \[of Simula/Smalltalk/CLU-like active data processing messages\] that is independent of any particular programming language.  
					  > Instances of Simula and Smalltalk classes and CLU clusters are actors”, but they are _non-concurrent_. The actor model is broader, including concurrent message-passing behaviour.  
					  Laws about (essentially) lexical scope. Laws about histories (finitude of activation chains; total ordering of messages inbound at an actor; etc.), including four different ordering relations. “Laws of locality” are what Miller was referring to on that erights.org page I mentioned above; very close to the capability laws governing confinement.  
					  Steps toward denotational semantics of actors.
					  * **1977\. Russell Atkinson and Carl Hewitt, “Synchronization in actor systems,” Proc. 4th ACM SIGACT-SIGPLAN Symp. Princ. Program. Lang., pp. 267–280.**  
					  Introduces the concept of “serializers”, a “generalization and improvement of the monitor mechanism of Brinch-Hansen and Hoare”. Builds on Greif’s work.
					  * **1981\. Will Clinger’s PhD dissertation. MIT**
					  
					  \#\#\# Actors as Concurrent Object-Oriented Programming
					  
					  * **1986\. Gul Agha’s book/dissertation.**
					  * **1990\. G. Agha, “Concurrent Object-Oriented Programming,” Commun. ACM, vol. 33, no. 9, pp. 125–141**  
					  Agha’s work recast the early “classic actor model” work in terms of_concurrent object-oriented programming_. Here, he defines actors as “self-contained, interactive, independent components of a computing system that communicate by asynchronous message passing”, and gives the basic actor primitives as `create`, `send to`, and `become`. Examples are given in the actor language Rosette.  
					  This paper gives an overview and summary of many of the important facets of research on actors that had been done at the time, including brief discussion of: nondeterminism and fairness; patterns of coordination beyond simple request/reply such as transactions; visualization, monitoring and debugging; resource management in cases of extremely high levels of potential concurrency; and reflection.  
					  > The customer-passing style supported by actors is the concurrent generalization of continuation-passing style supported in sequential languages such as Scheme. In case of sequential systems, the object must have completed processing a communication before it can process another communication. By contrast, in concurrent systems it is possible to process the next communication as soon as the replacement behavior for an object is known.  
					  Note that the sequential-seeming portions of the language are defined in terms of asynchronous message passing and construction of explicit continuation actors.
					  * **1997\. G. A. Agha, I. A. Mason, S. F. Smith, and C. L. Talcott, “A Foundation for Actor Computation,” J. Funct. Program., vol. 7, no. 1, pp. 1–72**  
					  Long paper that carefully and fully develops an operational semantics for a concrete actor language based on lambda-calculus. Discusses various equivalences and laws. An excellent starting point if you’re looking to build on a modern approach to operational semantics for actors.
					  
					  \#\#\# Erlang: Actors from requirements for fault-tolerance / high-availability
					  
					  * **2003\. J. Armstrong, “Making reliable distributed systems in the presence of software errors,” Royal Institute of Technology, Stockholm**  
					  A good overview of Erlang: the language, its design intent, and the underlying philosophy. Includes an evaluation of the language design.
					  
					  \#\#\# E: Actors from requirements for secure interaction
					  
					  * **2005\. M. S. Miller, E. D. Tribble, and J. Shapiro, “Concurrency Among Strangers,” in Proc. Int. Symp. on Trustworthy Global Computing, pp. 195–229.**  
					  As I summarised this paper for a seminar class on distributed systems: “The authors present E, a language designed to help programmers manage_coordination_ of concurrent activities in a setting of distributed, mutually-suspicious objects. The design features of E allow programmers to take control over concerns relevant to distributed systems, without immediately losing the benefits of ordinary OO programming.”  
					  E is a canonical example of the “communicating event loops” approach to Actor languages, per the taxonomy of the survey paper listed below. It combines message-passing and isolation in an interesting way with ordinary object-oriented programming, giving a two-level language structure that has an OO flavour.  
					  The paper does a great job of explaining the difficulties that arise when writing concurrent programs in traditional models, thereby motivating the actor model in general and the features of E in particular as a way of making the programmer’s job easier.
					  
					  \#\#\# Taxonomy of actors
					  
					  * **2016\. J. De Koster, T. Van Cutsem, and W. De Meuter, “43 Years of Actors: A Taxonomy of Actor Models and Their Key Properties,” Software Languages Lab, Vrije Universiteit Brussel, VUB-SOFT-TR-16-11**  
					  A very recent survey paper that offers a taxonomy for classifying actor-style languages. At its broadest, actor languages are placed in one of four groups:  
					  * The Classic Actor Model (create, send, become)  
					  * Active Objects (OO with a thread per object; copying of passive data between objects)  
					  * Processes (raw Erlang; receive, spawn, send)  
					  * Communicating Event-Loops (E; near and far references; eventual references; batching)  
					  Different kinds of “futures” or “promises” also appear in many of these variations in order to integrate asynchronous message_reception_ with otherwise-sequential programming.
	- [blockchain-new-paper](https://omnivore.app/me/u-7-e-8-dd-540-b-132-11-ee-b-631-eb-976-daebc-17-blockchain-new--18cfd24bdf0)
	  collapsed:: true
	  site:: [omnivore.app](https://omnivore.app/attachments/u/7e8dd540-b132-11ee-b631-eb976daebc17/blockchain-new-paper.pdf)
	  author:: MCA-46
	  date-saved:: [[01/12/2024]]
		- ### Content
		  collapsed:: true
			-