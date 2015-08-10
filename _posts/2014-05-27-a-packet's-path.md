---
layout: post
title: A Packet's Path 
tags: [Linux, Networks]
---

A nice backtrace captured by XF which shows a network packet's calling path in Linux kernel protocol stack.

<!--more-->

	#0 [ffff8800282030b0] machine_kexec at ffffffff8103210b
	#1 [ffff880028203110] crash_kexec at ffffffff810b7f05
	#2 [ffff8800282031e0] oops_end at ffffffff814ea360
	#3 [ffff880028203210] no_context at ffffffff810423fb
	#4 [ffff880028203260] __bad_area_nosemaphore at ffffffff81042685
	#5 [ffff8800282032b0] bad_area_nosemaphore at ffffffff81042753
	#6 [ffff8800282032c0] __do_page_fault at ffffffff81042e0d
	#7 [ffff8800282033e0] do_page_fault at ffffffff814ec33e
	#8 [ffff880028203410] page_fault at ffffffff814e96f5
    [exception RIP: skb_clone+208]
    RIP: ffffffff81415f40  RSP: ffff8800282034c0  RFLAGS: 00010246
    RAX: 000000000000f4cc  RBX: ffff88012bfb15c0  RCX: ffff880129a76000
    RDX: 0000000000000000  RSI: 0000000000000020  RDI: ffff88012fd10440
    RBP: ffff880028203510   R8: 0000000000000000   R9: d84156c5635688c0
    R10: 0000000000000000  R11: 0000000000000017  R12: ffff8801262b3df0
    R13: 0000000000000020  R14: ffff88012fd10440  R15: ffff8801261b6e60
    ORIG_RAX: ffffffffffffffff  CS: 0010  SS: 0018
	#9 [ffff880028203518] dev_hard_start_xmit at ffffffff81421d2e
	#10 [ffff880028203578] sch_direct_xmit at ffffffff8144093a
	#11 [ffff8800282035c8] dev_queue_xmit at ffffffff81426648
	#12 [ffff880028203618] neigh_resolve_output at ffffffff8142b668
	#13 [ffff880028203688] ip_finish_output at ffffffff8145f56c
	#14 [ffff8800282036c8] ip_output at ffffffff8145f800
	#15 [ffff8800282036f8] ip_local_out at ffffffff8145eae5
	#16 [ffff880028203718] ip_queue_xmit at ffffffff8145efd3
	#17 [ffff8800282037c8] tcp_transmit_skb at ffffffff814751de
	#18 [ffff880028203838] tcp_retransmit_skb at ffffffff814763ba
	#19 [ffff8800282038a8] tcp_xmit_retransmit_queue at ffffffff81478596
	#20 [ffff880028203908] tcp_ack at ffffffff814723f3
	#21 [ffff8800282039d8] tcp_rcv_state_process at ffffffff81473c38
	#22 [ffff880028203a18] tcp_v4_do_rcv at ffffffff8147b851
	#23 [ffff880028203ab8] tcp_v4_rcv at ffffffff8147d191
	#24 [ffff880028203b38] ip_local_deliver_finish at ffffffff814593cd
	#25 [ffff880028203b68] ip_local_deliver at ffffffff81459658
	#26 [ffff880028203b98] ip_rcv_finish at ffffffff81458afd
	#27 [ffff880028203bd8] ip_rcv at ffffffff81459095
	#28 [ffff880028203c18] __netif_receive_skb at ffffffff8142160b
	#29 [ffff880028203c78] netif_receive_skb at ffffffff81423968
	#30 [ffff880028203cc8] napi_skb_finish at ffffffff81423c90
	#31 [ffff880028203ce8] napi_gro_receive at ffffffff81426169
	#32 [ffff880028203d08] bnx2_poll_work at ffffffffa02ecd5f [bnx2]
	#33 [ffff880028203e18] bnx2_poll at ffffffffa02ed3a9 [bnx2]
