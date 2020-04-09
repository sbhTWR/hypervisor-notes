# hypervisor-notes
Notes I prepared while reading "Definitive Guide to the Xen hypervisor"

### Chapter 1 ###
-> Motivation for virtualization:
	1. Same as that of multi-tasking operating systems - if there are spare resources, why not utilize them? Organisations find that they have a lot of servrs all doing single tasks, or small clusters of tasks doing rlated job. Virtualizaition => number of virtual servers consolidated into a single physical machine, without losing the security gained by having isolated environments. A single job does not take up the entire rack dspace in the data center.
	2. Fault tolerance: Organisation runs multiple VMs for the same task, in case one fails. Easy to migrate VM accross physical servers (Keep redundant VM images sync. across physical machines)
	3. Clone VMs at a low cost: Easy to introduce patch into a production system. Clone VM, apply patch and see what breaks. Difficult to keep a production and a test machine in the same state.
	4. Migration: If hardware faults/upgrade scheduled => migrate VM to another physical machine. When original machine is restored, migrate back.
	5. Power usage: Consolidate services running in VMs on a smaller number of hosts can reduce the power costs considerably.
	6. Portability: While moving sevvices, can even save the state of a VM into a USB flash drive, (or maybe an iPod!). When you want to use it, plug it in and restore.
	7. Isolation: VM provides much greater degree of isolation than a process in OS. Read about virtual appliances (?) They dont take up any space (?), can be easily duplicated and run on more nodes if too heavily loaded ( or allocate more runtime on a large machine)

-> x86 Problem: Even without a virtual mode, processor would be virtualizable if (Popek and Goldberg), the set of control sensitive instructions is a subset of the set of priviledges instructions => Any instruction that modifies the configuration of resources in the system should either execute in priviledged mode OR trap if it doesn't. There are 17 instructions in x86 instruction set that do not satisfy this property.
	Example: LAR, LSL - Load info about a specific segment
			SIDT

-> Solutions: 
	1. Binary rewriting (VMware): 
	- Most virtual environement runs in userpscae, but performance penalty. Scan and identify sensitive instructions that are not privileged and re-written to point to emulated versions. Performance worse when doing I/O intensive jobs. Aggressive caching of locations of unsafe instructions can give speed boost.
	- Works similar to debugger with HW support - use DR0 and DR1 to point to instructions that need to be emulated after scanning them. When jmp encountered, scan the target and load DR0 and DR1 accordingly.
	2. Paravirtualization:
	- Don't care about the guest executing unsafe instructions. It will deal with consequences! So the instruction reqriting happens at compile time, rather than runtime.
	- In case of Xen, OS runs on ring 1 instead of ring 0. So it cannot perform any privileged instructions. Map unsafe instructions to hypercalls. Hypercalls similar to systemcalls in that push a value to eax and then execute int 80h to interrupt.
	- In case of Xen they use int 82h.
	- When int 80h is executed by the application, it gets trapped by hypervisor, which passes the control back to the OS. Unmodified apps can run, but small overhead.
	- Direct system calls allowed, but with modification of libc. 
	- Xen, linux, MS-DOS (not FreeBSD) Hypercall parameters stored in registers (ebx) rather than stack.
	- Recent Xen: hypercalls through a function in a shared memory page, with args in registers. Allows fast transitions to and from ring 0 (Intel and AMD)
	3. HW-assisted virtualization (Hardware virtual machine - HVM): 
	- Intels VT-x: Kind of adding ring -1, ring -2, etc... so that OS is where it expects to be, excep that a new privilege mode that can trap instructions.
	- ring -1 <=> VMX root mode. 
	- + A bitmap indicating whether instruction should be passed to the OS in ring 0 or to hypervisor running in VMX root mode.
	- AMD-V: 
		-- Shadow page tables:
			Mark page tables as read only, catch the fault to hypervsior instead of guest kernel, and then map them to real physical addressed.
		-- Nested page tables:
			Nested page tables allow this to be done in hardware. 
			MMU: virtual -> physical
			HW-assists: physical -> real physical
		-- allows running of unmodified operating systems. Unmodified guests however do not know of virtual environments, so they cannot make use of hardware support, so slower.
		-- VMware uses hybrid system: some virtualization in hardware, some in software (best of both worlds).
		-- HW assistance: A) faster transitions to ring 0 and back (because of ring -1: because we tell HW whether to give instruction to hypervisor or the guest OS at ring 0)	B) Lesser hypercalls for virtual memory operations, HW support for physcal to real physical. 
		-- Paravirtualization: Faster I/O interface, rather than emulated hardware.

-> Xen philosophy:
	1. Separate policy and Mechanism
		Xen => implement mechanisms
		Dom 0 guest => policies
	 	If a device can be accessed, look in XenStore. Multiple guests want to use it? look there. I/O device advertised in XenStore. But if you want to access it, guests co-operate with some policy, not enforced by Xen.
	2. Xen code base made as small as possible to audit and ensure bug-free. Xen implements shared memory for communicating between domains. How it is used is upto guests.
	3. Ease of administration - rely on Domain 0 features, because a Linux admin has already spent a lot of time learning it.
	4.  

