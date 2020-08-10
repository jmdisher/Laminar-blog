---
layout: post
title: Moving on to examples
---

Now that Laminar is effectively done, I am moving on to building 1-2 examples of how it can be used in actual practice.  The first such example is an in-memory key-value store with a basic REST interface.  Not only will this be a basic way to demonstrate how to interact with Laminar, but will also act as the basis for testing out packaging concerns, etc.

To start with, I am going to build and package this with Maven, since that will be more familiar and _automatic_ than my prefered system, Ant.  This will also give me an opportunity to play around with different ways of distributing Maven artifacts (since [something very promising and decentralized doesn't exist](https://github.com/AionJayT/MavenOnChain)).  This is somewhat annoying since Maven, while being a good dependency management system and passable build system, is a pretty terrible testing system.  Its inflexible-by-design approach has created a culture of poorly tested software.  This is going to be even more apparent for this project, than it would be for Laminar, since it has even more moving parts.  Bah, maybe I will just do the integration system testing in a shell script.  That will also make it more applicable and obvious (since `curl` invocations are pretty easy to understand and replicate).

The project will be interesting for another way, as well:  It will be a good example of "Focus on the field, not the entity".  That is a refrain from a talk I watched several years ago where someone was discussing micro-service designs, pointing out that micro-service architectures around key-value stores often miss the point by just shoving massive documents into storage and then trying to make sense of them or evolve them, much like a schema upgrade.  The argument of this talk was that doing this is focussing on the "entity" whereas designs should be embracing the natural decoupling of such systems and focus on dealing with the "fields", specifically.

This idea really stuck with me as it is simple and quite elegant.  Instead of thinking about these composite units as first-class citizens (much as would be required for designing an SQL table schema - nothing against SQL, it is awesome in its own ways), it allows these composites to be ad-hoc read-only views of an arbitrarily complex collection of attributes.  This creates a flat data model where each field is considered an ends unto itself and is only united by the foreign key which is the UUID of whatever "root" concept is considered in the system.  This reduces the data modeling scope and rigidity to defining only these roots and their join points, but nothing else.

This works very well with Laminar, where it is heavily simplified by restricting messages to 64 KiB.  A 64 KiB "user document" is very small, but a 64 KiB field is huge.  Further, this allows validation programs to be defined on individual fields, independently, which makes sense:  I don't care what your phone number is when determining if the balance transfer is valid.

So, the design of this system will be based around read-write interaction with various fields and read-only views of compositions.  I looked at the potential of using something like GraphQL for this but it seemed to include functionality which didn't really map, and I am not entirely sure is a great idea (or at least one which can be generally implemented in way to give consistent responses with reasonable overhead).  Of course, I suspect its value is more related to external data consumers, where their queries can't otherwise be effectively described, as opposed to internal data flow (so, it would be overkill, at best, for this example).

Playing around with this has also brought me back to playing with Jetty.  Whenever I need to use this, I wonder why anyone uses Spring for web services.  Jetty works so well, is so easy to hook into, is so light-weight, embeds without forcing you to do things in odd ways, and starts up so quickly (a few hundred ms versus multiple seconds).  It really is a dream to use when you need to hook into a web server.

In any case, I will probably post this as a new GitHub project within the next few days.  I need to get it to the point where it can do enough to actually demonstrate something useful, but should be a nice project to play with.  For now, I will just embed the Laminar client in a faked-up, project-local Maven repo until I settle on a solid plan and create a release I want to keep around forever (part of why project-local resources are nice - you never download a lemon and it will not fail to find a dependency at some future point).  GitHub Packages doesn't sound that great so I may just use the approach of keeping a branch with built products in it (a basic solution without more infrastructure requirements).

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/16)