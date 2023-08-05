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
	- A Sum Types tags are known as its **variants**. #card
	- Sum types are often referred to as "tagged unions" or "disjoint unions" #card
- Key Properties:
	- each alternative value in a sum type is typically associated with a unique tag or discriminator.
	- Only a single variant can exist simultaneously.
- ## Morphism
- ### Catamorphism (unfold)
	- A function which deconstructs a structure into a single value. #card
	- Examples:
		- `reduceRight` is an example of a catamorphism for  the`Array` prototype.
- ```js
  // sum is a catamorphism from [Number] -> Number
  const sum = xs => xs.reduceRight((acc, x) => acc + x, 0)
  - sum([1, 2, 3, 4, 5]) // 15
  ```
- ## Fun Fact
  background-color:: red
- **cata** comes from the Ancient Greek word **κατά** meaning *"downwards"* #card
- ### Anamorphism (fold)
	- A function that builds up a structure by repeatedly applying a function to its argument. #card
	- `unfold` is an example which generates an array from a function and a seed value.
- **This is the opposite of a catamorphism.**
- ### Hylomorphism
	- A function that composes catamorphism and anamorphism.
	- ```js
	  const hylo = x => ana(cata(x)
	  ```
-
- ## Representables
- A functor is dubbed Representable if it isomorphic to Function.
- This tends to be the case when we have a fixed number of “slots” in our functor and we can represent it with some other type as an index. To do so, we just need an isomorphism and some other type with which to index.
- *Id* can be represented by *undefined* since it has 1 slot and *undefined* has 1 instance
  ```js
  
  const Id = x =>
  ({
   x,
   map: f => Id(f(x))
  })
  
  // to :: Id a -> (() -> a)
  const to = ({x}) => () => x
  
  // from :: (() -> a) -> Id a
  const from = f => Id(f())
  
  from(to(Id(‘hi’))) // Id(‘hi’)
  to(from(() => ‘hi’)) // () => ‘hi’
  ```
- https://medium.com/@drboolean/laziness-with-representable-functors-9bd506eae83f
- ## Posts
	- [[Contravariant Functors]]
	- [[Morphism in JS]]