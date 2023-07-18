- ![](https://static.observableusercontent.com/files/4fc86c4132208e83a3bd8cfc789a16e3fa9962112a8327916234927ae48062ee38608f0df3fce0ed04389be3c42bc1e6e9c2d89814fa321b24aa29b044f9a70c)
- # Exploring Fantasy Land
- Fantasy Land is a specification for creating values that adhere to  
  algebraic  This series is meant to take readers on a journey through the
  Fantasy Land Specification by diving
- ## Lesson #1:  `Setoid`  and  `Ord`
- If you refer to the image above you will see all of the different 
  interfaces that exist within the Fantasy Land Spec. These interfaces are
  referred to as "Algebras".
- To quote the spec's documentation:
- >
- An algebra is a set of values, a set of operators that it is closed under and some laws it must obey.
- Put differently, an algebra is an object encapsulated with methods 
  that adhere to specific laws. Let's see this in action and begin 
  implementing a `Setoid`.
- ## `Setoid`
- ## Equivalence
- ### Setoid
- `a['fantasy-land/equals'](a) === true` (reflexivity)
- `a['fantasy-land/equals'](b) === b['fantasy-land/equals'](a)` (symmetry)
- If `a['fantasy-land/equals'](b)` and `b['fantasy-land/equals'](c)`, then `a['fantasy-land/equals'](c)` (transitivity)
- #### `fantasy-land/equals`  method
- ```
  fantasy-land/equals :: Setoid a => a ~> a -> Boolean
  ```
- A value which has a Setoid must provide a `fantasy-land/equals` method. The
  `fantasy-land/equals` method takes one argument:
- ```
  a['fantasy-land/equals'](b)
  ```
- -
- `b` must be a value of the same Setoid
- If `b` is not the same Setoid, behaviour of `fantasy-land/equals` is
  unspecified (returning `false` is recommended).
- -
- `fantasy-land/equals` must return a boolean (`true` or `false`).
- ### Ord
- A value that implements the Ord specification must also implement
  the [Setoid](https://observablehq.com/d/03d2e486126755df#setoid) specification.
- `a['fantasy-land/lte'](b)` or `b['fantasy-land/lte'](a)` (totality)
- If `a['fantasy-land/lte'](b)` and `b['fantasy-land/lte'](a)`, then `a['fantasy-land/equals'](b)` (antisymmetry)
- If `a['fantasy-land/lte'](b)` and `b['fantasy-land/lte'](c)`, then `a['fantasy-land/lte'](c)` (transitivity)
- #### `fantasy-land/lte`  method
- ```
  fantasy-land/lte :: Ord a => a ~> a -> Boolean
  ```
- A value which has an Ord must provide a `fantasy-land/lte` method. The
  `fantasy-land/lte` method takes one argument:
- ```
  a['fantasy-land/lte'](b)
  ```
- -
- `b` must be a value of the same Ord
- If `b` is not the same Ord, behaviour of `fantasy-land/lte` is
  unspecified (returning `false` is recommended).
- -
- `fantasy-land/lte` must return a boolean (`true` or `false`).