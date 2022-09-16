---
layout: post
title:  "Notes About CRDTs"
date:   2022-09-16 13:55:00 +0700
categories: notes
---
> Note: This post is more of a space for me to jot down my notes as I read a long survey paper

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

## Survey Paper: A comprehensive study of Convergent and Commutative Replicated Data Types
![Link to paper](https://hal.inria.fr/inria-00555588/document)


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