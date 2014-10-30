---
layout: post
title: Forwarding Information Base
---

## Introduction

In the IPv4 routing subsystem, several critical structures should be discussed.

## fib_table##

Defined in `include/net/ip_fib.h`.

Instantiated in `struct fib_table *fib_hash_table(u32 id)` in `net/ipv4/fib_hash.c`.

We first concern about the routing table `fib_table`. According to *Linux Kernel Networking - Implementation and Theroy*, it is a table of entries where each entry determines which nexthop should be chosen for traffic destined to a subnet (or to a specific IPv4 destination address).

	struct fib_table {	
		struct hlist_node tb_hlist;
		u32             tb_id;
		int             tb_default;
		int             (*tb_lookup)(struct fib_table *tb, const struct flowi *flp, struct fib_result *res);
		int             (*tb_insert)(struct fib_table *, struct fib_config *);
		int             (*tb_delete)(struct fib_table *, struct fib_config *);
		int             (*tb_dump)(struct fib_table *table, struct sk_buff *skb, struct netlink_callback *cb);
		int             (*tb_flush)(struct fib_table *table);
		void            (*tb_select_default)(struct fib_table *table, const struct flowi *flp, struct fib_result *res);
		
		unsigned char   tb_data[0];
	};

 The fib_table holds a series of function pointers. And at the end of fib_table is a variable-length structure. As we can see in `struct fib_table *fib_hash_table(u32 id)`, this structure is a `fn_hash`.

## fn_hash ##

Defined in `net/ipv4/fib_hash.c`

	struct fn_hash {
		struct fn_zone  *fn_zones[33];
		struct fn_zone  *fn_zone_list;
	};

A packet is forwarded to its destination subnet, which has a subnet IP. The subnet positions have different lengths, which are not more than 32. Routing information of each destination subnet length is stored in one zone. Those zones is `fn_zones`.

The `fn_zone_list` is just a list of **fn_zone** sorted by their subnet position lengths.

## fn_zone ##

Defined in `net/ipv4/fib_hash.c`.

	struct fn_zone {
		struct fn_zone          *fz_next;       /* Next not empty zone  */
		struct hlist_head       *fz_hash;       /* Hash table pointer   */
		int                     fz_nent;        /* Number of entries    */
		
		int                     fz_divisor;     /* Hash divisor         */
		u32                     fz_hashmask;    /* (fz_divisor - 1)     */
	#define FZ_HASHMASK(fz)         ((fz)->fz_hashmask)
		
		int                     fz_order;       /* Zone order           */
		__be32                  fz_mask;
	#define FZ_MASK(fz)             ((fz)->fz_mask)
	};

As is mentioned before, a `fn_zone` is a zone containing a group of routing information, whose destination subnets have a same position length. They are stored in a hash table in `fn_zone`, called `fz_hash`.

## fib_node ##

Defined in `net/ipv4/fib_hash.c`.

	struct fib_node {
		struct hlist_node       fn_hash;
		struct list_head        fn_alias;
		__be32                  fn_key;
		struct fib_alias        fn_embedded_alias;
	};

The hash map `fz_hash` stores different routing information. And each entry is a `fib_node`. In `fib_node`, a `fn_key` is defined to represent the destination IP address.

## fib_alias ##

Defined in `net/ipv4/fib_lookup.h`.

	struct fib_alias {
		struct list_head        fa_list;
		struct fib_info         *fa_info;
		u8                      fa_tos;
		u8                      fa_type;
		u8                      fa_scope;
		u8                      fa_state;
	#ifdef CONFIG_IP_FIB_TRIE
		struct rcu_head         rcu;
	#endif
	};

A structure in `fib_node` is `fib_alias`, where stores type of service, type, etc. Besides these, a more important structure is `fib_info`.

## fib_info ##

Defined in `include/net/ip_fib.h`.

	struct fib_info {
		struct hlist_node       fib_hash;
		struct hlist_node       fib_lhash;
		struct net              *fib_net;
		int                     fib_treeref;
		atomic_t                fib_clntref;
		int                     fib_dead;
		unsigned                fib_flags;
		int                     fib_protocol;
		__be32                  fib_prefsrc;
		u32                     fib_priority;
		u32                     fib_metrics[RTAX_MAX];
	#define fib_mtu fib_metrics[RTAX_MTU-1]
	#define fib_window fib_metrics[RTAX_WINDOW-1]
	#define fib_rtt fib_metrics[RTAX_RTT-1]
	#define fib_advmss fib_metrics[RTAX_ADVMSS-1]
		int                     fib_nhs;
	#ifdef CONFIG_IP_ROUTE_MULTIPATH
		int                     fib_power;
	#endif
		struct fib_nh           fib_nh[0];
	#define fib_dev         fib_nh[0].nh_dev
	};

`fib_info` holds many routing states and other information. At the end of it, there's a variant-length structure called `fib_nh`, meaning the next hop.

## fib_nh ##

Defined in `include/net/ip_fib.h`.

	struct fib_nh {
		struct net_device       *nh_dev;
		struct hlist_node       nh_hash;
		struct fib_info         *nh_parent;
		unsigned                nh_flags;
		unsigned char           nh_scope;
	#ifdef CONFIG_IP_ROUTE_MULTIPATH
		int                     nh_weight;
		int                     nh_power;
	#endif
	#ifdef CONFIG_NET_CLS_ROUTE
		__u32                   nh_tclassid;
	#endif
		int                     nh_oif;
		__be32                  nh_gw;
	};

In `fib_nh`, we can see clearly which net device to forward the packet, and the next hop gateway.
