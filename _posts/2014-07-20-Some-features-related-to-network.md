(0) The SMP Affinity of IRQ
  Description: bind an IRQ to a core, when an interrupt is triggered on an IRQ, it'll be handled by a certain core
  Checking:
		If not set, the %si of some core(s) may run high.
		IRQ balance should be disabled to avoid modification of affinities. 
	Setting:
		`echo [mask] > /proc/irq/[irq_no]/smp_affinity`

(1) Interrupt Throttle Rate
	Description: Control the interrupt trigger rate of a NIC, to let the kernel read more packages from NIC for one interrupt.
	Checking:
		The interrupt rate of a NIC looks abnormal, for example, very high, if this is not set properly.异常情况，网卡产生的中断速率异常升高
		`cat /proc/interrupt # to get one or more IRQ ids of a nic`
		`sar -I [int_id,int_id,...] 1`
		In my case, 3000 is an acceptable value.
	Setting:
		The `options ixgbe` in `/etc/modprobe.d/modprobe.conf`
		Also can be set by `ethtool -C [eth5] rx-usecs 333`

(2) ATR
	Description: Implemented by hardware, it samples the packages, for example, while they are sent by NIC. It takes a quintuple as the key, a CPU core as the value to store entries in a flow table. When a package arrive, the NIC will scan the flow table for a target core to gain connection locality.
	Checking:
		`ethtool -S [eth5] | grep -i fdir` to check the hits and misses
	Setting:
		For Intel 82599, ATR will be used when NTUPLE is down.

(3) NTUPLE
	Description: Writing rules to dispatch received packages to certain queues.
	Checking:
		`ethtool -k [eth5]` to check nutple is whether on or off.
	Setting:
		ethtool -K eth5 ntuple [on|off]

(4) RSS
	Description: Receiving queues of a NIC. when a package arrives, it hash the quintuple to a certain IRQ.
	Checking:
		If NTUPLE is turned on (so ATR is skipped) but no rule is hit
		`ethtool -k [eth5]` to check ntuple is on
		`ethtool -u [eth5]` to check no rule is hit
		Meanwhile `ethtool -S [eth5] | grep -i fdir` will show some misses.
	Setting:
		Turn on NTUPLE: ethtool -K eth5 ntuple on
		Clear the NTUPLE rules: ethtool -U [eth5] delete xxx
