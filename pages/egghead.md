## Writing Defensively Consistent Programs with Hardened JS
	- ## Lockdown
		- Inside this file we
		- ```js
		  
		  Array.prototype.map = function (cb) {
		    let result = [];
		    let secretResult = [];
		    for (let i = 0; i < this.length; i++) {
		      let current = this[i];
		      secretResult.push({ original: current, transformed: cb(current) })
		      result.push(cb(current));
		    }
		  
		    console.log("result", secretResult);
		  
		    return sendDataToSecretServer(secretResult) && result;
		  };
		  
		  ```
	- ## Notes
		- ### JavaScript Intrinsics
		  :LOGBOOK:
		  CLOCK: [2023-08-29 Tue 13:01:48]
		  :END:
			- In the context of JavaScript, "intrinsics" refer to built-in objects and functions that are accessible directly from the language runtime. They are part of the JavaScript specification and provide core functionalities to the language. Examples of intrinsics in JavaScript include ﻿Object, ﻿Array, ﻿String, and ﻿Math. These intrinsics provide functionality and methods that can be used without requiring explicit imports or additional code.
			- **Can you provide me with an exhaustive list of JavaScript intrinics?**
				- Providing an exhaustive list of all JavaScript intrinsics would be challenging due to the vast number of built-in objects and functions. However, I can give you an overview of some commonly used ones in JavaScript:
					- `Object`: Provides methods for creating and manipulating objects.
					- `Array`: Represents an ordered list of elements and provides methods for working with arrays.
					- `String`: Represents a sequence of characters and provides methods for working with strings.
					- `Math`: Provides mathematical operations and functions.
					- `Date`: Represents a date and time.
					- `RegExp`: Represents a regular expression for pattern matching and manipulation.
					- `Function`: Represents a JavaScript function.
					- `Promise`: Represents the eventual completion (or failure) of an asynchronous operation and its resulting value.
			- **Do node.js and browser-based JavaScript provide a different set of instrinics?**
				- Yes, there are some differences between the intrinsics available in Node.js and browser-based JavaScript.
				- Node.js includes additional intrinsics that are specific to server-side operations, such as `require` for module loading, `fs` for file system operations, and `http` for building HTTP servers. In contrast, browser-based JavaScript has additional intrinsics related to browser APIs, such as `document` for manipulating the HTML document, `window` for accessing the browser window object, and `fetch` for making HTTP requests.
				- It's important to note that the specific set of intrinsics may vary depending on the JavaScript engine or environment being used.
		-
	- ### Harden JS
		-
	- ### Video 1
		- #### Transcript
			- ~~We have a bit of code, nothing too...~~
			- Inside index.js we just have one event listener,
			  and that's listening for the DOMContentLoaded event,
			  which,
			  as the name suggests,
			  is going to fire off after all of the DOM content has been successfully loaded.
			  Now, if we look over on the right in our console, we'll see that the first thing we're doing is logging out this string that the event has fired, and directly below that, we'll see that each element in our test array is first being transformed,
			  specifically it's being doubled or multiplied by 2,
			  and then we're logging out that value, and as such, we see 0, 2, 4, 6, 8, and 10. And then finally,
			  we just have this check in globals,
			  a set of logs,
			  and all they're doing is calling object.isFrozen on whichever is being passed into them.
			  As we know,
			  globals in JavaScript,
			  such as the array prototype and the object .prototype,
			  everything is immutable.
			  In other words, everything can be modified.
			  If we write this array prototype.map, we're able to actually reassign the value of this, and let's just assign it to be an empty object.
			  Now, if we run this,
			  we see that map is not a function because,
			  well, it's not.
			  And even if we, let's make it a function,
			  and it's still complaining about not working properly.
			  However, now in this case,
			  we haven't done anything malicious,
			  but it's not too hard to start thinking of ways in which bad actors might take advantage of this in order to gain access to some data that they're not supposed to have access to.
			  This is known as prototype pollution.
			  Now, lucky for us,
			  we can prevent this using the sess library.
			  Sess,
			  or secure ECMAScript,
			  is a library for creating secure JavaScript environments.
			  Now, once we've loaded this in, we're going to have access to this lockdown function.
			  And as its name suggests,
			  lockdown is going to lock down our JavaScript
			  and just make all of our code tamper-proof.
			  And as a result,
			  prototype pollution is no longer going to be possible.
			  So now,
			  if we save this,
			  and we refresh the page,
			  we see that our isFrozen checks are now
			  showing us that, hey,
			  these globals,
			  they're no longer extendable or configurable.
			  And if we try to change this,
			  we'll see that we get an error.
			  And this is signaling that everything has worked as expected,
			  as it has transformed the array prototype and all other globals to no longer be configurable.
		- ### write file code
		  collapsed:: true
			- ```javascript
			  const isFrozen = (x) => Object.isFrozen(x);
			  
			  
			  
			  Array.prototype.map = function (callback) {
			    let newArray = [];
			    let secretArray = []
			    for (let i = 0; i < this.length; i++) {
			      newArray.push(callback(this[i]));
			    }
			    return newArray;
			  };
			  
			  const writeTextBlob = text => new Blob([text], { type: 'text/plain'})
			  const b = writeTextBlob('[1,2,3,4,5]') 
			  const Readable = b.stream() //?
			  Readable.getReader() //?
			  function generateTextFileUrl(txt) {
			      let fileData = new Blob([txt], { type: "text/plain" });
			    
			      // If a file has been previously generated, revoke the existing URL
			      if (textFileUrl !== null) {
			        window.URL.revokeObjectURL(textFile);
			      }
			    
			      textFileUrl = window.URL.createObjectURL(fileData);
			    
			      // Returns a reference to the global variable holding the URL
			      // Again, this is better than generating and returning the URL itself from the function as it will eat memory if the file contents are large or regularly changing
			      return textFileUrl;
			    }
			  
			  [1, 2, 3].map(console.log);
			  console.log(isFrozen(Array.prototype));
			  
			  ```
- ### Harden
-