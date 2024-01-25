- A SwingSet machine has two sets of vats. The first are the *Static Vats*, which are defined in the configuration object, and created at startup time.
- The root objects of these vats are made available to the `bootstrap()` method, which can wire them together as it sees fit.
- The second set are the *Dynamic Vats*. These are  created after startup, with source code bundles that were not known at 
  boot time. Typically these bundles arrive over the network from external providers. In the Agoric system, contracts are installed as dynami vats (each spawned instance goes into a separate vat).
- This document describes how you define and configure the static vats.