---
layout: post
title: All Hail the Micro-Commit
---

Anyone looking at the commit structure of Laminar may notice something odd:  The commits are generally quite small and a single feature often requires many (10+) commits to complete.  While this won't seem to make sense to everyone, I wanted to take a moment to explain.

The **unit of a change** in git, in fact most version-control systems, is the commit.  By this, I mean that the tooling can only talk about commits as atomic units, not as a collection of individual changes.

When you look at a **diff**, you are seeing the commit.  When you look at the **history of a file or project**, you are seeing a list of commits.  When you are using the **blame view** to get 4-dimensional context, you are seeing this context in units of commits.

Further, the tooling can only reason about changes in units of commits.  When **rebasing**, you are applying a sequence of commits to a new base revision.  When you are **bisecting**, you are looking for a problem introduced in a single commit.

Due to this, it is very valuable to keep commits small and tightly **focused on a specific idea**.  To this end, relatively few meaningful changes can be made in a single commit.  Bug-fixes sometimes can but new features almost never can.  This is because, when you think about it, new features are rarely just _the feature_ but all the supporting machinery to integrate it into an existing system.  This supporting machinery can often be considered as distinct from the core change to introduce or switch to a new feature.

When it comes to **collaborating with others**, small commits also makes much of what is changing easier to manage.  When doing a **code review**, for example, staring at a 1000+ line diff and claiming you understand it is just a lie.  Instead, this should be carved up into smaller pieces and you should walk through each of the commits **with the original author**.  This allows time to **discuss the implications** of each piece while also getting an understanding to how their change actually works and how it **interacts with the existing system**.

I have notice that my own code, and the code of those who I have reviewed using this process, has **substantially increased in quality** due to small commits and in-person (or at least in-voice) reviews.  When decomposing a large change or describing part of it to someone else is when you truly realize its **scope**, unexpected **implications**, or opportunities to **update documentation** or fill in **gaps in testing**.

Plus, there are few things as **satisfying** as investigating a regression and having `git bisect` show you exactly the idea which caused it or looking into some questionable code and having the `git blame` explain the context of why it is the way it is.  Not to mention the **velocity and de-risking** you can achieve by submitting the harmless plumbing of your change to production while you dive deeply into the last remaining corner case of your new feature.

The exact way to decompose a large change into a sequence of commits can be tricky and likely has many good approaches.  I will talk about my approach in greater detail, in the future.

For now, the key thing to remember is that **every commit must build and test clean** (modulo intermittent problems).  Beyond that, each one should represent **one complete idea**.  This second point is less clearly defined and everyone will put their own interpretation in a slightly different place but it is at least the **philosophical goal**.

Also, while you are at it, make sure your commit messages **describe the high-level meaning** of why the change exists.  It is an area where you pretty well can't have too much information.
