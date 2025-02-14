+++
authors = ["Lone Coder"]
title = "PintOS"
date = "2024-05-13"
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
tags = [
    "hugo",
    "markdown",
    "css",
    "html",
]
categories = [
    "theme demo",
    "syntax",
]
series = ["Theme Demo"]
aliases = ["migrate-from-jekyl"]
+++

PintOS is an educational operating system for the x86 CPU architecture which supports multithreading, loading and running user programs, and a file system. As part of our OS class, my team and I built PintOS over the course of a 5-month long semester. Unfortunately, because this was a project for a university course, I cannot post the code publicly. 

### Argument Passing

First, we extended the barebones Pintos operating system to support argument passing and a robust set of system calls for process management and file operations. Initially, Pintos lacked the ability to handle command-line arguments, which caused many test programs to crash. We implemented argument passing, allowing user processes to correctly receive `argc` and `argv`, ensuring compatibility with standard command-line conventions.

### Process Control Syscalls

In addition to argument passing, we developed several essential system calls. We started by implementing exit, which allows processes to terminate and return status codes, followed by halt, which powers down the OS safely. We then implemented exec, enabling the creation of new processes, and wait, which allows a parent process to wait for a child process to terminate. The fork system call was particularly challenging, as it required duplicating the process address space while maintaining correct parent-child relationships.

### File Operation Syscalls

Beyond process management, we implemented basic file operation system calls, including `create`, `remove`, `open`, `filesize`, `read`, `write`, `seek`, `tell`, and `close`. These system calls interface with Pintos’ basic file system, ensuring processes can read and write data efficiently while maintaining proper synchronization. Given that Pintos’ file system is not inherently thread-safe, we implemented a global lock mechanism to prevent concurrent access issues.

### Efficient Alarm Clock

We created an efficient alarm clock mechanism in Pintos by redesigning the `timer_sleep` function to eliminate busy-waiting. The existing implementation inefficiently consumed CPU cycles while waiting for the timer to elapse, so we modified it to maintain a list of sleeping threads and only wake them when their designated wait time had passed. This approach leveraged Pintos' existing timer interrupt handler to check the list of sleeping threads and wake them up efficiently, improving overall performance significantly. Additionally, we ensured that our implementation handled edge cases such as concurrent sleep requests and potential race conditions by properly synchronizing access to the sleep queue.

### Priority Scheduling

We also played a key role in modifying Pintos' priority scheduler to enforce strict priority scheduling and implement priority donation for locks. The default scheduler did not respect priority values, leading to suboptimal scheduling behavior. We reworked the scheduling logic to always select the highest-priority thread for execution while maintaining round-robin scheduling for threads with the same priority. Furthermore, we implemented priority donation to prevent priority inversion issues by temporarily boosting the priority of lower-priority threads holding locks needed by higher-priority threads. Our implementation accounted for nested donations and correctly restored original priorities when locks were released. Through these contributions, we improved Pintos' scheduling fairness and responsiveness, ensuring that high-priority tasks were not unnecessarily delayed.

### Thread Synchronization Syscalls

Along with kernel threads, we also extended the system’s functionality by implementing new syscalls to support pthreads and user-level synchronization. This involved designing and integrating system calls that allow user-space threads to interact efficiently with the kernel, enabling better thread management and synchronization primitives such as mutexes and condition variables. By implementing these syscalls, we improved the ability of multi-threaded programs to leverage fine-grained control over resource sharing and scheduling.

One of the key challenges was ensuring that the syscalls operated efficiently without introducing unnecessary context-switching overhead. We designed the interface to minimize system call latency and optimize thread scheduling policies, allowing user-space applications to take advantage of lightweight synchronization mechanisms. This section deepened my understanding of kernel-space and user-space interactions, thread synchronization issues, and how operating systems handle concurrent execution.

### Buffer Cache

To improve file system performance, we implemented a buffer cache that optimizes disk block access in Pintos. Previously, every read or write operation accessed the disk directly, leading to significant I/O overhead. By introducing a 64-block buffer cache with a write-back policy, we were able to reduce redundant disk operations, improve read latency, and coalesce multiple writes into fewer disk accesses. The cache follows an LRU-based replacement policy to approximate the MIN algorithm, ensuring that frequently accessed blocks remain in memory while older or less-used blocks are evicted efficiently. Additionally, we enforced synchronization mechanisms to prevent concurrent eviction of active blocks and avoid race conditions when multiple threads access the same cache entry.

Beyond basic caching, we also optimized disk writes by deferring updates until eviction or system shutdown, reducing the number of slow disk writes while ensuring data consistency. Furthermore, we implemented an optional read-ahead feature to prefetch data asynchronously, improving sequential read performance. This project required careful handling of concurrency, synchronization, and memory management, deepening my understanding of file system design and low-level optimizations for disk-based storage.

### File Operations

We also implemented hierarchical directory support in the Pintos file system, enhancing its functionality to allow user programs to interact with directories. Previously, the file system only allowed files to be placed in the root directory, but our modification introduces the ability to manipulate directories through the implementation of new system calls: `chdir`, `mkdir`, `readdir`, and `isdir`. These syscalls enable processes to change the current working directory, create directories, list directory entries, and check if a file descriptor represents a directory, respectively. Additionally, we updated existing syscalls like `open`, `close`, `exec`, `remove`, and `inumber` to handle directories, including support for relative and absolute file paths, which allows seamless navigation and file manipulation within subdirectories.

A critical aspect of this implementation was ensuring that the file system supports hierarchical directory trees, where directories can point to both files and other directories. We also made sure directories can expand beyond their initial size, just like regular files. We focused on maintaining the independence of processes' current working directories, ensuring that child processes inherit the parent's current directory. Furthermore, we handled the special file names `.` and `..`, which represent the current and parent directories, respectively, and implemented robust error handling for scenarios like directory removal. This enhancement improves the versatility and usability of the Pintos file system by allowing user programs to manage directories more intuitively, aligning the system with modern filesystem paradigms.
