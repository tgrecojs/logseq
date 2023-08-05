- ## Overview
- Matches buyers and sellers (counter parties) with overlapping loan preferences and executes the exchanging of allocations.
-
- across all active offers within a market.
-
-
- ## Design Description
- Amount data type held inside of an `Intersection`
	- #Semigroup
	- find intersections across all active offers within a market..
	- FIFO queue
		- When there are more than one matches, the duration decides which offers will be executed. The order that has been on the market longest wins.
-
- #### Intersection
	- ```js
	  const Intersection = values => ({
	    - values,
	    - concat: other => 
	    		Intersection(valus.filter(x => other.values.some(y => x === y)))
	  - })
	  ```
	- https://egghead.io/lessons/javascript-real-world-example-pt3
		- #drboolean, #[[Functional Programming]]
		-
		-
		-
		-
		-
		-
		- tags:: #[[lari finance]]