- [19. Semigroupoid and Category · Tom Harding](https://omnivore.app/me/fantas-eel-and-specification-19-semigroupoid-and-category-tom-ha-18fa66a8248)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20230301115944/http://www.tomharding.me/2017/07/10/fantas-eel-and-specification-19/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[07/09/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20230301115944/http://www.tomharding.me/2017/07/10/fantas-eel-and-specification-19/
- ## Fantas, Eel, and Specification 19: Semigroupoid and Category  
      
    **It’s not goodbye**, Fantasists. We’ll have other projects, new memories, more chance encounters. Let’s end on a high: talking about the humble `Category`, and how we’ve been learning this since the beginning. While it may not be the most *immediately useful* structure, it’s a **gem for the curious**.  
      
    Before we go any further, let’s get the **methods** and **laws** on the table. A `Semigroupoid` has one method, called `compose`:  
      
  
    ```llvm
    compose :: Semigroupoid c
          => c i j
          ~> c j k
          -> c i k
    
    ```
      
    There’s also one **law**, which is going to look *boringly* familiar:  
      
  
    ```gcode
    // Associativity..
    a.compose(b.compose(c)) ===
    a.compose(b).compose(c)
    
    ```
      
    A `Category` is a `Semigroupoid` with **one new method**:  
      
  
    ```livescript
    id :: Category c => () -> c a a
    
    ```
      
    Now, if you’ve been following the series, there are no prizes for guessing the two laws that come with this:  
      
  
    ```gcode
    // Left and right identity!
    a === C.id().compose(a)
    === a.compose(C.id())
    
    ```
      
    Yep, it’s another `Monoid`-looking structure. Let’s all **pretend to be surprised**.  
      
  
  ---
      
    *Now*, there’s a reason to get excited about these things. You may not **directly** interact with them every day, but they’re *everywhere* underneath the surface:  
      
  
    ```javascript
    Function.prototype.compose =
    function (that) {
      return x => that(this(x))
    }
    
    Function.id = () => x => x
    
    ```
      
    Yep. Under function **composition**, our functions form a `Category`! Do notice that `Function\#compose` is defined the exact same way as `Function\#map`, too. Of course, this might be a bit easier to introduce to your codebase than `Function\#map`:  
      
  
    ```jboss-cli
    // Much more colleague-friendly?
    const app =
    readInput
    .compose(clean)
    .compose(capitalise)
    .compose(etCetera)
    
    ```
      
    We can think of `s a b` (for some `Semigroupoid s`) as “a relationship from `a` to `b`” (the set of which, in **fancy talk**, are called **morphisms**), and `Category` types as having “an identity relationship”. When we did [the deep dive with Monad](https://web.archive.org/web/20230301115944/http://www.tomharding.me/2017/06/05/fantas-eel-and-specification-15/), we actually looked at another `Category`:  
      
  
    ```javascript
    //- We need a TypeRep for M to do `id`.
    //- Note that, for a `Chain` type, we could
    //- only make a `Semigroupoid`, not a
    //- `Category`!
    const MCompose = M => {
    //+ type MCompose m a b = a -> m b
    const MCompose_ = daggy.tagged('MCompose', ['f'])
    
    //+ compose :: Chain m
    //+         => MCompose m a b
    //+         ~> MCompose m b c
    //+         -> MCompose m a c
    MCompose_.prototype.compose =
      function (that) {
        return x => this.f(x).chain(that.f)
      }
    
    //+ id :: Monad m => () -> MCompose m a a
    MCompose_.id = () => MCompose_(M.of)
    
    return MCompose_
    }
    
    ```
      
    Note that we originally wrote a `Monoid` for operations `a -> m a`. With a `Category`, we can talk about operations `a -> m b`, and we have much more **freedom**. Symmetrically, [our Comonad friends](https://web.archive.org/web/20230301115944/http://www.tomharding.me/2017/06/19/fantas-eel-and-specification-17/) give us an almost identical `Category`:  
      
  
    ```javascript
    //- Extend for Semigroupoid, Comonad for
    //- Category!
    //+ type WCompose w a b = w a -> b
    const WCompose = daggy.tagged('WCompose', ['f'])
    
    //+ compose :: Extend w
    //+         => (w a -> b)
    //+         ~> (w b -> c)
    //+         -> (w a -> c)
    WCompose.prototype.compose =
    function (that) {
      return x => this.f(x).extend(that.f)
    }
    
    //+ id :: Comonad w => () -> WCompose w a a
    WCompose.id = () =>
    WCompose(x => x.extract())
    
    ```
      
    Why limit ourselves? We have **composition**! Let’s turn [the Applicative](https://web.archive.org/web/20230301115944/http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/)’s composition into a `Category`:  
      
  
    ```javascript
    //+ Apply for Semigroupoid, Applicative for
    //+ Category!
    
    const ApCompose = (A, C) => {
    //+ type ApCompose f c a b = f (c a b)
    const ApCompose_ = daggy.tagged('ApCompose', ['f'])
    
    //+ compose :: Apply f
    //+         => Semigroupoid s
    //+         => f s a b
    //+         ~> f s b c
    //+         -> f s a c
    ApCompose_.prototype.compose =
      function (that) {
        return that.f.ap(this.f.map(
          x => y => x.compose(y)))
      }
    
    //+ id :: Applicative f
    //+    => Category s
    //+    => () -> f s a a
    ApCompose_.id = () => A.of(C.id())
    }
    
    ```
      
    The really **neat** thing here is that, instead of limiting ourselves to the `Category` of *functions* within `Applicative` context, we’ve generalised `Function` to `Category`! It’s all getting a bit **abstract** and **scary**, right? **Don’t panic**.  
      
  
  ---
      
    With the **helpful exception** of `Function`, these examples may all seem a bit impractical. *Granted*, you might find that `MCompose\#compose` gives you a nice, declarative way to chain together **monadic actions**, or something to that effect, but this largely seems a bit too… well, **abstract**!  
      
    While this *may* not be the one you use every day, the `Semigroupoid` and `Category` structures form a *very* important idea. We hinted at this earlier: *`Category` is like a `Monoid` with __more freedom__*. Well, if everything we’ve seen ended up looking like `Semigroup` or `Monoid`… effectively, the **base concept** been `Category` all along!  
      
  
    > I had originally wanted to show `Monoid` in terms of `Category`. Let’s just say the **page of required declarations** compiled with the **help of three other people** made me think that, if *I* couldn’t understand it, I probably shouldn’t put it here!  
  
      
    It’s not *crucial* that we understand why. [Discussion around categories](https://web.archive.org/web/20230301115944/https://graphicallinearalgebra.net/2017/04/16/a-monoid-is-a-category-a-category-is-a-monad-a-monad-is-a-monoid/) leads me to **all sorts** of headaches, and to no avail. What is important is that I make a **small confession**…  
      
    *This __entire__ series has been about a branch of maths called __Category Theory__: the theory of __categories__*.  
      
    **Blam**! Here in programmer land, using our shiny new structures to write **fault-tolerant database systems** and **reactive event streams**, we thought we were safe from maths… but **no**! The whole **purpose** of Fantasy Land is to utilise concepts from category theory that help us to write **safer code**. “Was `Functor` maths?” *I’m afraid so*. “Surely our `Semigroup` was innocent, though?” *I wish I could tell you so*. “Not `Alt` though, right? … Right?” *All category theory. Please stop asking.*  
      
    Perhaps you’ll never need to think about `Category` again, and that’s **fine**. Function composition is more than valuable enough to justify its existence. *However*, sooner or later, if you continue down this **rabbit hole**, you’ll start asking **why** and **how**, and I’m sure you’ll find your way back here…  
      
    Simply, there’s really not much (immediately practical) to say about `Category` and `Semigroupoid` at this level of generality, but it’s a neat little concept, and I thought it was nice to put a name to the pattern we’ve been seeing **all the time**! Next time you define a weird type of **relationship** (perhaps one involving `Monad`?), give a thought to the humble `Category` we’ve just seen, and see whether you can simplify your code with a more specific `compose` implementation.  
      
  
  ---
      
    The keen-eyed among you will have noticed that this is **the last Fantasy Land structure**, and that we’ve really come to **the end of the series**. Well… **don’t panic**! I have **one more** article to publish before we call this whole thing a day. Call it the *encore* that no one wanted.  
      
    On that note, next time, we’ll be talking about **bringing concepts together** to build some cool little projects, and where you can go next! Otherwise, **thank you _so much_** for reading through **at least** 19 pages of my mindless rambling, and I hope that it has been useful in some small way.  
      
    Now, **write beautiful JavaScript**.  
      
    ♥ ♥ ♥
- [18. Bifunctor and Profunctor](https://omnivore.app/me/fantas-eel-and-specification-18-bifunctor-and-profunctor-tom-har-18fa66a0046)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20230301115931/http://www.tomharding.me/2017/06/26/fantas-eel-and-specification-18/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[06/25/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20230301115931/http://www.tomharding.me/2017/06/26/fantas-eel-and-specification-18/
- ## Fantas, Eel, and Specification 18: Bifunctor and Profunctor  
      
    **The worst is behind us**. We’ve [mastered the Monad](https://web.archive.org/web/20230301115931/http://www.tomharding.me/2017/06/05/fantas-eel-and-specification-15/), [conquered the Comonad](https://web.archive.org/web/20230301115931/http://www.tomharding.me/2017/06/19/fantas-eel-and-specification-17/), and [surmounted the Semigroup](https://web.archive.org/web/20230301115931/http://www.tomharding.me/2017/03/13/fantas-eel-and-specification-4/). Consider these last two posts to be a **cool-down**, because **the end is in sight**. Today, to enjoy our first week of rest, we’re going to revise **functors**. A number of times, we’ve seen types with **two** inner types: `Either`, `Pair`, `Task`, `Function`, and so on. However, in all cases, we’ve **only** been able to `map` over the **right-hand** side. Well, Fantasists, the reason has something to do with a concept called **kinds** that we won’t go into. Instead, let’s look at **solutions**.  
      
    We’ll take a type like `Either` or `Pair`. These types have **two** inner types (*left* and *right*!) that we could `map` over. The `Bifunctor` class allows us to deal with both:  
      
  
    ```livescript
    bimap :: Bifunctor f
        => f a c
        ~> (a -> b, c -> d)
        -> f b d
    
    ```
      
    It’s pretty much **exactly like `Functor`**, except we are mapping **two at a time**! What does this look like in *actual* code?  
      
  
    ```javascript
    Either.prototype.bimap =
    function (f, g) {
      return this.cata({
        Left: f,
        Right: g
      })
    }
    
    Pair.prototype.bimap =
    function (f, g) {
      return Pair(f(this._1),
                  g(this._2))
    }
    
    Task.prototype.bimap =
    function (f, g) {
      return Task((rej, res) =>
        this.fork(e => rej(f(e)),
                  x => res(g(e))))
    }
    
    ```
      
    Hopefully, no huge surprises. We apply the **left function** to any mention of the **left value**, and the same for the right. Even the laws are just **doubled-up** versions of the `Functor` laws:  
      
  
    ```reasonml
    // For some functor U:
    
    // Identity
    U.map(x => x) === U
    
    // Composition
    U.map(f).map(g) ===
    U.map(x => g(f(x)))
    
    // For some bifunctor C:
    
    // Identity
    C.bimap(x => x, x => x) === C
    
    // Composition
    C.bimap(f, g).bimap(h, i) ===
    C.bimap(l => h(f(l)),
            r => i(g(r)))
    
    ```
      
    A lot of brackets, but look closely: the laws are the same, but we have **two independent “channels”** on the go!  
      
  
    > *Fun fact: if you have a `Bifunctor` instance for some type `T`, you can automatically derive a fully-lawful `Functor` instance for `T a` with `f => bimap(x => x, f)`. __Hooray, Free upgrades__!*  
  
      
    So, is this useful? **Yes**! Let’s look at a neat little example using `Either` for a second:  
      
  
    ```javascript
    //- Where everything changes...
    const login = user =>
    isValid(user) ? Right(user)
                  : Left('Boo')
    
    //- Function map === "piping".
    //+ failureStream :: String
    //+               -> HTML
    const failureStream =
    (x => x.toUpperCase())
    .map(x => x + '!')
    .map(x => '<em>' + x + '</em>')
    
    //+ successStream :: User
    //+               -> HTML
    const successStream =
    (x => x.name)
    .map(x => 'Hey, ' + x + '!')
    .map(x => '<h1>' + x + '</h1>')
    
    //- We can now pass in our two
    //- possible application flows
    //- using `bimap`!
    login(user).bimap(
    failureStream,
    successStream)
    
    ```
      
    Note that this would also work with `Task`! We’re in a situation where we want to transform a potential success *or* failure, and `bimap` lets us supply both at once. *Cool*, right? **Straightforward**, too!  
      
  
  ---
      
    Now, `Function` is a *slightly* different story. Effectively, we can `contramap` over its **left-hand type** (the **input**) and `map` over its **right-hand type** (the **output**). It turns out there’s a fancy name for this sort of thing, too: `Profunctor`.  
      
  
    ```stylus
    promap :: Profunctor p
         => p b c
         ~> (a -> b, c -> d)
         -> p a d
    
    ```
      
    You can think of it as adding a **before** and **after** step to some process. Naturally, the laws look like a **mooshmash** of `Contravariant` and `Functor`:  
      
  
    ```gcode
    // For some profunctor p:
    
    // Identity...
    P.promap(a => a, b => b) === P
    
    // Composition...
    p.promap(f, i).promap(g, h) ===
    p.promap(a => f(g(a)),
             b => h(i(b)))
    
    ```
      
  
    > *Guess what? You can build a functor `P a` out of any profunctor `P`: `f => promap(x => x, f)` is all it takes. So many __free upgrades__!*  
  
      
    The **left-hand** side looks like `Contravariant`, and the **right-hand** side like `Functor`. Of course, we’ve seen a `Profunctor` already: **`Function`**! However, to give a slightly more **exciting** example, let’s look at one of my **favourites**: `Costar`.  
      
  
    ```typescript
    //- Fancy wrapping around a specific type
    //- of function: an "f a" to a "b"
    //+ Costar f a b = f a -> b
    const Costar = daggy.tagged('Costar', ['run'])
    
    //- Contramap with the "before" function,
    //- fold, then apply the "after" function.
    //+ promap :: Functor f
    //+        => Costar f b c
    //+        -> (b -> a, c -> d)
    //+        -> Costar f a d
    Costar.prototype.promap = function (f, g) {
    return Costar(x => g(this.run(x.map(f))))
    }
    
    //- Takes a list of ints to the sum
    //+ sum :: Costar Array Int Int
    const sum = Costar(xs =>
    xs.reduce((x, y) => x + y, 0))
    
    //- Make every element 1, then sum them!
    const length = sum.promap(_ => 1, x => x)
    
    //- Is the result over 5?
    const isOk = sum.promap(x => x, x => x > 5)
    
    //- Why not both? Is the length over 5?
    const longEnough = sum.promap(
    _ => 1, x => x > 5)
    
    // Returns false!
    longEnough.run([1, 3, 5, 7])
    
    ```
      
    `Costar` allows us to take the idea of `reduce` and wrap it in a `Profunctor`. We can use `promap` to **prepare** different inputs for the reduction *and* **manipulate** the result. You may have heard of this idea before: **map/reduce**. No matter how **complex** the process, there’s a good chance that you can express it in a `Profunctor`!  
      
  
  ---
      
    These are, of course, *very* quick overviews of `Bifunctor` and `Profunctor`. That said, if you’re comfortable now with `Functor` and you remember the post on `Contravariant`, there’s **nothing new** to learn! We’re really just building on ideas we’ve already had. `Bifunctor` might seem a little *underwhelming* now that we’ve seen the **power** of `Monad` and `Comonad`, but it’s *twice* as powerful as `Functor`: we can define a flow for **success** and **error**, for **left** and **right**, for… well, any **two things**!  
      
    As for `Profunctor`, it’s a pretty massive topic once you start digging. `Costar` is the opposite of `Star`, which is an `a -> f b` function; why not think about how to implement that? Would you need any **special conditions** to make it a `Profunctor`?  
      
    Take [the article’s gist](https://web.archive.org/web/20230301115931/https://gist.github.com/richdouglasevans/891564c2d13363b46e49187f28a28ae8), and `bimap` and `promap` until the cows come home, Fantasists, for there is only **one** article left: `Semigroupoid` and `Category`. Expect **high drama**, **hard maths**, and **herds of monoids**. Well, maybe not the second thing…  
      
    Until then!  
      
    ♥
- [16. Extend](https://omnivore.app/me/fantas-eel-and-specification-16-extend-tom-harding-18fa6698e51)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20231206140740/http://www.tomharding.me/2017/06/12/fantas-eel-and-specification-16/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[06/11/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20231206140740/http://www.tomharding.me/2017/06/12/fantas-eel-and-specification-16/
		    
		  You’re **still** here? That means you survived `Monad`! See? Told you it isn’t that scary. It’s nothing we haven’t **already seen**. Well, *today*, we’re going to revisit `Chain` with one *slight* difference. As we know, `Chain` takes an `m a` to an `m b` with some help from an `a -> m b` function. It **sequences** our ideas. However, *what if I told you_… we could go __backwards__? Let’s `Extend` your horizons.
		  
		  Don’t get too excited; the disappointment of `Contravariant` will come flooding back! We’re certainly not saying that we have a magical `undo` for any `Chain`. What we _are* saying, though, is that there are some types for which we can get from `m a` to `m b` via `m a -> b`. Instead of **finishing** in the context, we **start** in it!  
		    
		  Two questions probably come to mind:  
		    
		  
		  * **How** is this useful?
		  * … See question 1?  
		  
		  Well, we’ll be answering **at least** one of those questions today! *Before* that, though, let’s go back to our *old* format and **start** with the **function**:
		   ```livescript
		  extend :: Extend w => w a ~> (w a -> b) -> w b
		  
		   ```
		   > Notice we’re using `w` here. I kid you *not*, we use `w` because it looks like an upside-down `m`, and `m` was what we used for `Chain`. It’s literally there to make the whole thing look **back-to-front** and **upside-down**. I’m told that mathematicians call this a **joke**, so do *try* to laugh!  
		  
		  *As we said*, it’s `Chain` going backwards. It even has the same laws backwards! Say hello, once again, to our old friend **associativity**:
		   ```jboss-cli
		  // RHS: Apply to f THEN chain g
		  m.chain(f).chain(g)
		  === m.chain(x => f(x).chain(g))
		  
		  // RHS: extend f THEN apply to g
		  w.extend(f).extend(g)
		  === x.extend(w_ => g(w_.extend(f)))
		  
		   ```
		   > Wait, so, if `extend` has **associativity**, and if it looks a bit like a `Semigroup`… (*all together now*) **where’s the `Monoid`**?!  
		  
		  It’s just like `chain`, but backwards. **Everything is backwards**. It’s really the only thing you need to remember!  
		  
		  ?thgir ,taen ytterP
		  ---
		  
		  So, *aside* from it being backwards, is there anything *useful* about `Extend`? *Or*, are we just getting a bit tired at this point? **Both**! Hopefully more the former, though…  
		  
		  Let’s start with an old friend, [the Pair (or Writer) type](https://web.archive.org/web/20231206140740/http://www.tomharding.me/2017/04/27/pairs-as-functors/). When we `chain`, we have *total* control over the **output** of our function: we say what we *append* to the **left-hand side**, and what we want the right-hand value to be. There was, however, one thing we *can’t* do: see what was already *in* the left part!  
		  
		  `Pair` really gave us a wonderful way to achieve **write-only** state, but we had no way of **reading** what we’d written! Wouldn’t it be **great** if we had a `map`-like function that let us take a **sneaky peek** at the left-hand side? Something like this, perhaps:
		   ```livescript
		  map :: (m, a) ~> (a  -> b) -> (m, b)
		  
		  -- Just like map, but we get a sneaky peek!
		  sneakyPeekMap :: (m, a)
		  ~> ((m, a) -> b)
		  -> (m, b)
		  
		   ```
		  It’s really just like `map`, but we get some **context**! Now that we can, whenever we like, take a look at how the left-hand side is doing, we can feed our **hungry adventurer**:
		   ```crmsh
		  //- Sum represents "hunger"
		  const Adventurer = Pair(Sum)
		  
		  //+ type User = { name :: String, isHungry :: Bool }
		  const exampleUser = {
		  name: 'Tom',
		  isHungry: false
		  }
		  
		  // You get the idea... WorkPair again!
		  
		  //+ slayDragon :: User -> Adventurer User
		  const slayDragon = user =>
		  Adventurer(Sum(100), user)
		  
		  //+ slayDragon :: User -> Adventurer User
		  const runFromDragon = user =>
		  Adventurer(Sum(50), user)
		  
		  //- Eat IF we're hungry
		  //+ eat :: User -> Adventurer User
		  const eat = user =>
		  user.isHungry
		  ? Adventurer(Sum(-100), {
		  ... user,
		  isHungry: false
		  })
		  
		  : Adventurer(Sum(0),   user)
		  
		  //- How could we know when we're hungry?
		  //- This function goes the other way...
		  //+ areWeHungry :: Adventurer User -> User
		  const areWeHungry =
		  ({ _1: { value: hunger }, _2: user }) =>
		  hunger > 200
		  ? { ... user, isHungry: true }
		  : user
		  
		  // Now, we do a thing, check our hunger,
		  // and eat if we need to!
		  
		  // WE ARE SELF-AWARE.
		  // SKYNET
		  
		  slayDragon(exampleUser)
		  .sneakyPeekMap(areWeHungry).chain(eat)
		  // Pair(Sum(100), not hungry)
		  
		  .chain(slayDragon)
		  .sneakyPeekMap(areWeHungry).chain(eat)
		  // Pair(Sum(200), not hungry)
		  
		  .chain(runFromDragon)
		  .sneakyPeekMap(areWeHungry).chain(eat)
		  // Pair(Sum(150), not hungry)!
		  
		   ```
		  Just with this `sneakyPeekMap`, we can now inspect our character stats and feed that *back* into our actions. This is *so* neat: any time you want to update one piece of data depending on another, `sneakyPeekMap` is **exactly** what you need. Oh, and by the way, it has a much more common name: `extend`!
		  ---
		  
		  *So, can I just think of `extend` as `sneakyPeekMap`?* I mean, you basically can; that intuition will get you a **long** way. As an homage to [Hardy Jones’ functional pearl](https://web.archive.org/web/20231206140740/https://joneshf.github.io/programming/2015/12/31/Comonads-Monoids-and-Trees.html), let’s build a [**Rose Tree**](https://web.archive.org/web/20231206140740/https://en.wikipedia.org/wiki/Rose%5FTree):
		   ```php
		  //- A root and a list of children!
		  //+ type RoseTree a = (a, [RoseTree a])
		  const RoseTree = daggy.tagged(
		  'RoseTree', ['root', 'forest']
		  )
		  
		   ```
		  *Maybe*, you looked at that type and a little voice in the back of your mind said `Functor`. If so, **kudos**:
		   ```javascript
		  RoseTree.prototype.map = function (f) {
		  return RoseTree(
		  f(this.root), // Transform the root...
		  this.forest.map( // and other trees.
		  rt => rt.map(f)))
		  }
		  
		   ```
		  **No problem**; transform the root node, and then *recursively* map over all the child trees. **Bam**. *Now*, imagine if we wanted to *colour* this tree’s nodes depending on how many **children** they each have. Sounds like we’d need to take… a **sneaky peek**!
		   ```javascript
		  //+ extend :: RoseTree a
		  //+        ~> (RoseTree a -> b)
		  //+        -> RoseTree b
		  RoseTree.prototype.extend =
		  function (f) {
		  return RoseTree(
		  f(this),
		  this.forest.map(rt =>
		  rt.extend(f))
		  )
		  }
		  
		  // Now, it's super easy to do this!
		  MyTree.extend(({ root, forest }) =>
		  forest.length < 1
		  ? { ... root, colour: 'RED' }
		  : forest.length < 5
		  ? { ... root, colour: 'ORANGE' }
		  : { ... root, colour: 'GREEN' })
		  
		   ```
		  With our new-found **superpower**, each node gets to pretend to be the **one in charge**, and can watch over their own forests. The trees in those forests then do the same, and so on, until we `map`-with-a-sneaky-peek **the entire forest**! Again, I linked to Hardy’s article above, which contains a much **deeper** dive into trees specifically; combining `RoseTree` with Hardy’s ideas is enough to make your own **billion-dollar React clone**!
		  ---
		  
		  Let’s cast our minds *waaay* back to [the Setoid post](https://web.archive.org/web/20231206140740/http://www.tomharding.me/2017/03/09/fantas-eel-and-specification-3/), when we looked at `List`. List, it turns out, is a `Functor`:
		   ```javascript
		  //- Usually, if you can write something for
		  //- Array, you can write it for List!
		  List.prototype.map = function (f) {
		  return this.cata({
		  // Do nothing, we're done!
		  Nil: () => Nil,
		  
		  // Transform head, recurse on tail
		  Cons: (head, tail) =>
		  Cons(f(head), tail.map(f))
		  })
		  }
		  
		  // e.g. Cons(1, Cons(2, Nil))
		  //      .map(x => x + 1)
		  // === Cons(2, Cons(3, Nil))
		  
		   ```
		  Now, for this **convoluted example**, let’s imagine we have some weather data for the last few days:
		   ```javascript
		  const arrayToList = xs => {
		  if (!xs.length) return Nil
		  
		  const [ head, ... tail ] = this
		  return Cons(head, arrayToList(tail))
		  }
		  
		  List.prototype.toArray = function () {
		  return this.cata({
		  Cons: (head, tail) => ([
		  head, ... tail.toArray()
		  ]),
		  
		  Nil: () => []
		  })
		  }
		  
		  // Starting from today... (in celsius!)
		  const temperatures = arrayToList(
		  [23, 19, 19, 18, 18, 20, 24])
		  
		   ```
		  What we want to do is `map` over this list to determine whether the temperature has been warmer or cooler than the day before! To do that, we’ll probably need to do *something* sneakily… any ideas?
		   ```javascript
		  //+ List a ~> (List a -> b) -> List b
		  List.prototype.extend = function (f) {
		  return this.cata({
		  // Extend the list, repeat on tail.
		  Cons: (head, tail) => Cons(
		  f(this), tail.extend(f)
		  ),
		  
		  Nil: () => Nil // We're done!
		  })
		  }
		  
		  // [YAY, YAY, YAY, YAY, BOO, BOO, ???]
		  temperatures
		  .extend(({ head, tail }) =>
		  tail.length == 0 ? '???'
		  : head < tail.head
		  ? 'BOO' : 'YAY')
		  .toArray()
		  
		   ```
		  We only used the *head* of the tail this time, but we could use the whole thing if we wanted! We have the entire thing available to peek sneakily.
		   > For example, we could use this technique for lap times to record whether they’re **faster or slower** than the **average** so far! **Have a go!**  
		  
		  As we said, we can mimic the same behaviour for `Array` to save us all the to-and-fro with `List`:
		   ```javascript
		  Array.prototype.extend = function (f) {
		  if (this.length === 0) return []
		  
		  const [ _, ... tail ] = this
		  return [ f(this), ... tail.extend(f) ]
		  }
		  
		  // Now, we can use array-destructuring :)
		  ;[23, 19, 19, 18, 18, 20, 24].extend(
		  ([head, ... tail]) =>
		  tail.length == 0
		  ? '???'
		  : head < tail[0]
		  ? 'BOO' : 'YAY')
		  
		   ```
		  ---
		  
		  I brought these examples up for a reason, Fantasists. *Often*, `extend` isn’t just `f => W.of(f(this))` for some `W` type; that’s what `of` is for! `extend` is about being able to `map` while being aware of the surrounding **construct**.  
		  
		  Think of it like this: when we used `Chain`, we had total **write** access to the **output** *constructor* and *values*. We could turn `Just` into `Nothing`, we could fail a `Task`, and we could even change the length of an `Array`. **Truly, we were powerful**.  
		  
		  With `Extend`, we get full **read** access to the **input** *constructor* and *values*. It’s the **opposite idea**.  
		  
		  Whereas `Chain` lets us **inform the future** of the computation, `Extend` lets us **be informed by the past**. *This is the kind of sentence that ends up on a mug, you know!*
		   ```xl
		  -- chain: `map` with write-access to output
		  -- extend: `map` with read-access to input
		  
		  map    :: f a -> (  a ->   b) -> f b
		  chain  :: m a -> (  a -> m b) -> m b
		  extend :: w a -> (w a ->   b) -> w b
		  
		   ```
		  ---
		  
		  There are lots of cool examples of `Extend`, but they are often overlooked and generally considered a more “advanced” topic. After all, with `Monad`, we’re free to build **anything**; why bother continuing? Well, I hope this gives you an idea of how they work and where you can find them! After all, these are all just **design patterns**: just use them when they’re **appropriate**!  
		  
		  *So, `Semigroup` is to `Monoid` as `Chain` is to `Monad` as `Extend` is to…?* We’ll find out next time! Before we go, though, here’s a very tricky challenge to keep you busy:
		   > With `Writer`, we needed a `Semigroup` on the left to make a `Chain` instance, but we didn’t for `Extend`. `Reader` has an `Extend` instance; can you think of how we might write that?  
		  
		  Until then, take a look through [this article’s code](https://web.archive.org/web/20231206140740/https://gist.github.com/richdouglasevans/61eae2a787bd616f04e63a642a0dca5d), and take care!  
		  
		  ♥
- [15. Monad](https://omnivore.app/me/fantas-eel-and-specification-15-monad-tom-harding-18fa669234f)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/06/05/fantas-eel-and-specification-15/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[06/04/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/06/05/fantas-eel-and-specification-15/
		    
		  **Today is the day**, Fantasists. We all knew it was coming, but we *hoped* it wouldn’t be so soon. Sure enough, though, **here we are**. We’ve battled through **weeks** of structures, and reached the dreaded `Monad`. Say your goodbyes to your loved ones, and **let’s go**.  
		    
		  *Ahem.*  
		    
		  **A `Monad` is a type that is both a `Chain` and an `Applicative`**. That’s… well, it, really. We’re done here. Next time, we’ll be looking at the **new-wave wizardry** of `Extend`. Until then, take care!  
		    
		  ♥  
		    
		  
		  ---
		    
		  … Right, so maybe we could say a *little* more, but only if we *want* to! Honestly, though, the above is enough to get going. We’ve seen a few examples of `Semigroup`-and-`Monoid`-feeling relationships, but let’s focus on two in particular:  
		    
		  In [the Apply post](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/04/10/fantas-eel-and-specification-8/), we said that `ap` felt a bit `Semigroup`-ish. Then, in [the Applicative post](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/), we saw that adding `of` gave us something `Monoid`-ish.  
		    
		  Later, [we looked at Chain](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/05/15/fantas-eel-and-specification-13/), where `chain` gave us something like a `Semigroup`. *So*, you’re asking, *where’s the `Monoid`?* Well, with a `Monad`, `of` doubles up as the `empty` to **`Chain`**’s `Semigroup`, too!  
		    
		  We’re going to go through a pretty **mathematical** definition of `Monad` first, so don’t be discouraged if it doesn’t make sense on the **first few reads**. This is really just *background knowledge* for the curious; **skip ahead** if you just want to see a **practical** example!  
		    
		  
		  ---
		    
		  
		  > To skip ahead, **start scrolling**!  
		  
		    
		  Let’s do some **mind-blowing**; it’s the `Monad` post after all, right? Let’s first define two composition functions, `compose` and `mcompose`:  
		    
		  
		  ```javascript
		  //- Regular `compose` - old news!
		  //+ compose ::      (b -> c)
		  //+         -> (a -> b)
		  //+         ->  a   ->    c
		  const compose = f => g => x =>
		  f(g(x))
		  
		  //- `chain`-sequencing `compose`, fancily
		  //- known as Kleisli composition - it's the
		  //- K in Ramda's "composeK"!
		  //+ mcompose :: Chain m
		  //+          =>        (b -> m c)
		  //+          -> (a -> m b)
		  //+          ->  a     ->    m c
		  const mcompose = f => g => x =>
		  g(x).chain(f)
		  
		  ```
		    
		  *I’ve tried to line up the types so it’s a bit clearer to see __left-to-right__ how this works… I hope that it helped in some way!*  
		    
		  `compose` says, “_Do `g`, then `f`*”. `mcompose` says __the same thing__, but does it with some kind of __context__ (*[little language extension bubble](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/03/27/fantas-eel-and-specification-6/), remember?*). That `m` could be `Maybe` in the case of two functions that may fail, or `Array` in the case of two functions that return multiple values, and so on. What’s important is that, to _use* `mcompose`, our `m` **must** be a `Chain` type.  
		    
		  Now, you can make something very monoid-looking with regular `compose`:  
		    
		  
		  ```javascript
		  const Compose = daggy.tagged('Compose', ['f'])
		  
		  //- Remember, for semigroups:
		  //- concat :: Semigroup s => s -> s -> s
		  //- Replace s with (a -> a)...
		  //+ concat ::      (a -> a)
		  //+        -> (a -> a)
		  //+        ->  a   ->    a
		  Compose.prototype.concat =
		  function (that) {
		    return Compose(
		      x => this(that(x))
		    )
		  }
		  
		  //- We need something that has no effect...
		  //- The `id` function!
		  //+ empty :: (a -> a)
		  Compose.empty = () => Compose(x => x)
		  
		  ```
		    
		  Mind blown yet? **Function composition is a monoid**! The `x => x` function is our `empty` (because it doesn’t do anything), and composition is `concat` (because it combines two functions into a pipeline). See? **Everything is just monoids**. Monoids *all* the way down.  
		    
		  
		  > Typically, the `Compose` type is used for other things (remember [the Traversable post](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/05/08/fantas-eel-and-specification-12/)?), but we’re using it here as just a nice, *clear* name for this example.  
		  
		    
		  Now, here’s the **real wizardry**: can we do the same thing with `mcompose`? Well, we could certainly write a `Semigroup`:  
		    
		  
		  ```javascript
		  const MCompose = daggy.tagged('MCompose', ['f'])
		  
		  //- Just as we did with Compose...
		  //+ concat :: Chain m
		  //+        =>        (a -> m a)
		  //+        -> (a -> m a)
		  //+        ->  a     ->    m a
		  MCompose.prototype.concat =
		  function (that) {
		    return MCompose(
		      x => that(x).chain(this)
		    )
		  }
		  
		  ```
		    
		  `concat` now just does `mcompose` instead of `compose`, as we expected. If we want an `empty`, though, it would need to be an `a -> m a` function. Well, **reader mine**, it just so happens that we’ve already seen that very function: from `Applicative`, the `of` function!  
		    
		  
		  ```reasonml
		  //- So, we need empty :: (a -> m a)
		  //+ empty :: Chain m, Applicative m
		  //+       => (a -> m a)
		  MCompose.empty = () =>
		  MCompose(x => M.of(x))
		  
		  // Or just `MCompose(M.of)`!
		  
		  ```
		    
		  
		  > Note that, as with lots of interesting `Monoid` types, we’d need a `TypeRep` to build `MCompose` to know which `M` type we’re using.  
		  
		    
		  To make `MCompose` a full `Monoid`, we need our `M` type to have an `of` method *and* be `Chain`able. `Chain` for the `Semigroup`, plus `Applicative` for the `Monoid`.  
		    
		  Take a breath, Fantasists: I’m aware that I *might* be alone here, but I think this is **beautiful**. No matter how clever we *think* we’re being, it’s all really just `Semigroup`s and `Monoid`s at the end of the day. Under the surface, it **never** gets more **complex** than that.  
		    
		  Let’s not get *too* excited just yet, though; remember that there are **laws** with `empty`. Think back to [the Monoid post](https://web.archive.org/web/20230610043014/http://www.tomharding.me/2017/03/21/fantas-eel-and-specification-5/): it has to satisfy **left and right identity**.  
		    
		  
		  ```reasonml
		  // For any monoid x...
		  x
		  // Right identity
		  === x.concat(M.empty())
		  
		  // Left identity
		  === M.empty().concat(x)
		  
		  
		  // So, for `MCompose` and some `f`...
		  MCompose(f)
		  
		  // Right identity
		  === MCompose(f).concat(MCompose.empty())
		  
		  // Left identity
		  === MCompose.empty().concat(MCompose(f))
		  
		  //- In other words, `of` can't disrupt the
		  //- sequence held inside `mcompose`! For
		  //- the sake of clarity, this just means:
		  
		  f.chain(M.of).chain(g) === f.chain(g)
		  f.chain(g).chain(M.of) === f.chain(g)
		  
		  ```
		    
		  And there we have it: `of` **cannot disrupt the sequence**. All it can do is put a value into an **empty** context, placing it somewhere in our sequence. No **tricks**, no **magic**, no **side-effects**.  
		    
		  So, for your most **strict** and **correct** definition, `M` is a `Monad` if you can substitute it into our `MCompose` without breaking the `Monoid` laws. **That’s it**!  
		    
		  
		  ---
		    
		  
		  > For those skipping ahead, **stop scrolling**!  
		  
		    
		  Ok, big deal, `Monad` is to `Chain` as `Monoid` is to `Semigroup`; why is everyone getting so *excited* about this, though? Well, remember how we said we could use `Chain` to define **execution order**?  
		    
		  
		  ```javascript
		  //+ getUserByName :: String -> Promise User
		  const getUserByName = name =>
		  new Promise(res => /* Some AJAX */)
		  
		  //+ getFriends :: User -> Promise [User]
		  const getFriends = user =>
		  new Promise(res => /* Some more AJAX */)
		  
		  // e.g. returns [every, person, ever]
		  getUser('Baymax').chain(getFriends)
		  
		  ```
		    
		  With this, we can define [**entire programs** using map and chain](https://web.archive.org/web/20230610043014/https://twitter.com/am%5Fi%5Ftom/status/850082511900332033)! We can do this because we can **sequence** our actions. What we get with `of` is the ability to *lift* variables into that context whenever we like!  
		    
		  
		  ```reasonml
		  const optimisedGetFriends = user
		  user.name == "Howard Moon"
		  ? Promise.of([]) // Lift into Promise
		  : getFriends(user) // Promise-returner
		  
		  ```
		    
		  We know that `getFriends` returns a `Promise`, so our speedy result needs to do the same. Luckily, we can just *lift* our speedy result into a **pure** `Promise`, and we’re **good to go**!  
		    
		  Although it may *seem* improbable, we actually now have the capability to write **any** `IO` logic we might want to write:  
		    
		  
		  ```javascript
		  const Promise = require('fantasy-promises')
		  
		  const rl =
		  require('readline').createInterface({
		    input: process.stdin,
		    output: process.stdout
		  })
		  
		  //+ prompt :: Promise String
		  const prompt = new Promise(
		  res => rl.question('>', res))
		  
		  //- We use "Unit" to mean "undefined".
		  //+ speak :: String -> Promise Unit
		  const speak = string => new Promise(
		  res => res(console.log(string)))
		  
		  //- Our entire asynchronous app!
		  //+ MyApp :: Promise String
		  const MyApp =
		  // Get the name...
		  speak('What is your name?')
		  .chain(_ => prompt)
		  .chain(name =>
		  
		    // Get the age...
		    speak('And what is your age?')
		    .chain(_ => prompt)
		    .chain(age =>
		  
		      // Do the logic...
		      age > 30
		  
		      ? speak('Seriously, ' + name + '?!')
		        .chain(_ => speak(
		          'You don\'t look a day over '
		            + (age - 10) + '!'))
		  
		      : speak('Hmm, I can believe that!'))
		  
		    // Return the name!
		    .chain(_ => Promise.of(name)))
		  
		  //- Our one little impurity:
		  
		  // We run our program with a final
		  // handler for when we're all done!
		  MyApp.fork(name => {
		  // Do some database stuff...
		  // Do some beeping and booping...
		  
		  console.log('FLATTERED ' + name)
		  rl.close() // Or whatever
		  })
		  
		  ```
		    
		  That, *beautiful* Fantasists, is (basically) an **entirely purely-functional** app. Let’s talk about a few cool things here.  
		    
		  Firstly, **every step** is `chain`ed together, so we’re **explicitly** giving the **order** in which stuff should happen.  
		    
		  Secondly, we can **nest** `chain` to get access to **previous values** in **later actions**.  
		    
		  Thirdly, `chain` means we can do **everything with arrow functions**. Every command is a *single-expression* function; it’s **super neat**! Try re-formatting this example on a bigger screen; all my examples are written for *mobile*, but this example can look far more readable with 80-character width!  
		    
		  Fourthly, following on from the *first* point, there’s **no mention** of **async** with `chain` - we specify the **order**, and `Promise.chain` does the promise-wiring **for us**! At this point, async behaviour is literally **just** an **implementation detail**.  
		    
		  Fifthly (*are these still words?*), `MyApp` - our whole program - **is a value**! It has a type `Promise String`, and we can use that `String`! What does *that* mean? **We can chain programs together**!  
		    
		  
		  ```javascript
		  //+ BigApp :: Promise Unit
		  const BigApp =
		  speak('PLAYER ONE')
		  .chain(_ => MyApp)
		  .chain(player1 =>
		  
		    speak('PLAYER TWO')
		    .chain(_ => MyApp)
		    .chain(player2 =>
		  
		      speak(player1 + ' vs ' + player2)))
		  
		  ```
		    
		  **OMGWTF**! We took our **entire** program and used it as a **value**… **twice**! As a consequence, we can just write lots of little programs and **chain** (*compose, concat, bind, whatever you want to say*) them together into **bigger ones**! Remember, too, that `Monad`s are all also `Applicatives`…  
		    
		  
		  ```javascript
		  //+ BigApp_ :: Promise Unit
		  const BigApp_ =
		  lift2(x => y => x + ' vs ' + y,
		    speak('PLAYER ONE').chain(_ => MyApp),
		    speak('PLAYER TWO').chain(_ => MyApp))
		  
		  ```
		    
		  **Oh yeah**! Our programs are now **totally composable** `Applicative`s, just like any other value. Our **entire programs**! All a functional program *really* does is collect some *little* programs together with `ap` and `chain`. It really is **that neat**!  
		    
		  
		  > Why do we use `fantasy-promises` instead of the built-in `Promise`? Our **functional** `Promise` doesn’t execute until we call `fork` - that means we can **delay** the call until we’ve **defined** its behaviour. With a built-in `Promise`, things start happening immediately, which can lead to **non-determinism** and **race conditions**. This way, we maintain **full control**!  
		  
		    
		  Of course, maybe the syntax is a bit *ugly*, but that’s what **helper functions** are for! Also, why stop at `Promise`? This fanciness works for `Maybe`, `Array`, `Either`, `Function`, `Pair`, and **so many more**!  
		    
		  Keep fiddling, using those `Task`/`Promise` isomorphisms to do things in parallel, using `Maybe` to avoid `undefined` / `null` along the way, using `Array` to return multiple choices; if you can handle all that, you’re a **fully-fledged** functional aficionado!  
		    
		  
		  ---
		    
		  You might be wondering what the *rest* is for if we now have all the tools we’ll ever need, and that’s certainly a good question. The rest are **optional**; monadic functional programming doesn’t **require** an understanding of `Comonad` or `Profunctor`, but nor does it require an understanding of `Alt` or `Traversable`; these are just **design patterns** to help our code to be as **polymorphic** as possible.  
		    
		  As always, there’s [a **Gist** for the article](https://web.archive.org/web/20230610043014/https://gist.github.com/richdouglasevans/ea96fb5fc8bb55d832a8a20f8c14d4ed), so have a **play** with it! Here’s a little idea for an exercise: write a **monadic** functional CLI app to play “higher or lower”. You know everything you need to know; **trust me**!  
		    
		  ♥
	- ### Highlights
	  collapsed:: true
		- > fantas-eel-and-specification [⤴️](https://omnivore.app/me/fantas-eel-and-specification-15-monad-tom-harding-18fa669234f#3a9eed79-21b5-41dd-9f3f-38bb914c6df7)
- [14. ChainRec](https://omnivore.app/me/fantas-eel-and-specification-14-chain-rec-tom-harding-18fa668c647)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/05/30/fantas-eel-and-specification-14/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[05/29/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/05/30/fantas-eel-and-specification-14/
		    
		  **Happy Tuesday**, Fantasists! Sorry for the wait; I’ve been chasing around [an issue to change this entry](https://web.archive.org/web/20230924043628/https://github.com/fantasyland/fantasy-land/issues/250)! No movement on that yet, so let’s **soldier on**! We’ve seen that `chain` allows us to **sequence** our actions, which means we can now do pretty much **anything we want**! As fellow JavaScripters, this is of course the time we get cynical; it can’t be *that* simple, right? **Absolutely not**, and let’s take a look at a **convoluted example** to show us why!  
		    
		  Have a gander at this little number, which emulates the behaviour of the UNIX `seq` command, with the help of [our old friend Pair](https://web.archive.org/web/20230924043628/http://www.tomharding.me/2017/04/27/pairs-as-functors/):  
		    
		  
		  ```kotlin
		  // Click the above link for a full
		  // explanation of this type!
		  const Writer = Pair(Array)
		  
		  //+ seq :: Int -> Writer [Int] Int
		  const seq = upper => {
		  
		  //+ seq_ :: Int -> Writer [Int] Int
		  const seq_ = init =>
		    init >= upper
		  
		    // If we're done, stop here!
		    ? Writer([init], upper)
		  
		    // If we're not...
		    : Writer([init], init + 1)
		      .chain(seq_) // ...chain(seq_)!
		  
		  // Kick everything off
		  return seq_(1)
		  }
		  
		  seq(5)._1 // [1, 2, 3, 4, 5]
		  
		  ```
		    
		  Everything looks **fine** so far: `seq_` is just a **recursive** function. Until `init` exceeds `upper`, we **log** the current value and call `seq_` on the **next** integer. Pick any number and see that it works: `10`, `100`, `1000`, … **oh**.  
		    
		  
		  ```perl
		  > seq(10000)._1
		  RangeError: Maximum call stack size exceeded
		  
		  ```
		    
		  Now, we’re supposedly using **computers**. They take people to the **moon**, control **nuclear reactors**, and get us **pictures of cats**; *surely* it’s not too big an ask to count to `10000` without exploding?  
		    
		  Our problem here is that, every time we `chain(seq_)`, we store the *calling* function’s local state on the **stack** until the *called* function returns; as you can imagine, it’s actually pretty expensive to save **ten thousand** of these states. When we fill up the stack and try to carry on, we cause a **stack overflow** error. *If only there were some website to help us…*  
		    
		  *Typically*, in our non-functional JavaScript, we get round this problem with a **loop**. It’s no secret that we could write `seq` as:  
		    
		  
		  ```javascript
		  const seq = upper => {
		  const acc = []
		  
		  for (let i = 1; i <= upper; i++)
		    acc.push(i)
		  
		  return acc
		  }
		  
		  seq(10000) // Bam, no trouble
		  
		  ```
		    
		  See? **No stack overflow**; we use `acc` to store our state at each point **explicitly**, without recursion, and everything just works… so, have we been **defeated**? Do we *really* have to choose between **prettiness** - purity, composition, etc. - and **performance**?  
		    
		  
		  ---
		    
		  **Never**! We’re **functional programmers**; we solve these problems with **abstraction**, and `ChainRec` turns out to be *exactly* what we need. We’re going to start off with a slightly different definition of this spec’s function, and work towards the real one. First of all, we’ll introduce a **new type**, `Step`:  
		    
		  
		  ```ada
		  // type Step b a = Done b | Loop a
		  const { Loop, Done } = daggy.taggedSum({
		  Done: ['b'], Loop: ['a']
		  })
		  
		  ```
		    
		  We’re using these two constructors to mimic the two possible states of our imperative solution: are we `Done`, or do we still need to `Loop`?  
		    
		  
		  > We *could* make this a `Functor` over `a`, but we don’t need to; still, it’s an **exercise** if you’re looking!  
		  
		    
		  With that in mind, let’s define `chainRec`. Remember here that all `ChainRec` types are also `Chain` types:  
		    
		  
		  ```livescript
		  chainRec :: ChainRec m
		         => (a -> m (Step b a), a)
		         -> m b
		  
		  ```
		    
		  Well, well, well. We start off with a function of type `a -> m (Step b a)`, and a value of type `a`. We end up with `m b`. I reckon the first step is probably to *call* that function, and end up with `m (Step b a)`. When we’ve done that, we’ll be in one of two situations:  
		    
		  
		  * That `Step b a` will be a `Done b`, and we can pull out the `b` (by mapping over the `m`) and return the answer!
		  * That `Step b a` will be a `Loop a`, and we still don’t have a `b` to make the answer. What do we do then? We **loop again**!  
		  
		  We `chain` our starter function to our `m a`, and repeat this whole process until we finally get a `Done` value, and we can **finish up**!  
		  
		  This might be a little **heavy**, so let’s implement `chainRec` for our `Writer` type to clear it up:
		   ```javascript
		  // We need a TypeRep for the left bit!
		  const Pair = T => {
		  const Pair_ = daggy.tagged('_1', '_2')
		  
		  // ...
		  
		  //+ chainRec
		  //+   :: Monoid m
		  //+   => (a -> Pair m (Step b a), a)
		  //+   -> (m, b)
		  Pair_.chainRec = function (f, init) {
		  // Start off "empty"
		  let acc = T.empty()
		  
		  // We have to loop at least once
		  , step = Loop(init)
		  
		  do {
		  // Call `f` on `Loop`'s value
		  const result = f(step.a)
		  
		  // Update the accumulator,
		  // as well as the current value
		  acc  = acc.concat(result._1)
		  step = result._2
		  } while (step instanceof Loop)
		  
		  // Pull out the `Done` value.
		  return Pair_(acc, step.b)
		  }
		  }
		  
		   ```
		  We store our `acc`, starting with the “empty” value for that `Monoid`, and update it with every loop. This continues until we hit a `Done`, when we make a pair with the final `acc` and the value inside the `Done`!  
		  
		  It might take a couple read-throughs, but see what’s happened: instead of doing **recursion**, we’ve used `Step` to let a function *tell us* when it *wants* to recurse. With that extra bit of **abstraction**, we can hide a `while` loop under the hood, and no one will be any the wiser!  
		  
		  So, with all this talk of **accumulated results**, let’s get back to that `seq` example, and see what we can do:  
		  
		  ```kotlin  
		  //+ seqRec :: Int -> ([Int], Int)  
		  const seqRec = upper => Writer.chainRec(  
		  init => Writer([init],  
		  init >= upper ? Done(init)  
		  : Loop(init + 1)),
		  
		  0)  
		    
		  
		  ```
		  
		  Look at the **body**: there’s no recursion! We’ve abstracted all the recursion into `chainRec`, which can do its **sneaky optimisation**. All _we_ say in our `chainRec`\-using function is whether we’re `Done` or still need to `Loop` \- isn’t that **clearer**? What’s more, it’s now totally **stack-safe**!
		  
		  ```
		  > seqRec(100000)._1  
		  [1, 2, 3, 4, 5, ...]  
		  
		    
		  
		  ```
		  
		  Just a tiny little extra abstraction and we **scoff** at numbers like `10000`. Why stop the momentum? Next stop: **cat pictures on the moon**.
		  
		  ---
		  
		  Now, we haven’t quite matched the spec, but you’ll see why. The actual spec gives us no `Step` type, which means we end up with the following:
		  
		  ```
		  chainRec :: ChainRec m  
		         => ((a -> c, b -> c, a) -> m c, a)  
		         -> m b  
		    
		  
		  ```
		  
		  Now, I have a few problems with this definition, most of which are outlined in [the aforementioned GitHub issue](https://web.archive.org/web/20230924043628/https://github.com/fantasyland/fantasy-land/issues/250), so we won’t go into them too much. What we’ll do _instead_ is define a function to turn our `Step`\-based functions into **spec-compliant** functions. It’s a little number I like to call `trainwreck`:
		  
		  ```
		  //+ trainwreck  
		  //+  :: Functor m  
		  //+  => (a -> m (Step b a))  
		  //+  -> ((a -> c, b -> c, a) -> m c)  
		  const trainwreck = f =>  
		  (next, done, init) => f(init).map(  
		    step => step.cata({  
		      Loop: next, Done: done  
		    })  
		  )  
		    
		  // Or, with ES6 and polymorphic names...  
		  const trainwreck = f => (Loop, Done, x) =>  
		  f(x).map(s => s.cata({ Loop, Done }))  
		    
		  
		  ```
		  
		  With this cheeky thing, we can get the best of both worlds; we just wrap our definition of `Pair_.prototype.chainRec` in the `trainwreck` function and it’s good to go! Whether you choose to implement `chainRec` on your types with the `trainwreck-Step` approach or with the spec’s approach is up to you, but I think it’s pretty clear that I have a **favourite**!
		  
		  > Interestingly, the _spec’s_ (more formal) encoding is based on the [Boehm-Berarducci](https://web.archive.org/web/20230924043628/http://okmij.org/ftp/tagless-final/course/Boehm-Berarducci.html) encoding of the `Step` type; hat-tip to [Brian McKenna](https://web.archive.org/web/20230924043628/https://twitter.com/puffnfresh) for this one, as I’d **never** heard of this!
		  
		  ---
		  
		  Last and certainly not least: the **laws**. Wait, you didn’t think I’d forgotten, did you? **Ha**! We’ll define these with `Step` to make them a bit more **expressive**, but [the original definitions](https://web.archive.org/web/20230924043628/https://github.com/fantasyland/fantasy-land\#chainrec) are of course available on the spec.
		  
		  ```
		  // For some ChainRec m, predicate p,  
		  // and some M-returning d and n...  
		  M.chainRec(  
		  trainwreck(  
		  
		  * n(v).map(Next)),  
		  i)
		  
		    
		  // Must behave the same as...  
		    
		  (function step(v) {  
		  
		  * n(v).chain(step)  
		  }(i))
		  
		    
		  
		  ```
		  
		  By now, we’ve seen the _actual_ difference: the second one will explode before we get anywhere _close_ to the moon. However, in _theory_ (assuming we had an **infinite stack**), these two expressions would always end up at the same point. Again, we’re just asserting that `Done` and `Next` provide an **abstraction** for our recursion.
		  
		  The other law, which is one of my _favourites_, is that `chainRec(f)` must produce a stack no larger than `n` times `f`’s stack usage, for some **constant** `n`. In other words, whether I’m looping once or a million times, the stack **must not keep growing**. With this wordy little law, we **ensure** stack safety.
		  
		  The `ChainRec` spec is best-suited towards ideas like [Free structures](https://web.archive.org/web/20230924043628/https://github.com/safareli/free) and [asynchronous looping](https://web.archive.org/web/20230924043628/https://github.com/fluture-js/fluture): when we could end up with a _very_ complicated structure, `chainRec` makes sure we don’t hit an unexpected ceiling.
		  
		  In your day-to-day coding, it’s probably not something you’ll see an awful lot. In fact, it usually tends to be hidden in implementation detail, rather than being exposed to the user. Stick to the simple, **golden rule**: when `Chain` blows your stack, `ChainRec` is there to solve the problem. Every `Chain` type can implement `ChainRec`, so this will _always_ be an available option!
		  
		  Whether we use it regularly or not, we now have a practical, stack-safe, and **functional** answer to the danger of `chain`ing huge computations. We can **sequence** instructions safely, build up large computations, and then **let ‘em rip**.
		  
		  Next time, we’ll look at the **last piece of the puzzle**, and charge through some examples. Brace yourself for the _terrifying_, _incomprehensible_, and _downright impolite_ `Monad`! Trust me: it’s actually **none** of those things. In fact, it’s nothing new at all - **rest easy**, Fantasists!
		  
		  ♥
		  
		  As usual, feel free to play with [the post’s code gist](https://web.archive.org/web/20230924043628/https://gist.github.com/richdouglasevans/4c0a197c3d8312961a1c7fba557f4425) to experiment further!
- [13. Chain](https://omnivore.app/me/fantas-eel-and-specification-13-chain-tom-harding-18fa6688789)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20231206142142/http://www.tomharding.me/2017/05/15/fantas-eel-and-specification-13/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[05/14/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20231206142142/http://www.tomharding.me/2017/05/15/fantas-eel-and-specification-13/
		  
		  _You told me to **leave you alone**. My Papa said, “**Come on home**”. My doctor said, “**Take it easy**”, but your lovin’ is **much too strong**. I’m added to your… `Chain`, `Chain`, `Chain`_! Maybe we didn’t compose _that_ one, but we’re going to `compose` plenty of things today!
		  
		  Let’s take a moment to recap a few previous posts. When [we covered Functor](https://web.archive.org/web/20231206142142/http://www.tomharding.me/2017/03/27/fantas-eel-and-specification-6/), we saw that functors created **contexts** in which our “language” could be **extended**:
		  
		  ```
		  a :: a -> [b]        -- Multiple results  
		  m :: a -> Maybe b    -- Possible failure  
		  e :: a -> Either e b -- Possible exception  
		  t :: a -> Task e b   -- Async action  
		    
		  
		  ```
		  
		  We’ve also seen that [Applicative functors](https://web.archive.org/web/20231206142142/http://www.tomharding.me/2017/04/10/fantas-eel-and-specification-8/)’ contexts can be **combined** to give us cool tricks like parallel `Task` actions and [Traversable](https://web.archive.org/web/20231206142142/http://www.tomharding.me/2017/05/08/fantas-eel-and-specification-12/):
		  
		  ```
		  // Just([-1, -2, -3])  
		  ;[1, 2, 3].traverse(Maybe, x => Just(-x))  
		    
		  // Logs 50 (after only 2000ms!)  
		  lift2(  
		  x => y => x + y,  
		    
		  new Task((*, res) =>
		    setTimeout(
		      () => res(20),
		      2000)),
		  
		  new Task((*, res) =>  
		    setTimeout(  
		      () => res(30),  
		      1000)))  
		  .fork(  
		  _ => console.error('???'),  
		  x => console.log(x))  
		    
		  
		  ```
		  
		  There’s one thing we _can’t_ do, though. What if we want to `compose` two `Functor`\-returning functions? Take a look at this example:
		  
		  ```
		  //+ prop :: String -> StrMap a -> Maybe a  
		  const prop = k => xs =>  
		  
		  * Nothing
		  
		    
		  const data = { a: { b: { c: 2 } } }  
		  const map = f => xs => xs.map(f)  
		    
		  // How do we get to the 2?  
		  prop('a')(data) // Just({ b: { c: 2 } })  
		  .map(prop('b')) // Just(Just({ c: 2 }))  
		  .map(map(prop('c'))) // Just(Just(Just(2)))  
		    
		  // And if we fail?  
		  prop('a')(data) // Just({ b: { c: 2 }})  
		  .map(prop('badger')) // Just(Nothing)  
		  .map(map(prop('c'))) // Just(Nothing)  
		    
		  
		  ```
		  
		  So, we **can** get to the `2`, but not without a lot of **mess**. We `map` more and more each time, when all we _really_ want is a `Just 2` if it works and a `Nothing` if it doesn’t. What we’re missing is a way to **flatten** layers of `Maybe`. Enter `join`:
		  
		  ```
		  //+ join :: Maybe (Maybe a) ~> Maybe a  
		  Maybe.prototype.join = function () {  
		  return this.cata({  
		    Just: x => x,  
		    Nothing: () => Nothing  
		  })  
		  }  
		    
		  prop('a')(data) // Just({ b: { c: 2 } })  
		  .map(prop('b')).join() // Just({ c: 2 })  
		  .map(prop('c')).join() // Just(2)  
		    
		  prop('a')(data) // Just({ b: { c: 2 } })  
		  .map(prop('badger')).join() // Nothing  
		  .map(prop('c')).join() // Nothing  
		    
		  
		  ```
		  
		  We seem to have **solved** our problem! Each time we `map` with a `Maybe`\-returning function, we call `join` to flatten the two `Maybe` layers into one. `map`, `join`, `map`, `join`, and so on. Back in [the Traversable post](https://web.archive.org/web/20231206142142/http://www.tomharding.me/2017/05/08/fantas-eel-and-specification-12/), we saw that the `map`/`sequence` pattern was common enough that we gave it an alias: `traverse`. Wouldn’t you know it, the same applies here: the `map`/`join` pattern is so common, we call it `chain`.
		  
		  ```
		  //+ chain :: Maybe a ~> (a -> Maybe b)  
		  //+                  -> Maybe b  
		  Maybe.prototype.chain = function (f) {  
		  return this.cata({  
		    Just: f,  
		    Nothing: () => this // Do nothing  
		  })  
		  }  
		    
		  // Just like `sequence` is `traverse` with  
		  // `id`,  `join` is `chain` with `id`!  
		  //+ join :: Chain m => m (m a) ~> m a  
		  const join = xs => xs.chain(x => x)  
		    
		  // Our example one more time...  
		  prop('a')(data) // Just({ b: { c: 2 } })  
		  .chain(prop('b')) // Just({ c: 2 })  
		  .chain(prop('c')) // Just(2)  
		    
		  
		  ```
		  
		  **So many parallels!** Now, this fancy little pattern is useful far beyond an occasional `Maybe`. It turns out that we can unlock a **lot** of behaviour this way:
		  
		  ```
		  //+ chain :: Either e a  
		  //+       ~> (a -> Either e b)  
		  //+       -> Either e b  
		  Either.prototype.chain = function (f) {  
		  return this.cata({  
		    Right: f,  
		    Left: _ => this // Do nothing  
		  })  
		  }  
		    
		  const sqrt = x => x < 0  
		  
		  * Right(Math.sqrt(x))
		  
		    
		  Right(16)  
		  .chain(sqrt) // Right(4)  
		  .chain(sqrt) // Right(2)  
		    
		  Right(81)  
		  .chain(sqrt)  // Right(9)  
		  .map(x => -x) // Right(-9) 😮  
		  .chain(sqrt)  // Left('Hey, no!')  
		  .map(x => -x) // Left('Hey, no!')  
		    
		  Left('eep')  
		  .chain(sqrt) // Left('eep')  
		    
		  
		  ```
		  
		  Just as `Maybe`’s `chain` would carry forward a `Nothing`, `Either`’s will carry forward the first `Left`. We can then chain a bunch of `Either`\-returning functions and get the first `Left` if anything goes wrong - just like **throwing exceptions**! With `Either`, we can model exceptions in a **completely pure way**. **Pure**. Savour this moment. We **fixed** exceptions.
		  
		  This one’s **exciting**, right? Let’s see what `Array` can do:
		  
		  ```
		  //+ chain :: Array a  
		  //+       ~> (a -> Array b)  
		  //+       -> Array b  
		  Array.prototype.chain = function (f) {  
		  // Map, then concat the results.  
		  return [].concat(... this.map(f))  
		  }  
		    
		  // NB: **totally** made up.  
		  const flights = {  
		  ATL: ['LAX', 'DFW'],  
		  ORD: ['DEN'],  
		  LAX: ['JFK', 'ATL'],  
		  DEN: ['ATL', 'ORD', 'DFW'],  
		  JFK: ['LAX', 'DEN']  
		  }  
		    
		  //- Where can I go from airport X?  
		  //+ whereNext :: String -> [String]  
		  const whereNext = x => flights[x] || []  
		    
		  // JFK, ATL  
		  whereNext('LAX')  
		    
		  // LAX, DEN, LAX, DFW  
		  .chain(whereNext)  
		    
		  // JFK, ATL, ATL, ORD, DFW, JFK, ATL  
		  .chain(whereNext)  
		    
		  
		  ```
		  
		  With `Array`, we `map` with some `Array`\-returning function, then flatten all the results into one list. `map` and `join`, just like everything else! Here, we’re essentially **traversing a graph**, and building up a list of **possible positions** after each **step**. In fact, languages like [**PureScript** use chain to model **loops**](https://web.archive.org/web/20231206142142/https://github.com/purescript/purescript/wiki/Differences-from-Haskell\#array-comprehensions)_\*_:
		  
		  ```
		  //- An ugly implementation for range.  
		  //+ range :: (Int, Int) -> [Int]  
		  const range = (from, to) =>  
		  [... Array(to - from)]  
		  .map((_, i) => i + from)  
		    
		  //- The example from that link in JS.  
		  //+ factors :: Int -> [Pair Int Int]  
		  const factors = n =>  
		  range(1, n).chain(a =>  
		    range(1, a).chain(b =>  
		      a * b !== n  
		  
		  * [ Pair(a, b) ]))
		  
		    
		  // (4, 5), (2, 10)  
		  factors(20)  
		    
		  
		  ```
		  
		  ---
		  
		  Now, let’s talk about [our Task type](https://web.archive.org/web/20231206142142/https://github.com/folktale/data.task)†. We’ve seen that, with its `Apply` implementation, we get **parallelism for free**. However, we haven’t actually talked about how to get **serial** behaviour. For example, what if we wanted to do some AJAX to get a _user_, and then use the result to get their _friends_? Well, `chain` would appear to be exactly what we need:
		  
		  ```
		  //+ getUser :: String -> Task e Int  
		  const getUser = email => new Task(  
		  (rej, res) => API.getUser(email)  
		                .map(x => x.id)  
		                .then(res)  
		                .catch(rej))  
		    
		  //+ getFriendsOf :: Int -> Task e [User]  
		  const getFriends = id => new Task(  
		  (rej, res) => API.getFriends(id)  
		                .then(res)  
		                .catch(rej))  
		    
		  // Task e [User] - hooray?  
		  getUser('test@test.com').chain(getFriends)  
		    
		  
		  ```
		  
		  It turns out this behaviour **is** defined on our `Task` type, so we can `chain` together `Task` objects to get **sequencing** when we need it, and `ap` them together for **free parallelism**. Before we get _too_ excited, though, let’s talk about what’s **wrong** here.
		  
		  If a type **lawfully** implements `Chain` (and hence `Apply` and `Functor`), we can **derive** `ap` using `chain` and `map`:
		  
		  ```
		  //+ ap :: Chain m => m a ~> m (a -> b) -> m b  
		  MyType.prototype.ap = function (fs) {  
		  return fs.chain(f => this.map(f))  
		  }  
		    
		  
		  ```
		  
		  **So what**? Well, this `ap` doesn’t behave like the `ap` we already have. Most importantly, there’s **no parallelism**! Houston, **we have a problem**. In the case of `Task`, either the `ap` or `chain` method is therefore **unlawful**.
		  
		  > There have been [similar discussions](https://web.archive.org/web/20231206142142/https://github.com/ekmett/either/pull/38\#issuecomment-95688646) that are worth reading for more background, but this point is quite an _advanced_ discussion, so brace yourself.
		  
		  This means that, to write **law-obiding** code, we need to accept that either our `ap` method is wrong, or our `chain` method shouldn’t exist. _Personally, I’ve always chosen to view `chain` as the “problem method”, and asserted that `Task` is an `Applicative` but **not** a `Chain`_. What we **should** do is **convert** our `Task` to some “sequential” type when we want to do something sequential:
		  
		  ```
		  // A "sequential" async type.  
		  const Promise = require('fantasy-promises')  
		    
		  // For "errors" within Promise.  
		  const { Left, Right }  
		  = require('fantasy-eithers')  
		    
		  //- Convert a Task to a Promise  
		  //+ taskToPromise :: Task e a  
		  //+               -> Promise (Either e a)  
		  const taskToPromise = task => Promise(  
		  res => task.fork(e => res(Left(e)),  
		                   x => res(Right(x))))  
		    
		  //+ promiseToTask :: Promise (Either e a)  
		  //+               -> Task e a  
		  const promiseToTask = promise =>  
		  new Task((rej, res) =>  
		    promise.fork(either =>  
		      either.cata({  
		        Left: rej,  
		        Right: res  
		      })  
		    )  
		  )  
		    
		  //- Finally...  
		  //+ andThen :: Task e a ~> (a -> Task e b)  
		  //+                     -> Task e b  
		  Task.prototype.andThen = function (f) {  
		  return promiseToTask(  
		    taskToPromise(this)  
		    .chain(either => either.cata({  
		      // We "lift" failure using Promise'  
		      // Applicative instance.  
		      Left: _ => Promise.of(either),  
		    
		      Right: x => taskToPromise(f(x))  
		    }))  
		  )  
		  }  
		    
		  //- ... which gives us:  
		  getUser('test@test.com')  
		  .andThen(getFriends)  
		    
		  
		  ```
		  
		  **Big and scary**, but don’t panic. Here, we’re taking advantage of the **natural transformation** between `Task` and the composition of `Promise` and `Either e` in **both** directions: we move to a **sequential** context, do something sequential, then move back to the **parallel** context.
		  
		  With `promiseToTask` and `taskToPromise`, we can convert any `Promise` into a `Task` and vice versa. This is exactly what we need in order to say that `Promise` and `Task` are **isomorphic**!
		  
		  > **Isomorphisms** come up a lot to avoid these problems: if we can “convert” a type into another type with a required **capability**, and then convert it back, it’s as good as having that capability in our **original** type! The difference is, however, that we’re not breaking the **laws**.
		  
		  Of course, you could just as easily go ahead and use `chain`, and accept that it is **badly-behaved**. That’s cool, as long as you’re **aware** of this, and know to expect some **unexpected** results. You could also write a simple implementation of `andThen`:
		  
		  ```
		  //+ No intermediate type!  
		  Task.prototype.andThen = function (f) {  
		  return new Task((rej, res) =>  
		    this.fork(rej, x =>  
		      f(x).fork(rej, res))  
		  )  
		  }  
		    
		  
		  ```
		  
		  It’s really down to you, but the _lengthier_ method means that we neatly separate our **parallel** and **sequential** types, and can know what behaviour is expected in each. `Task`’s author wrote a [blog post on async control](https://web.archive.org/web/20231206142142/http://robotlolita.me/2014/03/20/a-monad-in-practicality-controlling-time.html), which may shed more light here.
		  
		  ---
		  
		  We’ve seen what `chain` does: `map` then `join`. This is why you’ll hear it called `flatMap` sometimes: it `maps` and then “flattens” two layers of `Chain` type into one. You might also hear `concatMap` (named after `Array`’s particular implementation) or `bind`.
		  
		  We’ve also seen that we can define `ap` in terms of `chain` in a way that will work for **any** law-obiding type. _If you don’t believe me, try it with `Maybe`, `Either`, or `Array`_! This, however, isn’t the only **rule** that we have to follow. There’s one, very _familiar_, **law** that comes with this class:
		  
		  ```
		  // Associativity.  
		  m.chain(f).chain(g)  
		  === m.chain(x => f(x).chain(g))  
		    
		  // Remember Semigroups?  
		  a.concat(b).concat(c)  
		  === a.concat(b.concat(c))  
		    
		  
		  ```
		  
		  Just as `Semigroup` was associative at **value**\-level, and `Apply` was associative at **context**\-level, we can see that `Chain` seems to be associative at an **order**\-level: unlike `Apply`, which we saw would freely allow for parallelism, `Chain` **implies order**. Just look at the type of `chain`:
		  
		  ```
		  chain :: Chain m => m a  
		                 ~> (a -> m b)  
		                 -> m b  
		    
		  
		  ```
		  
		  Our `chain` function is useless until we know what the `a` in `m a` is. How can we `chain` a function on a `Promise` until the first part has completed? How can we `chain` a function on a `Maybe` until we know whether the first part failed? With `chain`, we finally have a way to **encode** order into our programs, and strictly **forbid** parallelism when we need to.
		  
		  I think it’s **amazing** that we managed to achieve all that we have so far _without_ this concept! We’re so used to order being determined by the ordering of the lines in our file, but that’s because of **state**: we can’t mix those lines up because they may depend on one another.
		  
		  With **purely-functional** code, we know that can’t be true, and that all dependencies are **explicit**. Who cares which side of an `ap` gets calculated first? Who cares whether a `map` happens now or later? Until we `fork` or `extract` a value from a `Functor`, it’s just not important - it’s an **implementation detail**!
		  
		  A note to end on: for `Semigroup`, we had `Monoid`, which brought the **empty value**. For `Apply`, we had `Applicative`, which brought the **pure context**. Seems logical that `Chain` would have a similar partner, right? We’ll get to it in a _fortnight_.
		  
		  Until then, mess around with [this post’s Gist](https://web.archive.org/web/20231206142142/https://gist.github.com/richdouglasevans/d706a914453bdcbdf0df9adf114cfcb6), and **`chain` all the things**! With **parallel** and **sequential** power, you actually now have **everything** you need to write any app in an **entirely pure** manner. Check out [the fantasy-io library](https://web.archive.org/web/20231206142142/https://github.com/fantasyland/fantasy-io) to see how we can encode **any** IO in a `Chain` type and completely **remove** the need for **state**. Go on: **I dare you**!
		  
		  ♥
		  
		  _\* `do` notation really just connects each line with a `chain` call (or `bind`, as it’s known in Haskell/PureScript). Here’s [more information on do](https://web.archive.org/web/20231206142142/https://en.wikibooks.org/wiki/Haskell/do%5Fnotation\#Just%5Fsugar), but I wouldn’t worry too much at this stage._
		  
		  _† Hat-tip to [@safareli](https://web.archive.org/web/20231206142142/https://twitter.com/safareli/), who reminded me to include this bit!_
- [12. Traversable](https://omnivore.app/me/fantas-eel-and-specification-12-traversable-tom-harding-18fa66810f1)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20231206143607/http://www.tomharding.me/2017/05/08/fantas-eel-and-specification-12/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[05/07/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20231206143607/http://www.tomharding.me/2017/05/08/fantas-eel-and-specification-12/
		  
		  It’s **`Traversable` Monday™**, everyone! Granted, _tomorrow_ would have made for a catchier opening, but I wasn’t thinking this far ahead when I picked the day to release these. Still, I bet you can’t _wait_ for `Monad`s now! Putting all that aside, does everyone remember how great the `insideOut` function from [the Applicative post](https://web.archive.org/web/20231206143607/http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/) was? Well, today’s post is all about your **new favourite typeclass**.
		  
		  This little class comes with, hands down, the most _frightening_ signature we’ve seen so far! Just _look_ at this thing:
		  
		  ```
		  traverse :: Applicative f, Traversable t  
		         => t a -> (TypeRep f, a -> f b)  
		         -> f (t b)  
		    
		  
		  ```
		  
		  _Right_? Let’s break it down. We can take a `Traversable` structure (whatever _that_ means) with an inner `a` type, use this `a -> f b` function (where `f` is some `Applicative`), and we’ll land on `f (t b)`. _Wait, **what**_?
		  
		  The little bit of **magic** here is that, if we used the `a -> f b` with `map`, we’d end up with `t (f b)`, right? The `f b` replaces the `a` in the `t a`; nothing we haven’t seen before. **However**, with `traverse`, the `f` and `t` come out **the other way round!**
		  
		  ---
		  
		  In a **shocking** development, let’s see some examples _before_ we trouble ourselves with the laws, as they’re a bit **intimidating**. Let’s say you have an `Array` of `Int` user IDs, and we need to do some **AJAX** for each one to get some details:
		  
		  ```
		  // getById :: Int -> Task e User  
		  const getById = id => // Do some AJAX  
		    
		  
		  ```
		  
		  Luckily, we have our wonderful `Task` applicative to encapsulate the AJAX requests. We map over our `Array` with `getById`, and we end up with an `Array` of `Task` objects. **Neat**! However, whatever we’re doing with them, we probably want to do it on the _entire_ `Array`, so wouldn’t it be better to combine them into a single `Task e [User]`? Well, our luck just continues, because we have our nifty little `insideOut`:
		  
		  ```
		  // insideOut :: Applicative f  
		  //           => [f a] -> f [a]  
		  const insideOut = (T, xs) => xs.reduce(  
		  (acc, x) => lift2(append, x, acc),  
		  T.of([]))  
		    
		  // paralleliseTaskArray  
		  //   :: [Int] -> Task e [User]  
		  const paralleliseTaskArray = users =>  
		  insideOut(Task, users.map(API.getById))  
		    
		  
		  ```
		  
		  First we `map`, then we `insideOut`. Thanks to `Task`’s `Applicative` instance, we’ve successfully reached our goal! On top of _that_, the wonder of `Apply` means all our AJAX will happen **in parallel**, with no extra effort!
		  
		  Well, it turns out that there’s a name for `insideOut` that encompasses more outer types than just `Array`, and we call it `sequence`! What’s more, a `map` immediately followed by `sequence` has a more common name: `traverse`!
		  
		  ```
		  Array.prototype.traverse =  
		  function (T, f) {  
		    return this.reduce(  
		      //    Here's the map bit! vvvv  
		      (acc, x) => lift2(append, f(x), acc),  
		      T.of([]))  
		  }  
		    
		  // Don't worry, though: `sequence` can also  
		  // be written as a super-simple `traverse`!  
		  const sequence = (T, xs) =>  
		  xs.traverse(T, x => x)  
		    
		  
		  ```
		  
		  So, whenever we see `map` followed by `sequence`, we can just use `traverse`. Whenever all we want to do is **flip the types**, we can use `sequence`.
		  
		  > _I usually define `sequence` **and** `traverse` on my `Traversable` types because they both get plenty of use._
		  
		  Thinking about inner types, why stop with `Task`? What if we use another `Applicative` instead? Let’s play with `Either`:
		  
		  ```
		  // toChar :: Int -> Either String Char  
		  const toChar = n => n < 0 || n > 25  
		  
		  * Right(String.fromCharCode(n + 65))
		  
		    
		  // Right(['A', 'B', 'C', 'D'])  
		  ;[0,  1,  2,  3].traverse(Either, toChar)  
		    
		  // Left('-2 is out of bounds!')  
		  ;[0, 15, 21, -2].traverse(Either, toChar)  
		    
		  
		  ```
		  
		  By using `traverse` with an inner type of `Either`, we get back the first `Left` if _any_ of the values `map` to a `Left`! In other words, we get back the **first error**. Consider, for a minute, how **incredibly useful** this is for **form validation**.
		  
		  > _What if we just want the successes, and to filter out the rest_? We `map` our function, then `map(map(x => [x]))` to get all the `Right` values into a singleton list, then `fold` the list with the `Either` **semigroup** starting with `Right([])`. I swear, I can’t get through a single post without mentioning some sort of fold!
		  
		  In fact, why stop with `Array`? We can define `Traversable` instances for all sorts of things: `Maybe`, `Either`, `Pair`, and even our `Tree` from last time! What’s the _secret_, though?
		  
		  A `Traversable` type needs to know how to **rebuild itself**. It pulls itself apart, lifts each part into the `Applicative`, and then puts itself back together. With the wonderful help of `of` and `ap`, it’s not hard to get all the pieces _into_ the `Applicative`, so the only trickery is the work on either side. _Luckily_, this is often very straightforward:
		  
		  ```
		  // Transform *2, then `map` in the _1!
		  Pair.prototype.traverse = function (*, f) {  
		  return f(this._2).map(  
		    x => Pair(this._1, x))  
		  }  
		    
		  // Keep the nothing OR map in the Just!  
		  Maybe.prototype.traverse = function (T, f) {  
		  return this.cata({  
		    Just: x => f(x).map(Maybe.Just),  
		    Nothing: () => T.of(Maybe.Nothing)  
		  })  
		  }  
		    
		  // Lift all the bits, then rebuild!  
		  Tree.prototype.traverse = function (T, f) {  
		  return this.cata({  
		    Node: (l, n, r) => lift3(  
		      l => n => r =>  
		        Tree.Node(l, n, r),  
		    
		      l.traverse(T, f),  
		      f(n),  
		      r.traverse(T, f))  
		    Leaf: () => T.of(Tree.Leaf)  
		  })  
		  }  
		    
		  
		  ```
		  
		  > _If you’ve been following the [Reader](https://web.archive.org/web/20231206143607/http://www.tomharding.me/2017/04/15/functions-as-functors/)/[Writer](https://web.archive.org/web/20231206143607/http://www.tomharding.me/2017/04/27/pairs-as-functors/)/State series, we’ll actually be taking a look at the `Pair` traversable in the finale!_
		  
		  If you can `map` and `reduce` your type (i.e. it’s a `Functor` and a `Foldable`), there’s a really good chance that it can be a `Traversable`, too. Trust me: `Task`’s `Applicative` instance is _enough_ reason to get excited about this!
		  
		  > `Task` is also a good example of a type that _isn’t_ `Traversable`, despite it having a similar structure to `Either`. What’s the difference? Well, consider a `Task` that returns a `Maybe` to denote success. If we _could_ traverse `Task`, we’d get back `Just` a successful task, or `Nothing`. See why this is impossible? We don’t _know_ whether the `Task` succeeds until we run it!
		  
		  ---
		  
		  Before you get _too_ excited and go all **super-villian** on me with your new-found **superpowers**, let’s end on **the laws**. _Brace yourselves_. We’ll start with **identity**, as it’s definitely the simplest:
		  
		  ```
		  u.traverse(F, F.of) === F.of(u)  
		    
		  
		  ```
		  
		  If we take a `Traversable` structure of type `T a`, and _traverse_ it with `F.of` (which is `a -> F a` for some `Applicative F`), we’ll get back `F (T a)`. **Map and turn inside-out**. What _this_ law says is that we’d have ended up in the same place if we’d just called `F.of` on the whole `T a` structure. Squint a little, and ignore the `TypeRep`: `U.traverse(F.of) === F.of(U)` doesn’t look a million miles from `U.map(id) === id(U)` (the identity law for `Functor`), does it?
		  
		  Next up is **naturality**, which is a bit of a mess in the spec, so let’s try to **clean it up**. Let’s imagine we have two `Applicative` types, `F` and `G`, and some function `t :: F a -> G a`, that does nothing but **change the `Applicative`**.
		  
		  > A function that transforms one `Functor` into another _without_ touching the inner value is called a **natural transformation**!
		  
		  ```
		  t(u.sequence(F)) === u.traverse(G, t)  
		    
		  
		  ```
		  
		  So, we start with some `U (F a)`\-type thing, `sequence` it, and land on `F (U a)`. We then call `t` on the result, and finally land on `G (u a)`. **Naturality** says that we could just call `t` directly in a `traverse` and end up in the same place! I read this as saying, “_A `traverse` should behave the same way regardless of the inner `Applicative`_”; it doesn’t matter whether we do the transformation **during** or **after**.
		  
		  > This law is actually _implied_ by **parametricity**, which is a topic we might cover in the future. Basically, it means that the type signature of `traverse` is restricted enough that this can’t **not** be true for any `Traversable` that follows the **other two** laws!
		  
		  Last up is **composition**, and we’re going to need to introduce a type we haven’t seen before. `Compose` is a way of combining two `Functor`s into one (and even two `Applicative`s into one!), and it goes a little something like this:
		  
		  ```
		  // Type Compose f g a = Compose (f (g a))  
		  const Compose = (F, G) => {  
		  const Compose_ = daggy.tagged('Compose', ['stack'])  
		    
		  // compose(F.of, G.of)  
		  Compose_.of = x =>  
		    Compose_(F.of(G.of(x)))  
		    
		  // Basically map(map(f))  
		  Compose_.map = function (f) {  
		    return Compose_(  
		      this.stack.map(  
		        x => x.map(f)  
		      )  
		    )  
		  }  
		    
		  // Basically lift2(ap, this, fs)  
		  Compose_.ap = function (fs) {  
		    return Compose_(  
		      this.stack  
		          .map(x => f => x.ap(f))  
		          .ap(fs.stack)  
		    )  
		  }  
		    
		  return Compose_  
		  }  
		    
		  
		  ```
		  
		  We’re not going to spend too much time on this type, so have a **couple of looks** to make sure you’re following. The key point here is that we’ve stacked two `Applicative`s to form a **composite** `Applicative` \- how **amazing** is that? Even more excitingly, this rule generalises to **any number** of nested `Applicative`s - they compose **completely mechanically**!
		  
		  > `Compose` will _also_ be an important part of the upcoming `State` post, so don’t think we’ve seen the last of it!
		  
		  Anyway, let’s get back to the point of introducing this type _here_. We’ll use `F` and `G` as our `Applicative` placeholders again, and end up with the law below. [The spec](https://web.archive.org/web/20231206143607/https://github.com/fantasyland/fantasy-land\#traversable) only uses `traverse`, but this should make things a bit clearer:
		  
		  ```
		  const Comp = Compose(F, G)  
		    
		  // The type signature helps, I think:  
		  // t (F (G a)) -> Compose F G (t a)  
		  u.map(Comp).sequence(Comp) ===  
		  Comp(u.sequence(F)  
		        .map(x => x.sequence(G)))  
		    
		  
		  ```
		  
		  This one’s the **ugliest** of the bunch! We start with some `u` of type `t (F (G a))`, where `t` is `Traversable`. On the **left-hand** side, we `map` over this with `Comp` and land on `t (Compose F G a)`. Because `Compose F G` is an `Applicative`, we can turn this **inside out** and land on `Compose F G (t a)`. Whew!
		  
		  The **right-hand** side says we can `sequence` the `t (F (G a))` to get us to `F (t (G a))`, then `map` a `sequence` over it to get us to `F (G (t a))`. If we pass this into `Comp`, we land on `Compose F G (t a)`, and we **must land** on the **same result** as the left-hand side did!
		  
		  This is a _really_ dense law, but **just remember this**: we can either `Compose` the `Applicative`s **inside** the `Traversable` and _then_ `sequence`, or `sequence` and _then_ `Compose`. **It shouldn’t matter**. I read this as a kind of **re-inforcement** of **naturality**; not only should the `Traversable` leave the `Applicative` alone, but it should **respect `Applicative` composition**.
		  
		  ---
		  
		  `Traversable` has some _thorough_ laws, but this is a **good thing**! The tighter the **restrictions**, the better we can **optimise** the instances we write: we’ve already had a _taste_ of the power behind the `Traversable` / `Applicative` relationship! _However_, there’s a problem that we still haven’t addressed:
		  
		  We know we can do **parallel** computation with our `Applicative`, but how do we do **serial** computation? How do we **compose** functions that return `Functor`s? These questions, and others, will be solved **next week** when we cover `chain`.
		  
		  Until then, `traverse` all the things! A Gist of this post’s [motivating examples](https://web.archive.org/web/20231206143607/https://gist.github.com/i-am-tom/0fa536e81787df1c94a55f1b4c92e94e) is available. `Array` will, by far, be the one you use the most, but don’t be afraid to experiment. I look forward to your creations!
		  
		  ♥
- [[p2p-hackers] part 2: proxying and introduction: the two fundamental operations of emergent networks](https://omnivore.app/me/p-2-p-hackers-part-2-proxying-and-introduction-the-two-fundament-18fa41cbe86)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/000296.html)
  author:: unknown
  date-saved:: [[05/23/2024]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/000296.html
		  
		  **zooko at zooko.com** [zooko at zooko.com](https://web.archive.org/web/20140618094030/mailto:p2p-hackers%40zgp.org?Subject=%5Bp2p-hackers%5D%20part%202%3A%20proxying%20and%20introduction%3A%20the%20two%20fundamental%20operations%20of%20emergent%20networks&In-Reply-To= "[p2p-hackers] part 2: proxying and introduction: the two fundamental operations of emergent networks")  
		  _Sun Sep 16 09:05:01 UTC 2001_ 
		  * Previous message: [\[p2p-hackers\] BitTorrent now integrates into Internet Explorer!](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/000295.html)
		  * Next message: [\[p2p-hackers\] part 2: proxying and introduction: the two fundamental operations of emergent networks ](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/000297.html)
		  * **Messages sorted by:** [\[ date \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/date.html\#296) [\[ thread \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/thread.html\#296) [\[ subject \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/subject.html\#296) [\[ author \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/author.html\#296)
		  
		  ---
		  
		  About two weeks ago I posted a message entitled "proxying and introduction: the
		  two fundamental operations of emergent networks":
		  
		  [http://zgp.org/pipermail/p2p-hackers/2001-August/000258.html](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-August/000258.html)
		  
		  
		  an anonymous fan wrote:
		  >> _This is great stuff, Zooko.  It has really got me thinking.  ...too bad that the_
		  > _following thread was about MN more than how the resulting emergent network_
		  > _behaves._
		  
		  Thanks!
		  
		  Yes, I was sort of hoping that Oskar or someone would pick up the implicit
		  challenge.  See, AFAICT, Freenet and most other networks have focussed
		  exclusively on proxying and neglected introduction.
		  
		  There is a sound theoretical reason for concentrating on proxying, as Oskar can
		  lucidly explain: that with introduction alone, and no proxying, the size of
		  your horizon is proportional to the size of your local state (i.e. you have to
		  remember everyone's id and address, or whatever, in order to use them).
		  Therefore introduction is "non-scalable" in Oskar's opinion.
		  
		  The counterargument to that is that introduction is *required*.  A new node is
		  created and it is not connected to anyone in the network.  How does it get
		  connected?  That is the problem of Original Introduction.  People who neglect
		  introduction end up with some kind of kludge to do Original Introduction (e.g.
		  node-lists on HTTP, the MN Meta Trackers, or manually configuring your node to
		  connect to other nodes), and with no transitive introduction at all.
		  
		  But that gives a central point of failure (e.g. you can take out the HTTP
		  server that newcomers depend on in order to connect to the network), or at
		  least passes the buck for having a robust, emergent introduction service off to
		  the HTTP or the manual configuration or whatever.
		  
		  So a really good emergent network needs both: proxying *and* introduction.
		  
		  
		  IMO a really good emergent network is going to have:
		  
		  1. good Transitive Introduction
		  
		  2. good Original Introduction, which should utilize the features of the
		    Transitive Introduction -- instead of being a wholly separate behavior with
		    different properties
		  2.a. easy-for-users Original Introduction such as "type in the DNS or IP of
		    your friend who is already connected to the Emergent Net", or "scan local
		    net / local wireless area for Emergent Net nodes"
		  2.b. an easily accessible default original introduction service i.e. a set of
		    redundant original introducer nodes like the MN Meta Trackers
		  
		  3. Proxying for superlinear effective horizon
		  
		  
		  But in the immediate term, I don't really care about number 3: proxying of
		  operations (although I do care about relaying of messages) until the number of
		  nodes on my network times the amount of local space it takes to carry on a
		  relationship is approaching the amount of local space that I am willing to
		  allocate on each node.  Quick back-of-the-envelope calculations go like this:
		  
		  128 bytes for addressing information (could do with less, but let's be a bit
		  pessimistic), 128 bytes for crypto information (could do with less, again), and
		  let's just say 256 bytes for local reputation (i.e. what you know about that
		  other node, how it performs, etc.) for a nice round number of 512 bytes per
		  counterparty.  (Note: if you really wanted to squeeze I believe you could get
		  this down to 20 bytes for crypto and addressing, and maybe 20 bytes for local
		  reputation for a total of 40 bytes per counterparty, but let's be pessimistic
		  while writing on the back of this particular envelope...)
		  
		  So if users are willing to allocate up to 128 MB of local persistent storage
		  for maintaining their relationships in the network, then each node can have
		  direct relationships with at least 2^18 == 262,144 other nodes.
		  
		  By the time this 2^18 limit is actually hurting my network, Freenet v0.5 will
		  be out, and I can learn from all of the applied research that Freenet has done
		  in effective proxying techniques and then add those proxying techniques into my
		  network.  Of course, it's also possible that harddrives will be bigger by then
		  and users will be willing to allocate more than 128 MB just for peer
		  relationships.
		  
		  
		  Now my purpose here is definitely not to criticize proxying as such!  Proxying
		  is very important for a lot of reasons.  For one thing, if you don't have
		  proxying then local network usage is *also* proportional to the effective
		  network horizon of a given operation (although there of several things that can
		  ameliorate that problem including multicast) and for another thing, smaller
		  devices with tighter storage constraints are more likely to need proxying.
		  Another reason is that there may be some theoretical or engineering benefits
		  to combining message-relay with higher-layer operations (as Freenet does) as
		  opposed to making them separate abstraction layers (as Mojo Nation does).
		  Finally, as Lucas Gonze pointed out in private e-mail, proxying allows for more
		  complex relationships, for example maybe you don't *want* to introduce your two
		  friends to each other, because you don't want them to be able to exchange
		  information without giving you access to it.
		  
		  
		  So my purpose here is *not* to denigrate proxying, but to draw attention to the
		  important of introduction.  Here is a quick recap of the reasons why
		  introduction is vitally important to emergent networks:
		  
		  1. Introduction is required, in order to have a network at all in the first
		   place.
		  2. You can do non-transitive Original Introduction, but then it must be
		   centralized and/or manually managed by humans.  (Which is okay for a lot of
		   applications.)
		  3. If you are going to implement transitive, automatic Introduction in order to
		   have robust, automatic joining of the network, then why not use it?  e.g.:
		  a. You already use it to go from 1 neighbors to K neighbors (where K is the
		   minimum number of neighbors that you need to be part of the network), then
		   why not use it to go from K neighbors to M neighbors, where M is a higher
		   number for greater efficiency in some cases.
		  b. Use automatic transitive introduction to dynamically heal and optimize the
		   network.
		  4. Introduction may make for more efficient networks than proxying in some
		   cases (those cases where higher degree of connectivity is better).
		  a. One such case seems to be when the total number of nodes on the entire
		   network is sufficiently small, which the current state of Mojo Nation.
		  
		  
		  Regards,
		  
		  Zooko
		  
		  P.S.  Thanks again, for those who didn't read my earlier message, to Oskar,
		  Adam Langley, and Bram for getting me thinking about this last year, and to
		  Mark Miller for teaching me the ways of Granovetterism (== Introductionism).
		  
		  P.P.S.  I don't know that much about Freenet, and it probably already has some
		  transitive introduction features, but the Freenet people have not discussed it
		  in terms of "Proxying and Introduction: The Two Fundamental Network Operations"
		  before now as far as I know.
		  
		  
		  ---
		  
		  * Previous message: [\[p2p-hackers\] BitTorrent now integrates into Internet Explorer!](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/000295.html)
		  * Next message: [\[p2p-hackers\] part 2: proxying and introduction: the two fundamental operations of emergent networks ](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/000297.html)
		  * **Messages sorted by:** [\[ date \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/date.html\#296) [\[ thread \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/thread.html\#296) [\[ subject \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/subject.html\#296) [\[ author \]](https://web.archive.org/web/20140618094030/http://zgp.org/pipermail/p2p-hackers/2001-September/author.html\#296)
		  
		  ---
		  
		  [More information about the P2p-hackers mailing list](https://web.archive.org/web/20140618094030/http://zgp.org/cgi-bin/mailman/listinfo/p2p-hackers)
- [11. Foldable](https://omnivore.app/me/fantas-eel-and-specification-11-foldable-tom-harding-18fa667d836)
  site:: [web.archive.org](https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/05/01/fantas-eel-and-specification-11/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[04/30/2017]]
	- ### Content
		- The Wayback Machine - https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/05/01/fantas-eel-and-specification-11/
		  
		  Welcome back, Fantasists! This week has been **hectic**, so I haven’t caught up the [companion repository](https://web.archive.org/web/20231206151632/http://github.com/i-am-tom/fantas-eel-and-specification) as I’d hoped to. However, I should have some time to devote to it this week, so watch this space! Anyway, why don’t we have some **down time** before we get onto the **really grizzly** parts of the spec? Let’s take a look at `Foldable`.
		  
		  Wouldn’t you know it, this one comes with a **new function** to add to our repertoire, and it’s one that might look a bit _familiar_ to some readers:
		  
		  ```
		  reduce :: Foldable f =>  
		    f a ~> ((b, a) -> b, b) -> b  
		    
		  
		  ```
		  
		  Do we already know of any `Foldable` types? I’ll suggest one:
		  
		  ```
		  Array.prototype.reduce ::  
		    [a] ~> ((b, a) -> b, b) -> b  
		    
		  
		  ```
		  
		  Straight away, `Array` is a valid `Foldable` type. In fact, Fantasy Land’s `Foldable` was _deliberately_ modelled after `Array`. Why? Because this structure **generalises** `Array.reduce`. **Bam**. With that in mind, let’s look at its law:
		  
		  ```
		  const toArray xs => xs.reduce(  
		    (acc, x) => acc.concat([x]), []  
		  )  
		    
		  u.reduce(f) === toArray(u).reduce(f)  
		    
		  
		  ```
		  
		  This is a **really** weird law because it’s not very… rigorous. You’re unlikely to write a `Foldable` and get this wrong, because the `reduce` in `toArray` probably works in the exact same way as the `reduce` outside. In fact, it’s probably best to look at this more as a **behaviour** than a **law**. _You still have to obey it, though!_
		  
		  There’s a _lot_ of space for interpretation here, which isn’t necessarily a good thing. **On the other hand**, it makes it really easy to give some examples!
		  
		  ```
		  // reduce :: Maybe a  
		  //        ~> ((b, a) -> b, b) -> b  
		  Maybe.prototype.reduce =  
		    function (f, acc) {  
		      return this.cata({  
		        // Call the function...  
		        Just: x => f(acc, x),  
		    
		        // ... or don't!  
		        Nothing: () => acc  
		      })  
		    }  
		    
		  // reduce :: Either e a  
		  //        ~> ((b, a) -> b, b) -> b  
		  Either.prototype.reduce =  
		    function (f, acc) {  
		      return this.cata({  
		        // Call the function...  
		        Right: x => f(acc, x),  
		    
		        // Or don't!  
		        Left: _ => acc  
		      })  
		    }  
		    
		  
		  ```
		  
		  Because `Nothing` and `Left` represent **failure**, we just return the accumulator. Otherwise, we can use our `f` function once. These examples really just highlight that you don’t _need_ a structure with (potentially) multiple values in order to write a `Foldable` instance. However, it’s definitely **most useful** when that’s the case. For example, let’s build a **binary tree**:
		  
		  ```
		  // BTree a  
		  const BTree = daggy.taggedSum('BTree', {  
		    // Recursion!  
		    // Node (BTree a) a (BTree a)  
		    Node: ['left', 'x', 'right'],  
		    
		    // Leaf  
		    Leaf: []  
		  })  
		    
		  
		  ```
		  
		  So, `Node`s represent “branch points” of the tree with values, and `Leaf`s represent the ends of branches. Because `left` and `right` are also `BTree` instances, this gives us a recursive tree construction:
		  
		  ```
		  const { Node, Leaf } = BTree  
		    
		  //      3  
		  //     / \  
		  //    1   5  
		  //   /|   |\  
		  //  X 2   4 X  
		  //   /|   |\  
		  //  X X   X X  
		  const MyTree =  
		    Node(  
		      Node(  
		        Leaf,  
		        1,  
		        Node(  
		          Leaf,  
		          2,  
		          Leaf ) ),  
		      3,  
		      Node(  
		        Node(  
		          Leaf,  
		          4,  
		          Leaf ),  
		        5,  
		        Leaf ) )  
		    
		  
		  ```
		  
		  _I’m too ashamed to say how long I spent drawing that tree._ Now, `Array.reduce` combines all our values together (_if your [Semigroup](https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/03/13/fantas-eel-and-specification-4/) or [Monoid](https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/03/21/fantas-eel-and-specification-5/) klaxon just sounded, then I’m super proud ♥_) from left to right, so that’s probably what we want to imitate with this tree. How do we do that? With **magical recursion**:
		  
		  ```
		  BTree.prototype.reduce =  
		    function (f, acc) {  
		      return this.cata({  
		        Node: (l, x, r) => {  
		          // Reduce the tree on the left...  
		          const left = l.reduce(f, acc)  
		    
		          // Plus the middle element...  
		          const leftAndMiddle = f(left, x)  
		    
		          // And then the right tree...  
		          return r.reduce(f, leftAndMiddle)  
		        },  
		    
		        // Return what we started with!  
		        Leaf: () => acc  
		      })  
		    }  
		    
		  MyTree.reduce((x, y) => x + y, 0) // 15  
		    
		  
		  ```
		  
		  **Woo**! We reduce the tree starting from the **left-most** element and work across. Whenever we hit a `Node`, we just **recurse** and do the same thing!
		  
		  We saw at the end of the last snippet that we can find the **sum** of all the elements, and we could just as easily write `min`, `max`, `product`, or whatever. In fact, [you can do anything with Array.reduce](https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/02/24/reductio-and-abstract-em/) and generalise it to all `Foldable` types immediately! Whether we then have a `Maybe`, an `Array`, or even a `BTree`, our functions will **Just Work™**!
		  
		  ---
		  
		  `Product`? `Min`? `Max`? This really does sound like `Monoid` again, doesn’t it? Well, there are **no coincidences** in Fantasy Land. Back in [the Monoid post](https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/03/21/fantas-eel-and-specification-5/), we wrote the `fold` function:
		  
		  ```
		  // A friendly neighbourhood monoid fold.  
		  // fold :: Monoid m => (a -> m) -> [a] -> m  
		  const fold = M => xs => xs.reduce(  
		    (acc, x) => acc.concat(M(x)),  
		    M.empty())  
		    
		  
		  ```
		  
		  With our new-found `Foldable` knowledge, we now know that this type signature is **too specific**. Let’s fix it:
		  
		  ```
		  fold :: (Foldable f, Monoid m)  
		       => (a -> m) -> f a -> m  
		    
		  
		  ```
		  
		  Yes, this function will in fact work with **any** `Foldable` structure and **any** `Monoid` \- you’ll never need to write `reduce` again! Get comfortable with [using Monoid for reductions](https://web.archive.org/web/20231206151632/https://joneshf.github.io/programming/2015/12/31/Comonads-Monoids-and-Trees.html); it’s definitely a good way to make your code more **declarative**, and hence more **readable**. I know you’re probably _sick_ of hearing about monoids, but they really are **everywhere**!
		  
		  ---
		  
		  It turns out that _many_ of your favourite `Functor` types have sensible implementations for `reduce`. However, there are **exceptions**:
		  
		  We can’t `reduce` [the Task type](https://web.archive.org/web/20231206151632/https://github.com/folktale/data.task/) because we don’t know what the inner values are going to be! It’s the same with `Function`: we don’t know what the return value is going to be until we give it an **argument**. Remember: **functors’ inner values aren’t always reachable**.
		  
		  > Incidentally, if a `Functor` _does_ have an always-reachable inner value, we can call it [a Copointed functor](https://web.archive.org/web/20231206151632/https://hackage.haskell.org/package/pointed-5/docs/Data-Copointed.html). Remember how [Applicative’s of is a function of Pointed](https://web.archive.org/web/20231206151632/http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/) functors? Think about the relationship between `Pointed` and `Copointed`. There are **no coincidences** in Fantasy Land!
		  
		  ---
		  
		  This week, why not make a `Foldable` **Rose Tree**? Or **Set**? There are plenty of opportunities to practise here. Before we go, some of you may have noticed that you can `reduce` most things by just returning the **initial value** given to you:
		  
		  ```
		  MyType.prototype.reduce = (f, acc) => acc  
		    
		  
		  ```
		  
		  It satisfies the “law”, right? This is what I mean: this law is **not a good’un**, and leaves too much room for interpretation. Still, there’s no point in _moaning_: we’ll see next time that `Traversable` (my **all-time favourite** part of the Fantasy Land spec!) saves the day! From now on, Fantasists, the excitement is **non-stop**.
		  
		  ♥
- [10. ALT, Plus, and Alternative](https://omnivore.app/me/fantas-eel-and-specification-10-alt-plus-and-alternative-tom-har-18fa666fde5)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20240209071417/http://www.tomharding.me/2017/04/24/fantas-eel-and-specification-10/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[04/23/2017]]
	- ### Content
		- The Wayback Machine - https://web.archive.org/web/20240209071417/http://www.tomharding.me/2017/04/24/fantas-eel-and-specification-10/
		  
		  \#\# Fantas, Eel, and Specification 10: Alt, Plus, and Alternative
		  
		  We’re in **double digits**! Isn’t this exciting? It also means that, by my estimations, we’re **well over half way**! Before we get _too_ excited by `Profunctor` and `Comonad`, though, might I tempt you with an… `Alternative`?
		  
		  Today, we’re going to bundle together **three** very well-related entries in the spec, starting with `Alt`. This little typeclass has one function:
		  
		  ```
		  alt :: Alt f => f a ~> f a -> f a  
		    
		  
		  ```
		  
		  As we know, with great algebraic structure, come great **laws**:
		  
		  ```
		  // Associativity  
		  a.alt(b).alt(c) === a.alt(b.alt(c))  
		    
		  // Distributivity  
		  a.alt(b).map(f) === a.map(f).alt(b.map(f))  
		    
		  
		  ```
		  
		  The **associativity** law is _exactly_ what we saw in [the Semigroup post](https://web.archive.org/web/20240209071417/http://www.tomharding.me/2017/03/13/fantas-eel-and-specification-4/) with `concat`! As we said back then, think of it as, “_Keeping left-to-right order the same, you can combine the elements however you like_”.
		  
		  **Distributivity** gives us another clue that we might be looking at something a bit `Semigroup`\-flavoured. We can **`map` first** over the elements and then `alt` **or** **`alt` first** and then `map` over the result, and we’ll end up at the **same value**. Either way, some kind of “combination” definitely seems to be going on here.
		  
		  So, **what is it**? Well… it’s like a **semigroup for functors**. It’s a way of combining values of a functor type _without_ the requirement that the inner type be a `Semigroup`. _Why_, you ask? Well, we get a little hint from the name.
		  
		  `Alt` allows us to provide some _alternative_ value as a “fallback” when the first “fails”. Of course, this is particularly relevant to types with some notion of **failure**:
		  
		  ```
		  Maybe.prototype.alt = function (that) {  
		    this.cata({  
		      Just: _ => this,  
		      Nothing: _ => that  
		    })  
		  }  
		    
		  
		  ```
		  
		  If we have a `Just` on the left, we return it. Otherwise, we **fall back** to the second value! Naturally, you can chain as many of these as you like:
		  
		  ```
		  // Just(3) - note the "Nothing"s are  
		  // usually the result of some functions.  
		  Nothing.alt(Nothing).alt(Just(3))  
		    
		  
		  ```
		  
		  It turns out there are **loads** of use cases for `alt`, which isn’t too surprising if you look at it as a **functor-level `if/else`**. You can do [database connection failover](https://web.archive.org/web/20240209071417/https://gist.github.com/i-am-tom/9651cd1e95443c4cbf3953429e988b07), [API/resource routing](https://web.archive.org/web/20240209071417/https://github.com/slamdata/purescript-routing/blame/master/GUIDE.md\#L96-L102), and, most magically of all, [text parsing](https://web.archive.org/web/20240209071417/https://github.com/purescript/purescript/blob/master/src/Language/PureScript/Parser/Declarations.hs\#L161-L169). _Those last two were in PureScript and Haskell respectively, but don’t worry: in these languages, `alt` has an operator, written as `<|>`._
		  
		  The key thing all these cases have in common is that you want to _try_ something with a contingency plan for _failure_. That’s all there is to `Alt`!
		  
		  ---
		  
		  If `Alt` will be our functor-level `Semigroup`, what’s our **functor-level `Monoid`**? In comes `Plus`, which extends `Alt` with one more function called `zero`:
		  
		  ```
		  zero :: Plus f => () -> f a  
		    
		  
		  ```
		  
		  Looks a bit like `Monoid`’s `empty`, right? Note that there’s no restriction on the `a`, so this `zero` value must work for **any type**. This one has **three laws**, but the first two will look really familiar to readers of [the Monoid post](https://web.archive.org/web/20240209071417/http://www.tomharding.me/2017/03/21/fantas-eel-and-specification-5/):
		  
		  ```
		  // Right identity - zero on the right  
		  x.alt(A.zero()) === x  
		    
		  // Guess what this one's called?  
		  A.zero().alt(x) === x  
		    
		  // The new one: annihilation  
		  A.zero().map(f) === A.zero()  
		    
		  
		  ```
		  
		  The left and right **identity** laws just say, “_`zero` makes no difference to the other value, regardless of which side of `alt` you put it_”. **Annihilation** gives us a stronger idea of what `zero` does: **nothing**! `Plus` types _must_ be functors; for a `map` call to do _nothing_ in all cases, the type must have the ability to be **empty**, whatever that means.
		  
		  Think of our `Maybe` type: what can we `map` over with _any_ function and not change the value? `Nothing`! In fact, `() => Nothing` is the **only valid** implementation of `zero` for `Maybe`.
		  
		  What about `Array`? Well, `map` transforms every value in the array, so the only array that _wouldn’t_ be changed is the empty one. `() => []` is the **only valid** implementation of `zero` for `Array`.
		  
		  > We didn’t cover `Array` as an `Alt` because it’s a bit of a funny one. Back when we discussed [functors](https://web.archive.org/web/20240209071417/http://www.tomharding.me/2017/03/27/fantas-eel-and-specification-6/), we saw that `Array` _extends_ our language to allow us to represent **several values** at once. This can be thought of as **non-determinism** if we see an `Array` as the set of **possible values**. Thus, the `alt` implementation for `Array` is the same as `concat` \- all we’re doing is combining the two sets of possibilities!
		  
		  So, `Plus` adds to `Alt` what `Monoid` adds to `Semigroup`, and, in fact, what `Applicative` adds to `Apply`: an **identity**. Are we bored of this pattern yet? I hope not, because we’re _still_ not done with it! Incidentally, we can write custom `Semigroup` and `Monoid` types to encapsulate this behaviour so we can reuse the functions we talked about in their posts:
		  
		  ```
		  // The value MUST be an Alt-implementer.  
		  const Alt = daggy.tagged('Alt', ['value'])  
		    
		  // Alt is a valid semigroup!  
		  Alt.prototype.concat = function (that) {  
		    return Alt(this.value.alt(that.value))  
		  }  
		    
		  // The value MUST be a Plus-implementer.  
		  // And, as usual, we need a TypeRep...  
		  const Plus = T => {  
		    const Plus_ = daggy.tagged('Plus', ['value'])  
		    
		    // Plus is a valid semigroup...  
		    Plus_.prototype.concat =  
		      function (that) {  
		        return Plus(  
		          this.value.alt(  
		            that.value))  
		      }  
		    
		    // ... and a valid monoid!  
		    Plus_.empty = () => Plus_(T.zero())  
		  }  
		    
		  
		  ```
		  
		  Monoids are **everywhere**, I tell you. Stare at something long enough and it’ll start to look like a monoid.
		  
		  ---
		  
		  The final boss level on this `Alt` quest is `Alternative`. There are **no special functions** for this one, as it is simply the name for a structure that implements both `Plus` _and_ `Applicative`. Still, I know how much you _love_ laws:
		  
		  ```
		  // Distributivity  
		  x.ap(f.alt(g)) === x.ap(f).alt(x.ap(g))  
		    
		  // Annihilation  
		  x.ap(A.zero()) === A.zero()  
		    
		  
		  ```
		  
		  **Distributivity** is exactly as the same law that we saw with `Alt` and `map` at the beginning of all this, but now for `ap`. We can either **`alt` first** and _then_ `ap` the result to `x`, **or** we can **`ap` first** to both separately, and then `alt`. Either way, we end up in the same place.
		  
		  **Annihilation** is a _really_ scary word for a not-so-scary idea, if you think back to the `zero` values we discussed earlier. You couldn’t apply a value to `Nothing`, right? Or an empty list of functions? The **annihilation** law defines this behaviour: if you try to do **something with nothing**, you get **nothing**. Whatever you were doing is considered a _failure_, and `zero` is returned.
		  
		  You’ll often hear `Alternative` types described as **monoid-shaped applicatives**, and this is a good intuition. We talked about `of` as being the **identity** of `Applicative`, but this is only at **context-level**. For an `Alternative` type, `zero` is the identity value at context- **and** value-level.
		  
		  ---
		  
		  `Maybe`, `Array`, `Task`, `Either`: we’ve seen a lot of types that can very naturally implement `Alternative`. You could even make `Function` an `Alternative` if you knew the output would be of a `Plus`\-implementing type. With that, you could then write a function whose body can do **extra computation** depending on the result; who needs `if/else`?
		  
		  That’s about all there is to it! `Alt`, `Plus`, and `Alternative` are **under-appreciated** typeclasses, particularly within functional JavaScript. Take some time to look through your code, glare at the `if/else`, `try/catch`, and `switch` blocks, and see whether they’re really just `alt`s in disguise!
		  
		  Next time, we’ll be looking into your **new favourite** typeclasses: `Foldable` and `Traversable`. Try to contain your excitement until then!
		  
		  Take care ♥
- [9. Applicative](https://omnivore.app/me/fantas-eel-and-specification-9-applicative-tom-harding-18fa662353b)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20240103060406/http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[04/16/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20240103060406/http://www.tomharding.me/2017/04/17/fantas-eel-and-specification-9/
		  
		  _I asked my **German** friend whether any of this series’ posts particularly stood out. They said **9**, so I’d better make this a good one!_ I told you we were doing jokes now, right? Moving on… _Today_, we’re going to finish up a topic we started last week and move from our `Apply` types to `Applicative`. If you understood [the Apply post](https://web.archive.org/web/20240103060406/http://www.tomharding.me/2017/04/10/fantas-eel-and-specification-8/), this one is _hopefully_ going to be pretty intuitive. **Hooray**!
		  
		  `Applicative` types are `Apply` types with one extra function, which we define in [Fantasy Land](https://web.archive.org/web/20240103060406/https://github.com/fantasyland/fantasy-land\#applicative) as `of`:
		  
		  ```
		  of :: Applicative f => a -> f a  
		    
		  
		  ```
		  
		  With `of`, we can take a value, and **lift** it into the given `Applicative`. That’s it! In the wild, _most_ `Apply` types you practically use will also be `Applicative`, but we’ll go through a counter-example later on! Anyway, wouldn’t you know, there are a few **laws** to go with it, tying `ap` and `of` together:
		  
		  ```
		  // For some applicative A...  
		    
		  // Identity.  
		  v.ap(A.of(x => x)) === v  
		    
		  // Homomorphism  
		  A.of(x).ap(A.of(f)) === A.of(f(x))  
		    
		  // Interchange  
		  A.of(y).ap(u) === u.ap(A.of(f => f(y)))  
		    
		  
		  ```
		  
		  With **identity**, we’ve lifted the identity function (`x => x`) into our **context**, applied it to the inner value, and, surprise, **nothing happened**.
		  
		  The **homomorphism** law says that we can lift a function and its argument **separately** and _then_ combine them, **or** combine them and _then_ lift the result. Either way, we’ll end up with the same answer! The `of` function can do _nothing_ but put the value into the `Applicative`. No tricks. **No side effects**.
		  
		  **Interchange** is a little more complex. We can lift `y` and apply it to the function in `u`, _or_ we can lift `f => f(y)`, and apply `u` to that. I think of this one as, “_Nothing special happens to one particular side of `ap` \- it just applies the value in the right side to the value in the left_”.
		  
		  ---
		  
		  So, why is this `of` thing _useful_? Well, the most exciting reason is that we can write functions generic to **any applicative**. Let’s write a function that takes a **list** of **applicative-wrapped** values, and returns an **applicative-wrapped list** of values. Note that `T` is a `TypeRep` variable - we’ve seen these before in [the monoid post](https://web.archive.org/web/20240103060406/http://www.tomharding.me/2017/03/21/fantas-eel-and-specification-5/). We need it in case the list is empty:
		  
		  ```
		  // append :: a -> [a] -> [a]  
		  const append = y => xs => xs.concat([y])  
		    
		  // There's that sneaky lift2 again!  
		  // lift2 :: Applicative f  
		  //       => (a -> b -> c, f a, f b)  
		  //       -> f c  
		  const lift2 = (f, a, b) => b.ap(a.map(f))  
		    
		  // insideOut :: Applicative f  
		  //           => [f a] -> f [a]  
		  const insideOut = (T, xs) => xs.reduce(  
		    (acc, x) => lift2(append, x, acc),  
		    T.of([])) // To start us off!  
		    
		  // For example...  
		    
		  // Just [2, 10, 3]  
		  insideOut(Maybe, [ Just(2)  
		                   , Just(10)  
		                   , Just(3) ])  
		    
		  // Nothing  
		  insideOut(Maybe, [ Just(2)  
		                   , Nothing  
		                   , Just(3) ])  
		    
		  
		  ```
		  
		  **First of all**, we lift an empty list into the applicative context. Then, value by value, we **combine** the contexts with a function to `append` the value to that inner list. _Neat_, right? We’ll see in a few weeks that this `insideOut` can be generalised further to become a super-helpful function called [sequenceA](https://web.archive.org/web/20240103060406/http://hackage.haskell.org/package/base-4.9.1.0/docs/Data-Traversable.html\#v:sequenceA) that works on a lot more than just lists!
		  
		  > This is pretty useful for AJAX, too. We can use our `data.task` applicative to create a list of requests - a `[Task e a]` \- and then combine them into a single `Task` containing the eventual results - a `Task e [a]` \- using `insideOut`!
		  
		  Notice that we only _need_ an `Applicative` because the list _could_ be empty. Otherwise, we could just `map` over the first with `x => [x]` and use that one as the accumulator - we’d only need `Apply`! _Anyone else having [flashbacks to monoids](https://web.archive.org/web/20240103060406/http://www.tomharding.me/2017/03/13/fantas-eel-and-specification-5/)_? They’re all **very strongly-connected**:
		  
		  * `concat` can combine any non-zero number of **values** (of the same type) into one.
		  * `ap` can combine any non-zero number of **contexts** (of the same type) into one.
		  * If we might need to handle zero **values**, we will need to use `empty`.
		  * If we might need to handle zero **contexts**, we will need to use `of`.
		  
		  Wizardry. **For our next trick**, let’s notice that you can turn _any_ `Applicative` into a valid `Monoid` if the inner type is a `Monoid`:
		  
		  ```
		  // Note: we need a TypeRep to get empty!  
		  const MyApplicative = T => {  
		    // Whatever your instance is...  
		    // const MyApp = daggy.tagged('MyApp', ['x'])  
		    // Put your map/ap/of here...  
		    
		    // concat :: Semigroup a => MyApp a  
		    //                       ~> MyApp a  
		    //                       -> MyApp a  
		    MyApp.prototype.concat =  
		      function (that) {  
		        return lift2((x, y) => x.concat(y),  
		                     this, that)  
		      }  
		    
		    // empty :: Monoid a => () -> MyApp a  
		    MyApp.prototype.empty =  
		      () => MyApp.of(T.empty())  
		    
		    return MyApp  
		  }  
		    
		  
		  ```
		  
		  The above will _always_ be **valid**. Notice that it doesn’t care about the shape of our `Applicative` at all - we can write these implementations using just the **interface** (`map`, `ap`, and `of`) for _any_ applicative. Of course, `Applicative` types _can_ use different implementations for `Monoid`. For example, look at `Maybe`:
		  
		  ```
		  // Usual implementation:  
		  Just([2]).concat(Just([3])) // Just([2, 3])  
		  Just([2]).concat(Nothing)   // Just([2])  
		  Nothing.concat(Just([3]))   // Just([3])  
		  Nothing.concat(Nothing)     // Nothing  
		    
		  Maybe.empty = () => Nothing  
		    
		  // With the above implementation:  
		  Just([2]).concat(Just([3])) // Just([2, 3])  
		  Just([2]).concat(Nothing)   // Nothing  
		  Nothing.concat(Just([3]))   // Nothing  
		  Nothing.concat(Nothing)     // Nothing  
		    
		  Maybe.empty = () => Just(  
		    MyInnerType.empty())  
		    
		  
		  ```
		  
		  A type _might_ have more than one implementation for any given typeclass (such as `Semigroup`); the choice is up to the **implementer** and the **users**! As we can see, `Maybe`’s usual implementation probably works better. Particularly the `empty` bit.
		  
		  Still, if we have an `Applicative` type, we know **for sure** that have at least _one_ valid `Monoid` definition! **Magical**, _right_?
		  
		  ---
		  
		  All the `Apply` types we’ve seen so far have, coincidentally, been `Applicative`. We can see it’s pretty easy in many cases to make an `of`:
		  
		  ```
		  Array.of = x => [x]  
		  Either.of = x => Right(x)  
		  Function.of = x => _ => x  
		  Maybe.of = x => Just(x)  
		  Task.of = x => new Task((_, res) => res(x))  
		    
		  
		  ```
		  
		  We’re really just constructing the simplest possible value within a type that can hold a value for us. No tricks, nothing fancy. Even `Task`, if you remember our `Promise` analogy, is about as routine as we could make it. However, let’s talk about **pairs**:
		  
		  ```
		  // Pair :: (l, r) -> Pair l r  
		  const Pair = daggy.tagged('Pair', ['x', 'y'])  
		    
		  // Map over the RIGHT side. The functor  
		  // is `Pair l` and `r` is the inner type.  
		  // map :: Pair l r ~> (r -> s)  
		  //                 -> Pair l s  
		  Pair.prototype.map = function (f) {  
		    return Pair(this.x, f(this.y))  
		  }  
		    
		  // Apply this to that, retain this' left.  
		  // ap :: Pair l r ~> Pair l (r -> s)  
		  //                -> Pair l s  
		  Pair.prototype.ap = function (that) {  
		    return Pair(this.x, that.y(this.y))  
		  }  
		    
		  // But wait...  
		  // of :: r -> Pair l r  
		  Pair.of = function (x) {  
		    return Pair({WHAT GOES HERE}, x)  
		  }  
		    
		  
		  ```
		  
		  … _Ah_. Look at the signature for `of` \- there’s a magical `l` value that just appears! We don’t know what type it is, so we don’t know what it can do, which means we can’t find a value to fill this gap. Sure, we can `ap` \- because we have two `l` values to choose from! - but we can’t `of`.
		  
		  How do we solve this? With the same _magical_ pattern that has been going on throughout this post: we require `l` to be a `Monoid`. If `l` is a `Monoid`, we know we can call `l.empty()` to get a value for that gap!
		  
		  ```
		  // TypeRep!  
		  const Pair = T => {  
		    Pair_ = daggy.tagged('Pair', ['x', 'y'])  
		    
		    // And now we're fine! Hooray!  
		    Pair_.of = x => Pair_(T.empty(), x)  
		    
		    return Pair_  
		  }  
		    
		  Array.empty = () => []  
		    
		  // SUCCESS!  
		  const MyPair = Pair(Array)  
		  MyPair.of(2) // Pair([], 2)  
		    
		  
		  ```
		  
		  > So, an `Apply` is an `Applicative` without `of`. An `Applicative` without `ap` also has a name: [Pointed](https://web.archive.org/web/20240103060406/https://hackage.haskell.org/package/pointed-5/docs/Data-Pointed.html). However, there are **no laws** attached to it independently, so it’s not particularly useful on its own - just a bit of trivia!
		  
		  ---
		  
		  That, good people of The Internet, is _all_ there is to `Applicative`. Most of this was covered in the last post, so there are hopefully no great surprises. The important take away is that the relationship between `Semigroup` and `Monoid` is _very_ similar to that of `Apply` and `Applicative`. This isn’t the last time we’ll see such a relationship, either! Isn’t it weird how everything’s **connected**? Baffles me, at least.
		  
		  Anyway, I hope this has been useful, and, as always, [I’m available on twitter](https://web.archive.org/web/20240103060406/http://twitter.com/am%5Fi%5Ftom) to answer any questions you might have. Next time, we’ll be tackling `Alt`. Don’t worry: it’s going to be pretty familiar-looking if you’ve been through all the posts in this series. Until then, though, go forth and `Apply`!
		  
		  Thank you _so much_ for reading, and take care ♥
- [8. Apply](https://omnivore.app/me/fantas-eel-and-specification-8-apply-tom-harding-18fa6664fff)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20240103060404/http://www.tomharding.me/2017/04/10/fantas-eel-and-specification-8/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[04/09/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20240103060404/http://www.tomharding.me/2017/04/10/fantas-eel-and-specification-8/
		  
		  Aaand we’re back - hello, everyone! Today, we’re going to take another look at those mystical [Functor types](https://web.archive.org/web/20240103060404/http://www.tomharding.me/2017/03/27/fantas-eel-and-specification-6/). We said a couple weeks ago that functors encapsulate a little world (**context**) with some sort of _language extension_. Well, what happens **when worlds collide**? Let’s talk about `Apply`.
		  
		  All `Apply` types are `Functor` types by requirement, so we know they’re definitely “containers” for other types. The exciting new feature here is this `ap` function:
		  
		  ```
		  ap :: Apply f => f a ~> f (a -> b) -> f b  
		  --                 a ->   (a -> b) ->   b  
		    
		  
		  ```
		  
		  If we ignore the `f`s, we get the second line, which is our basic **function application**: we _apply_ a value of type `a` to a function of type `a -> b`, and we get a value of type `b`. _Woo!_ What’s the difference with `ap`? All those bits are wrapped in the **context** of our `f` functor!
		  
		  That’s the, uh, “grand reveal”. _Ta-da_. I’m not really sure that, in _isolation_, this particularly helps our **intuition**, though, so let’s _instead_ look at `ap` within the `lift2` function:
		  
		  ```
		  // Remember: `f` MUST be curried!  
		  // lift2 :: Applicative f  
		  //       =>  (a ->   b ->   c)  
		  //       -> f a -> f b -> f c  
		  const lift2 = f => a => b =>  
		    b.ap(a.map(f))  
		    
		  
		  ```
		  
		  For _me_, this is **much** clearer. `lift2` lets us **combine** two **separate** wrapped values into one with a given function.
		  
		  > `lift1`, if you think about it, is just `a.map(f)`. The `lift2` pattern actually works for any number of arguments; once you finish the article, why not try to write `lift3`? Or `lift4`?
		  
		  _Wait, **combine**? Do I sense a_ [Semigroup](https://web.archive.org/web/20240103060404/http://www.tomharding.me/2017/03/13/fantas-eel-and-specification-4/)?
		  
		  Sort of! You can think of it this way: a `Semigroup` type allows us to merge **values**. An `Apply` type allows us to merge **contexts**. _Neat_, huh? Now, how could we forget the **laws**?!
		  
		  ```
		  // compose :: (b -> c) -> (a -> b) -> a -> c  
		  const compose = f => g => x => f(g(x))  
		    
		  // COMPOSITION LAW  
		  x.ap(g.ap(f.map(compose))) === x.ap(g).ap(f)  
		    
		  // But, if we write lift3...  
		  const lift3 =  
		    f => a => b => c =>  
		      c.ap(b.ap(a.map(f)))  
		    
		  // Now our law looks like this!  
		  lift3(compose)(f)(g)(x) === x.ap(g).ap(f)  
		    
		  // Remember: x.map(compose(f)(g))  
		  //       === x.map(g).map(f)  
		    
		  
		  ```
		  
		  By introducing some little helper functions, our law seems much clearer, and a little more familiar. It says that, just as `map` could **only** apply a function to the wrapped value, `ap` can **only** apply a wrapped function to the wrapped value. **No magic tricks!**
		  
		  Before we go any further, I challenge you take a moment to _try_ to build `lift2` without `ap`. Just think about _why_ we couldn’t do this with a plain old `Functor`. If we tried to write `lift2`, we might end up here:
		  
		  ```
		  // lift2F :: Functor f  
		  //        => (  a ->   b ->      c)  
		  //        ->  f a -> f b -> f (f c)  
		  const lift2F = f => as => bs =>  
		    as.map(a => bs.map(b => f(a)(b)))  
		    
		  
		  ```
		  
		  So, we can apply the inner values to our function - _hooray!_ \- but look at the **type** here. We’re doing a `map` inside a `map`, so we’ve ended up with **two levels** of our functor type! It’s clear that we can’t write a **generic** `lift2` to work with _any_ `Functor`, and `ap` is what’s missing.
		  
		  ---
		  
		  With all that out the way, let’s look at some examples, shall we? We’ll start with the [Identity type’s ap](https://web.archive.org/web/20240103060404/https://github.com/fantasyland/fantasy-land/blob/master/internal/id.js\#L42-L44) from our beloved spec:
		  
		  ```
		  const Identity = daggy.tagged('Identity', ['x'])  
		    
		  // map :: Identity a ~> (a -> b)  
		  //                   -> Identity b  
		  Identity.prototype.map = function (f) {  
		    return new Identity(f(this.x))  
		  }  
		    
		  // ap :: Identity a ~> Identity (a -> b)  
		  //                  -> Identity b  
		  Identity.prototype.ap = function (b) {  
		    return new Identity(b.x(this.x))  
		  }  
		    
		  // Identity(5)  
		  lift2(x => y => x + y)  
		       (Identity(2))  
		       (Identity(3))  
		    
		  
		  ```
		  
		  No frills, no magic. `Identity.ap` takes the function from `b`, the value from `this`, and returns the **wrapped-up** result. _Did you spot the similarity in **type** between `map` and `ap`, by the way_? Moving on, here’s the slightly more complex implementation for `Array`:
		  
		  ```
		  // Our implementation of ap.  
		  // ap :: Array a ~> Array (a -> b) -> Array b  
		  Array.prototype.ap = function (fs) {  
		    return [].concat(... fs.map(  
		      f => this.map(f)  
		    ))  
		  }  
		    
		  // 3 x 0 elements  
		  // []  
		  [2, 3, 4].ap([])  
		    
		  // 3 x 1 elements  
		  // [ '2!', '3!', '4!' ]  
		  [2, 3, 4]  
		  .ap([x => x + '!'])  
		    
		  // 3 x 2 elements  
		  // [ '2!', '3!', '4!'  
		  // , '2?', '3?', '4?' ]  
		  [2, 3, 4]  
		  .ap([ x => x + '!'  
		      , x => x + '?' ])  
		    
		  
		  ```
		  
		  I’ve put a little note with the answers so we can see what’s happening: we get **every** `a` and `b` pair. This is called the **cartesian product** of the two arrays. On top of that, when we `lift2` an `f` over two `Array` types, we’re actually doing something _quite familiar_…
		  
		  ```
		  return lift2(x => y => x + y)(array1)(array2)  
		    
		  // ... is the same as...  
		    
		  const result = []  
		    
		  for (x in array1)  
		    for (y in array2)  
		      result.push(x + y)  
		    
		  return result  
		    
		  
		  ```
		  
		  We get a really pretty shorthand for **multi-dimensional loops**. Flattening a **loop within a loop** gives us every possible pair of elements, and that’s what `ap` is for! If this feels weird, just **think of the types**. We have to use `Array (a -> b)` and `Array a` to get to `Array b` without violating the **composition** law; there aren’t many possibilities!
		  
		  ---
		  
		  There are loads of types with `ap` instances. Most, we’ll see, implement `ap` in terms of `chain`; we’ll look at the `Chain` spec in a week or two, so don’t worry too much. Most of them are fairly intuitive anyway:
		  
		  * `Maybe` combines **possible failures**. If either of the two `Maybe` values are `Nothing`, the result is `Nothing`.
		  
		  ```
		  Just(2).ap(Just(x => -x)) // Just(-2)  
		  Nothing.ap(Just(x => -x)) // Nothing  
		  Just(2).ap(Nothing)       // Nothing  
		  Nothing.ap(Nothing)       // Nothing  
		    
		  
		  ```
		  
		  * `Either` combines **possible failures with exceptions**. If either of the two are `Left`, the result is the first `Left`.
		  
		  ```
		  Right(2)    .ap(Right(x => -x)) // Right(-2)  
		  Left('halp').ap(Right(x => -x)) // Left('halp')  
		  Right(2)    .ap(Left('eek'))    // Left('eek')  
		  Left('halp').ap(Left('eek'))    // Left('eek')  
		    
		  
		  ```
		  
		  At some point, I’d like to write a follow up to [the Functor post](https://web.archive.org/web/20240103060404/http://www.tomharding.me/2016/12/31/yippee-ki-yay-other-functors/) to give some more _practical_ examples, but, for now, this is hopefully understandable (_please [tweet me](https://web.archive.org/web/20240103060404/http://twitter.com/am%5Fi%5Ftom) if I’m wrong!_). Whatever your `Functor` trickery, **`ap` is `map` with a wrapped function**. Before we go, though, I’d like to talk about _one last trick_ up `Apply`’s sleeve…
		  
		  A type we _haven’t_ talked about before is `Task`. This is similar to `Either` \- it represents either an error **or** a value - but the difference is that `Task`’s value is the result of a possibly-**asynchronous** computation. They look a _lot_ like `Promise` types:
		  
		  ```
		  const Task = require('data.task')  
		    
		  // Convert a fetch promise to a Task.  
		  const getJSON = url => new Task((rej, res) =>  
		    fetch(url).then(res).catch(rej))  
		    
		  
		  ```
		  
		  We can see that it holds a **function** that will eventually call a resolver. `Task`, just like `Promise`, sorts out all the async wiring for us. However, an interesting feature of `Task` is that it implements `Apply`. Let’s take a look:
		  
		  ```
		  const renderPage = users => posts =>  
		    /* Write some HTML with this data... */  
		    
		  // A Promise of a web page.  
		  // page :: Task e HTML  
		  const page =  
		    lift2(renderPage)  
		         (getJSON('/users'))  
		         (getJSON('/posts'))  
		    
		  
		  ```
		  
		  Just as we’d expect: we get the two results, and **combine** them into one using `renderPage` as the “glue”. _However_, we can see that `lift2`’s second and third arguments have **no dependencies** on one another. Because of this, the arguments to `lift2` can always be calculated **in parallel**. Do you hear _that_? These AJAX requests are **automatically parallelised**! Ooer!
		  
		  You can see [Task.ap’s implementation](https://web.archive.org/web/20240103060404/https://github.com/folktale/data.task/blob/master/lib/task.js\#L131-L183) for an exact explanation, but isn’t this _great_? We can **abstract** parallelism and never have to worry about it! When we have two parallel `Task`s and finally want to glue them back together, we just use `lift2`! Parallelism becomes an **implementation detail**. _Out of sight, out of mind_!
		  
		  ---
		  
		  I think `Task` gives a really **strong** case for `Apply` and why it’s immediately useful. When we look at `Traversable` in a few weeks, we’ll come back to `ap` and see just how _powerful_ it is. Until then, don’t overthink `ap` \- it’s just a mechanism for **combining contexts** (worlds!) together without unwrapping them.
		  
		  I had originally intended to mention `of` in this post and cover the full `Applicative`. However, it’s already quite a long post, so I’ll write up that post some time this week! I might even throw in some bigger _practical_ examples for good measure.
		  
		  If you’re still with me, **hooray**! I hope that wasn’t _too_ full-on. We’re definitely wading in **deeper waters** now, getting to the more advanced parts of the spec. All the more reason to keep [asking questions](https://web.archive.org/web/20240103060404/http://twitter.com/am%5Fi%5Ftom), though! I want to make this as clear as possible, so don’t hesitate to get in touch.
		  
		  For now until we talk about `Applicative`, though, it’s goodbye from me! Keep at it, `Apply` yourself (_zing - this blog has jokes now!_), and take care ♥
	- ### Highlights
	  collapsed:: true
		- > fantas-eel-and-specification [⤴️](https://omnivore.app/me/fantas-eel-and-specification-8-apply-tom-harding-18fa6664fff#dc9da986-ffd9-4a82-8de1-80c90b5ad8b9)
- [7. Contravariabt](https://omnivore.app/me/fantas-eel-and-specification-7-contravariant-tom-harding-18fa66599e6)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20231206151556/http://www.tomharding.me/2017/04/03/fantas-eel-and-specification-7/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[04/02/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20231206151556/http://www.tomharding.me/2017/04/03/fantas-eel-and-specification-7/
		  
		  Well, well, well. We’re a fair few weeks into this - I hope this is all still making sense! In the last article, we talked about **functors**, and how they’re really just containers to provide “language extensions” (or **contexts**). Well, today, we’re going to talk about another _kind_ of functor that looks… _ooky spooky_:
		  
		  ```
		  -- Functor  
		  map :: f a ~> (a -> b) -> f b  
		    
		  -- Contravariant  
		  contramap :: f a ~> (b -> a) -> f b  
		    
		  
		  ```
		  
		  It’s no typo: the arrow is **the wrong way round**. This _blew my mind_. It _looks_ like `contramap` can somehow magically work out how to _undo_ a function. Imagine the possibilities:
		  
		  ```
		  // f :: String -> Int  
		  const f = x => x.length  
		    
		  // ['Hello', 'world']  
		  ;['Hello', 'world'].map(f).contramap(f)  
		    
		  
		  ```
		  
		  **Impossible**, I hear you say? Well…
		  
		  … _Yes_, you’re right this time. `Array` isn’t a `Contravariant` functor - just a normal `Functor` (which we can more specifically call a **covariant** functor). In fact, there’s **no magic** here at all if we look at the examples of functors that _are_ potential `Contravariant` instances. Try this one for size:
		  
		  ```
		  // type Predicate a = a -> Bool  
		  // The `a` is the *INPUT* to the function!  
		  const Predicate = daggy.tagged('Predicate', ['f'])  
		    
		  // Make a Predicate that runs `f` to get  
		  // from `b` to `a`, then uses the original  
		  // Predicate function!  
		  // contramap :: Predicate a ~> (b -> a)  
		  //                          -> Predicate b  
		  Predicate.prototype.contramap =  
		    function (f) {  
		      return Predicate(  
		        x => this.f(f(x))  
		      )  
		    }  
		    
		  // isEven :: Predicate Int  
		  const isEven = Predicate(x => x % 2 === 0)  
		    
		  // Take a string, run .length, then isEven.  
		  // lengthIsEven :: Predicate String  
		  const lengthIsEven =  
		    isEven.contramap(x => x.length)  
		    
		  
		  ```
		  
		  The `Predicate a` is saying that, _if I can get from my type_ `a` _to_ `Bool`_, and you can get from your type_ `b` _to_ `a`_, I can get from_ `b` _to_ `Bool` _**via**_ `a`_, and hence give you a_ `Predicate b`! Sorry if that bit takes a couple of read-throughs; in short, `lengthIsEven` converts a `String` to an `Int`, then to a `Bool`. We don’t _care_ that there’s an `Int` somewhere in the pipeline - we just care about what the input value has to be.
		  
		  All smoke and mirrors, right? There’s no magical `undo` here; it’s just that we’re adding our actions to the _beginning_ of a mapping. Trust me: you’re not as **heartbroken** as I was.
		  
		  Still, if we can see _past_ this _betrayal_, we’ll of course see that there are some cool things going on here. First of all, our **laws** are basically just the same as `Functor`, but a little bit upside down:
		  
		  ```
		  // Identity  
		  U.contramap(x => x) === U  
		    
		  // Composition  
		  U.contramap(f).contramap(g)  
		    === U.contramap(x => f(g(x)))  
		    
		  
		  ```
		  
		  Identity is the same because “doing nothing” is still “doing nothing” if you do it backwards. _Probably not a sentence that’ll win me awards_. Composition is pretty much the same, but the functions are composed **the other way round**!
		  
		  A _lot_ \- probably the overwhelming majority - of `Contravariant` examples in the wild will be mappings _to_ specific types. Imagine a `ToString` type:
		  
		  ```
		  // type ToString a :: a -> String  
		  const ToString = daggy.tagged('ToString', ['f'])  
		    
		  // Add a pre-processor to the pipeline.  
		  ToString.prototype.contramap =  
		    function (f) {  
		      return ToString(  
		        x => this.f(f(x))  
		      )  
		    }  
		    
		  // Convert an int to a string.  
		  // intToString :: ToString Int  
		  const intToString =  
		    ToString(x => 'int(' + x + ')')  
		    .contramap(x => x | 0) // Optional  
		    
		  // Convert an array of strings to a string.  
		  // stringArrayToString :: ToString [String]  
		  const stringArrayToString =  
		    ToString(x => '[ ' + x + ' ]')  
		    .contramap(x => x.join(', '))  
		    
		  // Given a ToString instance for a type,  
		  // convert an array of a type to a string.  
		  // arrayToString :: ToString a  
		  //               -> ToString [a]  
		  const arrayToString = t =>  
		    stringArrayToString  
		    .contramap(x => x.map(t.f))  
		    
		  // Convert an integer array to a string.  
		  // intsToString :: ToString [Int]  
		  const intsToString =  
		    arrayToString(intToString)  
		    
		  // Aaand they compose! 2D int array:  
		  // matrixToString :: ToString Int  
		  const matrixToString =  
		    arrayToString(intsToString)  
		    
		  // "[ [ int(1), int(2), int(3) ] ]"  
		  matrixToString.f(1, 3, 4)  
		    
		  
		  ```
		  
		  It’s pretty clear to see how this approach could be used to develop a **serializer**: you could output **JSON**, **XML**, or even your own **new format**! It’s also a _great_ example of the beauty of **composition**: with functions like `arrayToString`, we’re using smaller `ToString` instances to make instances for other, more _complex_ types!
		  
		  Another good example that’s worth a look is the `Equivalence` type:
		  
		  ```
		  // type Equivalence a = a -> a -> Bool  
		  // `a` is the type of *BOTH INPUTS*!  
		  const Equivalence = daggy.tagged('Equivalence', ['f'])  
		    
		  // Add a pre-processor for the variables.  
		  Equivalence.prototype.contramap =  
		    function (g) {  
		      return Equivalence(  
		        (x, y) => this.f(g(x), g(y))  
		      )  
		    }  
		    
		  // Do a case-insensitive equivalence check.  
		  // searchCheck :: Equivalence String  
		  const searchCheck =  
		    
		    // Basic equivalence  
		    Equivalence((x, y) => x === y)  
		    
		    // Remove symbols  
		    .contramap(x => x.replace(/W+/, ''))  
		    
		    // Lowercase alpha  
		    .contramap(x => x.toLowerCase())  
		    
		  // And some tests...  
		  searchCheck.f('Hello', 'HEllO!') // true  
		  searchCheck.f('world', 'werld')  // false  
		    
		  
		  ```
		  
		  So, we’re saying we can compare anything that works with `===`, and we can therefore compare values of any type as long as they can be _converted_ to something that works with `===`. For `searchCheck`, this is really neat - we can supply **steps** for making a value comparable, for transforming _single_ values, and the `Contravariant` instance will compare the inputs after being transformed accordingly. **Hooray**!
		  
		  > If you fancy an exercise, why not play around with an `Equivalence` using our [Setoid comparison](https://web.archive.org/web/20231206151556/http://www.tomharding.me/2017/03/09/fantas-eel-and-specification-3/) \- perhaps a starter function of `(x, y) => x.equals(y)`? This should give a **lot** more control when comparing complex types.
		  
		  ---
		  
		  Well, that’s about it! There’s not much to `Contravariant` types, and they’re relatively rare. However, they’re a really good way of making your code more **expressive** (or **self-documenting**, or whatever we call it at the moment):
		  
		  ```
		  filter :: Predicate   a -> [a] ->  [a]  
		  group  :: Equivalence a -> [a] -> a  
		  sort   :: Comparison  a -> [a] ->  [a]  
		  unique :: Equivalence a -> [a] ->  [a]  
		    
		  
		  ```
		  
		  I’ll leave it to you to write the `Comparison` type and its `contramap` \- it’ll look quite a lot like `Equivalence` \- but you see that these type signatures make it _really_ clear what the functions are probably going to do.
		  
		  If all else fails, just remember:
		  
		  * When `f` is a (**covariant**) `Functor`, `f a` says, “_If you can give me an_ `(a -> b)`_, I can give you a_ `Functor b`”.
		  * When `f` is a `Contravariant` functor, `f a` says, “_If you can give me a_ `(b -> a)`_, I can give you a_ `Contravariant b`”.
		  
		  It’s the same - it’s just backwards. There’s sadly no way we could write `contramap` for an array, but do think about why we also couldn’t write a `map` for `Predicate` \- some things just aren’t meant to be! _Sigh_.
		  
		  **One last thing** before you go: many of these types are [monoids](https://web.archive.org/web/20231206151556/http://www.tomharding.me/2017/03/21/fantas-eel-and-specification-5/). See? _Everything’s connected_:
		  
		  ```
		  // It's like a function to our `All` monoid!  
		  Predicate.prototype.empty = () =>  
		    Predicate(_ => true)  
		    
		  Predicate.prototype.concat =  
		    function (that) {  
		      return Predicate(x => this.f(x)  
		                         && that.f(x))  
		    }  
		    
		  // The possibilities, they are endless  
		  Equivalence.prototype.empty = () =>  
		    Equivalence((x, y) => true)  
		    
		  Equivalence.prototype.concat =  
		    function (that) {  
		      return Equivalence(  
		        (x, y) => this.f(x, y)  
		               && that.f(x, y))  
		    }  
		    
		  
		  ```
		  
		  How about that? We can combine various `Predicate` and `Equivalence` instances of the same type to make new instances!
		  
		  > Imagine a **search** tool with options for search criteria and strictness, with each one represented as an `Equivalence` structure. When the user makes a selection, we just combine the selected structures, and we have our **purpose-built** search utility!
		  
		  Something to explore! Next time, we’ll talk about `Apply` (and probably `Applicative`) - my **second favourite** typeclass (after `Comonad` \- we’ll get to _that_ one in a few more weeks!) I hope you’re all well, and hopefully learning a thing or two along the way. See you in a week!
		  
		  Take care ♥
		-
		-
- [6. Functor](https://omnivore.app/me/fantas-eel-and-specification-6-functor-tom-harding-18fa665328f)
  collapsed:: true
  site:: [web.archive.org](https://web.archive.org/web/20231230183557/http://www.tomharding.me/2017/03/27/fantas-eel-and-specification-6/)
  author:: unknown
  labels:: [[fantas-eel-and-specification]] [[fantasyland]] [[functional programming]]
  date-saved:: [[05/23/2024]]
  date-published:: [[03/26/2017]]
	- ### Content
	  collapsed:: true
		- The Wayback Machine - https://web.archive.org/web/20231230183557/http://www.tomharding.me/2017/03/27/fantas-eel-and-specification-6/
		  
		  Fantasy Landers, **assemble**! We’ve been `concat`enating for two weeks now; are you ready for something a bit different? Well, **good news**! If you’re humming, “_Oh won’t you take me… to functor town?_”, then this is the article for you. Today, friends, we’re going to talk about **functors**.
		  
		  Yes, so, at the very end of last year, I wrote an article about functors that went over several examples of `Functor` types. It’s a collection of examples of how these can be useful _in practice_, but it’s a bit light on **intuition**. Let’s right that wrong today!
		  
		  I’d recommend giving the above a quick read, as this post won’t spend much time going over what was said back then. tl;dr, `Functor` is just another **typeclass** (a structure just like `Semigroup` or `Monoid`) with one special method:
		  
		  ```
		  -- Any functor must have a `map` method:  
		  map :: Functor f => f a ~> (a -> b) -> f b  
		    
		  
		  ```
		  
		  … and (_surprise!_) a couple of **laws**. For _any_ functor value `u`, the following must be true:
		  
		  ```
		  // Identity  
		  u.map(x => x) === u  
		    
		  // Composition:  
		  u.map(f).map(g) === u.map(x => g(f(x)))  
		    
		  
		  ```
		  
		  They might not be _immediately_ clear, and the definitions in the other article are a bit wordy, but have a couple reads. In _my_ head, I tend to think to myself, _a call to `map` must return an identical **outer** structure, with the **inner** value(s) transformed._
		  
		  ---
		  
		  If this looks a bit _weird_ to you, then it’s probably because `Functor` is the **first** entry we’ve seen in Fantasy Land that _must_ **contain** some other type.
		  
		  Think about `String`. It has actually ticked all the boxes up until now - it’s a `Setoid`, `Semigroup`, _and_ a `Monoid` \- but we can’t make it a `Functor`. This is because `String` is just, well, strings!
		  
		  `Array a` has also ticked all the boxes so far, but it _is_ a `Functor`. This is because it _contains_ a value of type `a` (or, usually, several of them). Strings are just strings - you can’t have a “`String` _of_ a thing” in the same way as you can have an “`Array` _of_ a thing” or a “`Maybe` _of_ a thing”.
		  
		  **_Functor types are containers. Not all container types are functors._** Want an example? Let’s look at `Set a` \- a collection of _unique_ values of some type `a`. There’s a catch here, though - `a` must be a `Setoid`. Looking at the type signature for `map`, we see no `Setoid` restrictions - we could `map` our `Set a` values to some not-a-`Setoid` type `b` (like `Function`), and we’d have no way of ensuring uniqueness!
		  
		  A functor **does not care** about the type it’s holding. `Set`, on the other hand, _must_. Thus, `Set` can’t be a `Functor`. Don’t worry, though - there are plenty of [types that _are_ functors](https://web.archive.org/web/20231230183557/http://www.tomharding.me/2016/12/31/yippee-ki-yay-other-functors/)!
		  
		  ---
		  
		  Ok, everything has hopefully just about made sense up to here. Now’s when we get a bit… **hand-wavey**. Sorry in advance. We’ve been over the _laws_, and we’ve seen some examples. What’s the **intuition**, though? If all these wildly different types of things are functors, what behaviour do they all _share_?
		  
		  Imagine a world _without_ functors. All we have are basic values: `Int`, `String`, `Number`, that sort of thing. No `Array`, though. No `Set`, either, as you’d _internally_ need an array of values. No nullable values _at all_ (they’re not type-safe). No `Function`, even! We’d have an **extremely limited** set of tools and would probably throw our computers out the window before too long. But, when we start introducing `Functor` types:
		  
		  | Type       | Capability                           |
		  | ---------- | ------------------------------------ |
		  | a          | Represent a value.                   |
		  | Maybe a    | Represent a **possibly null** value. |
		  | Either e a | Represent a value **or exception**.  |
		  | Array a    | Represent **a number of values**.    |
		  | x \-> a    | Represent a **mapping to values**.   |
		  | …          | …                                    |
		  
		  A functor is like a little “world” in which our boring, functor-less language has been _extended_ in some (hopefully useful!) way. When we put a value into one of these worlds, we say that we’re **lifting** the value _into_ the functor.
		  
		  **More than one extension**? What if we want to represent _a number of mappings to values_? We make an `Array` of `Function`s! What about a _mapping to a possibly null value_? We write a `Function` that maps to `Maybe`s! If we want to combine these capabilities, we simply **nest** them.
		  
		  > We’ll see that we’re not always so lucky with composition-by-nesting when we get on to more complex structures!
		  
		  We’re there. **That’s what a functor does**. It provides some **extended behaviour**, with total, gorgeous **type safety**. Note that our language isn’t _changed_ inside a functor type - just _extended_. This is why the **functor laws** hold. Let’s write a couple of little proofs:
		  
		  ```
		  // For ANY functor *constructor* U:  
		  // e.g. [x].map(f) === [f(x)]  
		  U(x).map(f) === U(f(x))  
		    
		  // Read: `map` just applies the function to  
		  // the inner value. Using only this rule,  
		  // we win!  
		    
		  const id = x => x  
		    
		  // Identity...  
		  U(x).map(id)  
		    
		    === U(id(x))  
		    
		    === U(x) // id(x) === x  
		    
		  // Composition...  
		  U(x).map(g).map(f)  
		    
		    === U(g(x)).map(f)  
		    
		    === U(f(g(x)))  
		    
		    === U(x).map(x => f(g(x)))  
		    
		  ```  
		    
		  Our laws hold, everything works. If it helps your intuition, just think of `U(x).map(f) === U(f(x))` every time you run into a functor. More generally (and correctly), **`map` applies a function to the value(s) within a functor** and *nothing else*. That, friends, is all there is to it!  
		    
		  
		  ---
		    
		  So, I’m sorry that it got a bit less *clearly*-defined towards the end. You can see it’s quite difficult to define an all-encompassing intuition for functors, but I hope this has given you somewhere to start!  
		    
		  If you want more examples, the Haskell docs contain [loads of Functor types](https://web.archive.org/web/20231230183557/https://hackage.haskell.org/package/base-4.9.1.0/docs/Data-Functor.html\#control.i:Functor) for you to see! The intuition will come with time. Until then, as always, feel free to [tweet me](https://web.archive.org/web/20231230183557/http://twitter.com/am%5Fi%5Ftom) and I’ll do my best to clarify my ramblings.  
		    
		  Next time, we’ll look at another type of functor: the **contravariant functor**. Get excited! As always, until then,  
		    
		  Take care ♥
-