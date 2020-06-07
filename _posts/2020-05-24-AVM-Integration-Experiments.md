---
layout: post
title: AVM Integration Exeriments
---

The first forays into AVM integration have gone pretty well.  The only immediate changes I needed to make to the AVM were related to changing it from JDK10 to JDK8, since Laminar is JDK8.  This was a pretty straight-forward change as it was only using a few helpers added to that version of the JCL, and those were simple enough to re-write for my purposes.  Beyond that, it was just a matter of removing the `module-info.java` and rebuilding some dependencies.  The branch with the changes is here:  [https://github.com/jmdisher/AVM/tree/Java8](https://github.com/jmdisher/AVM/tree/Java8)

I really wish JDK9 didn't introduce that module system:  It is such a pain and I have yet to see an actual improvement related to it.  I really wish they had only added this change to visibility at some advisory level instead of some core VM-level concept which has implications throughout the entire platform.  JDK8 was awesome but everything since has been stained by this problem (hence why Laminar is just JDK8 - I saw no benefit to this headache for the purposes of this experiment).

The next step was to build some simple tests of how I want to approach AVM integration on the Laminar side.  This was actually pretty straight-forward and I do have a test which works.  It is still an isolated test, though, and not integrated into the core of Laminar.  This development can be seen here:  [https://github.com/jmdisher/Laminar/tree/avm_experiment](https://github.com/jmdisher/Laminar/tree/avm_experiment)

All that said, it is still some distance to the point where anything will be merged into `master`.  I suspect that I will minimize the intrusion into the core server by building this _AVM bridge_ into a distinct module which the server can easy choose to build in, or not (part of this will just be to simplify the intermediate build by balling-up the dependencies so their relationship is clearer).

Of course, it is probably a good idea to make the actual AVM changes to fit better with Laminar prior to anything going to `master` since these intermediary states are interesting for the version history of the modified AVM, but they would mean almost nothing from the perspective of Laminar.

An open question is still how much of the AVM functionality should be mapped into a Laminar concept and how many AVM features could be heavily optimized by living in the Laminar world.

A few such features:

* Should cross-topic calls be supported and what would those mean?  I suspect not since it makes things like per-topic compaction essentially impossible
* Should the AVM run all ready-to-commit mutations, even if they aren't programmable?  This is something the AVM does with normal balance transfers since they are part of the dependency graph, so batching more concurrent transactions is far more aggressive with this suppoort.  I suspect that it should also be the case for Laminar, even though these are even simpler, here.
* How should billing be interpreted?  This should still be in Laminar, but only to stop errors from halting the cluster, not from aggressive denial-of-service, as is the concern of AVM.  Still unknown if the design or values AVM uses should be changed, though.

A big optimization which is possible when embedded in Laminar is object graph serialization.  Laminar only needs this for restartability support and potentially some state synchronization, in the future (assuming some aggressive and clever compaction is implemented).  Laminar does need a similar, but much simpler feature:  Revert support.  A failed transaction should have no side-effects within the object graph or its event stream, meaning it isn't as easy as just mutating the objects in memory.  This doesn't even begin to scratch the surface of making this graph of unbounded size, which is also possible in Laminar.

Still, in terms of the basics, the path forward is reasonably clear.  The first priorities are around API and ABI:  They need to be quite different, but the changes should be small.  From there, internals of how AVM works are really more optional changes.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/8)
