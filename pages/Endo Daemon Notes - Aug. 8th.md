### `daemon-node-powers`
	- When storing a resource, the endo daemon processes its data in chunks using an iteraor, `Writer`
	- The result - **Each time we store a resource, we u
		- endo creates a hash for the file using `crypto.createHash(’sha512’`)
		-
		- `sha-512` object is created `makeSha512` wh
		-
	- ```Javascript
	  
	    /**
	     * @param {string} path
	     */
	    const makeFileReader = path => {
	      const nodeReadStream = fs.createReadStream(path);
	      return makeNodeReader(nodeReadStream);
	    };
	  
	    /**
	     * @param {string} path
	     */
	    const makeFileWriter = path => {
	      const nodeWriteStream = fs.createWriteStream(path);
	      return makeNodeWriter(nodeWriteStream);
	    };
	  
	  ```
-
- tags::  #@endo, #@endo/daemon, #@endo/CapTP