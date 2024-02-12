hypothesis-uri:: https://web.archive.org/web/20160404122250/http://wiki.ecmascript.org/doku.php?id=strawman:concurrency
hypothesis-title:: strawman:concurrency [ES Wiki]
hypothesis-naming-scheme:: 0.2.0

- *
  hid:: A0iDaMk3Ee64SPMoSQl0Vg
  updated:: 2024-02-11T23:41:01.019695+00:00
   On both the browser and the server, JavaScript’s de-facto concurrency model is increasingly “shared nothing” communicating event loops.*
- *Aggregate objects into process-like units called vats. Objects in one vat can only send asynchronous messages to objects in other vats. Promises represent such references to potentially remote objects. Eventual message sends queue eventual-deliveries in the work queue of the vat hosting the target object. A vat’s thread processes each eventual-delivery to completion before proceeding to the next.* #[[Async]] #[[Event-loops]]
  hid:: A3wRyMk4Ee6K4ZtDmb6ufg
  updated:: 2024-02-11T23:58:28.344056+00:00
  > ## Terminology
  - **Vats**
      - Process-like units.
      - Corresponds to the miimal possible unit of concurrency.
      - Each Vat consists of:
          -  a single sequential thread of control,\
          - a single call-return stack,
          - a single fifo queue holding eventual-deliveries,
          - an internal object heap,
          - and incoming and outgoing remote references.
  
      - `then` registers a callback to happen ina seperate turn once a promise is resolved, *providing the callback with the promise's resoluton.
  
  - **Turn**
      - The name for each processing step that occurs within a vat.
  
  
  ## Why Communicating Event Loops
  - Free of conventional race condition or deadlock bugs.
  - State isolation
      - While a turn is in progress, it has mutually exclusive access to all state which it has synchronous access (i.e. all state within its vat).
      - This results in programs avoiding race conditions.
  - Non-blocking
      - This model provides no locks or blocking constructs of any kind, although it does not forbid a host environment from blocking constructs (i.e. `alert`)