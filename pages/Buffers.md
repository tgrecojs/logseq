## TypingMind Convo
	- ## Part 1: Buffers from 0 to 100
	  
	  Buffers, as they are referred to in the Node.js, are designed to handle raw binary data. Each buffer corresponds to some raw memory allocated outside V8's heap. V8 is the JavaScript engine that powers Node.js and browsers. 
	  
	  1. Role of Buffers: Node.js is primarily used for I/O operations, and sometimes these include working with streams of data (files, network, console). The Buffer class in Node.js serves as an interface to interact with streams of binary data. They are part of Node.js's global scope and thus, you do not require to import it using any 'require' statement. Think of buffers as the temporarily stored 'chunks' of data that can be shipped around as packets.
	  
	  2. Buffer Utility: Buffers are crucial for working with binary data, which is central to tasks such as reading and writing to the filesystem, handling binary data in TCP streams, or even performing operations in cryptography where one needs to have access to every bit in a byte.
	  
	  3. JavaScript without Buffers: Without Buffers, in JavaScript, working with binary data efficiently would be nearly impossible, or at least, significantly more complex. JavaScript was created for the browser, not to handle binary streams of data, so Buffers help bring this ability to the language.
	  
	  4. Common Use Cases: Buffers are critical when you want to operate on raw data in a fast, controlled way. They are extensively used in Stream handling or reading/writing from any I/O device, TCP streams, server-side operations like reading file, data processing, and in various encoding related tasks in programmatic advertisements and marketing.
	  
	  ```javascript
	  // Creating a buffer
	  var buf = Buffer.from('Hello', 'utf8');
	  console.log(buf);  // <Buffer 48 65 6c 6c 6f>
	  console.log(buf.toString());  // 'Hello'
	  console.log(buf.toJSON());  // { type: 'Buffer', data: [ 72, 101, 108, 108, 111 ] }
	  
	  // Modifying a buffer
	  buf.write('wo');
	  console.log(buf.toString());  // 'wollo'
	  ```
	  
	  In contrast to typical arrays in JavaScript, Buffer instances are 'fixed-size' (modifiable on creation) and are capable of manipulating binary data directly, providing a closer interaction with OS I/O calls.
	- ## Part 2: Buffers continued, *WTH are Lone Surrogates*
	  
	  Lone surrogates are code units that are part of a surrogate pair but appear without their corresponding other half. Surrogate pairs are a mechanism by which Unicode allows for the encoding of characters outside the Basic Multilingual Plane (BMP) in fixed-width,16-bits encodings like UTF-16.
	  
	  1. Relevance of Lone Surrogates: Mozilla web docs include a section on lone surrogates because they represent an edge case that developers need to handle explicitly when dealing with Unicode data. Leaving them unchecked can lead to the creation of invalid Unicode strings, which can result in bugs in your JavaScript code related to text processing.
	  
	  2. When it Matters: Particularly, you'd need to consider lone surrogates when you need to deal with a wide range of Unicode symbols, e.g. emojis or some languages' symbols. If your JavaScript code is processing strings of text byte by byte, improperly handling a lone surrogate could lead to malformed text appearing in your applications.
	  
	  3. Lone surrogates and Coding: When a string is a lone surrogate, you should be careful while slicing, iterating, or manipulating the string as all these processes can produce an invalid Unicode which may cause subsequent operations to fail.
	  
	  ```javascript
	  // String can contain paired surrogates
	  const strA = '😂';
	  console.log(strA.length);  // 2
	  
	  // Slice can create strings with unpaired or lone surrogates
	  const strB = strA.slice(0, 1);
	  console.log(strB.length);  // 1
	  
	  // Buffer.from does not enforce proper Unicode encoding and can create buffers with unpaired surrogates
	  const bufA = Buffer.from(strA);
	  console.log(bufA.length);  // 4
	  
	  const bufB = Buffer.from(strB);
	  console.log(bufB.length);  // 3, this can cause unexpected behaviour
	  ```
	  
	  Being aware of how JavaScript deals with these encoding concerns can save your debugging hours and confusion, particularly if your system needs to handle multilingual input or binary data storage and transmission.
-
- tags::