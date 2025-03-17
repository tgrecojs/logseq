## Overview
	- Algebraic data types provide us with a means for transforming simple objects into  more complex objects. This transformation occurs by way of applying some operation (on an object); the most pervasive ways to combine structures are:
		- Product Type
		  logseq.order-list-type:: number
			- a.k.a. **composition**
			- $$A=(B,C)$$
			-
		- Coproduct Type (Union Type)
		  logseq.order-list-type:: number
			- a.k.a. **inheritence**
			- $$A= B + C $$
		- Recursive Type
		  logseq.order-list-type:: number
			- a.k.a. **a type that contains the same type**
			- Example:
				- `rest` is a `List`
				- `const List = (value, rest) => ({value, rest})`
				- $$A = (B, A) + C$$
			-
	- Algebraic Data Types provide us with a unified interface for composing across types.
	- > The fact that a data type can be formed by applying some operations between them gives them the name algebraic. An algebraic data type is a kind of composite type.
- ### `List`
	- The algebraic definition for a list can be described as:
		- A list is either
			- an empty list.
			  logseq.order-list-type:: number
			- a concatenation of an element and a list.
			  logseq.order-list-type:: number
- ###
-