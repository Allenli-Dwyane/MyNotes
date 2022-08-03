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