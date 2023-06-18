- ## TL;DR
- Kleisi Composition refers to the composition of Monads.
-
- ### History
- Kleisli composition is named after the mathematician and computer scientist, Heinrich Kleisli. In the 1960s, Kleisli developed a theory of categories that included a generalization of the notion of a function to include functions that return values in a monadic context.
- In Kleisli's theory, a Kleisli arrow is a function that takes a value and returns a monadic value. Kleisli arrows can be composed using a special composition operator that takes two Kleisli arrows and returns a new Kleisli arrow. This composition operator is known as Kleisli composition.
- Kleisli arrows and Kleisli composition are important concepts in functional programming, particularly in the context of monads. Monads provide a way to sequence computations that involve side effects or non-determinism, and Kleisli composition provides a way to compose functions that return monadic values.
  In the context of programming languages, Kleisli composition is often used to compose functions that return values in a monadic context, such as promises in JavaScript or futures in Scala.
- ###
-