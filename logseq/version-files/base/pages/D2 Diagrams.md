## Overview
	- Robust tool for creating code diagrams.
	- Docs: https://d2lang.com/
- ## Features
	- ### Positions
		- `top-left`, `top-center`, `top-right`,
		- `center-left`, `center-right`,
		- `bottom-left`, `bottom-center`, `bottom-right`
	- ### [Connections](https://d2lang.com/tour/connections)
		- Defined via
			- `—`
			- `->`
			- `<-`
			- `<->`
			- ### Connection Chaining #useful
				- For readability, it may look more natural to define multiple connection in a single line.
				- The following code:
					- ```
					  High Mem Instance -> EC2 <- High CPU Instance: Hosted By
					  ```
				- Produces the following output:
					- ![connection chaining](https://media.cleanshot.cloud/media/56139/5CsSeDwCVBsmsbFStz7PcgFG0XnC2Zcc8Un9A0wl.jpeg?Expires=1692611988&Signature=h1cWrEwZVOFE7TSJFfRgUg7PkGwuD2gFQYFE54kWtQLLwJVGaSv9~4Hqj4YvphtqyGGysFap6jTUjbeUYr3Vxx5JixumkCoX-0pfeB6JNkXNqcgieMIA0AoxkbGcXWjK8x76TNaHrbf21RTjCU7oCRR93t8yjpGDU~q-9oS5RvFugFBdK0J1tWTpgzFD4xkJpAGHKDiJAIzUVZL~S3N1LmaEsIf9RbsjQM5f7WHdYfnwxSvpuYzjzoaiwJ3W2kvmEGAd6TVZvWQZcD3s48ho8JouAfiHSYocGX3crzgtR~bw644eiCddJV~g-K5lHEhRWfHMuo~4fVsVqE~Iig~-~w__&Key-Pair-Id=K269JMAT9ZF4GZ)
	- ### [Containers](https://d2lang.com/tour/containers)
		- Easily create shapes nested within parents.
		- `server` - creates a container “server”
		- `server.process` - creates a container “process” inside of the container “server”.
		- Use underscore (`_`) to reference a parent container.
	- ### Compositon
		- Each d2 diagram exists within a **board**.
		- D2 provides a system for board composition.
			- **Layers** - Boards that do not inherit. They are a new base.
			- **Scenarios** - Boards which inherit from the base layer
			- **Steps** - Boards which inherit from the previous step.
			  tags:: #Charting
- ### Examples
	- [[D2 Examples]]
	-
-
- ### Useful References
	- ![📄 D2-CheatSheet (Merged).pdf](file:///Users/tgreco/Desktop/D2-CheatSheet (Merged).pdf)