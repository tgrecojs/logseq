- the `main` function seems is where “all of it” starts.
	- Endo implements the powerbox pattern to enforce POLA.
	- In the powerbox pattern... the “main” module of the application (where execution begins) hands out to each module just enough authority to do the job needed by that particular module. - [[How Emily Tamed the Caml]]
	- There are 2 `main` modules in Endo:
		- `src/daemon.js`
			- the top-level `main` module.
			- it's through this module that `worker` objects that hold power
		- `worker`
			- the `main` module for
	- Endo
- where does `main0` come from?
- Regarding the locator...
- what is the difference between:
- statePath
	- ephemeralStatePath
	- sockPath
	- cachePath
- after coming across the term "noncelocator" a number of times while reading erights.org, I'm wondering what is the locator within the context of SwingSet?
- What is the significance of `child.on` within the endo daemon's `start` function? What exactly occurs when a “message” event occurs?
- **What is the significance of `child.on` within the endo daemon's `start` function? What exactly occurs when a “message” event occurs?**
	- This is one of the wonders of daemonization. Daemonization is the process of spawning a child process, removing it from the parent process’s session, and handing it over to process 0 (init) to manage until it dies. On UNIX, by default, a child process remains in the parent process’s session, sees all of the same signals, and exits when the parent exits. Daemons are longer-lived.
	- When a client like the CLI attempts to open a connection to the Endo daemon, there’s a possibility that the daemon is not running. So, the client will spawn and daemonize the daemon. But, owing to the vagaries of timing, the daemon will not be ready to receive connections from the client until it has run a while and started listening.
	- This is a problem for the client which wants to connect to the daemon the moment it becomes available. We’re using the Node.js IPC port to send a message from the child process (daemon) to the parent process (cli) that it can connect.
	- The alternative is a retry loop, which will always be slower than this approach.
- What facets does `@endo/daemon` make accessible upon calling the `start` function?
- Confusion around the `bootstrap` object.
- `makeMessagesCapTP`
- This function takes a `bootstrap` object as an argument, which is then passed into `makeCapTP` in exchange for a few values, one of which being `getBootstrap`.
	- I'm trying to figure out how imports/exports of bootstrap objects work.
-