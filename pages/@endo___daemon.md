## `servePort`
id:: 64fc1e40-f285-40df-8355-0c82a67ae8d5
collapsed:: true
	- investigate usage of websockets within endo
	- ### Action Item
	- ## Adding `rename` behaviors
	-
	- September 9th
-
-
- ### Aug. 1st - Endo Daemon Investigtion #@endo/daemon
  =======
- ### Aug. 1st - Endo Daemon Investigtion #@endo/daemon
  >>>>>>> 617d85e (fix: related to the mp4 files.)
  =======
  ### Aug. 1st - Endo Daemon Investigtion #@endo/daemon
  >>>>>>> 617d85e (fix: related to the mp4 files.)
	- #### `makeMessageCapTP`
	  id:: 6562d4c4-0953-49a3-9940-93c68411c197
	  collapsed:: true
		- creates a CapTP connection.
		- ```javascript
		  export const makeMessageCapTP = (
		    name,
		    writer,
		    reader,
		    cancelled,
		    bootstrap,
		  ) => {
		    /** @param {any} message */
		    const send = message => {
		      return writer.next(message);
		    };
		  
		    const { dispatch, getBootstrap, abort } = makeCapTP(name, send, bootstrap);
		  
		    const drained = (async () => {
		      for await (const message of reader) {
		        console.log('inside drained:: for await(const message of reader)', {
		          message,
		        });
		        dispatch(message);
		        console.log('inside drained:: message dispatched');
		      }
		    })();
		  
		    const closed = cancelled.catch(async () => {
		      abort();
		      await Promise.all([writer.return(undefined), drained]);
		    });
		  
		    return {
		      getBootstrap,
		      closed,
		    };
		  };
		  ```
	- ## CapTP Messages
		- #### Message Types
			- `CTP_BOOTSTRAP`
			  logseq.order-list-type:: number
			- `CTP_RETURN`
			  logseq.order-list-type:: number
			- `CTP_CALL`
			  logseq.order-list-type:: number
			- `CTP_DISCONNECT`
			  logseq.order-list-type:: number
		- Example: `endo list`
			- `send` function
				- `{ type: 'CTP_BOOTSTRAP', epoch: 0, questionID: 'q-1' }`
			- what is this private facet we receive back immediately?
				- ```js
				  /**
				      {
				      type: 'CTP_RETURN', epoch: 0, answerID: 'q-1',
				      result: {
				        body: '{"@qclass":"slot","iface":"Alleged: Endo private facet","index":0}',
				        slots: [Array]
				      }
				    }
				  */
				  ```
- ## Looming Questions
  id:: 64c8aef0-0261-43b0-a139-ad8c5b3ce0c1
	- `SIGINT` and `process.exitCode`
	  collapsed:: true
		- `process.once('SIGINT', () => cancel(new Error('SIGINT')));`
			- Researching looks like this will fire
			- https://nodejs.org/api/process.html#signal-events
				- > Sending `SIGINT`, `SIGTERM`, and `SIGKILL` will cause the unconditional termination of the target process, and afterwards, subprocess will report that the process was terminated by signal.
				- > Sending signal `0` can be used as a platform independent way to test for the existence of a process.
				- > Signals are not available in `Worker` threads.
		- used when calling `main` in `daemon-node.js` & `worker-node.js`.
		- success
			- process.exitCode = 0;
		- failure
			- process.exitCode = 1;
	- what does harbinger mean?
	  collapsed:: true
		- is this our safety net?
		  <<<<<<< HEAD
		  <<<<<<< HEAD
		  =======
- >>>>>>> 617d85e (fix: related to the mp4 files.)
  =======
- >>>>>>> 617d85e (fix: related to the mp4 files.)
-
-