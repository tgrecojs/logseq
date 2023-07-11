- ## Fantasy Land Specification
- *Specification for interoperability of common algebraic structures in JavaScript*
	- a.k.a "Algebraic JavaScript Specification"
- ### Overview
- The Fantasy Land spec is made up of 20+ "Algebras".
- An Algebra is a type that contains a value, as well as a set of operations.
	- Algebras have concrete laws that they must abide by.
	- These laws change depending on the type.
		- Ex. `Functor` vs `Monad`
- Often time, an Algebra is created by extending a different Algebra.
	- Ex. `Functor -> Monad`
		- Since Monad is also a functor, it must abide by the Functor Laws as well as the Monadic Laws.
- ## Meet the Algebras
-
- ###
-
- ![fl-spec.png](../assets/fl-spec_1688872479067_0.png)
- ### Setoid
-
-
- ### Commutative Diagrams
	- functions need to be **pure** so that they uphold the commutative law.
- ### on types..
	- lowercase `a`, `b`, `c`, etc. refer to a type.
	- ```js
	  // getName:: Object -> String
	  const getName = ({name}) => name;
	  // length::String -> Number
	  const length = ({length}) => length;
	  ```
	-
-
-