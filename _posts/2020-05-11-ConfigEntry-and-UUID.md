---
layout: post
title: ConfigEntry and UUID
---

I was recently looking into replacing all the addressing done by ClusterManager with the actual node UUID, as opposed to the ConfigEntry it is currently using.  I thought this would be pretty straight-forward (the underlying UUID is always extracted, anyway, for indexing into maps, etc).  I eventually decided against this for reasons which I thought were somewhat interesting.  This isn't the _purely right_ answer but it does keep things simpler, at least for now.

## Some History

The fact that ConfigEntry is being used for addressing, at all, probably seems odd and unnatural.  This started in the earlier points in the project where there was no need to come up with a different identifier scheme beyond how a node was described in the config.

For a while, I even had `.equals()` and `.hashcode()` defined on it in order to make it usable directly in hashed collections, etc.

The problem which eventually arose was the problem of canonical addressing of nodes on the internet.  Well, this problem was there from the beginning (binding a port to `0.0.0.0` and connecting via `127.0.0.1` is a common pattern and those numbers are not the same!) but I modified the testing to ignore the problem until I could address it, more directly.

When I got closer to public source release (and had been annoyed by new tests periodically just kind of _doing nothing_ until I manually fixed the IP), I decided to address this.  I knew that I didn't want to require a STUN server for canonicalizing IPs, nor did I want them in the critical path, so I wanted to do something else.  The obvious answer was assign a UUID.

At first, I toyed with the idea of a node self-assigning, when it starts up, and _discovering itself_ in any new config it received.  While this was a simple and versatile idea, it would have added some additional states to the system which seemed like using a sledgehammer to drive in a thumb-tack so I went back to the drawing board.

Instead, I decided to make the UUID into part of the config, itself.  A node would either be given a UUID or self-assign, when it started up.  Clients could then ask it for this UUID when building new configs.  Internally, each node could immediately know if they were in the config since they would see their own UUID.

From there, I changed the rest of the system to stop using ConfigEntry as a key, removed the `.equals()` and `.hashcode()`, instead digging into the ConfigEntry to access the UUID for these cases.

## So, why can't the UUID be used for internal addressing?

I recently tried to change the internal addressing to no longer dig into the ConfigEntry but just use the UUID as the first-class addressing element.

For the most part, this was going well (aside from my concern that peers and clients were now addressed by the same type), until I ran into the requirement that a redirect can be given to a client when a follower node is contacted, directly.  In this case, the ConfigEntry is used by the NodeState to remember who the leader is and the underlying socket data is needed to give the client the redirect.

Replacing this would have required additional states to be maintained in the NodeState.  Specifically, it would now need to directly know the lifecycle of upstream peers so it could resolve them from the UUID so it could tell the ClientManager for redirects.  Currently, the NodeState doesn't know anything about upstream peer lifecycle and only knows it received some relevant information from a given ConfigEntry.

Therefore, changing to the UUID across the system would have added complexity to the system.  Hence, I decided against it.

In the future, if the NodeState has a good reason to need this more detailed lifecycle information, maybe I will revisit this issue.

## Why is this interesting?

This shows another example of the difference between storing state in parameters versus in a long-lived scope (like the NodeState's instance variables).

This keeps the system somewhat more _functional_, reducing the cognitive scope which must be considered when looking at any part of the system.  This also, of course, makes testing and other forms of embedding easier since it makes fewer demans of the surrounding environment (it brings what it needs with it).

Personally, I have always favoured moving complexity into the parameters, instead of cluttering a long-lived scope.  I suppose part of me never wanted to leave the old UW lecture halls where Scheme and ML were everywhere.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/3)
