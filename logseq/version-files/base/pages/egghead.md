### Secure ECMAScript
	- ### Video 1
		- Here we have an index.html file which loads in this index.js file using the script element.
		- Let's make use of this isFrozen function things to investigate things that we know to be true. Specifically, let's check out our program's Array prototype.... if we reload the page, we'll see that this executes to be false. Although this is the expected behavior, the ability to mutate, or change the behavior of globals such as the Array prototype can lead to
		- Now this behavior, although
		- While this shouldn't come as a surprise, this behavior, although expected, can  this is the expected behavior.
		-
		-
		- we can investigate things that we know to be
		-
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
-