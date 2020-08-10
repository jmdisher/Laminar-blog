---
layout: post
title: Submission and Next Ideas
---

I submitted the Laminar whitepaper 2 weeks ago.  It will take up to 6 weeks to hear back about what they think about it and if they will accept it so I have been looking around at what other interesting things I can do in that time (while waiting for another source of formal employment to resume).

I realized that I can't do much of anything without a UI so I have been looking into building some basic declarative web UI construction, since then.  Note:  This is VERY slow-going since I don't like doing UI work or JavaScript work so motivation has been an up-hill battle.  The other problem has been how this is mostly a bottom-up approach.  I don't have a clear picture of what I am going to end up with or why I should bother, which is making this into a "tower built via Brownian motion" kind of endeavour.

In any case, it is coming along.  The key idea was to build a generator which can create an interactive UI for reading/writing data of the defined format.  A secondary goal was to allow declarative data binding within that system.  For the most part, this is working.  I can define strings, structs, and arrays, potentially including "actions" within the structs.  These elements can then be bound to the same underlying data structure so that changes in one part of the model are immediately reflected elsewhere, etc.

I am now running into the question of what I am going to do with this.  The first goal is probably one of those mentioned [in my last post](https://jmdisher.github.io/Laminar-blog/Membrane-and-data-patterns/):  a chatroom.  This is easy enough:  An array containing a struct defining author, time, and content.  Additionally with a string and action for input.  I have a pretty good sense for how all of those things could be build directly on top of Laminar, probably augmenting Membrane for listening to updates.

The problem is that I would need the usual website boiler-plate around this to make it meaningfully useful.  First and foremost, this would involve a registration and login mechanism.  Looking around, there really isn't much innovation in this space (made worse by the trend toward centralization).  My first thought was to see if I could use my Ledger Nano to just sign a string:  I could register with a public key on the device and sign a challenge to login.  This would be great but there seems to be no such mechanism for this (and most things with the Ledger are strongly coupled to Chrome - so much for a decentralized future).  My next thought was to use GPG to do the same thing.  While somewhat more manual, it would still work.

Interestingly enough, [someone described doing exactly this](http://neverfear.org/blog/view/3/Secure_website_authentication_using_GPG_keys)!  Unfortunately, this seems to be the last mention of such an idea and it is from 2007.  So, I will probably still go this way but I will need to roll it all, myself.

Thinking about this further, it really is a shame that there isn't more support for things like this, from directly within the browser.  After all, this could be used to do away with session cookies almost entirely.  In fact, anything more sophisticated could be tacked on using JWT (for richer permissions, etc).  However, any action to write, or even read, site content could be done via such signed challenges.  This would probably be impractical (how would you get all the challenges without lock-stepping the entire system) but is interesting and could of course use the JWT for these smaller bursts of interaction, just using the challenge signature to refresh it.

The other matter is whether to build some kind of permissions system directly into Membrane or build a thin auth layer on top of it.  This gets a little odd since I wouldn't want this directly in Membrane (as it is more of a light-weight DB than back-end business logic - where this should be applied) but the actual logic to be applied here would be very small.  After all, the read/write actions of this system can be bolted directly onto Membrane, as is.

Looking higher in the stack, I could almost do it with HAProxy (which I suggest people use for SSL termination), but that isn't quite a fit, either.

I guess I have something to play with, even if it can only be used for establishing the initial login cookie (even the JWT refresh won't be appropriate without some kind of automated browser support).

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/18)
