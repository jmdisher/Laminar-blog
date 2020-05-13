---
layout: post
title: Evolving the Design for Testing
---

Testing in Laminar is currently pretty rough.

This is due to its heritage as a prototype to demonstrate an idea.  It still hasn't progressed far enought to have demonstrated that idea.  This means there is still a long way to go.  That, in turn, means there is much more testing to do.

As a result of this heritage, much of the focus has been on integration testing since it was more important to ensure that the entire cluster behaves as expected, as opposed to unit testing a specific component.

Which leads to the next problem:  The design.  The internals of Laminar have sort of _blobbed out_ from an original prototype where almost all the logic was in one class (if you look into the history of NodeState, you can see this).  This means that much of the code _can't_ be unit tested as one would traditionally like.

Now, I am not going to say that the decisions which got the project this far were poor (over-design would have had similar problems, in a less obvious shape, and would have taken longer).  All it means is that refactoring is now needed.

Refactoring is something I am somewhat touchy on since I have worked with people who would periodically clutter version history or create conflicts with real work because they "liked refactoring" and that this made it _cleaner_ (I have no idea why many people think that there is such a thing as _objectively cleaner_).

That said, I don't think that refactoring is a bad thing.  In fact, I think it is an essential part of almost all code changes.  This is why I don't like it when it gets called out as "the purpose" of a change since refactoring must be a means to an end, not and an end unto itself.  In fact, my team in a recent job had a shirt made for me which says "You can't refactor in a vacuum!" simply because this was one of my ranting points.

So, the specific areas of interest here are the pieces of logic in NodeState and ClusterManager.  These are currently so tightly coupled to the idea of being run as an entire system, that the unit tests still depend on things like the network layer, the callback structure, or the threading model.  As these are some of the most complex parts of the system (just due to the fact that they implement all the interesting pieces of behaviour), I really need to extract their logical elements into more easily tested fragments.

The other difficulty in testing is observing intermittent failures in integration tests.  I don't think this problem is a big deal but it is one of the reasons I am happy to have as much control over testing as Ant gives me and why I am glad I didn't use some pre-existing framework for this.  Today, I ran all the integration tests with high logging on hundreds of times to detect a failure and luckily it didn't take hours.

So, the good news is that it appears as though these intermittent bugs are resolved since I didn't see anything go wrong after my last few fixes after a similarly long debugging and testing adventure, yesterday.

Next up is my plan to split out upstream and downstream logic management from ClusterManager (to make that easier to test) and work on introducing the real message dialect and topic splitting in order to make a real example.
