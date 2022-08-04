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

### Type Enforcement (TE)

TE introduces type labeling to every file system object, and is a prerequisite for MAC. Objects are labeled with a type, and a policy is defined in the kernel to specify which types are allowed to transition to which other types.

### Multilevel Security (MLS)

## Demo for Running LXC

See [rayden's blog](https://raydenchia.com/linux-containers-lxc/) for detailed description.

### LXC APIs
APIs for Linux containers in C can be found in [lxxcontainer.h](https://github.com/lxc/lxc/blob/master/src/lxc/lxccontainer.h).

Some of the most frequently used and important APIs are listed below (as well as some interpretation of the source code):

1. the struct lxc_container

    ```c
    struct lxc_container{
      char *name; // name of the container
      char *configfile; // full path to configuration file
      char *pidfile; // file that stores pid
      struct lxc_lock *slock; // container semaphore lock
      struct lxc_lock *privlock; // container private lock
      int numthreads; // number of references to this container, protected by privlock
      struct lxc_conf *lxc_conf; // container configuration
      ...
      char *config_path; // full path to configuration file
      bool (*is_defined)(struct lxc_container *c); // if the container exists
      const char *(*state)(struct lxc_container *c); // state of container.
      bool (*is_running)(struct lxc_container *c);
      bool (*freeze)(struct lxc_container *c);
      bool (*unfreeze)(struct lxc_container *c);
      pid_t (*init_pid)(struct lxc_container *c); // process ID of the containers init process.
      bool (*load_config)(struct lxc_container *c, const char *alt_file); // load configuration for the container from a specified file
      bool (*start)(struct lxc_container *c, int useinit, char * const argv[]);
      bool (*startl)(struct lxc_container *c, int useinit, ...);
      bool (*stop)(struct lxc_container *c);
      ...
      char *(*config_file_name)(struct lxc_container *c); //Return current config file name.
      bool (*wait)(struct lxc_container *c, const char *state, int timeout); // Wait for container to reach a particular state.
      bool (*set_config_item)(struct lxc_container *c, const char *key, const char *value); // Set a key/value configuration option.
      bool (*destroy)(struct lxc_container *c);
      bool (*save_config)(struct lxc_container *c, const char *alt_file); // save current configuration to the specified file
    
      bool (*create)(struct lxc_container *c, const char *t, const char *bdevtype,
			struct bdev_specs *specs, int flags, char *const argv[]);

      bool (*createl)(struct lxc_container *c, const char *t, const char *bdevtype,
			struct bdev_specs *specs, int flags, ...);
      ...
    };
    ```

    An important note here is that changing the order of struct members is an API change, as callers will end up having the wrong offset when calling a function.

2. the setup of a container
   
      ```c
      struct lxc_container *lxc_container_new(const char *name, const char *configpath); // name: container name, configpath: full path to configuration file, return: a newly allocated lxc_container pointer or NULL on error
      ```

3. creation of a container
   
      ```c
      struct lxc_container{
        ...
        /*
        lxc_container c: container
        const char *t: template to execute to instantiate the root filesystem and adjust the configuration.
        const char *bdevtype: bdevtype Backing store type to use (if \c NULL, \c dir will be used).
        struct bdev_specs *specs: Additional parameters for the backing store (for example LVM volume group to use).
        int flags: LXC_CREATE_* options
        ...: command line, e.g., "-d", "ubuntu", "-r", NULL
        */
        bool (*createl)(struct lxc_container *c, const char *t, const char *bdevtype,
			struct bdev_specs *specs, int flags, ...);
    
        ...
      };
      ```

  4. get and drop reference to a container

      ```c
      int lxc_container_get(struct lxc_container *c);
      int lxc_container_put(struct lxc_container *c)
      ```

### Running LXC via command-line

Steps are listed below:

1. create a user config directory for lxc (if it does not exist) and the default configuration file.

    ```bash
    $ mkdir -p ~/.config/lxc
    $ touch ~/.config/lxc/default.conf
    ```

2. creating a container
   
    ```bash
    lxc-create {-n name} [-f config_file] {-t template} [-B backingstore] [-- template-options]
    ```

    Four default templates are specified in /user/share/lxc/templates/

    1. download: downloads pre-built images and unpacks them
    2. local:  consumes local images that were built with the distrobuilder build-lxc command
    3. busybox:  consumes local images that were built with the distrobuilder build-lxc command
    4. oci: creates an application container from images in the Open Containers Image (OCI) format

    After creation, a container directory will be created at ~/.local/share/lxc/example/

3. Running a container:
   
    ```bash
    lxc-start [container name]
    ```

    Attaching container:

    ```bash
    lxc-attach [container name]
    ```

    **Notice that we are root inside the container, even though we created an unprivileged container. This behavior is the result of UID namespaces.**

4. Networking
   
    Require prerequistes on iptables, detailed knowledge can be found [here](https://wiki.archlinux.org/title/Iptables) as well as [this blog](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html#TRAVERSINGOFTABLES).

    *This part may make more sense to me after carefully taking a course on computer networks*.

