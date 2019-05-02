# Intro

We present some of our ongoing work on a new programming model for asynchronous and distributed programming. For now, we call it *function-passing* or *function-passing style*, and it can be thought of as an inversion of the actor model – keep your data stationary, send and apply your functionality (functions/spores) to that stationary data, and get typed communication all for free, all in a friendly collections/futures-like package!

We are going use Apache Spark for some motivation, but doesn't have to be programming model like Spark. Spark is "just" a distributed collection abstraction. 

Clearly, *f-p* is not restricted to improve Spark, but is a generalisation on RDDs. With *f-p* we improve many things at once. 

# Pickling

Note: those are the points you should focus on better-understanding:

* static pickler generation
* better type safety 
* more analysis happens at compile-time which means more safety

But when it gets into the model for pickler generation, for example, you don't have to be too concerned that's not important for you to know in-and-out. 

# Laziness

In general, laziness, enables operation fusion.

*Lazy heartbeats* would yield to less synchronisation and less book-keeping. 

# Silo

A silo is a structure for storing bulk materials. A silo is a typed data container; stationary.

## Reference to a silo (`SiloRef`)

This is similar to `ActorRef`: "Immutable and serializable handle to an actor, which may or may not reside on the local host or inside the same ActorSystem." This is convenient because an `ActorRef` can be serialized, allowing it to be a proxy for a remote actor on some other machine. For the component acquiring an `ActorRef`, the location of the actor --- local in the same JVM or remote on some other machine --- is completely transparent. We call this property *location transparency*.

# Silo System

Each machine needs a silo system. Clients can create silos on remote hosts.

# Peer-to-Peer

Think about actor system, doing various things (healt check, ...), say, "multiple-machine on program". Peer-to-peer doesn't fit into it. Peer-to-peer is heterogenous: different nodes executing differrent logic (cf. SOA). 

# Uploading data

`SiloRef#pumpTo` takes a destination host, and a function that takes an element from the source and tells you what to put in the target. Note, one source element may be mapped onto many target elements.

# Caching

Give the user the possiblity to cache/ uncache.

# Failure detection

* How does Erlang do it?
* Model of fault tolerance handling in order to support the peer-to-peer case, as well as, the spark case (master/worker).
* Consensus required? What do we need?
* Always keep in mind the question: is there a relation between fault tolerance handling and caching?

# Literatur

