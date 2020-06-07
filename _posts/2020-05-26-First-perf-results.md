---
layout: post
title: First Perf Results
---

Built a small [write performance test](https://github.com/jmdisher/Laminar/blob/master/integration/test/com/jeffdisher/laminar/performance/PerfWritePerformance.java) to verify that the performance curve of Laminar is as expected and to see what the first rough cut of programmable topic performance is like.

All this test does is send `PUT` messages to a single topic.  The variables are how many clients are used to send them, how many messages each client sends, and how many nodes there are in the server cluster.  The clients are still run in one thread but the greater focus is on commit concurrency (although the observable effect should be similar to just sending that many more messages on a single client).

This doesn't test anything regarding cluster fail-over, partial/degraded cluster performance, write performance while syncing a node, or read performance of listeners.  The idea was to create a test which can be used to check the performance curve of the current system and use this as a basis for comparison against programmable topics and a real persistence layer.

In terms of the numbers observed, they were very much what I expected:

* more messages resulted in a lower average per-message time (since that washes out constant costs)
* more nodes resulted in slightly lower performance (since worst-case majority commit time is greater)
* programmable topics were about 20+x slower than _vanilla_ topics (no optimization of the AVM or even its invocation so this is replacing an object allocation with a thread hand-off, execution of code, and serialization of the object graph)

## Vanilla Results

![alt text](https://raw.githubusercontent.com/jmdisher/Laminar-blog/master/images_for_posts/2020-05-26/perf_vanilla.jpg "Non-programmable topic performance")

This is for 10,000 messages per client.  The result here is about what would be expected and shows the best time being an average of `25 us` per message when 5 clients each send 10,000 messages to a single node.  The worst time is an average of `269 us` when 1 client sends 10,000 messages to a 7-node cluster.

## Programmable Results

![alt text](https://raw.githubusercontent.com/jmdisher/Laminar-blog/master/images_for_posts/2020-05-26/perf_avm.jpg "Programmable topic performance using AVM")

This is for 10,000 messages per client.  The result here is not surprising, given the very rough nature of the integration and shows the best time being an average of `1124 us` per message when 1 client sends 10,000 messages to a single node.  The worst time is an average of `4487 us` when 1 client sends 10,000 messages to a 7-node cluster.

## Performance Next Steps

Improving the AVM integration so that all ready-to-commit messages can be submitted at the same time would be an improvement and not terribly complex.  While this test uses just one topic (and the topic is the unit of concurrency in the AVM integration), it would still amortize the cost of entering/exiting the AVM for every call.

Beyond that, a very big advantage could be found in modifying the AVM's graph serialization decisions since Laminar only needs to save out the graph, periodically.  This would remove a lot of heavy-weight effort from the critical path but would be a more substantial change.

That said, performance isn't actually the point of this experiment, until much later on.

## Next Priorities

Before fixating on performance, purely, more work must be done to fully trace out the usefulness of the system and better demonstrate why I was building Laminar, in the first place.  This means that my current thinking on priorities (although this changes often as I realize shorter paths) is this:

1. Adapt Laminar shape to be ready for the AVM bridge
1. Remove hacks from the AVM integration and merge that to master (not sure if this should be done before or after the next)
1. Change AVM API to better fit the needs of Laminar programs, as opposed to blockchain programs
1. Build complex examples of solving specific problems with AVM
1. Build the persistence layer and restart support

The progress of the project, thus far, is meaning that I need to determine precisely what point is **done** for this experiment.  I want to get this to the point where some kind of paper can be written about what problems it solves and what trade-offs it introduces.  I am not sure exactly what that point looks like (and if something like _unbounded lazy-loading object graphs_ are required).

Still, being to the point where some numbers and patterns can be shown is interesting.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/9)
