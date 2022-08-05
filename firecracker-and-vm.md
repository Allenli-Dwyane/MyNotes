# Notes on [Firecracker: Lightweight Virtualization for Serverless Applications](http://css.csail.mit.edu/6.858/2022/readings/firecracker.pdf) NSDI'20

## Background Knowledge on Virtual Machines

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
