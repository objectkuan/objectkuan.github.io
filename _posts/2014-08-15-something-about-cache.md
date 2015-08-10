---
layout: post
title: Something about Cache
tags: Linux
---

Currently I'm optimizing the performance of the well-known in-memory key-value server - redis, from the system and network perspectives.

<!--more-->

That means codes of redis are not expected to be modified. What to do is analyze and propose optimization on network configuration or kernel modification.

All started from connection locality researching, followed by scalability analysis. The lock issues and cross-NUMA acceses are taken into research, and finally everything stopped at the LLC issues. LLC, which is known as the last level cache, is a 16MB space cache shared by CPU cores, just before they access the main memory.

With multi instances and high accessing concurrency, the LLC miss rate can go up to 40%. And this harms the scalability of redis service a lot. After deeper survey, quite a few misses arise while accessing members of some large data structure such as skb.

It attracts me to watch how the members are accessed of skb. With hardware breakpoints (which is also known as watchpoints), we can keep watchpoints on memory accessing. But each watchpoint can only monitor the access of one byte (I'm not sure it's one byte or sevral bytes, but it couldn't monitor the R/W accesses of the address 4 bytes beyond the registered address). On an x86_64 computer, at most four watchpoints can be set, but this is far from enough for a large struct.

Referring to related work in MIT, if monitor four certain bytes each time, the whole many-byte structure's accessed pattern is restored after repeated monitoring. This is a very mathematical and I think emulator may help, too.
