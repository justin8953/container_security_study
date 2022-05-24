# Fundamental Concept

## 1. [Namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)

### 1.1. Description

A namespace wraps a **global system resource** in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource. Changes to the global resource are visible to other processes that are members of the namespace, but are invisible to other processes. One use of namespaces is to implement containers.

### 1.2. Namespace types

| Namespace | Flag            | Isolates                            |
|:---------:|:---------------:|:-----------------------------------:|
| Cgroup    | CLONE_NEWCGROUP | Cgroup root directory               |
| IPC       | CLONE_NEWIPC    | System V IPC, POSIX message queues  |
| Network   | CLONE_NEWNET    | Network devices, stacks, ports,etc. |
| Mount     | CLONE_NEWNS     | Mount points                        |
| PID       | CLONE_NEWPID    | Process IDs                         |
| Time      | CLONE_NEWTIME   | Boot and monotonic clocks           |
| User      | CLONE_NEWUSER   | User and group IDs                  |
| UTS       | CLONE_NEWUTS    | Hostname and NIS domain name        |

### 1.3. Namespaces API

the namespaces API includes the following system calls:

- clone(2)
- setns(2)
- unshare(2)
- ioctl(2)

## 2. [Cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html)

### 2.1. Description

Control groups, usually referred to as cgroups, are a Linux kernel feature which allow processes to be organized into hierarchical groups whose usage of various types of resources can then be limited and monitored.  The kernel's cgroup interface is provided through a pseudo-filesystem called cgroupfs.  Grouping is implemented in the core cgroup kernel code, while resource tracking and limits are implemented in a set of per-resource-type subsystems (memory, CPU, and so on).

## 3. [Seccomp](https://man7.org/linux/man-pages/man2/seccomp.2.html)

### 3.1 Description

The seccomp() system call operates on the Secure Computing (seccomp) state of the calling process.

## 4. [Capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)

### 4.1 Description

For the purpose of performing permission checks, traditional UNIX implementations distinguish two categories of processes: privileged processes (whose effective user ID is 0, referred to as superuser or root), and unprivileged processes (whose effective UID is nonzero).  Privileged processes bypass all kernel permission checks, while unprivileged processes are subject to full permission checking based on the process's credentials (usually: effective UID, effective GID, and supplementary group list). Starting with kernel 2.2, Linux divides the privileges traditionally associated with superuser into distinct units, known as capabilities, which can be independently enabled and disabled. Capabilities are a per-thread attribute.

## 5. [LSM](https://www.kernel.org/doc/html/v4.13/admin-guide/LSM/index.html) (Linux Security Module)

### 5.1 Description

The primary users of the LSM interface are Mandatory Access Control (MAC) extensions which provide a comprehensive security policy. Examples include SELinux, Smack, Tomoyo, and AppArmor. In addition to the larger MAC extensions, other extensions can be built using the LSM to provide specific changes to system operation when these tweaks are not available in the core functionality of Linux itself.

## 6. [OverlayFS](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html)

### 6.1 Description

An overlay-filesystem tries to present a filesystem which is the result over overlaying one filesystem on top of the other.