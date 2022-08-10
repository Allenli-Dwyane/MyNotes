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

### Motivation

* The economics and scale of serverless applications demand that workloads from multiple customers run on the same hardware with minimal overhead, while preserving strong security and performance isolation. 

* Two solutions can be used to address such isolation problems:
  
  1. containers, such as Docker or LXC
  2. hypervisor-based virtualization, as used in AWS EC2

  Both solutions obtain their own advantages and shortbacks. 
  
  * Containers are more light-weighted (introducing less overheads) and flexible (users can implement their own authority level to the containers), but they suffer from the trade-off between compatibility and security. Implementing stronger security imposes more strict limits on the syscalls available in the containers, but limited syscalls constrain the code compatibility inside a container (e.g., some syscalls might be missing).

  * Hypervisor-based virtualization provides a full execution environment for serverless computing, making it no need to worry about code compatibility. However, hypervisor-based virtualization is expensive and inefficient in terms of performance and  resource utilization.

### Contribution

* Firecracker, an open source Virtual Machine Monitor (VMM) specialized for serverless workloads, but generally useful for containers, functions and other compute workloads within a reasonable set of constraints.

* replace QEMU to build a new Virtual Machine Monitor (VMM),
device model, and API for managing and configuring MicroVMs. Firecracker contains about 50k lines of Rust code, which is a lot less than the original QEMU codes.

### Goals 

* Isolation: "It must be safe for multiple functions to run on the same hardware, protected against privilege escalation, information disclosure, covert channels, and other risks."

* Overhead and Density: possible to run numerous of functions on a single machine with minimal overheads

* Performance: running functions in serverless cloud service should be similar to running in the native environment, and should also be consistent.

* Compatibility

* Fast Switching

* Soft Allocation

### Implementation Details

* Overview: uses the Linux Kernel’s KVM virtualization infrastructure to provide minimal virtual machines (MicroVMs), supporting modern Linux hosts, and Linux and OSv guests; provides REST-based API, device emulation, networking and serial console and configurable rate limiting for network and disk throughput and request rate.

* Design Philosophy:
  
  1. **One Firecracker process runs per MicroVM.**
  2. **rely on components in Linux rather than re-implementing**. Reasons for this include the long-proven quality of Linux components and most users are programmers who have long used to the Linux programming model.

#### Device Model

limited number of emulated devices, including network and block devices (for storage rather than file system passthrough), serial ports and partial i8042 support.

#### API

REST API for configuring, managing and stoping MicroVMs.

#### Constraints and Configuration on Machines

* The amount of memory and number of cores used by each MicroVM can be configured via REST API, so do the cpuid bits.
  
* The limits imposed on block and network devices, such as IOPS and bandwidth, can also be configured via the built-in API. This rate limiters are similar to cgroups in containers, but less flexible and powerful.

#### Security

Implements jailer, an additional level of protection against unwanted VMM behaviors. The jailer implements a wrapper around Firecracker which places it into a restrictive sandbox before it boots the guest, including running it in a chroot, isolating it in pid and network namespaces, dropping privileges, and setting a restrictive seccomp-bpf profile.

### Implement Firecracker in Real Production

The architecture of AWS Lambda:

* Invoke traffic arrives via the *Invoke* REST API and is authenticated and checked for authorization. During this process, metadata is loaded.

* The Worker Manager takes the job of routing. Once identifies the next worker to run the code on, it advises the invoke service to send the payload directly to the worker. A concurrency protocol is deployed to solve race conditions.

* Each lambda worker offers a number of *slots*. Each slot provides a pre-loaded execution environment for a function and only for such function.

* The Placement service is used to request a new slot be created for the function when no slot is available. Also, Placement is responsible for limiting the lifetime of slots, terminating slots and other similar activities.

The architecture of the Lambda Worker (where Firecracker plays its role):

* Each worker runs thousands of MicroVMs (each providing a single slot) according to the resources each VM consumes. Each MicroVM is set up and managed by one Firecracker process,and provides the minimal environment for a single customer function.

* MicroManager is a per-worker process which is responsible for managing the Firecracker processes. It communicates the information of details of slot size with the shim process inside a MicroVM via TCP/IP. It also keeps a small pool of pre-booted MicroVMs, ready to be used when Placement requests a new slot.

Soft-allocation:

* Each slot has three states: initializing, idle, busy. The slot in each slot consumes different amounts of resources.


## Some Thoughts

The novelty of Firecracker lies in that as a VMM, Firecracker is able to adjust the scale of the MicroVM according to the requested services, while providing a full execution environment. In my opinion, since that each MicroVM runs one function, the MicroVM seems like a 'hyper-container', which executes a single function with small scale, just like a container, but obtains a full environment, more than just a container.

Anyways to crack this service?

I can not think of many potential scenarios. Since everycode inside MicroVM is untrusted, it would be hard for attackers to use their own codes to crash this service. Perhaps some vulnerability might occur during the re-use of each MicroVM? I am pretty confused.