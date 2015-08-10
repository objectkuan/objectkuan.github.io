---
layout: post
title: Reusing Linux Device Driver in OSV
tags: OSV
---

I still cannot figure out all the details how the linux drivers work.

And OSV kernel is not simple.

Tough days start from now on.

<!--more-->

## Working report ##

Now, Linux virtio block and net drivers has been copied to the OSV source. And with the help of DDE (Device Driver Environment), compiling and linking is not a problem now, i.e. the syntax problem is solved.

But the sematic problem puzzles me. As the OSV initialization goes on, page fauls and lock errors follows. That must be the problem that OSV's kernel has too many assumptions for its drivers, while the ported Linux drivers violates them.  


## Working report ##

After analysing the codes, I try the simple case that only the Linux virtio block device is used in OSV, and it crashes sometimes.

I found that while thread A is doing something on the mempool (alloc or free), an interrupt is fired and the interrupt routine also performs some operations on the mempool.

OSV's mempool management doesn't provide enough protection in such case.


## Working report ##

The backtrace while OSV aborts indicates that the error arises in the function *free* in core/mempool.cc. So why not have a look at the codes there.

	void pool::free(void* object)
	{
	    trace_pool_free(this, object);
	    WITH_LOCK(preempt_lock) {
	
	        free_object* obj = static_cast<free_object*>(object);
	        page_header* header = to_header(obj);
	        unsigned obj_cpu = header->cpu_id;
	        unsigned cur_cpu = mempool_cpuid();
	
	        if (obj_cpu == cur_cpu) {
	            // free from the same CPU this object has been allocated on.
	            free_same_cpu(obj, obj_cpu);
	        } else {
	            // free from a different CPU. we try to hand the buffer
	            // to the proper worker item that is pinned to the CPU that this buffer
	            // was allocated from, so it'll free it.
	            // assert(arch::irq_enabled());
	            free_different_cpu(obj, obj_cpu);
	        }
	    }
	}

When free is called, the object to be reclaimed may be owned by the current cpu or another one. So OSV mempool perform this action in two ways, in *free_samp_cpu* and *free_different_cpu*.

And as we can see, the operation in either case is protected by a preempt lock, which means that, during the freeing process, preempt is disabled.

In free_same_cpu, of course, this can be guaranteed. But in free_different_cpu, the preempt lock will be dropped for a while.

Anyway, we are sure that a preempt lock is used to protect the free operation. If we use an irq lock instead, any interrupt is disabled during the memory operation, then the problem may solve. 

So I implement a lock with both preemption and irq features, using some functions from the DDE:

	class irq_preempt_lock_t {
	private:
	spinlock_t spinlock;

	int dde_spin_lock_irqsave() {
	    int ret = arch::irq_enabled();
	    arch::irq_disable_notrace();
	    spinlock.lock();
	    return ret;
	}

	void dde_spin_unlock_irqrestore(int enabled) {
	    spinlock.unlock();
	    if (enabled)
		arch::irq_enable();
	}

	public:
	    void lock() {
		sched::preempt_disable();
		irq_preempt_lock_irq = dde_spin_lock_irqsave();
	    }
	    void unlock() {
		dde_spin_unlock_irqrestore(irq_preempt_lock_irq);
		sched::preempt_enable();
	    }

	    irq_preempt_lock_t()  {}
	    ~irq_preempt_lock_t() {}
	};

Then OSV with a Linux virtio block driver works in a single-cpu mode. I am so happy about that, although it still crashes in the multi-cpu mode.

### Work Report ###

Since free_same_cpu works fine, a way to fix this bug is to avoid the calling of free_different_cpu.

I'm trying to set up a request queue for each cpu, storing the free requests on each cpu. Instead of calling free_different_cpu, a request enters the queue of the owner cpu of the victim object (the victim object means the object to be freed). 

And when free_same_cpu is called, mempool goes through the queue of that cpu, to handle the freeing requests.

This may cause other problems, maybe introducing performance overhead, accumulating useless data.

### Work Reprot ###

According to the design, I simply try to add an array of std::queues in mempool class.

	typedef std::queue<free_object*> delegate_queue;
	delegate_queue dqueues[sched::max_cpus];

And modify the free function:

	delegate_queue& dq = dqueues[obj_cpu];
	if (obj_cpu == cur_cpu) {
		while(!dq.empty()) {
			free_same_cpu(dq.front(), obj_cpu);
			dq.pop();
		}
		free_same_cpu(obj, obj_cpu);
	} else {
		dq.push(obj);
	}

The irq_preemt_lock is not neccessary now, and with the origin preempt_lock, the Linux virtio block driver works in OSV.
