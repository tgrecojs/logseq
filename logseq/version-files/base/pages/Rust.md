### Ownership Systems
	-
-
- ### Macros
	- Macros provide us with a way to "write code that writes other code". In other words, **macros unlock metaprogramming**
	- > *Metaprogramming is the process of writing code that writes code*
	- In rust, the term *macro*refers to a family of features in Rust: *declarative* macros with `macro_rules!` and three kinds of *procedural* macros:
	  id:: 67e46f11-e76e-42b8-bed5-69ec3f57d040
		- Custom `#[derive]` macros that specify code added with the `derive` attribute used on structs and enums
		- Attribute-like macros that define custom attributes usable on any item
		- Function-like macros that look like function calls but operate on the tokens specified as their argument