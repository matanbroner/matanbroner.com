---
layout: post
title:  "Notes About Distributed Data Types"
date:   2022-09-16 00:00:00 +0700
categories: notes
---

# Introduction
A while ago, I built an app called "Tryout", which uses the notorious ShareDB library to map operational transforms onto
a collaborative coding environment. The app was fun to build, but it left me with a lasting desire to better
understand how distributed data structures work, and how I might rebuild "Tryout" using my very own structure (or at least one I can understand the internals of).

The "big" thing in distributed data structures is *Conflict Free Replicated Data Types*, or CRDTs. They offer the strong
eventual consistency guarantees that one might expect from any distributed data structure, but they include
the added benefits of more efficient memory usage and scalability beyond two peers in practical settings.

# So what is this post?
I am collecting a bunch of notes and links about the topic in order to collect my thoughts. I will be reading a few papers
and I figured my understanding of them may improve if I take the time to summarize certain parts. Furthermore, I will be curating
a list of links I find useful and interesting.

## Seminal Paper: A comprehensive study of Convergent and Commutative Replicated Data Types
[Link to paper](https://hal.inria.fr/inria-00555588/document)

### Background
* Eventual consistency allows for asynchronous replication with other users such that they all reach the same state *eventually*.
* CRDTs require no synchronization, allowing users to apply their updates immediately.
* CRDTs do not use consensus under the hood.
* Certain limitations requiring expensive operations, these can be delayed to a later period when the network is well connected.
* An *atom* is a basic data structure which is contained within an *object*.
* Objects can be replicated, and are independent of each other within the process in which they are located.
* An *operation* is applied on an object by a client, first applied to a source replica and is then propagated asynchronously to all other replicas.
* Operations can be state or operation based.

#### State Based Replication
An update occurs entirely at the source and is then propagated by transferring the modified payloads between replicas.
Updates may have some preconditions or may be null, depending on the object at hand.
For example, incrementing/decrementing a counter has no precondition, but removing elements from a set has the precondition that the item being removed exists in the set.
Causal history is modified during an update such that `f, C(f(xi)) = C(xi) U {f}` and states are merged through `xi, xj, C(merge(xi, xj)) = C(xi) U C(xj)`.

* Happens-Before relationship: `f happens before g <-> C(f) ⊂ C(g)`.

We assume that due to liveness that each replica is able to receive all other replicas' causal history.

#### Operation Based Replication
Instead of sending state, replicas send their operations, using the same method of applying locally then propagating the operations to other replicas.
Causal history of a replica again starts as null, and after executing the downstream propagation it will be (given an operation `f` on replica `xi`): `C(f(xi)) = C(xi) U {f}`.
Again, we assume that with liveness that a delivery order exists such that each each update is reliably broadcasted to all other replicas.
With the same happens-before relation, we say that f happened before g if f is delivered before g is delivered.

#### Convergence
For two replicas to eventually converge, we must meet safety and liveness conditions, meaning that for two replicas:
* If their causal history is the same, their abstract states are equivalent (their `query` operations return the same value).
* If f is in the causal history of a causal history, this implies that it *eventually* gets added to this history.

### CRDTs

#### State based CRDT: CvRDT
These CRDTs send their entire state to all other replicas, which must be merged by a commutative, associative, and idempotent function.'
In order to merge, the states of two replicas must form a *semilattice*.
* What is a semilattice in plain terms?...
Updating the state must monotonically increase the internal state count.
It is proven in the paper that as long as replicas can deliver their states to one another, they will eventually converge.

#### Operation based CRDT: CmRDT
Operations are reliably broadcasted between replicas. They must arrive without duplication but can arrive in any order, meaning they are commutative but not idempotent (ie. can be applied in any order but not multiple times without changing the result.)

> Reliable causal delivery does not require agreement, and is immune to partitions in the network.

## Links

### Papers

**Unique CRDT and Implementations**

* [Peritext](https://www.inkandswitch.com/peritext/)
* [Chronofold](https://arxiv.org/pdf/2002.09511.pdf)
    * [Github](https://github.com/dkellner/chronofold)

### Blog Posts

**Unique CRDT and Implementations**


* [Conclave](https://conclave-team.github.io/conclave-site/)
* [Antimatter](https://braid.org/antimatter) (self Pruning, guys from Braid)

**Tutorials**

* [Operation-based CRDTs: JSON document](https://bartoszsypytkowski.com/operation-based-crdts-json-document/)

### Forums and Discussions

* [Be aware that CRDTs like automerge are solving a different (and harder) problem than Replicache.](https://news.ycombinator.com/item?id=22175530)
* [Main takeaways from toying with both Yjs and Automerge:](https://news.ycombinator.com/item?id=29507948)


### Lists

* [Awesome CRDT](https://github.com/alangibson/awesome-crdt)
* [Choosing a realtime collaboration library](https://convergencelabs.com/realtime-collaboration-technology-guide/)


### CRDT Repos

* [Automerge](https://automerge.org/docs/hello/) (Martin Kelppman)
* [Diamond Types](https://github.com/josephg/diamond-types) (Seph Gentle)
* [YJS](https://github.com/yjs/yjs)


### ShareDB 

* [ShareDB Postgres](https://github.com/share/sharedb-postgres) (outdated)
* [Official Docs](https://share.github.io/sharedb/)


### Random

* [RepliCache](https://replicache.dev/#why) (ie. has central authority?)
* [How NAT Traversal works](https://tailscale.com/blog/how-nat-traversal-works/)