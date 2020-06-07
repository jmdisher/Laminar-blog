---
layout: post
title: AVM Integration Plans
---

As the project progresses, the most important part is finally getting closer:  AVM integration to support programmable topics.  In fact, the project is at the point where this could be attempted (although I am weighing that against building some sort of example and better support for specific IP binding and identification).  This leads to the question of how to do this.

At first, I thought that this would be a very drawn out process:  Fork the AVM, whittle it down to the point where it is no longer blockchain-specific but is more geared toward event store activities, then integrate it and build tests of what it can do.  Thinking about it this morning, however, I realized that there is a quicker path to demonstrating this:  Drop the AVM in, as is.

One of the reasons why I wanted to use the AVM is that it was designed to be an embedded module, from the start.  This means that integrating it really just requires loading it as a dependency, implementing a few interfaces, and passing in the right types.  Beyond that, it will "just work".

Also, since I only need to support tests which demonstrate event store functionality, I can just not make use of the features which I will stub out as inappropriate.  Really, my current thinking is to disable basically everything and just expose the key-value write API (which I will use to synthesize output events - it is a better fit than the existing event log API) and maybe something like sending in the mutation offset number as the "block number".

From there, I will need to expose deployment support on topic creation (which will be easy, aside from concerns that I might run into my 64 KiB message size limit - not sure if I will need to change that through the duration of this) and write some basic tests and a demo.

As for the demo, I was thinking of writing a work item ticket system.  This is a contrived example but is simple and demonstrates the core benefit of using this model (since it is a bizarre use of Laminar but possible whereas it would be impractical/impossible on non-programmable stores).

From there, I can proceed with changing the APIs to be a better fit and expose more appropriate information/functionality to the deployed programs.

By that point, the scope of the project will be realized and I will just need to build the storage system, find an approach to restartability, and add some more optimizations at the AVM level by removing complexity required for blockchain support.  After that, it is just hardening, testing, and studying.

Those are not small tasks but at least the full scope will allow for rich conversations and considerations of the merits/drawbacks of the approach.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/7)
