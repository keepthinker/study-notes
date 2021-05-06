# Linux Kernel

## Overview

Typical components of a kernel are interrupt handlers to service interrupt requests, a scheduler to share processor time among multiple processes, a memory management system to manage process address spaces, and system services such as networking and interprocess communication.  

Applications running on the system communicate with the kernel via system calls (see Figure 1.1). An application typically calls functions in a library—for example, the C library—that in turn rely on the system call interface to instruct the kernel to carry out tasks on the application’s behalf.   

![relationshipAppKernelHardware](relationshipAppKernelHardware.png)

The kernel also manages the system’s hardware. Nearly all architectures, including all systems that Linux supports, provide the concept of interrupts.When hardware wants to communicate with the system, it issues an interrupt that literally interrupts the processor, which in turn interrupts the kernel.  

## Process

Processes are, however, more than just the executing program code (often called the text section in Unix).They also include a set of resources such as **open files** and **pending signals**, **internal kernel data**, **processor state**, **a memory address space with one or more memory mappings**, one or more **threads of execution**, and **a data section containing global variables**.  

### Thread

Each thread includes a unique **program counter**, **process stack**, and **set of processor registers**.  The kernel schedules individual threads, not processes.

Linux has a unique implementation of threads: It does not differentiate between threads and processes. To Linux, **a thread is just a special kind of process.**  

On modern operating systems, processes provide two virtualizations: **a virtualized processor** and **virtual memory**. The virtual processor gives the process the illusion that it alone monopolizes the system, despite possibly sharing the processor among hundreds of other processes. Virtual memory lets the process allocate and manage memory as if it alone owned all the memory in the system.  

threads share the virtual memory abstraction, whereas each receives its own virtualized processor.  

A process begins its life when, not surprisingly, it is created. In Linux, this occurs by means of the fork() system call, which creates a new process by duplicating an existing one.The process that calls fork() is the parent, whereas the new process is the child.  The fork() system call returns from the kernel twice: once in the parent process and again in the newborn child.  

The kernel stores the list of processes in a circular doubly linked list called the task list.

![process-task-list](process-decriptor-task-list.png)

### Process State 

- **TASK_RUNNING**—The process is runnable; it is either currently running or on a runqueue waiting to run.This is the only possible state for a process executing in user-space; it can also apply to a process in kernel-space that is actively running.

- **TASK_INTERRUPTIBLE**—The process is sleeping (that is, it is blocked), waiting for some condition to exist.When this condition exists, the kernel sets the process’s state to TASK_RUNNING.The process also awakes prematurely and becomes runnable if it receives a signal.  

- **TASK_UNINTERRUPTIBLE**—This state is identical to TASK_INTERRUPTIBLE except that it does not wake up and become runnable if it receives a signal.This is used in situations where the process must wait without interruption or when the event is expected to occur quite quickly. Because the task does not respond to signals in this state, TASK_UNINTERRUPTIBLE is less often used than TASK_INTERRUPTIBLE.  

- **TASK_TRACED**—The process is being traced by another process, such as a debugger, via ptrace.

- **TASK_STOPPED**—Process execution has stopped; the task is not running nor is it eligible to run.This occurs if the task receives the SIGSTOP, SIGTSTP, SIGTTIN, or SIGTTOU signal or if it receives any signal while it is being debugged.  

- ![process-state](D:process-state.png)

  Normal program execution occurs in user-space.  When a program executes a system call or triggers an exception, it enters kernel-space.  

### The Process Family Tree 

A distinct hierarchy exists between processes in Unix systems, and Linux is no exception. All processes are descendants of the init process, whose PID is one. The kernel starts init in the last step of the boot process. The init process, in turn, reads the system
initscripts and executs more programs, eventually completing the boot process. 

Every process on the system has exactly one parent. Likewise, every process has zero or more children.   

### Process Creation

The first, fork(), creates a child process that is a copy of the current task. It differs from the parent only in its PID (which is unique), its PPID
(parent’s PID, which is set to the original process), and certain resources and statistics, such as pending signals, which are not inherited. The second function, exec(), loads a new executable into the address space and begins executing it. The combination of fork() followed by exec() is similar to the single function most operating systems provide.  

