## Ownership System
	- > If you're not careful, you can easily introduce **memory leaks**, **buffer overflows**, and other memory-related bugs. Rust tries to solve this problem by providing a set of rules and restrictions that prevent you from making these mistakes. It's like having a safety net that catches you when you fall. This set of rules is called the **[ownership](https://www.rustfinity.com/learn/rust/ownership) system**.
	- The goal of ownership is to have the same memory safety and management as garbage collector without the overhead.
	- **The rules of ownership**
		- Each value in Rust has an owner.
		- There can only be one owner at a time.
		- When the owner goes out of scope, the value will be dropped.
	- in Rust and once the scope ends, the value is dropped and they no longer take up memory.
		- ```rust
		  fn main() {
		      let x = 5; // x is valid from this point
		      {
		          let y = 10; // y is valid from this point
		          println!("The value of y is: {}", y);
		      } // y goes out of scope here
		      println!("The value of x is: {}", x);
		  }
		  ```
	- ### Transferring Ownership
		- When you assign a value to another variable, you are moving the value to the variable.
		- If you re-assign the variable to another one, the value is moved in such a way that results the initial variable becomes invalid.
- ## Variables
	-
- ## Macros
	- Macros provide us with a way to "write code that writes other code". In other words, **macros unlock metaprogramming**
	- > *Metaprogramming is the process of writing code that writes code*
	- In rust, the term *macro*refers to a family of features in Rust: *declarative* macros with `macro_rules!` and three kinds of *procedural* macros:
	  id:: 67e46f11-e76e-42b8-bed5-69ec3f57d040
		- Custom `#[derive]` macros that specify code added with the `derive` attribute used on structs and enums
		- Attribute-like macros that define custom attributes usable on any item
		- Function-like macros that look like function calls but operate on the tokens specified as their argument