- ## The Powers of the Prototype
  JavaScript takes a lot of flack on social media sites. The idea of JavaScript being "inferior" to other languages is nothing new. While I'm certainly baised, I sincerely do not understand why this is the case. I'm sure it's related to it being the most popular language. Anyways, in this article I'm going to explore object composition in JavaScript in an attempt to demonstrate why it is so powerful.
-
- ```js
  const withConstructor = (constructor) => (o) => ({
    // create the delegate [[Prototype]]
    __proto__: {
      // add the constructor prop to the new [[Prototype]]
      constructor,
    },
    // mix all o's props into the new object
    ...o,
  });
  ```
-