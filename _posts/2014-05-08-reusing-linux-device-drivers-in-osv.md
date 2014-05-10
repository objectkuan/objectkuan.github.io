---
layout: post
title: Reusing Linux Device Driver in OSV
---

I still cannot figure out all the details how the linux drivers work.

And OSV kernel is not simple.

Tough days start from now on.

## Working report ##

Now, Linux virtio block and net drivers has been copied to the OSV source. And with the help of DDE (Device Driver Environment), compiling and linking is not a problem now, i.e. the syntax problem is solved.

But the sematic problem puzzles me. As the OSV initialization goes on, page fauls and lock errors follows. That must be the problem that OSV's kernel has too many assumptions for its drivers, while the ported Linux drivers violates them.  


## Working report ##

After analysing the codes, I try the simple case that only the Linux virtio block device is used in OSV, and it crashes sometimes.

I found that while thread A is doing something on the mempool (alloc or free), an interrupt is fired and the interrupt routine also performs some operations on the mempool.

OSV's mempool management doesn't provide enough protection in such case.
