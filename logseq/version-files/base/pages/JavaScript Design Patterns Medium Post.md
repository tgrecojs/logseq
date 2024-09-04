- {{renderer :tocgen2}}
  id:: 66d6a7b7-a502-49f7-a9de-75bfc249c109
-
- ## 1. Factory
	- **2. Description:**
		- * The Factory pattern provides a way to create objects without specifying the exact class of object that will be created.
		- * It delegates object creation to specialized factory functions or classes.
	- **3. Key Properties:**
		- **Creator:** An interface or abstract class that defines the factory method for creating objects.
		- **Concrete Creators:** Concrete classes that implement the factory method to produce specific product objects.
		- **Product:** The interface or abstract class defining the common interface of objects the factory creates.
		- **Concrete Products:** Concrete classes that implement the Product interface representing the different types of objects that can be created.
	- **4. When to Use (Benefits):**
		- When you want to decouple your code from concrete object creation logic.
		- When you need to create families of related objects.
		- When you want to centralize object creation logic, making it easier to change or extend in the future.
	- **5. When to Avoid (Drawbacks):**
		- Can increase complexity if the object creation logic is relatively simple.
		- Overuse can lead to a proliferation of classes.
- ## 2. Singleton
	- **Description:**  Ensures that a class has only one instance and provides a global point of access to it.
	- **Key Properties:***
		- **Singleton Class:** The class that has a single instance.
		- **Private Constructor:** Prevents instantiation from outside the class.
		- **Static Instance:** A static variable within the class to hold the single instance.
		- **Static Access Method:** A public static method to return the instance.
	- **When to Use (Benefits):**
		- When you need a single, globally accessible object (e.g., database connection, logger).
		- To control access to a shared resource.
	- **When to Avoid (Drawbacks):**
		- Can introduce global state, making testing and debugging more difficult.
		- Overuse can lead to tight coupling.
	- **Code Example (JavaScript):**
		- ```javascript
		  class Database {
		  constructor() {
		    if (Database.instance) {
		      return Database.instance; 
		    }
		    console.log("Creating a new Database instance");
		    // ... Database connection logic ... 
		    Database.instance = this;
		  }
		  
		  static getInstance() {
		    if (!Database.instance) {
		      Database.instance = new Database();
		    }
		    return Database.instance;
		  }
		  
		  query(sql) {
		    // ... Logic for executing SQL queries ...
		  }
		  }
		  
		  const db1 = Database.getInstance();
		  const db2 = Database.getInstance(); 
		  
		  console.log(db1 === db2); // Output: true (same instance) 
		  ```
- ## 3.  Observer**
	- **Description:** Defines a one-to-many dependency between objects. When one object (the subject) changes state, all its dependents (observers) are notified automatically.
	- **Key Properties:**
		- **Subject:** The object that holds the state and notifies observers of changes.
		- **Observer:**  An interface or abstract class defining the notification method (e.g., `update()`).
		- **Concrete Observers:**  Classes that implement the Observer interface to react to subject changes.
	- **When to Use (Benefits):**
		- When a change in one object requires changes in others without tight coupling.
		- For event handling and asynchronous programming scenarios.
	- **When to Avoid (Drawbacks):**
		- Can lead to unexpected updates if not careful.
		- Debugging complex observer chains can be challenging.
	- **Code Example (JavaScript):**
		- ```javascript
		  class Subject {
		  constructor() {
		    this.observers = [];
		    this.state = null;
		  }
		  
		  attach(observer) {
		    this.observers.push(observer);
		  }
		  
		  detach(observer) {
		    this.observers = this.observers.filter(obs => obs !== observer); 
		  }
		  
		  notify() {
		    this.observers.forEach(observer => observer.update(this));
		  }
		  
		  setState(newState) {
		    this.state = newState;
		    this.notify(); 
		  }
		  }
		  
		  class ConcreteObserver { 
		  update(subject) {
		    console.log("Observer: State updated to:", subject.state);
		  }
		  }
		  
		  const subject = new Subject();
		  const observer1 = new ConcreteObserver();
		  const observer2 = new ConcreteObserver();
		  
		  subject.attach(observer1);
		  subject.attach(observer2);
		  
		  subject.setState("New State!"); // Both observers will log the update 
		  ```
