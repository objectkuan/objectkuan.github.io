---
layout: post
title: OSV Page Structure 
---

While modifying the mechanism of OSV's mempool, I learn the structure of a page in OSV.

<!--more-->

In *mempool.cc*, according to `add_page`, which allocates a page, initializes it and adds it to the free list `_free`, the structure of a page in OSV looks like:

	|<----------- page size ---------->|
	|------|-----------|---|-----------|
	|Header|free_object|...|free_object|
	|------|-----------|---|-----------|

The page header holds the meta info of a page, including the cpu id, the owner mempool, allocated amount, and the free objects.

Each free object is just a continuous memory.
