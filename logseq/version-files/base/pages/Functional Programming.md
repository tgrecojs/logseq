- [[FP Terminology]]
- #[[Category Theory]]
- ## Concepts
- ### Polymorphic
- A polymorphic function is a function that can operate on different types.
- ```js
  // polymorphic concat
  const concat = x => y => x.concat(y);
  
  concat([1],[2,3])
  concat("hello ", "there) 
  ```
- ### Sum types
- A type that can have **multiple alternative values.**
- Each value is distinguished by a "tag" or a "discriminant" which indicate what the current value is.
- A Sum Types tags are known as its **variants**.
- Sum types are often referred to as "tagged unions" or "disjoint unions"
- Key Properties:
	- each alternative value in a sum type is typically associated with a unique tag or discriminator.
	- Only a single variant can exist simultaneously.
- Variants are tagged in order to distinguish them from one another at runtime. Scriptum supplies the union auxiliary function to facilitate the construction of tagged unions in Javascript:
-