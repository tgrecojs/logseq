### Ownership Systems
	- > If you're not careful, you can easily introduce **memory leaks**, **buffer overflows**, and other memory-related bugs. Rust tries to solve this problem by providing a set of rules and restrictions that prevent you from making these mistakes. It's like having a safety net that catches you when you fall. This set of rules is called the **[ownership](https://www.rustfinity.com/learn/rust/ownership) system**.
	- The goal of ownership is to have the same memory safety and
- ### Macros
	- Macros provide us with a way to "write code that writes other code". In other words, **macros unlock metaprogramming**
	- > *Metaprogramming is the process of writing code that writes code*
	- In rust, the term *macro*refers to a family of features in Rust: *declarative* macros with `macro_rules!` and three kinds of *procedural* macros:
	  id:: 67e46f11-e76e-42b8-bed5-69ec3f57d040
		- Custom `#[derive]` macros that specify code added with the `derive` attribute used on structs and enums
		- Attribute-like macros that define custom attributes usable on any item
		- Function-like macros that look like function calls but operate on the tokens specified as their argument