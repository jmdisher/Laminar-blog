---
layout: post
title: When you Disagree with a Candidate
---

The Raft paper is somewhat light on explaining what happens when a node sees a request for votes, in a new term, from a node further behind it.

Earlier on in the cluster experiments with Laminar, I found that it wasn't sufficient to just ignore the request.  This is because the node which timed out, even if it was the only one, has no way of rejoining the cluster in that case.  The single node times out, starts an election, times out since nobody will vote for it, and repeats.

In order to allow it to rejoin the cluster, the other nodes in the cluster **must start an election** in the same term, in order to keep everyone in sync.  It may result in a split vote, causing another timeout and election, but it at least keeps all the nodes in the same term.  This means that they will sync up and reform consensus, at some point.

Recently, I found that not even this is sufficient in some cases.  I noticed this in the `testElectionTimeout` test in `TestCluster` as I started to add the persistence layer.  These factors are not logically related but it did widen a timing hole related to this case.  The problem is actually related to what happens when the leader is stopped and immediately restarted with a blank state.  Consider this situation (5 nodes marked L, C, F, for leader, candidate, and follower, respectively, and their received offsets in brackets):

```
1:  L(32)
2:  F(32)
3:  F(32)
4:  F(7)
5:  F(5)
```

In this case, stopping and restarting 1 ends up with the following situation:

```
1:  F(0)
2:  F(32)
3:  F(32)
4:  F(7)
5:  F(5)
```

Here, we can see the problem:  The cluster had a majority consensus of 32 before 1 was restarted.  After the restart, it would be possible for node 4 to become candidate, get votes from 1 and 5, thus becoming the leader.  This would reverse the cluster consensus from 32 to 7, which is clearly a problem if any of those intentions have been committed (which is what triggered the failure - nodes are never allowed to reverse committed intentions).

I am not entirely happy with my solution, since all it does is dramatically shrink the timing hole, but it isn't perfect.  I suspect that this may just be an issue of dealing with degraded clusters.

The solution I did use was to **start an election in the next term** when disagreeing with the candidate.  This forces the the original candidate to abandon its election and vote for the new leader.  This works but it is still possible for the incorrect candidate to win and receive (and potentially commit) new intentions before it receives this other request for votes.

I suspect that a better answer would be to forbid new servers from voting in elections, since they can only be new if there is no leader, meaning they are in this situation.  It would mean that a majority of the remaining servers would still need enough votes to lead the cluster, which would force a server who contains all the committed intentions to become the leader, hence resolving the problem.

The more I think about it, that is probably a better solution but does still leave a problem of that server participating in the next election, if there is some kind of network fault around the real leader, and there would be no way to handle that case.

Of course, taking this a step further might make most sense:  a server cannot vote for anyone until it catches up to the lead server's commit point with its received intention list, at least once.  After all, an append attempt or a hearbeat tells the node what the leader's commit offset is, so it could then enter such a mode.

The further extreme, of instead only voting for a candidate whose received offset is at least whatever the leader previously told you was its commit offset is also an option, but doesn't really solve any problem, since that can also be heavily out of sync, beyond the degenerate case of not voting if you are a new server.

Unfortunately, there probably aren't elegant solutions when the cluster is in a degraded state.  A degraded cluster may always be in a fragile state until its new leader is able to commit at least one intention.

I am still trying to think through the best option but [I did merge that "shrink the hole" solution to master](https://github.com/jmdisher/Laminar/commit/ed68e552a9d4ed1a3428a6b3c1849fbf52d0a457).  I am not entirely sure but I suspect that is probably still a good change, even if it doesn't solve the entire problem, since it will tend to favour "better" leaders, either way (that is, avoiding redundant client resends).  Still, it does add extra delay on leader selection, so I have not ruled out reverting it.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/13)
