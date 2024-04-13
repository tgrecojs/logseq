- Removed the following from dckc feedback
	- ### stateMachine
	  
	  the contract exists in a number of "states" which change throughout its lifecycle. 
	  the stateMachine object provides us with the following methods:
	- `transitionTo`
	- `canTransitionTo`
	- `getState`
	  
	  using the `makeStateMachine` funct states are represented as a `string`  and collectively, these string every possible value b track using a state machine object. thanks to the state machine  status  acts as an indicator and a tremendously useful one at that!  specific state  specific time-frame  lifecycle, it ca contract contains a `status` string  can be in any one of the predefined status  `  state.status` is tracked in the exo object's state object.
	-