---
layout: post
title: AVM Integrated and Paper Draft
---

The AVM support merged to master a few days ago.  While the AVM would need some non-trivial modification to truly fit Laminar well, its default state allows for demonstration of the core thesis of Laminar.  Further changes may be considered but they would be more related to making the API a better fit for contracts (since there is currently a lot of _interpretation_ of the existing blockchain API) or improving performance (since none of the concurrent execution capabilities of the AVM are currently leveraged).

This means that it is now possible to break ground on the first draft of the white paper to describe what Laminar can do.  That, in its very roughest form (more of a _sketch_ than a _draft_, really), is on the Laminar wiki:  https://github.com/jmdisher/Laminar/wiki/Draft-White-Paper

The next core Laminar task is implementing persistence and restart support.  This shouldn't be too difficult, but some details will be hard to nail down (such as what consistitutes an atomic file operation - it is hard to find information around this when you focus on durability instead of cross-process visibility).  For the most part, the design of Laminar won't change but this does require some changes corresponding to some other pieces of data which represent _now_:

* list of valid topics
* contract transformed code
* contract object graph
* current cluster configuration

These questions are not terribly difficult but are far from trivial.

After that is done, there are probably only 2 other directions which may or may not be worth pursuing:

1. Modify AVM to be a better fit with Laminar and enable concurrent execution
1. Productize Laminar

I will likely need to see if I or someone else ends up using this before considering those points.

Also, I am trying out that idea of using GitHub issues to track feedback from blog posts so let me know what you think about any of this.

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/11)
