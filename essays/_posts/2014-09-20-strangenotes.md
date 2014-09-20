---
layout: post
title: StrangeLoop
---

I went to [StrangeLoop][1]. It was awesome! Basically a polyglot conference
that pulled together academia and industry. Very focused on Functional
Programming (FP), but had other ideas too. I took notes, pasted in somewhat raw
fashion below. I highly recommend going next year!

## 2014-09-17

### Future of Programming Workshop x Emerging Languages Camp

I attended a bonus day camp on the Future of Programming. I was expecting stuff
like I've [seen out of Brett Victor][11], and while I didn't get quite that, I
did see some really interesting stuff. All videos should be published by 12/1
this year.

### Jonathan Edwards / Subtext

* "Leader of the Programmer Liberation Front"
* [http://www.subtext-lang.org/](http://www.subtext-lang.org/)
* Experimental language trying to make it easier to build websites
* inspired by excel / lighttable
* wysiwyg with explosion of computation
* All arrays implemented as CRDTs
* [Lenses][2] all the way down
* Transaction based
* Causality is monotonic in the tree
* Guaranteed change-commit cycle has nice properties, especially in
  parallel and distributed computation
* assertions are final phase of commit, operations are strictly ordered
* "Two-way Dataflow"

### Mark / Storyteller

* Visual VCS with Story Board
* His primary complaint is that he doesn't see how the programmer
  thought about the problem and solved it.
* I really think you can get all the benefit of this with smaller, more
  verbose commit messages on a standard VCS, which is general
  best-practices anyway
* [http://www.storytellersoftware.com](http://www.storytellersoftware.com)

### Toby Schachman / Shadershop

* @NYC MIT Media Lab analogue
* How do we communicate to a computer over a spacial channel?
* Shadershop = shaders in 'photoshop'
* a Shader is a fn of xy -> rgba
* This is cool!!
* Pseudo random number generator = high-amplitude sin sampled by fract
  (flatten out to fractional components)

### Kaya

* Declarative Reactive
* Object Query Language
* Better query language for tabular / spreadsheet data

### Céu

* Structured Reactive Programming
* `await`, composition, synchronous execution
* Neat GUI programming ideas
* [http://www.ceu-lang.org/](http://www.ceu-lang.org/)

### Joel / CodeHint

* Dynamic and Interactive Synthesis for Modern IDEs
* Autocomplete for the modern age
* incorporates some concepts from lighttable
* for Java only now

### Julien / Facebook / Hack

* Evolving PHP
* Julien = one of the language designers for Hack
   * oCaml programmer at heart
* Whatever you choose, in 10-15 years it'll be outdated
* FB PHP
    * over 10M sloc
    * Monolithic
    * reinvesting in mercurial and git to make them scale
    * pushing twice a day
    * mostly migrated to Hack (except for php libs)
* hack = statically typed language for hhvm, compatible with php, same
  representation at runtime
* instantaneous type-checking, with gradual typing

## 2014-09-18

### Keynote - The Mess We're In

* Joe Armstrong - one of the creators of Erlang
* What's going wrong and why, physical limits of computation
* Software's getting worse over time. When it doesn't work we don't
  understand why
* We've created billions of man-hours of work to do in the future!

--

* Bad code is code you can't immediately understand, code that requires
  a brain shift to understand
* 6 32-bit integers can be in the same number of states as the number of
  atoms on earth
* need 2^7633587786 universes to have the same number of states as any
  one laptop

--

* failure - you cant handle failure with just one computer
* a neat exercise might be RSA in roman numerals
* new programmers have no common language (old-school: sh, make, C)
* efficiency vs. clarity - balance between abstraction and speed
* naming is hard
* byzantine general problem, causality
    * 2-phase commit breaks the laws of physics ;)
* simultaneity - observers see different things based on location
* entropy always increases
* c^2/h = the bremmerman limit - how fast things can vibrate, or change
  state. The fastest clock rate you can get
* the ultimate 1kg computer - a black hole
    * 10^51 ops/sec, lasts for 10^-21 second, emits data through quantum
      radiation
    * Hawkins radiation and quantum entanglement
    * conventional - 10^9 op/s
* the real ultimate computer
    * the entire universe - 10^123 ops so far, can store 10^92 bits
    * information theory - if it takes more than 10^123 to crack a code,
      it's uncrackable

* entropy reverser - "Let's break the 2nd law of thermodynamics!"
* Naming objects
    * URIs aren't perfect. hashes are useful, validatable, perfect.
      (a la darknet)
    * Algorithms for finding hashed data - chord, kademlia. basic P2P systems
    * combined you get gittorrent ;)
* collapse
    * find all files
    * merge all _similar_ files - maybe least compression difference:
      size(C(A)) ~ size(C(A+B)). if identical, compressed concat should
      be tiny bit larger. Works for small data, not big data. There are
      other better methods

* computing is about controlling complexity, and we've failed miserably
* make small, validatable systems, and connect them
* computers are using more energy than air traffic, etc. we need to make
  computers operate more efficiently
* let's clean up the mess we've made

### Unix smalltalk

* neat dynamic symbol loading, autoloading C libraries from Node as
  proof of concept

### Lolchat - League of Legends Chat

* lolwut?
* 7.5M concurrent players!
* 1b events routed per server per day
* 11k messages/sec!
* Using XMPP protocol, but with their own modifications
* Used ejabberd at the beginning - was good out of the box but they
  customized it later
* Riak - database
* rescheduling queries for later processing
* cloned Riak set for ETL jobs
* rewrote most of ejabberd - 5-10% is left from open-source!
    * removed unnedded code, optimized certain flow paths, wrote lots of
      tests
    * example: asymmetric friendships caused too much messaging
      overhead, changed protocol to be symmetric friendship
    * example: removed muc router for multi-user chat
* changes to erlang
    * patched erlang and otp to remove some generic stuff they didn't
      need for better overall performance
    * added transactional multi-server code reloading
* started with mysql but moved to riak for performance, stability, and
  flexibility (schema changing speed)
* Riak - truly masterless, so fully HA
    * CRDT libraries help with write conflicts (dropping C in CAP)
    * scaled linearly
* _lessons learned_
    * *key to success is to understand what our system is actually doing*
        * built over 500 realtime counters to monitor things
        * each metric and counter has threshholds to define normal and
          abnormal
            * using graphite, zabbix, and nagios
    * use Feature Toggles!
    * patch bugs on the fly
    * logging important - log healthy state as much as failure state
        * using [Honu?](https://github.com/jboulon/Honu/)
    * always load-test code. generate metrics on load-test environment
      and learn about it. Simulate failures in test too
    * be ready for failure - things will fail that are out of your
      contorl

### Sociology of programming languages

* who adopts what
* keeping perceived adoption pain low
* programming is actually social
* design smarter by exploiting social foundations
* ecological model of adoption
    * music spreads through friends (people)
* hammerprinciple data + 2-week intensive survey + mooc data + scraping
  sourceforge = [http://lmeyerov.github.io](http://lmeyerov.github.io)
* lots of room in the long tail for niche languages
* managed languages are much easier to learn
* Peter Norvig quote about learning at least half a dozen languages
* diffusion of innovation:
    * knowledge, persuasion, decision, trial, confirmation
    * diffusions catalysts: clear relative advantage, observable,
      trialability and compatibility
    * make something observable and it'll catch on much better
* dataviz and renderfarms tonight [@lmeyerov][3]

### Rules / [@ryanbrush][4]

* intelligent systems derive their power form the knowledge they posses,
  not how it's encoded
* structure your rules system as a graph / network
    * Forgy's thesis gave rise to expert systems
    * They were overhyped - lots of intellectually dishonest promises
      were made
* expert systems were mostly forgotten in the 90s
* Martin Fowler on rules engines in 2009
    * it doesn't end well...
    * good for simple problems but problems don't stay simple
* talking about logical constraint systems (prolog-ish)
* [https://github.com/ulfurinn/wongi-engine](https://github.com/ulfurinn/wongi-engine)
* Agile works when iteration is cheap
* tools from unagile industries:
    * blueprints - unambiguous specification that describes our system
    * floorplan - simplified projection of a blueprint
* rules are a tool for developer to simplify code by minimizing semantic
  gap

### Idris

* Dependent types = types dependent on values
* NICTA on SEO4 - formally verified kernel
* Adventures in Extraction - Wouter Swierstra
* extraction = taking formally verified dependently typed code and
  creating executable code
    * extraction was inefficient 4 years ago at least
* idris is making dependent types more practical than coq
* iridium - window manager like xmonad (60% idris, 40% obj-c for mac
  apis)
* *brain melt*
* Algebraic laws lead to correctness guarantees

### CRDTs etc

* CP systems - concensus protocol, like Chubby, Doozer, ZooKeeper,
  Consul, etcd (`Raft` is latest hipster CP)
* AP - Cassandra, Riak, Mongo, Couch
    * eventual consistency
    * all are different and rarely provable guarantees
* delay, drop, out of order, duplication
    * CP abstracts these away
    * AP allow them to happen and deal with it
* CALM principle
    * Consistency As Logical Monotonicity
    * A system as it's accepting messages should only grow in one
      direction (a counter that only gets bigger) bloom lang
    * ACID 2.0 - Associative Commutative Idempotent Distributed
        * reification of CALM
        * CRDT
            * conflict-free replicated data type, provably consistent
              without strong consensus

#### CRDTs

* increment-only counter - increment or read
    * Need associative, commutative, and idempotent function
    * addition is associative, commutative, not idempotent
    * Set union - associative, commutative, idempotent
    * on track plays, you can use userid as set key to union - gset crdt
    * read - read all values and rely on ACI to give correct value
* convenient solution to a problem but it requires bending the problem
* devops - acknowledged that developers are the best people to run their
  own code in production. Devs are ops.
* soundcloud event - timestamp, actor, verb, thing
    * distribution methods:
        * fan-out on write
        * fan-in on read (much nicer for products mostly)
            * can be slow
    * CRDT Sets - G-set, 2p-set, or-set, roshi set
        * roshi - single logical set represented by 2 physical sets -
          add set and remove set
    * built with sharded redis zset with lua extension
* basically reimplemented Cassandra in go
* [http://github.com/soundcloud/roshi](http://github.com/soundcloud/roshi)

### Finagle and Clojure - Sam Neubart

* finagle-clojure - highly concurrent rpc on the jvm
* "your server as a function" - finagle paper
* makes distributed systems obvious
* finagle works with zipkin
* build with netty - async io on jvm
* uses Futures!
    * 3 states - undefined (pending), success, error
* blocks on IO, locks, and waiting for futures
    * composes futures to stay async

## 2014-09-19

### Nada Amin / Programming should eat itself

* Programming to understand programming
* Reflective language Black in Scheme
* TABA / convolution
* very academic

### Evan Czaplicki - Elm / FRP

* [@czaplic][7] wrote Elm
* FRP - dealing with the world
    * in elm, it's the signal graph
    * explicitly structured time
    * signals are infinite
* FRP lets you do time-travel
* [higher-order functional reactive programming in bounded space][5]
    * linear types (out of idris)
    * maybe not actually useful
* async dataflow - imperative FRP
    * flatten : Signal (Signal a) -> Signal a
    * ReactiveExtensions ReactiveCocoa Bacon.js
* arrowized FRP - Yampa, Netwire, Automaton (elm)
    * swapping in and out Signal handlers
    * abstraction replacement for pure functions and modules
    * helpful for music..
* you can hook up to elm from non-elm stuff (with something like
  bacon.js etc)
* [http://elm-lang.org/papers/concurrent-frp.pdf](http://elm-lang.org/papers/concurrent-frp.pdf)

### Purescript - [@bodil][8]

* JS sux d00d
* game engine - functional, reactive, animation stream in game loop
* "elm - if haskell were designed by a usability expert"
* constraint - low deliverable size and unpredictable environment
* originally built with typescript and baconjs (rxjs is more powerful)
  and html canvas
    * game loop reactive stream
    * in production
* once you've done haskell you can't go back
* purescript focuses on producing very neat js code
    * haskelly syntax
    * close to the js
    * actually pure - purely functional
    * effects & ffi (monads and js access)

### Rich Hickey - Transducers

* Transducers - extracting the essence of map, filter, et al, recasted
  as process transformations
* you can transduce a process that is a succession of steps that ingests an input
* a seeded left reduce, more as a process than something that's building
  a thing
* transduce - to lead across
* example: put the baggage on the plane, as you do that
    * break apart pallets,
    * remove bags that smell like food,
    * label heavy bags
* conveyances, sources, and sinks are irrelevant in process definition
* transducers are fully composable
* hard to model in strictly typed systems because each step returns a
  different fixed type
    * sounds like a good problem for Ruby! (trololololol)
* early termination support (take-while non-ticking?)

### Clojurescript

* C.A.R. Hoare - building without deficiencies. There's a significant
  difference between simple and easy. Ease doesn't necessarily imply
  simplicity.
* @Prismatic: Still working our their core product, need to iterate
  fast. Using clojure on the backend
* global mutable state = trouble
* yeaaaah no sell on why I'd do this instead of others

### Haste: Full-Stack Haskell for Non-PhD Candidates

* [@lasericus][9] and [@coopernurse][10]
* industry developers, wanted to build some stuff in Haskell but didn't
  know where to go
* language properties: performance, productivity, safety
* Haste - Haskell -> JS compiler
* Haste project people would like to write something that overall spans
  client and server, but that's not required
* cabal install haste; haste-boot
* netwire mentioned again
* Has FFI support - ccall and ffi
* ccall is much faster but requires type declaration
* exported functions go in Haste js namespace
* compiled js is not awesome
* you can test with Hspec
* Haskell's Integer is not the same as JS's Number! Hspec can lie!
* Pure-haskell libraries will work! (from hackage) with haste-inst
* could use a better API for DOM manip
* upenn CS 194 - learn haskell better [http://www.seas.upenn.edu/~cis194/](http://www.seas.upenn.edu/~cis194/)

### 8 Fallacies of Distributed Computing - Stephen Asbury

* list on wikipedia
* Fallacy: a wrong belief, a false idea, or a mistaken idea
* backhoes suck - they kill your internets
* seen this all before, going to another talk...

### game in haskell

* eleria - FRP haskell
* network in her examples is the set of things that make state
* generate signals for all the things!
* never use unsafecoerce!
* used openAL for sound, not threadsafe but worked
* keyframe interpolated or skeletal animation
* made objects in blender to make keyframes (it's not user-friendly)
* physics - 2d library = chipmunk / hipmunk (haskell bindings)
    * not easy to combine with FRP...
* it's far from perfect..
* making a game is much harder to get the UX right than the code
* [Nikki and the robots] is commercially made hasklel with QT,
  open-source code
* game studio made haskell sdl window on mobile
* chucklefish is hiring haskell game devs (wayward tides)

### Visualizing Persistent Data Structures / Dann Toliver

* Data structures are "accidentally" complicated
* people love to complicate data structure language
* abstract vs. concrete data structures
    * abstract - stack, queue, list, array
    * concrete - linked list, array
    * array - list with random access support and the data structure
      that supports it
    * adjectives usually mean concrete, unless there are too many, like
      self-balancing binary search tree... or too few, like priority
      queue
    * proper name in it then it's always concrete ;)
* persistent data structure
* fully-persistent data structures - includes changes over time, like
  git
* confluently persistent - you can merge a branch back into the main
* *batched queue* - 2 stacks, push onto one, pop the other, when pop stack
  empties pop/push all from push stack to pop stack
* *[hash array mapped trie][6]* - fast, great for maps, dandy vectors, constant
  time
* very cool js viz of data structure values
* mori vs. fb vector queues

### Our shared joy of programming

* take care of your personal joy:
    * Get enough sleep
    * eat well, exercise
    * keep interested and keep learning
    * code can pull you out of darkness
* Sharing joy increases our personal joy, binds us as a community, lifts
  us up, and makes us more than ourselves
* [A little video ;)](http://youtu.be/Pcu2DoBPtCc)

[1]: https://thestrangeloop.com/
[2]: https://github.com/ekmett/lens
[3]: https://twitter.com/lmeyerov
[4]: https://twitter.com/ryanbrush
[5]: https://www.mpi-sws.org/~neelk/popl074-krishnaswami.pdf
[6]: http://en.wikipedia.org/wiki/Hash_array_mapped_trie
[7]: https://twitter.com/czaplic
[8]: https://twitter.com/bodil
[9]: https://twitter.com/lasericus
[10]: https://twitter.com/coopernurse
[11]: http://worrydream.com/#!2/LadderOfAbstraction
