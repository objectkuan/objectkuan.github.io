---
layout: post
title: Linux Kernel Data Structure - Hash List
categories: [Linux Kernel, Data Structure]
---

Hash lists are different from normal doubly linked lists.

As we discussed before, dobuly linked lists have a unique node type regardless of the head or the others, which contains a prev pointer and a next pointer.

When we implement a hash table, if we use normal doubly linked lists to store the item, each list will have a head node containing two pointers. If the hash table is large, each head with two pointers waste much space. But with a hash list whose head has only one pointer, we can save half of the space.

	  ┏━━━━━━━━━━━━┓
	  ┃  HashTable ┃
	  ┣━━━━━━━━━━━━┫
	  ┃     ...    ┃
	  ┣━━━━━━━━━━━━┫   ┏━━━━━━━━━━━┓   ┏━━━━━━━━━━━┓
	┏━╋━>hlist_head┣━━>┃ hlist_node┃┏━>┃ hlist_node┃    
	┃ ┣━━━━━━━━━━━━┃   ┣━━━━━━━━━━━┫┃  ┣━━━━━━━━━━━┫
	┃ ┃     ...    ┃ ┏━╋━> next    ┃┛  ┃    next   ┣━>...
	┃ ┗━━━━━━━━━━━━┛ ┃ ┣━━━━━━━━━━━┫   ┣━━━━━━━━━━━┫
	┗━━━━━━━━━━━━━━━━╋━┫    pprev  ┃   ┃    pprev  ┣━┓
	                 ┃ ┗━━━━━━━━━━━┛   ┗━━━━━━━━━━━┛ ┃
	                 ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

Definition in <include/linux/list.h>:

	struct hlist_head {
		struct hlist_node *first;
	};
	
	struct hlist_node {
		struct hlist_node *next, **pprev;
	};

According to the definition, the head of hast list contains only one pointer. And each node contains two. Another difference is that each node holds the pprev instead of prev. 

The pointer `pprev` points to the next pointer of the prev node. This brings in convenience while implementing the list operations like `hlist_del`. `hlist_head` is different from `hlist_node`, so we need such a unified mechanism to treat the first node and the following ones.

More operation interfaces are in the .h file.