#### Copy on Write

In Linux, fork() is implemented through the use of copy-on-write pages. Copy-on-write (or COW) is a technique to delay or altogether prevent copying of the data. Rather than duplicate the process address space, **the parent and the child can share a single copy.**

**The duplication of resources occurs only when they are written; until then, they are shared read-only.**  

The only overhead incurred by fork() is the duplication of the parent’s page tables and the creation of a unique process descriptor for the child.  

### The Linux Implementation of Threads  

To the Linux kernel, there is no concept of a thread. Linux implements all threads as standard processes. The Linux kernel does not provide any special scheduling semantics or data structures to represent threads.  Instead, **a thread is merely a process that shares certain resources with other processes.**  

This approach to threads contrasts greatly with operating systems such as Microsoft Windows or Sun Solaris, which have explicit kernel support for threads (and sometimes call threads lightweight processes).  To Linux, threads are simply a manner of sharing resources between processes (which are
already quite lightweight).  

#### Creating Threads  

Threads are created the same as normal tasks, with the exception that the clone() system call is passed flags corresponding to the specific resources to be shared:  

``` clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);   ```

The previous code results in behavior identical to a normal fork(), except that the **address space**, **filesystem resources**, **file descriptors**, and **signal handlers** are shared. In other words, the new task and its parent are what are popularly called threads.  The flags provided to clone() help specify the behavior of the new process and detail what resources the parent and child will share. 



## Process scheduling

The **Completely Fair Scheduler** (CFS) is the registered scheduler class for normal processes, called SCHED_NORMAL in Linux (and SCHED_OTHER in POSIX). CFS is defined in kernel/sched_fair.c. The CFS algorithm and is germane to any Linux kernel since 2.6.23.  



# 用户登录文件执行顺序

## 全局文件名称
1. /etc/profile
2. /etc/bashrc(bash resource config)

## 用户私有文件

## 登录文件

1. ~/.bashrc
2. ~/.bash_profile

### 历史文件
~/.bash_history

### 退出文件
1. ~/.bash_logout


## 过程
在刚登录Linux时，首先启动/etc/profile文件，然后在profile文件里面去启动/etc/profile.d目录里面的脚本，然后再启动用户根目录下的 .bash_profile文件，查看.bash_profile文件，.bash_profile文件还会判断.bashrc文件是否存在并执行。

再查看.bashrc文件，它会再去判断并执行/etc/bashrc


## 各文件含义
（1）/etc/profile： 此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行. 并从/etc/profile.d目录的配置文件中搜集shell的设置。

（2）/etc/bashrc: 为每一个运行bash shell的用户执行此文件.当bash shell被打开时，该文件被读取。

（3）~/.bash_profile: 每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次!默认情况下，他设置一些环境变量，执行用户的.bashrc文件。

（4）~/.bashrc: 该文件包含专用于你的bash shell的bash信息，当登录时以及每次打开新的shell时，该该文件被读取。

（5）~/.bash_logout:当每次退出系统(退出bash shell)时，执行该文件. 另外，/etc/profile中设定的变量(全局)的可以作用于任何用户，而~/.bashrc等中设定的变量(局部)只能继承/etc /profile中的变量，他们是"父子"关系。

（6）~/.bash_profile 是交互式、login 方式进入 bash 运行的~/.bashrc 是交互式 non-login 方式进入 bash 运行的通常二者设置大致相同，所以通常前者会调用后者。

