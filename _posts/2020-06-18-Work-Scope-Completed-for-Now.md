---
layout: post
title: Work Scope Completed... for Now
---

I merged the persistence and restart support to `master` so this marks the end of the core Laminar requirements before focus shifts more completely to the paper.

I may return to fix bugs, polish, or otherwise optimize the project if I or someone else ends up using it for something but this is effectively the "Done" point, for the time being.

This does mean that a clearer plan for the contents of the research paper needs to be defined.  Work on this will likely involve adding the tests and demonstrations to the code base, though, so it won't be completely stagnant.

For the most part, I will probably structure the paper much like the Raft paper, since it is also introducing a new approach to an old problem and explaining its implications, as opposed to a direct comparison against something existing.

In terms of specific measurements, some initial ideas:

* emulations of native `PUT`/`DELETE` operations - to demonstrate the overhead of simple-case invocations of programmable topics
* the parameter validation demo - just to demonstrate the overhead of a practical application of programmable topics
* maybe some kind of small DB or cache - just to demonstrate the overhead of impractical programs

Also, the thinking around that rejoining race problem is still on my mind and it has implications for restartability.  I am starting to think that a straight-forward solution is to persist UUID, for restart, and just rely on fresh nodes having new UUIDs.  This will mean consistent elections for restarted nodes (since they were part of a consensus decision, before, so they may still be part of the majority which allowed progress) and new nodes only coming in through well-defined joint consensus.  Completing the restart support made this option more obviously correct.  Not sure if it is worth solving that until I find something to do with this, though.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/14)
