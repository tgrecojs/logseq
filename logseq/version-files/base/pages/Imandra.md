- ## Resources
- [Demos](https://www.imandra.ai/demos)
  logseq.order-list-type:: number
- [Documentation](https://docs.imandra.ai/imandra-docs)
  logseq.order-list-type:: number
-
- ## Stop Light Model
- link - https://docs.imandra.ai/imandra-docs/notebooks/simple-stoplight-model/
-
-
-
- ## UBS Dark Pool
- UBS' order priority logic was flawed due to **the order ranking not being transitive.**
- ### Analysis
- Defined a type corresponding to:
	- relevant market data signals.
	- the current National Best Bid and Offer (a.k.a. [NBB/O](https://www.investopedia.com/terms/n/nbbo.asp)). The NBB/O is the highest bid price and the lowest big (ask) price available across all markets.
- *The NBB/O is a composite of the tighest bid-ask spread in the market*
- ![Bid-Ask Spread](https://www.investopedia.com/thmb/bCGbSYtlVNkYGX1LhFg673U0SWI=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/bid-askspread-v4-dc6919600c5b4c5084b21e24fd3c7e5e.jpg)
- ### Market Conditions
- /pyth
	-
	-
-
- https://docs.imandra.ai/imandra-docs/notebooks/ubs-case-study/
-
- [[Transitive Law]]
-
- ```ocaml
  type mkt_cond = MKT_NORMAL | MKT_CROSSED | MKT_LOCKED
  
  let which_mkt mkt =
    if mkt.nbo = mkt.nbb then MKT_LOCKED
    else if mkt.nbo <. mkt.nbb then MKT_CROSSED
    else MKT_NORMAL
  
  let mid_point mkt = (mkt.nbb +. mkt.nbo) /. 2.0
  ```
-
- ```
  type mkt_cond = MKT_NORMAL | MKT_CROSSED | MKT_LOCKED
  let which_mkt mkt =
  if mkt.nbo = mkt.nbb then MKT_LOCKED
  else if mkt.nbo <. mkt.nbb then MKT_CROSSED
  else MKT_NORMAL
  let mid_point mkt = (mkt.nbb +. mkt.nbo) /. 2.0
  ```
- ```javascript
  const hiThere = string => string + !;
  ```
-
-
- ### FaceTime Bug
- @@html: https://core-europe-west1.imandra.ai/imandra-notebook/http/0d4a4520-b1e1-4262-800f-d9c04a3118bd/notebooks/Exploring%20the%20FaceTime%20Bug%20With%20ReasonML%20State%20Machines.ipynb#@@
-
- tags: [[formal verification]], [[ocaml]], [[functional programming]]