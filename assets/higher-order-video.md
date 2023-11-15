I'm Mark Miller,
and I'll be speaking today about higher-order smart contracts across chains.
The world economy,
the phenomenon of markets that has evolved in human society for thousands of years,
is a very rich,
complex phenomena that has given rise to all sorts of ways that humans cooperate with each other in large networks.
And the current way that we're building smart contracts is not able to reproduce that richness.
And in particular, I want to focus on one particular source of that richness that we can understand and try to do a better job to reproduce,
which is the abstractness of property rights.
Property rights evolved from a very simple concept, that there's physical goods and there's territory,
which became tradable land,
and you traded this thing for that thing. And it probably started without even a general concept that all these different things were under a common concept of property. But as people traded one thing for another,
there came to arise this general understanding that, oh, these are tradable things, there's this concept of property.
And then as people built institutions,
as institutions grew, institutional arrangements, ways of transacting with each other grew around the concept of property, that created this richness, this vocabulary of ways of cooperating with each other that were parameterized by whatever would feed into those arrangements under the label property.
So once we had that,
we had the ability to generalize the label, to label new things as property that had not previously been labeled property. And as we labeled new things property,
the existing vocabulary of arrangements of cooperation,
we were able to leverage them, to reuse them, to apply to any new thing that we came to call property.
So in computer science,
this process of building up a variety of abstractions that are generically parameterizable over some category of input is known as parametric polymorphism.
And the ability to take new concepts and fit it into an old category so that parametric things, things that were generally parameterizable could apply to them,
is known as reification.
So Hernando de Soto,
in the book The Mystery of Capital,
examines why it is that the informal economy, the economy that exists outside of formal legal systems,
he examines the nature of that economy. First of all, he explains that it's a very rich economy, it's a genuine economy.
A tremendous fraction of the world's overall economic activity occurs outside of any legal system, occurs in the informal economy.
There's certainly much creation of wealth, much cooperation there. But they're not able to
engage in the rapid growth,
the rapid exponential growth of wealth creation that is familiar in our world. And Hernando de Soto asks why not.
And he focuses on what he calls turning a property into capital.
That it's a failure to turn property into capital. So in our world, we're familiar with complex networks of abstractions of property, arranging composition of cooperative arrangement.
So we have the concept of, let's say, owning a house, but then we have the concept of mortgaging a house. Well, mortgaging a house creates this new valuable arrangement,
but then once you've mortgaged the house, you don't own the house. How is it that you can sell the house if you don't own the house? Well, what you do is you sell your position in the mortgage contract,
that the position in the mortgage contract itself becomes a new abstract form of property. You can think of it as a derived right where the house itself is the base right.
And the example that Hernando de Soto gives in Mystery of Capital is the inability to turn ownership of a house, what is genuinely,
from an economics perspective,
ownership of the house, but ownership that is not legally recognized by the external world legal systems. It's recognized by the systems of norms in the local community that a particular person is recognized as owning the house. Within the local community, they can sell the house and the new owner is recognized as owning the house, but they can't take that internal recognition.
It doesn't fit the vocabulary of rights that these other external built-up arrangements of cooperation understand.
So there's just this inability to reify. You can think of this as a reification failure. And the reification failure means that this whole built-up richness of institutions that we have in the rest of the world cannot be parameterized
by the property rights created in the informal economy.
So I
want to recast de Soto's insight very much in computer science terms so that we can use both the economic perspective and the computer science perspective on this one phenomenon
to build smart contracting systems that can enable a greater richness and complexity and composability of cooperative arrangements.
So in computer science, we have this notion of higher order composition.
And for each form of higher order composition,
it depends on some ability to reify.
And each time you have an ability to reify,
there's some case at which it runs out of steam and you no longer have an ability to reify within that system.
And now that failure to reify should provoke you to build a new higher order system that has new rules of reification so that you can now bring this higher order composition into that new realm. So this is all very, very abstract.
I'm going to start off a little bit less abstract.
So in the world of functional programming languages, where you have pure functions,
the really cool thing about functional programming languages is functions operate on values of some sort and will compute a new value given input values.
But a function itself is the kind of value that you can apply functions to.
And you can write many functions like a function composing function, which are generic, that will take any function as input. So the kind of thing they are as a means to manipulate values,
the kind of thing that they manipulate, which is the values that you can feed into the function, is also the kind of thing that they are.
So therefore they are the kind of thing that they manipulate.
And therefore you can compose them into these great networks of functions operating on each other and eventually on base values and create the richness that we associate with functional programming.
However,
functional programming is specifically mathematical -like functions with no side effects.
The world has stateful concepts like a bank balance.
And you can't take a mutable bank balance and reify it as an object, as a thing, that is the kind of thing that plays within the world of higher order composition of the functions.
You can write patterns of functions.
You can engage in these other patterns with monads and state passing and all that,
that end up creating a new level of abstraction.
But you can't simply apply a function to a stateful object and have it be the same stateful object after it changes state.
So instead we have object-oriented programming, which has this higher order nature with regard to stateful things.
That you can take a mutable object, store it in a table.
While it's in the table it might change state and then later you can pull it out of the table and it's the same object in the new state. That's an example of a kind of composition, a kind of higher order composition to create cooperative arrangements of computation
that you simply can't do in a direct manner in functional programming.
And when you build up patterns that do that you're creating,
for example,
a new object layer of abstraction that's really distinct from the functions.
However,
what is directly relevant to our mission of trying to bring the world economy online
through digital media, through blockchains, through smart contracts,
is that objects themselves
cannot directly represent exclusive rights.
You can create patterns of objects, as you will see,
by which you indirectly create the concept of exclusive rights. Indirectly manipulate the exclusive rights. But the exclusive right itself is not an object, is not reified as an object.
So in the same way that you can create patterns of functions that give rise to this new level of abstraction of objects,
we can create patterns of objects that give rise to a new level of abstraction of electronic rights. And that's what we're doing.
But we're not just trying to
reify electronic rights, we're trying to do it in a higher order manner.
So that contracts manipulate rights,
but a contract that unfolds over time, like a mortgage,
any contract that unfolds over time,
the continued participation in the contract is a derivative value. It is a valuable role. It's a valuable position to be in. And we want that valuable role to be a valuable tradable right and to conceive of the concept of rights
as a generic vessel,
so that a very great variety of kinds of rights can be fit into that vessel,
where the vessel has to be broad enough to fit many things into it, but it has to be strong enough such that you can write a very wide range of market institutions, of kinds of contracts,
exchange,
options,
various forms of derivatives, auctions,
et cetera, et cetera. We found that we could write these things in a generically parameterizable way, in a way that was parametrically polymorphic over rights,
in a way that is very much analogous to what we've seen before. So what we're trying to achieve is by enabling this higher-order composition of rights and contracts.
This is much of the richness of the real world, of the thousands of years of evolution of market institutions.
It's those deep networks of contracts
interacting with other contracts, of agreements and arrangements interacting with other agreements and arrangements that is the source of the richness of the world of the market.
Without this higher-order composition, we're not going to reproduce that richness. Where we have had higher-order composition,
functions and objects,
we have seen in the computer world a richness of composition,
where we have not seen higher-order composition in the existing world of blockchain and smart contracts, we have not seen deep networks of cooperation.
However, with proper humbleness, we should understand that we're probably taking a step that is also at some point going to run into a reification failure.
It's early days yet, so we don't yet have a painful reification failure to talk about,
but we should expect that eventually the process repeats again.
There is a reification failure and there will be a need to create a pattern at this level that can create yet another level of abstraction.
To provide orientation about what we're doing,
I'm just going to show the architecture Agoric is building,
but we're, in this talk, only concentrating on the upper layers of it. So just very briefly,
each rectangle here, each purple rectangle is an individual machine. Each stack is a logical machine. The stack on the left is a public blockchain.
It's a logical machine that's built out of massive multi-way agreement between many physical machines that all cross-check each other,
and that logical machine becomes a credible platform for running programs that need wide credibility, that need everyone to believe that what the program seems to have done is what the program legitimately did.
And then there's individual machines and non -blockchain replication,
and some of these things will be proprietary inside companies and some of them will be in the public.
What we want to do is we want to build a level of abstraction over that so that you can interact across the network
in very much the way we are used to with distributed systems across all of these different platforms.
And on top of that,
we're building our object capability layer.
So our object capability layer is a secure form of object-oriented programming
where the object reference,
the pointer to an object,
is a write and message passing as a form of writes transfer.
And we're building this to, again,
span over all of those platforms,
and I'll just briefly mention why that form of spanning is so powerful.
One of the things that we're used to with the web
is that the web is mostly a public system.
However, companies have internal websites behind the firewall that contains information that simply nobody intends to make public.
But the users within the firewall can follow links from internal webpages to external webpages in a completely seamless manner.
So we believe the same thing is true for the upcoming world of cooperative distributed secure computation that we want to span private and public where the public use benefits from a large volume of private use interconnecting to it and likewise the private use benefits. Markets are all about network effect,
and if we can span both of those worlds so that the logic of it
seamlessly interpenetrates, we can add value to both.
However,
this is a world of objects. It has exactly the reification failure I was just mentioning.
We're not able to reify a tradable write directly as an object.
So we build patterns of objects
that create this new level of abstraction which we call the e-writes and smart contracts. And there's this duality here. It's exactly the duality that I mentioned,
for example,
with selling a mortgage.
It's that contracts manipulate writes and a contract that unfolds over time creates tradable writes and many contracts can be generically parameterizable so they can form these cooperative networks.
And our system comes in a protocol stack
where each protocol builds each level of abstraction out of the previous level of abstraction.
We have something called VAT-TP which we're reconciling with the IBC system from Cosmos
that builds the VAT level of abstraction out of the machine level of abstraction. CAP-TP that builds the object level of abstraction out of the VAT level of abstraction.
Neither of those are relevant to the talk. What is relevant to the talk is ERTP which is the Electronic Writes Transfer Protocol
and that is the means by which starting with distributed secure objects we can build a system of tradable writes and smart contracts that facilitates higher order composition. So
an object reference in a memory safe object programming language is already a form of write even before you go from there to object capabilities.
If object Bob does not have a reference to object Carol
then in any memory safe object programming language object Bob simply cannot invoke Carol. He cannot make use of Carol's public API. He cannot provoke whatever behavior would be provoked by invoking Carol.
However, if Alice who has a reference to both Bob and Carol
invokes Bob passing Carol as an argument that is both Alice exercising her right to invoke Bob granted by her reference to Bob as well as Alice authorizing Bob to invoke Carol by providing the reference. And now that Bob receives the reference he can invoke Carol.
So to get from objects to object capabilities
we impose the constraint that these references between objects are the only carriers of causality. There is no other form of causality outside the system.
This makes the reference graph from the programming language literature identical to the access graph from the access control literature so that the richness that object programming already gives us at manipulating the reference graph we leverage that intuitively
as a direct extension of object intuitions for manipulating the access graph for manipulating security relationships.
And the result is that we can express
what would normally be thought of as complex security relationships very straightforwardly with simple object code that's easy to explain.
So on the left we have our object layer.
On the right we have Alice paying Bob in some amount of money.
As an example of the kind of erite that we want to support let's use the contrast between objects and money to explore the differences between these layers of abstraction.
So objects are shared.
Once Alice gives Bob access to Carol there's still this line on the left the diagonal line by which Alice still has access to Carol and Alice might drop the reference but Bob has no way to know that Alice has dropped it. Whereas for Bob to get paid to consider himself to have been paid he needs to know that he has exclusive access to the money that Alice gave him.
Objects are exercised. They perform actions.
The whole point of having reference to an object is you can do things by invoking it.
Whereas money is symbolic. It doesn't have any use value. It only has value in trading.
Objects are opaque.
When Bob receives this reference to Carol all Bob knows is he's received a reference to some object
that somebody who could invoke him decided to give him.
Whereas Bob in order to consider himself to be paid needs not just to have received the money he needs to know that he's received the money and he needs to know that he's received what money. To know that he's received three units
isn't any good if he doesn't know if those are gold coins or dollar bills or euros or whatever.
So he has to be receiving something where he understands the quantity it is that he's received.
Objects are specific.
Bob receives exactly Carol itself that particular object.
Money is fungible. Bob doesn't care what dollar bills he's received. He just cares about the total quantity he's left with at the end of the day.
So I'm drawing this contrast not to say that objects are the left column and e-writes are the right column.
Objects are indeed the left column. That's accurate.
But by e-writes I include all of these variations, all these degrees of variability
that this contrast has shown us four of the dimensions by which writes can be different from each other.
And by e-writes we want to include all of the variations and combinations that the contrast between the two has shown us as a world to explore and build.
So fortunately the patterns you need to express
in objects to build e-writes are straightforward.
This is the code needed to build
money with the attributes I just showed out of objects with the attributes I just showed.
So over here we have an outer function called makeMint
and each time we invoke makeMint
it makes an issuer object over here, a ledger object and a mint object
and it creates this layer in which those three objects are in bed with each other, they know each other.
So let's go ahead and,
so this layer, this plane represents
a logical new currency. So let's invoke makeMint four times
creating four different systems of currency all of which instantiate the code on the right.
The mint object is where new units of currency come from. It makes a purse initialized with those units.
So let's go ahead and create some new units and give Alice a purse containing $100 and let's invoke mint again creating a purse with $200 that we give to Bob.
So we consider this to be,
Alice has a right to 200 units of,
let's call it gold coins instead of dollars,
that Alice has a right to 100 of them and Bob has a right to 200 but notice that the rights themselves do not appear in the diagram and the rights themselves do not appear in the code.
They are not objects that you can store and retrieve from other generic objects.
Now let's say that in this arrangement Alice wants to buy a concert ticket from Bob.
So what Alice does first is she sends the withdraw method to the purse and what the withdraw method does is it makes a new purse.
So Alice's main purse creates a new payment purse and then consults the ledger and moves 10 units from the main purse to the payment purse.
Then Alice sends the payment to Bob
with this buy method,
this buy message where the buy message contains the payment and also a description of what Alice wants to buy and
now we have an interesting dilemma.
You might think that Alice has reified the payment in the payment purse. We even called the object payment.
So as far as Alice is concerned at this point she's paid Bob but we colored that reference error red because Bob has no way of knowing that Alice has dropped it.
Alice might still have access to the payment purse but that's not our only problem.
There's also,
Alice knows that she sent a purse whose purpose was to pay Bob but Bob doesn't know this yet.
He's just received some object from Alice. He doesn't know what it is.
But objects are exercisable. You can invoke them so we can use the behavior of the objects when invoked
to bring about the change of state that represents at our higher level of abstraction the motion of rights.
So Bob,
so going back to our code, Bob does this by sending a deposit message to his main purse saying deposit into yourself
10 units from the argument purse.
These purses consult the ledger, change the ledger's representation of that
and at no time has Bob revoked Alice's access to the payment.
Bob cannot revoke Alice's access to the payment object but it doesn't matter because now the payment object is associated with zero units.
Bob has revoked the usefulness of the payments object. Alice still has access to it but it does her no good.
The code that I showed you created
e-rights out of objects but it did not yet fit into our framework for creating higher order composition of contracts
in a smooth way.
So I want to be realistic about how much code it takes to do this for real and that's the code on the right
which I will not go through.
But I do want to emphasize something about the previous piece of code, this code and every piece of code you're about to see in this talk which is well these things are operating on blockchains and not blockchains and creating cryptographic interactions that represent distributed money protocols and other forms of cooperation.
Surely if they're doing this cryptographically
the code must be expressing cryptographic concepts
and the answer is no that it isn't. The answer is that that's one of the things that we're buying with our layer separation
is designing new cryptographic protocols, designing new arrangements of cryptographic primitives is bloody hard and experts get it wrong all the time and if every time we need a new complex cooperative arrangement we have to do it by figuring out a new arrangement of cryptographic primitives our new economy is sunk. We're not going to get there that way.
Instead our approach is to invest that effort once in the cryptographic
this is actually a first approximation it's not quite true but it's sufficiently true that it's the main truth of programming in the system
which is we invest the effort in building the cryptographic protocols once the very hard effort to create the system of distributed secure objects such that once we've got the power of distributed secure object capabilities we can now take a great variety of cooperative arrangements and express them simply in terms of objects and object behaviors without ever needing to refer back to cryptographic concepts.
So having built patterns
that express e-writes we're now going to think about things at this new level of abstraction that we've created
where the e-write itself
is now thought of as an object that moves so we'll use that as one visual language where you still have issuers and purses but the e -writes move between them.
We'll also use the visual language
where the e-write is just carried in a message and in this case Alice buying a ticket from Bob through the code on the right
once Alice receives the payment Alice is able, Bob is able I'm sorry, when Bob receives the payment Bob is able to know that he's been paid before releasing the concert ticket.
However,
this protects Alice from Bob this protects Bob from Alice it does not yet protect Alice from Bob as far as this arrangement is concerned
Bob could simply pocket the money and never release the ticket.
So this is still not an adequate set of primitives to start down on the road to rich composition of markets.
We need to go from this picture
to a picture that includes smart contracts
including a smart contract that brings about an exchange as an all or nothing exchange between two parties that Alice knows she won't lose her money unless she gets the ticket Bob knows he won't lose the ticket unless he gets the money.
So over here this is just a rearrangement of the elements you've already seen we have the money issuer in the upper left we have the ticket issuer in the upper right Alice has a main money purse and a main ticket purse and Bob likewise has a main money purse and a main ticket purse.
We introduce a new concept called a contract host and this is more than anything the kind of code that you would run on a public blockchain because this is the piece of code that needs the massive multi-way mutual credibility that you get by running code on a public blockchain.
So this is the thing that Alice and Bob and a million other contracting parties would have common knowledge of a particular contract host.
And what the contract host does is it accepts code from Alice and Bob and whoever else knows the contract host and the highlighted line here is where the contract host starts a contract.
So let's say that Bob submits a escrow exchange contract
so now the escrow exchange contract has been set in motion by the contract host and what the escrow exchange contract does is it creates two roles for participation in the escrow exchange
and then the contract host takes that and turns them into seats that Alice and Bob can sit in which is their ability to continue to participate in the contract.
And now that Alice and Bob can participate through these seats
Alice can send in the money that she would like to use to buy the ticket Bob can send in the ticket that he would like to offer to get the money in exchange
and the escrow exchange agent will just go to the issuers and obtain both of those rights and verify that it has obtained the rights that it has obtained from Alice the rights that Bob demands and it has obtained from Bob the rights that Alice demands.
And once it has verified that it can release the coin into Bob's main coin purse it can release the ticket into Alice's main ticket purse. So
an escrow exchange is fairly immediate.
A much more interesting example is the covered call option
which is,
let's say that the reason
that Bob wants to sell the ticket is he's bought a $500 ticket to a concert that he wants to go to but now he finds that oops, he has a conflict he's going to be out of town he wants to sell it to Alice but Alice doesn't know if she really wants to attend she needs to make some other plans
so she'd like to reserve the ticket.
So she says, okay Bob if I give you $10 can you just hold on to the ticket reserve it for me for a few days so I can make some other dependent plans.
So Bob's reservation if Bob agrees to reserve it that's a covered call option. It's very much like the covered call options the things called covered call options which are derivative instruments in the financial industry just logically the same relationship.
And the covered call option
it's more interesting than the escrow exchange it's more complicated why is it so much less code?
It's because this piece of code actually wraps the escrow exchange code it uses the escrow exchange and front ends it in order to create the differences between a covered call and an escrow exchange.
So how does a cover call differ from an escrow exchange? Well first of all the ticket is pre-escrowed until the covered call option has the ticket safely squirreled away where only it can transfer it
there is no covered call arrangement for Alice to participate in it doesn't exist until the ticket is in escrow.
Also with the escrow exchange either party can and will cancel the arrangement with the contract host Bob has given up the right to cancel the cancellation is instead held by a timer created by the contract host.
And other than that it's the same and it uses the escrow exchange underneath the surface it delegates all the hard work to the escrow exchange.
And the result is that now Alice is in this valuable arrangement.
But now that Alice has this reservation let's say Alice finds that oops she does need to travel anyway or she can't make use of it but Fred is she knows Fred happens to be interested maybe she'd like to sell the position in the reservation to Fred.
So
this is higher order composition this is every time we create a contract that unfolds over time we have created a derivative value we've created a situation
that is a valuable situation to be in.
And to bring about higher order composition
of rights and contracts we need those derivative values to themselves be derivative rights.
So we bring that about
by having these seats that are created by the contract hosts in reaction to the contracts that they're running asking the contract host to create a seat
that these seats are actually represented in a seat issuer that the covered call option
there is an issuer for the right to sit in that seat
that the contract host has created
and now for the covered call option this is basically functioning as the seat where Alice has the ability to participate but for the escrow exchange option down here it's acting as an issuer so to this contract the issuers are gold coins and tickets to this contract the issuers are the seat issuer and the silver coin issuer over there where Bob would buy the covered call option for Alice for these silver coins.
So these seats as derived rights as derived tradable rights
show us, demonstrate that this taxonomy we've opened up is a genuine taxonomy it really does open up a rich space of kinds of rights.
This seat, this ability to continue play in the first contract like money is an exclusive right.
The purpose of obtaining the seat is the possibility of exercising it of making use of it, of bringing about the behavior of the upper contract that's the right being traded in the lower contract is the ability to bring about behavior in the upper contract so therefore that right is exercisable the way an object is exercisable.
In order for Fred to buy it it needs to be assayable and this is actually where the interesting problems are
the interesting design issues is the upper right, the derivative right is created by an arbitrary program that is the smart contract that has to be designed to fit into the framework but it's some arbitrary program that has been designed to fit in the framework nevertheless,
Fred has to be able to describe
what role it is that he's trying to acquire what the nature is of what he's trying to be able to continue play in
so these seats are assayable in the way that money is assayable
and then the thing Fred
receives is the right to continue play in a particular contract this particular contract that Alice and Bob started so therefore it is specific the way objects are specific
and now I will take questions
More observation but
one of the things that I have
observed about ERC20 tokens or ERC721
is that they are reproducing some of what it means to have objects or object pointers in a clunky open coded way
but the fundamental thing that this in this spectrum that is different about them is they are only symbolic
with an ERC20 token or ERC721 it can't have exercise
so I can't have a concert ticket that I can ask the venue of and the other thing is that the fungible tokens and the non-fungible tokens are completely different from each other nobody writes contracts
over here if I write an auction
second price seal bid auction or English auction or Dutch auction or continuous double auction all of these things as well as these derivatives
this very wide range of things can be generically parameterizable where you can feed them where they can bring about transfer of things that are fungible or not fungible etc.
and the other that also addresses this issue of being able to
build and use and compose cooperative arrangements with confidence so that people of more normal skill people whose expertise is elsewhere in their subject matter that they want to cooperate about rather than having to be deep experts in smart contracting is the highly reusable nature of the components means that you can have the same kind of wide spectrum of effort and expertise that we're used to from normal software engineering where some components are designed very carefully but they're highly reusable so that they can now be so that now that they're heavily vetted and well understood they can be applied with confidence and then these simple wrappers the simple compositions can then be simple and simple to understand one of the key things about object capabilities as opposed to other security architectures is to support
compositional reasoning about correctness
even in regards to matters when there are multiple interests and rights at stake
this is something where the the main security paradigms identity based access control which Ethereum and the conventional
smart contracting world we see today is mostly trafficking in
we know that it does not support compositional reasoning about correctness with regard to such security matters
so high reuse plus compositional reasoning about correctness enables simple compositions of complex but highly reusable components to be done with confidence come
on, more questions
yes, I was struck by the the discovery aspect here that what's really exciting
discovery in right
and we've never had that before it's almost the opposite of that that you have a set of rights and then the discovery process is largely about how to exploit that or exploit the counterpart in this but this actually does create high power incentives
to discover rights that can be built on yes
well it's a wonderful observation I certainly appreciate it Kate, yes how
do you expect people to be able to
direct their computer agents to be able to
choose the right kinds of rights to buy and so forth
so
I'm I'm going to take the strategy of reducing this to a previously unsolved problem can you repeat the question? oh
in a world where there are millions of different designed
kinds of rights lots and lots of different contracts creating lots and lots of derivative rights how do you expect people or the programs people write to be able to navigate that
and understand what right they want to trade in and what that right means Kate, am I getting that? Okay, good
so reducing it to a previously unsolved problem
conventional software engineering is able to build up
tremendous rich
world of software that has really created the modern world without solving that problem either with regard to objects
but what we found is that we can go a great distance without solving the problem there's lots and lots of different interface types
and lots of different
APIs that essentially solve the same problem in ways that are not interoperable
and we've got
GitHub and just all sorts of communities and ways to interact with each other to become informed about what to use for what purpose
what things seem to be better at it, which are pleasant APIs to use it through
and
then we have things like package managers,
etc. that have various forms of description including prose and listing of dependencies so all of the great variety of categorization and explanation
and search and discovery that we have with regard to the great diversity of object APIs
I think
that that's a strength it means that the world is tolerant of having a great diversity that the richness of a great diversity gives us a richness of experimentation,
a diversity of ways to provide functionality and it does it at some cost at making discovery harder and then we build up these other institutions to try to ease the discovery process without reducing the variation
Bill?
So objects I'm sorry
the fact that these rights are exercisable does that give us an ability to I'm going to rephrase but tell me if I got it right does that give us an ability to represent within the system
rights to manipulate things outside the system in the physical world and the answer is yes and this very much builds on the fact that the answer with objects is already yes
we use our computer systems all the time to manipulate things in the world
let's just take the example of a printer in a given office
so we might take the printer driver and abstract it off up into a set of APIs that other objects can invoke for
commanding the printer
but now you can wrap that right in an ERTP interface and now you can
manage the allocation of that right let's say within this is a great example of use of it within an organization rather than a public blockchain is you can easily just transfer the right from one place to the other such that in any moment it might be an exclusive right to use the printer whether or not this makes sense for printer
several of the founders of Agorac had earlier founded a
well I don't need to go into that
the answer is yes that
it's very straightforward to create
that anytime you have software able to cause action in the world you can abstract a wrapper around that which is an exercisable right to provoke that software to affect the world
yeah
I'm a technical person I'm going to let Dean answer that one
oh sorry I'll repeat the question first who would be the top three companies let's say that would benefit by adopting these technologies
so
I'm also not going to answer that but I'm not going to answer that because
it's something where A it might not be a company adopting it to make the biggest difference right and B all of them will get different incomparable benefits from adopting this as we saw in Professor Jason Potts' talk you know the
ability to reduce the cost of trust across trading resources across trading stocks across communication across supply chain across transport
the World Trade Organization
noted that if they could deploy blockchain for tracking chain of custody of packages that would increase the global GDP by 5% and trade by
world trade by 15% it's one of those things where
you know the value in all these different areas is just too hard to compare yeah so I'm going to
pull from what you said
something that is sort of an answer to that question which is the real benefit of adopting this technology has to do with what happens between companies much more so than what happens within companies but both benefit