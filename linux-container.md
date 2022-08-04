# Linux Containers (LXC)

This is mainly a note for studying LXC on this [website](https://raydenchia.com/linux-containers-lxc/), which is also a note for the MIT 6.858 course.

* Definition
  
  ```
  A container is a collection of one or more processes that are isolated from the rest of the system.
  ```

* Why containerization important?
  
  * A solution to software portability: a container contains a full operating environment for software deployment, thus software of different versions can be deployed and tested in multiple environments.
  * A safer execution environment for external programs.

* Six Linux features for containerization
  
  1. cgroups
  2. capabilities
  3. seccomp
  4. mandatory access control
  5. namespaces
  6. chroot jails
   
## chroot

The job of chroot: **changes apparent root directory for current running process and its children.**

The man page of chroot(2) is as follows:

```
NAME
       chroot - change root directory
SYNOPSIS
       #include <unistd.h>

       int chroot(const char *path);
```
Limitations:

  * May be broken by a second chroot operation. 
  * Not intended for any kind of security purpose, nor sandboxing processes.

Breaking chroot(), see this [website](https://web.archive.org/web/20160127150916/http://www.bpfh.net/simes/computing/chroot-break.html#notes). The following steps require that the program is run with root privileges.
  
  * Create a temporary directory in its current working directory
  * Open the current working directory
  * Change the root directory of the process to the temporary directory using chroot().

The above three steps create a chroot()ed area, next we need to break from it.

  * Use fchdir() with the file descriptor of the opened directory to move the current working directory outside the chroot()ed area.
  * Perform chdir("..") calls many times to move the current working directory into the real root directory.
  * Change the root directory of the process to the current working directory, the real root directory, using chroot(".")

## Capabilities

The manual page of linux capabilities are as follows:

```
DESCRIPTION
    
       Starting with kernel 2.2, Linux divides  the  privileges  traditionally
       associated  with  superuser into distinct units, known as capabilities,
       which can be independently enabled and disabled.   Capabilities  are  a
       per-thread attribute.
```

Each capability is prefixed by CAP_ and each represents a dinstinct unit of privilege. Commands 'getcap' and 'setcap' are for getting and setting capabilities on a file.

Usage of 'getcap' and 'setcap':

* getcap
  
  ```bash
  SYNOPSIS
       getcap [-v] [-n] [-r] [-h] filename [ ... ]
  ```

* setcap
  
  ```bash
  setcap [-q] [-n <rootid>] [-v] {capabilities|-|-r} filename [ ... capabilitiesN fileN ]
  ```

Three capability flags:

1. E (effective): whether the capability is active
2. I (inheritable): whether the capability is inherited by child processes
3. P (permitted): whether the capability is permitted, regardless of parent’s capability set

## Control Groups (cgroups)

Control groups (cgroups) enables the limiting of system resource utilization based on user-defined groups of processes.

Functions that can be achieved by cgroup features:

* Limits: maximum limits can be specified on processor usage, memory usage, device usage, etc.
* Accounting: monitoring resource usage
* Priortization: resource usage can be prioritized over other cgroups
* Control: the state of processes can be controlled (e.g., stop, restart, suspend).
  
A cgroup is a set of one or more processes with the same set of defined limits. See this [website](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01) for more detailed documentation on cgroups.

Cgroups are organized hierarchically, meaning that child cgroups can inherit some of the attributes of the parents, much like the processes in Linux. But many differences exist:

* Linux process model is a single hierarchy (tree-like) since all processes are children of the 'init' process.
* Linux cgroup model includes many different hierarchies of crgoups simultaneously, meaning that the cgroup model is like one or more separate and unconnected trees of tasks.
* Each hierarchy of cgroups is attached to one or more **subsystems**, each of which represents a single resource, such as CPU time or memory.

There are ten subsystems (also called *resource controller or controller*) provided in Red Hat Enterprise Linux 6, as can be show from the website above.

The functioning of cgroups include the following steps:

```bash
cgcreate #create a cgroup, typical usage: cgcreate -g subsyste:path
cgset # set the limits, typical usage: cgset -r name=value cgroup_path
cgget # get the parameters of a cgroup
cgexec # run the task in the given cgroup
```

The limits set in each subsystem can be found in 

```
/sys/fs/cgroup/[subsystem name]
```

## Namespaces

Definition (*Note that Linux namespaces are different from the common-sense namespaces widely used in other computer science areas*):

* A namespace is an abstract object that encapsulates resources so that said resources have a view restricted to other resources in the same namespace.
* With the introduction of the PID namespaces, multiple process trees (instead of a single tree with the root of 'init' process), which are disjoint, can co-exist.

7 kinds of namespaces in Linux kernel:

1. PID – isolates processes
2. Network – isolates networking
3. User – isolates User/Group IDs
4. UTS – isolates hostname and fully-qualified domain name (FQDN)
5. Mount – isolates mountpoints
6. cgroup – isolates the cgroup sysfs root directory
7. IPC – isolates IPC/message queues

Namespaces defined on the system can be found via procfs.With namespaces introduced, the processes in a container X are unaware of the resources in another container Y.

Three system calls associated with namespaces (can be seen in detail via the manual page of namespaces):

```bash
The namespaces API
       As  well as various /proc files described below, the namespaces API in‐
       cludes the following system calls:

       clone(2)
              The clone(2) system call creates a new process.   If  the  flags
              argument  of  the  call  specifies one or more of the CLONE_NEW*
              flags listed below, then new namespaces  are  created  for  each
              flag,  and  the  child  process  is made a member of those name‐
              spaces.  (This system call also implements a number of  features
              unrelated to namespaces.)

       setns(2)
              The  setns(2)  system call allows the calling process to join an
              existing namespace.  The namespace to join is  specified  via  a
              file  descriptor  that refers to one of the /proc/[pid]/ns files
              described below.
       unshare(2)
              The unshare(2) system call moves the calling process  to  a  new
              namespace.   If  the flags argument of the call specifies one or
              more of the CLONE_NEW* flags listed below, then  new  namespaces
              are  created  for  each  flag, and the calling process is made a
              member of those namespaces.  (This system call also implements a
              number of features unrelated to namespaces.)

       ioctl(2)
              Various  ioctl(2) operations can be used to discover information
              about   namespaces.    These   operations   are   described   in
              ioctl_ns(2). 
```
**Important notes here**: A PID namespace can only be created at the time a new process is spawned using clone(), while other namespaces can also be created using unshare() system call. (see this [website](https://www.toptal.com/linux/separation-anxiety-isolating-your-system-with-linux-namespaces) for details).

The /proc/[pid]/ns directory:

* Each process has a /proc/[pid]/ns/ subdirectory  containing  one  entry for each namespace that supports being manipulated by setns(2).

**Distinguishing namespaces from cgroups**: namespaces limit resource **view** (i.e., how many resources that can be seen by each namespace, not that the resources that are allocated/assigned), while cgroups limit resource **utilization** (i.e., how many resources that can be used by each cgroup, not the resources that can be seen).

More on the namespaces can be found in the [RedHat blog](https://www.redhat.com/sysadmin/7-linux-namespaces), [Developers blog](https://www.toptal.com/linux/separation-anxiety-isolating-your-system-with-linux-namespaces) and the [Linux manual page](https://man7.org/linux/man-pages/man7/namespaces.7.html).
## Seccomp

Definition:

* seccomp protects against the threat of damage by a malicious process via syscalls, by limiting the number of syscalls a process is allowed to execute. 

In lxc, seccomp filters can be accessed through the container configuration file

```
~/.local/share/lxc/<container_name>/config
```

And the default disallowed system calls are specified in

```
/usr/share/lxc/config/common.seccomp
```

## Mandatory Access Control (MAC)

MAC is a centralized authorization mechanism that operates on the philosophy that information belongs to an organization (and not the individual members). 

Two concepts of MAC:

1. Type Enforcement (TE)
2. Multilevel Security (MLS)
