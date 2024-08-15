### What is a "composite type"? #card
	- In the context of software development, a "composite type" refers to a data type that is composed of multiple other types. These can include primitive types (like numbers, strings, etc.) and other composite types. Composite types can be used to create more complex data structures. #card
- ### Examples of Composite Types
	- **Arrays:** Collections of values, which can be of the same or different types.
	- **Objects:** Collections of key-value pairs, where the keys are strings (or symbols) and the values can be of any type.
	- **Classes:** Templates for creating objects, encapsulating data and behavior.
- ### Example in JavaScript
	- #### Array (Composite Type Example)
	  An array in JavaScript is a composite type because it can hold multiple values of different types.
		- ```javascript
		  // Array containing different types of values
		  let compositeArray = [42, "Hello", true, { name: "Alice" }, [1, 2, 3]];
		  
		  console.log(compositeArray);
		  // Output: [42, "Hello", true, { name: "Alice" }, [1, 2, 3]]
		  ```
- #### Object (Composite Type Example)
	- An object in JavaScript is another example of a composite type, as it can contain multiple key-value pairs, where the values can be of any type, including other objects or arrays.
		- ```javascript
		  // Object containing various types of values
		  let compositeObject = {
		    id: 1,
		    name: "Composite Object",
		    isActive: true,
		    details: {
		        description: "This is a nested object",
		        tags: ["example", "composite", "type"]
		    },
		    items: [10, 20, 30]
		  };
		  
		  console.log(compositeObject);
		  /* Output:
		  {
		    id: 1,
		    name: "Composite Object",
		    isActive: true,
		    details: {
		        description: "This is a nested object",
		        tags: ["example", "composite", "type"]
		    },
		    items: [10, 20, 30]
		  }
		  */
		  ```
- #### Class (Composite Type Example)
	- A class in JavaScript can be used to create objects with predefined properties and methods, making it a composite type.
		- ```javascript
		  // Class definition
		  class Person {
		    constructor(name, age, hobbies) {
		        this.name = name;
		        this.age = age;
		        this.hobbies = hobbies; // Array of hobbies
		    }
		  
		    greet() {
		        console.log(`Hello, my name is ${this.name} and I am ${this.age} years old.`);
		    }
		  }
		  
		  // Creating an instance of the Person class
		  let person = new Person("John", 30, ["reading", "swimming"]);
		  
		  console.log(person);
		  /* Output:
		  Person {
		    name: "John",
		    age: 30,
		    hobbies: ["reading", "swimming"]
		  }
		  */
		  
		  person.greet();
		  // Output: Hello, my name is John and I am 30 years old.
		  ```
- ### Summary
	- In summary, composite types in software development are data structures that combine multiple values, which can be of different types. In JavaScript, arrays, objects, and classes are common examples of composite types, allowing developers to create complex and structured data representations.
-