* [Failure detectors](http://www.cs.yale.edu/homes/aspnes/pinewiki/FailureDetectors.html)

* [Dealing With Failure in Actor Systems](http://danielwestheide.com/blog/2013/03/20/the-neophytes-guide-to-scala-part-15-dealing-with-failure-in-actor-systems.html)

# Misc
## Sundries
- We do not substitute `Fwd`! But leave them as there are, in order to also leave the data where it is. Otherwise replacing `Fwd` by `Val` yield copy of data.
- A well-typed prg. can be evaluated; one does not get stuck; progess
- `host` geht lineage hoch bis `Mat` und sagt wo dieses liegt; `host` traversiert den kompletten Graph um das Silo zu identifieren
- `h` würde gerne `r` haben damit es `c` komplementieren kann
- A client can not invent a SiloRefId. That's the job of the server
- `heap :: Location -> Value`; `heap` liefert zu eine location ein Laufzeitobject, welches verschiedene Formen haben kann
- Store typing (\Sigma) nur beim type checking
- `\Sigma :: Location -> Typ`
- `S :: DecentralizedId -> Value`
- "consume" ~ adjust silo store
- `\iota` haben nur eine Bedeutung im lokalen heap; `\omega` kann man auch verschicken
- Subject reduction ~ Preservation; Ergegnis der Reduzierung hat den selben Typ mit dem man gestartet hat; Beweis (1) sequential reduction rules (2) deterministic (3) non-deterministic; "Wenn die Conclusion gilt, dann gelten natürlich auch die Prämissen (rückwärtslesen)
- `\Sigma |- \mu`; Für jeden Wert in `\mu` muss `\Sigma` einen Typ  zuweisen
- Eval context: Wie evaluiere ich einen Term, der sich in einem grösseren Ganzen befindet (nested); Ein Evaluation context hilft uns das Regelwerk zu verringern, da wir davon ausgehen können dass wir die Params zu Werten reduziert haben; sonst bräuchten wir weitere Regeln die dafür sorgen, dass die Params erst zu Vals reduziert werden.
- Typregeln sind compositional
- Silo store typing `\Delta :: Id -> Typ` typing for silo store `S :: Id -> Val`
- Store typing `\Sigma :: Loc -> Typ` typing for heap store `\mu :: Loc -> Obj`
- Silo store typing and store typing must be consistent (\Delta ; \Sigma |- \mu) \Delta und \Sigma sind konsitent bzgl. einen Stores \mu
- \iota is a location (Future, Host)
- \omega is a dec. identifier (h,i)
- whenever a type T appears in a term, we must check that T is well formed
- decentralized Ids and lineages are the foundation of fault tolerance
- pure ~ ausschliessen, dass ein term ein subterm hat welcher eine Closure ist

## Volatile local fields

As to CPIS#77:

> Unlike Java, Scala allows you to declare local fields volatile (in this case, local to the closure of the enclosing for loop). A **heap object** with a volatile field is created for each local volatile variable used in some closure or a nested class. We say the variable is lifted into an object.

> var inc: () => Unit = null
> val t = thread { if (inc != null) inc() }
> private var number = 1
> inc = () => { number += 1 }
>
> The local number variable is captured by the lambda, so it needs to be lifted.
>
> number = new IntRef(1) // captured local variables become objects
> inc = new Function0 {
>   val $number = number // recall – vals are final!
>   def apply() = $number.elem += 1
> }

Do we consider this?

## Cyclic dependencies between lazy values

As to CPIS#81:

> Cyclic dependencies between lazy values are unsupported in both sequential and concurrent Scala programs. The difference is that they potentially manifest themselves as deadlocks instead of stack overflows in concurrent programming

## Spark

"Spark --- which is a system --- should be build atop of F-P --- which is a programming model, not a system". Thus, it does not make sense to compare Spark and F-P; that would be like comparing actors with Spark. 

F-P generalizes the Spark model. Recall, generalisation is a broadening of application to encompass a large domain of objects of the same or different type. 

F-P is more general and more flexible. Like, it's not only for master/slave scenarios but also for peer-to-peer ones (recall, peer-to-peer does not have the notion of a master/slave relationship! It's more like a coordination between data stores, for example). Further, F-P may utilize arbitrary data structures (`Silo[T]`) rather than `RDD`s only. To add a new type into the guts of Spark is quite complex.

Spark does have no typing discipline: `instanceOf`. Spark is surprisingly untyped!

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# Distributed programming via SCP

## Abstract

> However, widely-used programming models remain rather ad-hoc

What do you mean by "ad-hoc"?

> [...] aspects such as implementation trade-offs

What exactly do you denote by "implementation trade-offs"?

## 2. Programming Model

You stated:

> A *silo* is a typed data container. It is stationary in the sense that it does not move between machines. A silo remains on the machine where it was created.

So far so good. Then,

> Data stored in a silo is typically loaded from stable storage, such as a distributed file system.

As far as I know, a distributed file system (DFS) abstracts on data location. That is, the fact that the data is distributed is transparent to the user. Silos utilize this transparaency?

In other words, a silo might hold data, distributed over various notes? If yes, could I expand your statement as follows:

> A silo remains on the machine where it was created BUT due to DFS may hold data, distributed over various nodes.

Next, 

> Some programming patterns require combining data contained in silos located on different nodes (e.g., joins)

So, if my previous assumptions are correct, it might be the case that even if the silos are located on different nodes, the actual data might be located on one and the same node, so no "shuffling" is required?

Next,

> Since ref2 is derived from ref1, ref1's silo is located on the same node.

Why are both located on the same node?

Next,

> Partitioning and groupByKey. A groupByKey operation on a group of silos containing collections needs to create multiple result silos, on each node, with ranges of keys supposed to be shipped to destination nodes. These destination nodes are determined using a partitioning function.

I understand that a groupByKey operation on a group of silos, containing collections, needs to create multiple result silos --- for each partion one silo. But why "with ranges of keys supposed to be shipped to destination nodes"?

## A.1 Spore Syntax

> A spore is a closure with a specific shape that dictates how the environment of a spore is declared. [...] In particular, the spore closure is not allowed to capture variables in the environment.

"to capture variables in the environment" what environment? The spore closure is allowed to capture variables in the spore header but not beyond. Variables in the spore header, however, are allowed to capture variables outside the shape of the spore. Right? With "environment" you refer to "everything" outside the shape of the spore?
 
# Function Passing: A Model for Typed, Distributed Functional Programming

## Abstract

Why name change from "SCP" to "F-P"?

Next, 

> A key idea is to pass safe, well-typed serializable functions to immutable distributed data.

What do you mean by "safe"?

Next, 

> The F-P model itself can be thought of as a distributed persistent functional data structure, which stores in its nodes transformations to data rather than the distributed data itself.

What nodes? Up to this point, you introduce the F-P model as distributed, persistent, functional data structure but on this level of abstraction I don't see any "nodes" involved in the data structure as such.

"stores [...] transformations" is like transformations on Spark's RDDs (lineage)? You clarify matters, however, later in section 1 by stating "lineage-based fault recovery in a purely functional setting". Sounds like the main differentiation to Spark. I'd like to recommend to already at this point emphasise the functional rsp. typed approach of your model otherwise readers of the abstract might get distracted or get the impression "yet another spark". 

Next,

> by carefully incorporating laziness into our design [...], our model remains easy to reason about

Why does laziness ease reasoning? Or do you refer to "Deferred evaluation also enables optimizing distributed computations through operation fusion"? 

## 1. Introduction

> However, these frameworks are built atop of tall stacks of typically imperative and untyped code, losing most of the benefits enjoyed by the users of their high-level APIs
> [...]
> Some language features, like closures, aren’t able to be reliably distributed

For both statements an illustrative example would be worthwhile.

Next, 

> Our model can be thought of as somewhat of a dual to the actor model

Even with the footnote, I don't get the point of "dual to".

Next, 

> we keep data stationary and send functionality to the data

Sounds like something new, but it's the "general" big data approach, isn't it?

Next,

> Our model brings together immutable, persistent data structures, monadic higher-order functions, strong static typing, and lazy evaluation

What is the difference between "immutable" and "persistent"?

Next, 

> Interestingly, we found that laziness was an en- abler in our model 

Maybe a forward reference to the explanation?

Next,

> our model can be thought of as a generalization of the MapReduce/Spark computation model

Don't want to be too picky, but don't MapReduce and Spark have different computation models? The former one is already described within the name, and the latter is based on RDDs. So, here comes the nitpicking, I guess the `/` is misused in terms of it gives the impression that both have the same computation model.

## 2. Overview of Model

> We call subgraphs of a DAG lineages [...] To obtain the result of a computation, it is necessary to first "kick off" computation, or to force its lineage

If lineages are subgraphs of a DAG, then what does "to force its lineage" denote?

BTW, up to this point, I've the impression of reading a paper on Spark rsp. RDDS rsp. the concept of a lineage.

Next, 

> The characteristic property of a spore is that the spore body is only allowed to access its parameter, the values in the spore header, as well as top-level singleton objects (Scala's form of modules). The spore closure is not allowed to capture variables other than those declared in the spore header

First you refer to "spore body" and explain three "types" of accessible variables. Next, you call it "spore closure" --- which is confusing, at least for a new reader --- and what's even more confusing is that now you state "is not allowed to capture variables other than those declared in the spore header" which excludes the parameters and top-level singleton objects. I'd just leave out the second sentence. 

### 2.1 Basic Usage

I guess the link to vimeo isn't the proper one :-)

Next, 

> The only way to interact with distributed data stored in silos is through the use of SiloRefs.

Why use `SiloRefs` instead of directly using `Silo`s? Location transparency?

Next, 

> Users interact with this distributed data by applying functions to SiloRefs, which are transmitted over the wire

Are those functions directly transmitted over the wire, or not before a `send`?

In the verbalisation of the Fig. 1 you clarify matters: "forced".

Next, 

> the F-P framework queues up operations applied to SiloRef[S]

Shouldn't it be `SiloRef[T]`? The queuing starts at `SiloRef[T]`, no?

### 2.2 Primitives

In Fig. 2, shouldn't be there a silo for `vehicles`? Also, in contrast to Fig. 1, links to silos are not drawn through. This first confused me.

In `flatMap`, the formal parameter `ps` is confusing relating to the code snippet in `map`. I'd rather use `as` (adults).

Next, 

> With the use of flatMap, however, the call to localVehicles.map(..) creates the final result silo, whose data is then also contained in the silo returned by flatMap.

The "also" confused me. I'd even leave out the entire subordinate clause.

Next, 

> To enable materialization of remote silos to proceed concurrently, the send operation immediately returns a future

What exactly happens concurrently? For example, wrt. Fig. 1, what happens concurrently --- the materialization concurrent to, e.g., the main thread? Or is this only the case if more than one silo is involved?

Next, 

> Assuming we would also cache the owners SiloRef from the previous example, the resulting lineage graph would look as illustrated in Figure ??

Missing figure.

## 4. Formalization

    spore {   \overline{x : T = t}  ; (x: T) => t }
    spore { { \overline{x : T = t} }; (x: T) => t }

Next, 

    map(r, t[, t])
    flatMap(r, t[, t])

Second parameter `t`? Why not require the second parameter to be a spore (`p`)?

Next, 

What does $\overline{T}$ denote?

Next, 

`\iota` is ambiguous. 

# Codebase 

MERGE QUESTIONS.MD AND MEMENTO.MD
DELETE QUESTIONS WHICH ARE ALREADY IN PDF

## F-P

### SiloRef

* Why constrain by `T <: Traversable[W]`?

* `SiloRefId` vs `Int @@ Id`?
