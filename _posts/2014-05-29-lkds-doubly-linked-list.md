---
layout: post
title: Linux Kernel Data Structure - Doubly Linked List
tags: [Linux, Data Structure]
---

The doubly linked list is a commonly used data structure. The Linux kernel define the doubly linked list in the file <include/list/list.h>.

<!--more-->

	struct list_head {
		struct list_head *next, *prev;
	};

And the structure looks like:

	  ┏━━━━━━━━━━┓  ┏━━━━━━━━━━┓  ┏━━━━━━━━━━┓
	  ┃ list_head┃  ┃ list_head┃  ┃ list_head┃
	  ┣━━━━━━━━━━┫  ┣━━━━━━━━━━┫  ┣━━━━━━━━━━┫
	┏>┃   next   ┃━>┃   next   ┣━>┃   next   ┣━┓
	┃ ┣━━━━━━━━━━┫  ┣━━━━━━━━━━┫  ┣━━━━━━━━━━┫ ┃
	┃┏┫   prev   ┃<━┫   prev   ┃<━┫   prev   ┃<╋┓
	┃┃┗━━━━━━━━━━┛  ┗━━━━━━━━━━┛  ┗━━━━━━━━━━┛ ┃┃
	┗╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛┃
	 ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

When we want to use such a list of something, for example, if we have a structure called node, and we want a doubly linked list of `struct node`, we can define it as:

	struct node {
		int data;
		struct list_head list;
	}

And a piece of sample codes is:

	struct list_head head, *plist;
	struct node a, b;
	
	a.data = 1;
	b.data = 2;
	
	// Initialize a list
	INIT_LIST_HEAD(&head);
	list_add_tail(&a.list, &head); // Add to list
	list_add_tail(&b.list, &head); // Add to list

	list_for_each(plist, &head) // Iterating
	{
		// list_entry is used to get the struct of a node
		struct node *node = list_entry(plist, struct node, list);
		// do something with node...
	}
	
More operation interfaces can be find in the .h file.
