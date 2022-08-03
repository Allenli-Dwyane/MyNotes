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