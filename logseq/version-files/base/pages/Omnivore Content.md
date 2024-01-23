## All posts
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