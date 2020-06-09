---
layout: post
title: Select on Regular Files
---

While sketching out the design for the `DiskManager`, I noticed something surprising:  [FileChannel](https://docs.oracle.com/javase/7/docs/api/java/nio/channels/FileChannel.html) does not descend from [SelectableChannel](https://docs.oracle.com/javase/7/docs/api/java/nio/channels/SelectableChannel.html).  This means that `select()` cannot be applied to regular files in NIO.

This really surprised me as I was sure that `select()` could be used in precisely that way.  In my confusion, I went looking around and found [this thread on Stack Overflow](https://stackoverflow.com/questions/3955250/why-filechannel-in-java-is-not-non-blocking) discussing precisely this.  The chosen answer provides a link to [this blog post](https://www.remlab.net/op/nonblock.shtml) which explains the matter in greater detail.  The short answer is (as quoted from that post):  _"Regular files are always readable and they are also always writeable. This is clearly stated in the relevant POSIX specifications."_

Mind blown...

While I am thinking about it, that post has some other great points, as well, so **I recommend taking a read** (it is also concise, so the value density is high).  I had a similar argument, back in the days of the Chrome graphics stack, regarding _blocking_ versus _sleeping_ (although there were other factors in that situation - I made a valid point, but wasn't _totally correct_).

Still surprised by this, I decided to check it out for myself.  Sure enough, writing a small C program to test this does appear to imply this is the case in both specification and implementation (on my Ubuntu laptop - BtrFS on hard disk).  Writing 10K to a file in a loop took about `4 us` in `select()` but `25-33 us` in the `write()` (although it later changed to `0.9 us` and `6 us`, as the file grew).  Reading this same file showed about `3 us` in `select()` and `5-12 us` in `read()`.  All writes and reads also managed to use the entire 10K buffer in a single call.

While these numbers are so small that it isn't completely clear, it is an interesting pattern which implies that this does work as the blog implied.  The fact that even the man page for `write()` doesn't mention the possibility of any writing buffer in the kernel being full (which should allow non-blocking back-pressure requiring `select()` - essentially what the network does), and the page for `read()` was a little vague on this point does seem to imply that it might not be possible to normally do non-blocking file reads or writes.

While this isn't a big deal for Laminar (all invidiual records are smaller than 64 KiB), it doesn't sound right and I must be missing some detail, here.  Can anyone provide richer clarification?  Is this something I would see if larger buffers were used (I tried 100K and no real difference - IO calls just took longer but completed, in full)?

This is one of the nice things about the Laminar project, though:  There are many things I believed that I knew how to build but never had, and I wanted this opportunity to make sure I understood those things as well as I thought I did.  This is the first part where I actually realized my assumption was completely wrong.

Given these results and the small Laminar record size, this does mean a very simple blocking `DiskManager` design will be sufficient (it already runs in its own thread so this won't impact blocking behaviours of other components).

[Comments and Discussion](https://github.com/jmdisher/Laminar-blog/issues/12)
