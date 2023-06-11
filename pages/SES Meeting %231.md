- scripts can only run on this exact domain via script-src ‘self
### ShadowRealms
- Webkit shipped shadowRealms and then dropped support due to lack of HTML integration. This work still needs to be done, so it is pressing.
- Mark’s counterpoint - We should be using knowledge of HTML standarrds Influencing APIs to make browser integration easy, but only to a point.
### Metamask Question
- Creating a compartment that by default does not have any evaluators.
## Agenda
- Immutable Array Buffers
- Superfreeze
- A non transative version of freeze
- Different properties
## Immutable Array Buffers
- Need to send copy bytes on the wire.
- When you receive the copy bytes you can transform it into an object and pass it around without it being mutated.
### Solutions
- Arraybuffers are not allowed to be frozen
- Transfer an Arraybuffer itto a frozen state.
- Give the slice of a current buffer
- WebBlob API is a version of bytes.
- Gives out immutable copies
- Arraybuffer doesn’t have copy on write optimization
- Doesn’t subsume the need for an immutable array buffer
- Blob only creates a fresh copy because of the absence of the possibility of a frozen buffer.
- Sharing the blob, not the buffer.
- Blob maintains the contents of the type.
- It also has webstreams
- A blobby-thing without the mimeType would not subsume the type
- We already have UInt8Array, DataView
### Peter Hoddie
- They have taken a very light approach
- What is the runtime behavior?
- How do you create them?
- An immutable array buffer acts exactly like an array buffer (until you try to write to it(
- Created by:
- Creating a regular Arraybuffer using lock down
- When the build hits the lockdown phase, Arraybuufers become immutable.
- petrify function
- a way to make any object deeply immutable (transative)
- if you had moved the entire object to ROM, (buffers, internal slots)
- For moddable, they are focused on solutions that do not act any different than the regular, common JavaScript APIs → a frozen ArrayBuffer is just an ArrayBuffer when not frozen.
- Objects that inherit from a frozen object without override mistakes.
- XS Limitations
- a closure that captures signable variables
- Markm: a protocol that is opt-in or opt-out, and you still petrify
- Puttin a symbol on it that once the symbol is invoked will freeze its own state
- Mike Fig: what if the data is put in a side-table
  
  If you petrify a derived class, the base case might have a constructor
  
  when the derived class seems to be adding fields using the override property, how do you account for
  
  a derived class with private fields can not be frozen