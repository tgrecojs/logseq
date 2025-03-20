## Conjunction (a.k.a. the AND operator)
	- ### Properties of Conjunction
		- Associativity 
		  logseq.order-list-type:: number
		- Commutativity
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			- ![ss_09052024_000494.gif](../assets/ss_09052024_000494_1725568910729_0.gif)
			  logseq.order-list-type:: number
		- Truth Table
			- ![ss_09052024_000495.png](../assets/ss_09052024_000495_1725568973152_0.png)
		- ### Iterative Conjunction
			- *Given `n` propositions*...
				- $$P1, P2, .., Pn,$$
			- We can "take" their conjunction
				- $$p\equiv p_1\wedge p_2\wedge\cdots\wedge p_n.$$
				- #### Iterative Notation
			- $$ p\equiv\bigwedge_{i=1}^np_i. $$
			-
			- ![ss_09052024_000496-converted.mp4](../assets/ss_09052024_000496-converted_1725569123952_0.mp4)
			-
			-
	- ## Disjunction (a.k.a. the OR operator)
		- When 2 propositions are connected using the disjunction operation, it becomes a new proposition whose truth value depends on the truth values of the operands.
		- #### Properties of Disjunction
			- Associaativity 
			  logseq.order-list-type:: number
			- Commutativity
			  logseq.order-list-type:: number
		- |$$p$$|$$q$$|$$p\lor q$$|
		  |--|--|--|
		  |1|1|1|
		  |1|0|1|
		  |0|1|1|
		  |0|0|0|
		- $$p\vee q=\max(p,q).$$
		- ### Exclusive Disjuction
			- |$$p$$|$$q$$|$$p\lor q$$| $$p\oplus q$$|
			  |--|--|--|--|
			  |T|T|T| F|
			  |T|F|T| T |
			  |F|T|T| T |
			  |F|F|F| F |
- ## Negative (a.k.a. the NOT operator)
	- #### Properties of negation
		- It is a **unary** operation.
		- ### Notation
			- Negation is denoted with the following symbols
				- logseq.order-list-type:: number
				  $$N\equiv\neg H.$$
				- logseq.order-list-type:: number
				  $$\neg p\equiv\overline{p}.$$
	- ### Note
		- **We must be cautious when interpreting the meaning of a statement with "not" in it.**
			- Example:
			  collapsed:: true
				- $\ h$ Harry is taller than Smith
				- Why statement is $\neg h$?
					- A. Harry is shorter than Smith
					- B. Smith is taller than Harry
					- C. Harry and Smith are the same height.
					- D. Harry is not taller than smith.
			- Answer
				- **D**
	-
		- ![ss_03202025_000833.png](../assets/ss_03202025_000833_1742494894201_0.png)
-
- # Precedence of Operations and Negating Compound Propositions
	-
	- | Operation | Symbol | Precedence (Order of application) |
	  | ---- | ---- | ---- |
	  | Parenthesis | ( )| 1 |
	  | Negation | ¬ | 2 |
	  | Conjunction | ∧ | 3 |
	  | Disjunction | ∨ | 4 |
	-
- tags:: #Educative
-