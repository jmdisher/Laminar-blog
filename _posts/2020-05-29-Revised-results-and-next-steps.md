---
layout: post
title: Revised Results and Next Steps
---

After thinking more about the performance results in the last post, I started to wonder if I had correctly invoked the AVM such that it would enable its optimizations ("hot contracts" to better utilize the JIT on contract code and "hot objects" to avoid redundant deserialization of the graph).  As it turns out, in my haste to get the initial experiment working, I had avoiding bothering with deciding how to expose this opportunity.

These optimizations resulted in a better than 4x performance improvement over the previous results.  Still much slower than a vanilla topic, but much closer.

## Vanilla Results

![alt text](https://raw.githubusercontent.com/jmdisher/Laminar-blog/master/images_for_posts/2020-05-29/perf_vanilla.jpg "Non-programmable topic performance")

This is for 10,000 messages per client.  The result here is about what would be expected and shows the best time being an average of `26 us` per message when 5 clients each send 10,000 messages to a single node.  The worst time is an average of `264 us` when 1 client sends 10,000 messages to a 7-node cluster.  These are highly consistent with previous results as nothing big has changed.

## Programmable Results

![alt text](https://raw.githubusercontent.com/jmdisher/Laminar-blog/master/images_for_posts/2020-05-29/perf_avm.jpg "Programmable topic performance using AVM")

This is for 10,000 messages per client.  The result here is still slower, but not nearly as slow as implied by the initial results:  The best time being an average of `101 us` per message when 5 clients send 10,000 messages to a single node.  The worst time is an average of `1054 us` when 1 client sends 10,000 messages to a 7-node cluster.  These now show a more similar shape to the vanilla results and 4-10 times better than the initial AVM results.

## Next Steps in Laminar

The key purpose of Laminar is now in reach:  Solve the parameter validation problem (and potentially other interesting cases) using a programmable event store.  There is still a question of whether the persistence layer and/or adapting the AVM API should be done first or if they are required to prove the point, at all.

Current plan:

* Demonstrate parameter validation solution
* Demonstrate other idioms like concurrency primitives
* Create initial draft of paper (or just an unofficial summary of the project)
* Implement persistence
* Formally integrate AVM by merging its integration commit into `master`
* Write and publish official paper
* Consider future possibilities (including AVM API adaptations and optimizations possible within this domain)

The storage question is still with me but I do think that it is required before producing anything "official" since there is something _fake_ about not using it.  It also has implications for things like how the transformed code and serialized object graph should be persisted, which really should be addressed.

Beyond that, there is the question of what to do with the project, long-term:  Should it just be left to gather dust as an intellectual discussion topic or should it be productized into something someone could actually use?  I would love to do that productization work but I am no salesman so I am not sure if there is real value in the work beyond the fun of just doing it.

## Design of Parameter Validation Solution

When considering that "balance transfer" example, even the AVM doesn't make this fully trivial.  After all, a naive solution requires rebuilding a key-value store _inside_ the AVM.  While this might have merit in some far-future situations (where Laminar and AVM are just used as a generic distributed systems framework for building databases), it isn't appropriate in the current situation (finite store size, and all).

I realized a fairly simple approach which would work in constant space is to allow it to give false negatives if the client sending the transaction is too out-of-sync.  Essentially, this would start to feel like a compare-and-swap (CAS) loop:

* server has committed mutation offset `X`
* client has observed mutation offset `Y` via listener (for `Y < X`)
* client sends transaction:  Account key (`K`), value to withdraw (`V`), current balance (`B`) at offset `Y`, and `Y`
* topic program checks to see if it has this account in its cache:
	* (if not, this means it has been evicted by more-recent changes)
	* if not present and `Y >=` its oldest entry offset, enter `B - V` into cache key `K` with offset `X + 1` and return success
	* if not present and `Y <` its oldest entry, there is not enough information to proceed so return failure
	* if present and latest offset of `K > Y`, the client is stale so return failure
	* if present and the latest offset of `K <= Y`, enter `B - V` into cache key `K` with offset `X + 1` and return success
* if the client observes the failure, it can retry once its listener has made progress

This approach isn't perfect since many failed mutations will be sent and replicated when the system is under heavy load but it will still work and make progress (since it only updates the cache, potentially evicting elements, when an update succeeds).  Hence why I look at it like a CAS.

## Possible Comment Idea

Something which is an obvious blind spot of using this as a blog is that there is no real comment or feedback system.  I was thinking I might start opening issues and connecting them to posts in order to provide a simple mechanism for this.  It will be a little manual, and not exactly ideal, but it would work and allow for feedback.
