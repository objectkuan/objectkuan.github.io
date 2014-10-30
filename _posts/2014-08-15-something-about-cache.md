---
layout: post
title: Something about Cache
---

Currently I spent much time optimizing the performance of the well-known in-memory key-value server - redis, from the system and network perspectives.

<!--more-->

That means I'm not expected to modify any line of codes of redis, but to analyze and propose optimization on network configuration or kernel modification.

My work started from connection locality researching, which was followed by scalability analysis. I worked through the lock issues, cross-NUMA acceses, and finally stopped at the LLC issues. LLC, which is known as the last level cache, is a 16MB space cache shared by CPU cores, just before they access the main memory. 

In my redis work, I found that with multi instances and high accessing concurrency, the LLC miss rate can go up to 40%. And this does harm the scalability of redis service. According to deeper survey, quite a few misses arise while accessing members of some large data structure such as skb.

I'm interested in it and drive myself to watch how the members are accessed of skb. With hardware breakpoints, which is also known as watchpoints, I can keep watch on memory accessing. But unfortunately, each watchpoint can only monitor the access of one byte (I'm not sure it's one byte or sevral bytes, but it does failed to monitor the R/W accesses of the address 4 bytes beyond the registered address). On my x86_64 platform, at most four watchpoints can be set, but this is obviously not enough for a large struct.

Of course I think it may be a way to monitor four certain bytes each time, and after many times, the whole many-byte structure's accessed pattern is restored.

I feel that's quite mathematical and insteresting.

But I think emulator may help.

