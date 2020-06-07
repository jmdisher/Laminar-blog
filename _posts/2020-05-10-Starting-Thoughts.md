---
layout: post
title: Starting Thoughts
---

Ok, this blog [forked from here](https://github.com/barryclark/jekyll-now) seems to be working very well.  It is exactly what I wanted:  something simple, with an RSS feed, which didn't require me to sign up for something new.

So, time to get down to talking about Laminar.

As anyone would notice, it isn't really much of anything, yet.  My milestone for pushing the source publicly (the first 200 commits, or so, were just local) was to get all the basics required for core functionality working.  These include:

* network layer
* client library
* listener library
* client transparent reconnection
* cluster config changes (including joint consensus)
* cluster data replication
* timeout and leader election

These parts were all working so it seemed like it was finally worth pushing somewhere, where it could be discussed.

Next points of functionality required:

* simple examples to demonstrate it (as much hand-waving is currently required to explain this)
* change message dialect to key-value put/delete operations instead of just "message"
* topic splitting (there is currently just a single "topic")
* AVM introduction and modification to support programmable topics
* on-disk persistence and node restart from persistent state (as data is currently just stored in an asynchronous in-memory array)

One of the main elements of an event store which will *not* be considered in-scope for this experiment is log compaction.  This means that the data store will always grow without bound.  That said, I do have some thoughts on how this would be done (since programmable topic make this... interesting) which I will probably talk about at some point in the future (shouldn't get ahead of myself, now).

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/2)
