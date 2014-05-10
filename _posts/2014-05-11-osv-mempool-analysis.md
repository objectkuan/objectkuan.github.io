---
layout: post
title: OSV Mempool Analysis 
---

In order to modify the mechanism of OSV's mempool, it's good to anaylize its implementation first.

That is the *mempool.cc*.

### add_page ###

The function add_page allocates a page, initialize it and add it to the free list *_free*.

From this function we can get the structure of a page in OSV.

	|<----------- page size ---------->|
	|------|-----------|---|-----------|
	|Header|free_object|...|free_object|
	|------|-----------|---|-----------|

The page header holds the meta info of a page, including the cpu id, the owner mempool, allocated amount, and the free objects.

Each free object is just a continuous memory.