## 参考博客
[Linux 用户登录后执行的脚本](https://www.jianshu.com/p/a47c9e6f6ff3)

# CPU指标

## cpu utilization
CPU utilization is a measure of how busy the CPU is right now. It is useful to measure CPU time as **a percentage of the CPU's capacity**, which is called the CPU usage. When the CPU usage is **above 70%**, the user may experience lag. Such high CPU usage indicates **insufficient processing power**. Either the CPU needs to be upgraded, or the user experience reduced, for example, by switching to lower resolution graphics or reducing animations.

## cpu load averages
An idle computer has a load number of 0 (the idle process isn't counted). Each process **using** or **waiting for CPU** (the ready queue or run queue) increments the load number by 1. Each process that terminates decrements it by 1. Most UNIX systems count only processes in the running (on CPU) or runnable (waiting for CPU) states. However, Linux also includes processes in uninterruptible sleep states (usually waiting for disk activity), which can lead to markedly different results if many processes remain blocked in I/O due to a busy or stalled I/O system

## Details about cpu utilization and load averages
On Linux at least, the load average and CPU utilization are actually two different things. Load average is a measurement of how many tasks are waiting in a kernel run queue (not just CPU time but also disk activity) over a period of time. CPU utilization is a measure of how busy the CPU is right now. The most load that a single CPU thread pegged at 100% for one minute can "contribute" to the 1 minute load average is 1. A 4 core CPU with hyperthreading (8 virtual cores) all at 100% for 1 minute would contribute 8 to the 1 minute load average.

Often times these two numbers have patterns that correlate to each other, but you can't think of them as the same. You can have a high load with nearly 0% CPU utilization (such as when you have a lot of IO data stuck in a wait state) and you can have a load of 1 and 100% CPU, when you have a single threaded process running full tilt. Also for short periods of time you can see the CPU at close to 100% but the load is still below 1 because the average metrics haven't "caught up" yet.

I've seen a server have a load of over 15,000 (yes really that's not a typo) and a CPU % of close to 0%. It happened because a Samba share was having issues and lots and lots of clients started getting stuck in an IO wait state. Chances are if you are seeing a regular high load number with no corresponding CPU activity, you are having a storage problem of some kind. On virtual machines this can also mean that there are other VMs heavily competing for storage resources on the same VM host.


## 参考文献
[Load_(computing)-Wikipedia](https://en.wikipedia.org/wiki/Load_(computing))

[CPU_time-Wikipedia](https://en.wikipedia.org/wiki/CPU_time)

# inode
The inode (index node) is a data structure in a **Unix-style file system** that describes a file-system object such as a file or a directory. Each inode stores the attributes and disk block location(s) of the object's data. File-system object attributes may include metadata (times of last change, access, modification), as well as owner and permission data.

A file system relies on data structures about the files, beside the file content. The former are called metadata—data that describe data. Each file is associated with an inode, which is **identified by an integer number**, often referred to as an i-number or inode number.

Inodes store information about files and directories (folders), such as file **ownership, access mode (read, write, execute permissions), and file type**. On many types of file system implementations, the maximum number of inodes is fixed at file system creation, limiting the maximum number of files the file system can hold. A typical allocation heuristic for inodes in a file system is one percent of total size.

The inode number indexes a table of inodes in a known location on the device. From the inode number, the kernel's file system driver can access the inode contents, including the location of the file - thus allowing access to the file.

## Implications
Files can have multiple names. If multiple names hard link to the same inode then the names are equivalent; i.e., the first to be created has no special status. This is unlike symbolic links, which depend on the original name, not the inode (number).

A file's inode number stays the same when it is moved to another directory on the same device, or when the disk is defragmented which may change its physical location. This also implies that completely conforming inode behavior is impossible to implement with many non-Unix file systems, such as FAT and its descendants, which don't have a way of storing this invariance when both a file's directory entry and its data are moved around.

## 参考文献
[inode-Wikipedia](https://en.wikipedia.org/wiki/Inode)

# File Descriptor
In Unix and related computer operating systems, a file descriptor (FD, less frequently fildes) is an abstract indicator (handle) used to access a file or other input/output resource, such as a pipe or network socket.

To perform input or output, the process passes the file descriptor to the kernel through a system call, and the kernel will access the file on behalf of the process. The process does not have direct access to the file or inode tables.

On Linux, the set of file descriptors open in a process can be accessed under the path /proc/PID/fd/, where PID is the process identifier.

## 参考文献
[File descriptor-Wikipedia](https://en.wikipedia.org/wiki/File_descriptor)

# Inode number vs File Descriptor
An inode number unambiguously identifies a file or directory on a given device, but two files on **different mounts** may have **the same inode**. A file descriptor does not unambiguously identify anything by itself; in combination with a process ID it unambiguously identifies some resource on the system, even if you don't know which device it's on.

在不同的逻辑分区，两个文件的inode可以相同。

## 参考文献
[What's the difference between inode number and file descriptor?](https://www.quora.com/Whats-the-difference-between-inode-number-and-file-descriptor)


# 中断
## 硬中断
1. 硬中断是由硬件产生的，比如，像磁盘，网卡，键盘，时钟等。每个设备或设备集都有它自己的IRQ（中断请求）。基于IRQ，CPU可以将相应的请求分发到对应的硬件驱动上（注：硬件驱动通常是内核中的一个子程序，而不是一个独立的进程）。

2. 处理中断的驱动是需要运行在CPU上的，因此，当中断产生的时候，CPU会中断当前正在运行的任务，来处理中断。在有多核心的系统上，一个中断通常只能中断一颗CPU（也有一种特殊的情况，就是在大型主机上是有硬件通道的，它可以在没有主CPU的支持下，可以同时处理多个中断。）。

3. 硬中断可以直接中断CPU。它会引起内核中相关的代码被触发。对于那些需要花费一些时间去处理的进程，中断代码本身也可以被其他的硬中断中断。

4. 对于时钟中断，内核调度代码会将当前正在运行的进程挂起，从而让其他的进程来运行。它的存在是为了让调度代码（或称为调度器）可以调度多任务。

## 软中断

1. 软中断的处理非常像硬中断。然而，它们仅仅是由当前正在运行的进程所产生的。

2. 通常，软中断是一些对I/O的请求。这些请求会调用内核中可以调度I/O发生的程序。对于某些设备，I/O请求需要被立即处理，而磁盘I/O请求通常可以排队并且可以稍后处理。根据I/O模型的不同，进程或许会被挂起直到I/O完成，此时内核调度器就会选择另一个进程去运行。I/O可以在进程之间产生并且调度过程通常和磁盘I/O的方式是相同。

3. 软中断仅与内核相联系。而内核主要负责对需要运行的任何其他的进程进行调度。一些内核允许设备驱动的一些部分存在于用户空间，并且当需要的时候内核也会调度这个进程去运行。

4. 软中断并不会直接中断CPU。也只有当前正在运行的代码（或进程）才会产生软中断。这种中断是一种需要内核为正在运行的进程去做一些事情（通常为I/O）的请求。有一个特殊的软中断是Yield调用，它的作用是请求内核调度器去查看是否有一些其他的进程可以运行。

## 参考文献
[硬中断与软中断的区别！！！](https://blog.51cto.com/noican/1361087)

# 页缓存

# mmap

# 文件锁
文件锁（也叫记录锁）的作用是，当一个进程读写文件的某部分时，其他进程就无法修改同一文件区域。

能够实现文件锁的函数主要有2个：flock和fcntl。

lockf是在fcntl基础上构造的函数，它提供了一个简化的接口。它们允许对文件中任意字节区域加锁，短至一个字节，长至整个文件。

## 锁类型
共享读锁F_RDLCK，独占性写锁F_WRLCK，解锁F_UNLCK

## 使用锁的基本规则
- 任意多个进程在一个给定的字节上可以有一把共享的读锁（F_RDLCK），但是在一个给定的字节上只能有一个进程有一把独占性写锁（F_WRLCK）。

- 如果在一个给定字节上已经有一把或多把读锁，则不能在该字节上再加写锁，如果在一个字节上已经有一把独占性写锁，则不能再对它加任何读锁。

- 对于单个进程而言，如果进程对某个文件区域已经有了一把锁，然后又试图在相同区域再加一把锁，则新锁会替换旧锁。

- 加读锁时，该描述符必须是读打开，加写锁时，该描述符必须是写打开

## 参考
[件锁的使用浅析](https://blog.csdn.net/guotianqing/article/details/80044087)