-
- Secure protocol design is much like design of regular protocols, but it is more obsessive about errors. Real computer science is about dealing with errors, and cryptoplumbing just deals with more of reality, and more aggression. What might be achieved with a restart or reboot in other designs could cause disaster in a security context. Here are some protocol tips to guide you on your way.
  
  **#2.1 Protocols Divide Naturally Into Two Parts**
  
  Good protocols divide into two parts, the first of which says to the second,
  
  **trust this key completely!**
  
  Frequently we see this separation between a *Key Exchange* phase and *Wire Encryption* phase within a protocol. Mingling these phases seems to result in excessive confusion of goals, whereas clean separation improves simplicity and stability.
  
  Note that this principle is recursive. That is, your protocol might separate around an exchange of keys, being part 1 for key exchange, and part 2 for encryption. The first part might then separate into two more components, one based on public keys and the other on secret keys, which we can call 1.a and 1.b. Some protocols separate further, for example primary (or root) public keys and local public keys, and, session negotiation secret keys and payload encryption keys.
  
  As long as the borders are clean and simple, this game is good because it helps us to conquer complexity. But when the borders are breached, we will add complexity which adds insecurity.
  
  **A Warning.** An advantage from this hypothesis might appear to be that one can swap in a different Key Exchange, or upgrade the protection protocol, or indeed repair one or other part without doing undue damage. But bear in mind the [#1](https://iang.org/ssl/h1_the_one_true_cipher_suite.html): just because these natural borders appear doesn't mean you should slice, dice, cook and book like software people do.
  
  **#2.2 KISS**
  
  *... you can make a secure system either by making it so simple you know it's secure, or so complex that no one can find an exploit.*
  
  allegedly Dan Geer, as reported by Jon Callas.
  
  In the military, KISS stands for *keep it simple, stupid* because soldiers are dumb by design (it is very hard to be smart when being shot at). Crypto often borrows from the military, but we always need to change a bit. In this case, KISS stands for
  
  **Keep the Interface Stupidly Simple.**
  
  When you divide your protocol into parts, KISS tells you that even a programmer should find every part easy to understand [[1](https://iang.org/ssl/h2_divide_and_conquer.html#ref_1)]. This is more than fortuitous, it is intentional, because the body programmer is your target audience. Remember that your hackers have to also understand all the other elements of good programming and systems building, so their attention span will be limited to around 1% of their knowledge space.
  
  A good example of this is a block cipher. A smart programmer can take a paper definition of a secret-key cipher or a hash algorithm and code it up in a weekend, and know he has got it right.
  
  Why is this? Three reasons:
- The interface is simple enough. There's a secret key, there's a block of input, and there's a block of output. Each is generally defined as a series of bits, 128 being common these days. What else is there to say?
- There is a set of numbers at the bottom of that paper description that provides test inputs and outputs. You can use those numbers to show basic correctness.
- The characteristic of cryptography algorithms is that they are designed to screw up fast and furiously. Wrong numbers will 'avalanche' through the internals and break everything. Which means, to your good fortune, you only need a very small amount of testing to show you got it very right.
  
  An excellent protocol example of a simple interface is SSL. No matter what you think of the internals of the protocol or the security claims, the interface is designed to exactly mirror the interface to TCP. Hence the name, Secure Socket Layer. An inspired choice! The importance of this is primarily the easy understanding achieved by a large body of programmers who understand the metaphor of a stream.
  
  **#2.3 Do not use anything you can't explain**
  
  By now, you should be getting the idea that any complicated mumbo jumbo from the cryptographers should be excluded. If you can't apply KISS, and you can't understand it, drop it. If the crypto blah blah divides you, your protocol will be conquered.
  
  To paraphrase Adi Shamir, [only the simplest crypto survives](http://www.financialcryptography.com/mt/archives/000147.html). There is no requirement for you to feel even the slightest embarrassment by failing to understand. It is the crypography world's failure, not your problem. Move on, use something simpler.
  
  Practically speaking, the things to understand from the cryptography world are secret key block ciphers (AES), modes (CCB), hashes (SHA1 and cousins), and HMACs. Not much more, and even these you should not understand more than you understand the networking side: coordination problem, atomic actions, sliding windows, request/response models, datagrams, etc.
  
  **#2.4 Avoid Public Key Cryptography like the Plague**
  
  Public key cryptography is the kiss of death to simplicity. The problem is that it is not simple, not amenable to KISS, and full of traps that will swallow a battleship. Although the very basic idea is understandable and elegant, none of the instantiations of public key cryptography can create simple interfaces that are free of minefields.
  
  Full-sized protocols will often need something like public key cryptography, but that doesn't mean you have to place them center stage. You should work aggressively to reduce your reliance on the public key phase.
  
  Look around for help in this area. This might be the one place where you bring in an experienced cryptoplumber to help with the design and interface design. Also, ask around how people reduce or even avoid their dependence on this phase.
  
  Highly successful applications are often (but not always) characterised by this one feature: how they successfully dumbed down the public key phase to the extent that it couldn't hurt them.
  
  **#2.5 Bootstrap Relationships into Key Exchanges**
  
  Key sharing is hard. We have no unified or simple theory for key sharing, and more work than we care to admit has broken on this simple problem. Most long-term problems occur in this one area.
  
  Use the application's natural paths of communication to share keys. Almost all applications have natural manners and means by which information is already transmitted. Nothing exists in a vacuum: banks send out snail mail, maillists send passwords out in email, chat programs work to link people who are often already linked in meatspace. For most applications, there are already many means you can use to share keys. Study them carefully, and look for ways to piggy back sufficient information on them to get enough key material across.
  
  The goal here is to seamlessly piggy-back (#3) the initial key sharing operations over some other channel or process, so as to
- generate enough key material to securely bootstrap to something strong, and
- to not require any additional work on behalf of the users.
  
  This latter (b) is perhaps the most important principle in cryptoplumbing for users, and also the most forgotten. Kerckhoffs said it first in 1883: if it is too much work for the user, it will not be used (his 6th principle).