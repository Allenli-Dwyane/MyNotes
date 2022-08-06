# Notes on [Firecracker: Lightweight Virtualization for Serverless Applications](http://css.csail.mit.edu/6.858/2022/readings/firecracker.pdf) NSDI'20

## Background Knowledge on Virtual Machines and Containers

### Hypervisor (or Virtual Machine Monitor)

According to a [RedHat blog](https://www.redhat.com/en/topics/virtualization/what-is-a-hypervisor), "A hypervisor is software that creates and runs virtual machines (VMs).  A hypervisor, sometimes called a virtual machine monitor (VMM), isolates the hypervisor operating system and resources from the virtual machines and enables the creation and management of those VMs."  

The duties of a hypervisor:

* reallocate resources (e.g., CPU, memory, storage) among existing VM guests or to new virtual machines.
* perform some operating system-level functions, e.g., memory management, process schedule, security and etc., to run VMs
* summarizing its duties, "the hypervisor gives each virtual machine the resources that have been allocated and manages the scheduling of VM resources against the physical resources."

Two types of hypervisors:

* Type 1: also referred to as a native or bare metal hypervisor, runs directly on the host’s hardware to manage guest operating systems. It takes the place of a host operating system and VM resources are scheduled directly to the hardware by the hypervisor. 
  > commonly seen in data centers or server-based environments. For example, KVM, Microsoft Hyper-V.

* Type 2: also known as a hosted hypervisor, and is run on a conventional operating system as a software layer or application. It works by abstracting guest operating systems from the host operating system. VM resources are scheduled against a host operating system, which is then executed against the hardware. 
  > better for individual users who want to run multiple OSs on persornal computers, e.g., Oracle VirtualBox.

### KVM (Kernel-based Virtual Machine)

> This note is mainly based on [this RedHat blog](https://www.redhat.com/en/topics/virtualization/what-is-KVM)

By definition, according to [this RedHat blog](https://www.redhat.com/en/topics/virtualization/what-is-KVM), KVM is "an open source virtualization technology built into Linux®. Specifically, KVM lets you turn Linux into a hypervisor that allows a host machine to run multiple, isolated virtual environments called guests or virtual machines (VMs)".

The features of KVM:

* type 1 hypervisor, i.e., can run on bare-metal and need some OS components. **KVM is part of the Linux kernel**, and every VM is implemented as a regular Linux process. Also, **Linux is part of KVM, either.**

* Use a combination of security-enhanced Linux (SELinux) and secure virtualization (sVirt) for enhanced VM security and isolation.

* able to use any storage supported by Linux, including some local disks and network-attached storage (NAS). Multipath I/O may be used to improve storage and provide redundancy. KVM also supports shared file systems so VM images may be shared by multiple hosts. Disk images support thin provisioning, allocating storage on demand rather than all up front.

* KVM can use a wide variety of certified Linux-supported hardware platforms.
  
* KVM inherits the memory management features of Linux, including non-uniform memory access and kernel same-page merging. 

* KVM supports live migration, which is the ability to move a running VM between physical hosts with no service interruption. 

* In the KVM model, a VM is a Linux process, scheduled and managed by the kernel.

### Pulling Together: Virtual Machines

> This part is mainly based on [this RedHat blog](https://www.redhat.com/en/topics/virtualization/what-is-a-virtual-machine).

By definition, "a virtual machine (VM) is a virtual environment that functions as a virtual computer system with its own CPU, memory, network interface, and storage, created on a physical hardware system (located off- or on-premises)."

Hypervisor: a software that separates the machine’s resources from the hardware and provisions them appropriately so they can be used by the VM. (As is shown above.)

Host-guest model: The physical machines, equipped with a hypervisor , is called the host machine; The many VMs that use its resources are guest machines.

### Containers

See [this note](./linux-container.md) for details.

## [Firecracker: Lightweight Virtualization for Serverless Applications](http://css.csail.mit.edu/6.858/2022/readings/firecracker.pdf)