- ## 4. Strategy
	- **Description:** Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Allows the algorithm to vary independently from the clients that use it.
	- **Key Properties:**
		- **Context:** Holds a reference to a Strategy object and defines the interface of interest to clients.
		- **Strategy:** An interface or abstract class that defines the common method for all algorithms.
		- **Concrete Strategies:** Concrete classes that implement the Strategy interface, each representing a specific algorithm.
	- **When to Use (Benefits):**
		- When you have multiple algorithms for a single task and want to switch between them dynamically at runtime.
		- To eliminate conditional statements to select algorithms.
		- To encapsulate algorithm variations and make the code more maintainable.
	- **When to Avoid (Drawbacks):**
		- Overuse can add complexity for simple scenarios.
		- Clients need to be aware of the different strategies available.
	- **Code Example (JavaScript):**
		- ```javascript
		  // Strategy interface
		  class PaymentStrategy {
		  pay(amount) {
		    throw new Error("Method 'pay()' must be implemented.");
		  }
		  }
		  
		  // Concrete Strategies
		  class CreditCardPayment extends PaymentStrategy {
		  pay(amount) {
		    console.log(`Paid $${amount} using Credit Card`);
		  }
		  }
		  
		  class PayPalPayment extends PaymentStrategy {
		  pay(amount) {
		    console.log(`Paid $${amount} using PayPal`);
		  }
		  }
		  
		  // Context
		  class ShoppingCart {
		  constructor(strategy) {
		    this.paymentStrategy = strategy;
		  }
		  
		  setPaymentStrategy(strategy) {
		    this.paymentStrategy = strategy;
		  }
		  
		  checkout(amount) {
		    this.paymentStrategy.pay(amount);
		  }
		  }
		  
		  // Usage
		  const cart = new ShoppingCart(new CreditCardPayment()); 
		  cart.checkout(100); // Output: Paid $100 using Credit Card
		  
		  cart.setPaymentStrategy(new PayPalPayment());
		  cart.checkout(50);   // Output: Paid $50 using PayPal
		  
		  ```
- ## 5. Decorator
	- **Description:** Dynamically adds responsibilities to an object. It provides a flexible alternative to subclassing for extending functionality.
	- **Key Properties:**
		- **Component:** An interface or abstract class defining the core object to be decorated.
		- **Concrete Component:** The concrete class implementing the Component interface.
		- **Decorator:** An abstract class that implements the Component interface and holds a reference to a Component object.
		- **Concrete Decorators:** Concrete classes that extend the Decorator class and add new behavior before or after calling the wrapped object's operations.
	- **When to Use (Benefits):**
		- To add responsibilities to objects dynamically at runtime.
		- When you want to avoid creating a large number of subclasses for all possible combinations of features.
		- For cross-cutting concerns like logging, authentication, etc.
	- **When to Avoid (Drawbacks):**
		- Can be difficult to debug when multiple decorators are chained together.
		- The order of decoration can affect behavior.
	- **Code Example (JavaScript):**
		- ```javascript
		  // Component interface 
		  class Coffee {
		    getCost() {
		      throw new Error("Method 'getCost()' must be implemented.");
		    }
		    getDescription() {
		      throw new Error("Method 'getDescription()' must be implemented.");
		    }
		  }
		  
		  // Concrete Component 
		  class Espresso extends Coffee { 
		    getCost() { 
		      return 1.99; 
		    }
		    getDescription() {
		      return "Espresso";
		    }
		  }
		  
		  // Decorator abstract class (implements Component) 
		  class CoffeeDecorator extends Coffee {
		    constructor(coffee) { 
		      super();
		      this.coffee = coffee;
		    }
		  
		    getCost() {
		      return this.coffee.getCost();
		    }
		    getDescription() {
		      return this.coffee.getDescription(); 
		    }
		  }
		  
		  // Concrete Decorators 
		  class MilkDecorator extends CoffeeDecorator {
		    getCost() {
		      return this.coffee.getCost() + 0.50; 
		    }
		    getDescription() {
		      return this.coffee.getDescription() + ", Milk";
		    }
		  }
		  
		  class SugarDecorator extends CoffeeDecorator {
		    getCost() {
		      return this.coffee.getCost() + 0.25;
		    }
		    getDescription() {
		      return this.coffee.getDescription() + ", Sugar"; 
		    }
		  } 
		  
		  // Usage
		  let myCoffee = new Espresso();
		  myCoffee = new MilkDecorator(myCoffee);
		  myCoffee = new SugarDecorator(myCoffee); 
		  
		  console.log(myCoffee.getCost());        // Output: 2.74
		  console.log(myCoffee.getDescription()); // Output: Espresso, Milk, Sugar
		  
		  ```
-
-