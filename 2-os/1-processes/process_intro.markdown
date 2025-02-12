---
title: Processes and Process API
parent: Operating System
nav_order: 1
---
# Processes
Reference: [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/), [Github](https://github.com/remzi-arpacidusseau/ostep-homework)

Processes is one of the most fundamental **abstraction** that OS provides to users. Formally, a process is just defined as a **running program**.

We often need to run **multiple programs**, and processes aims to provide *the illusion* of many CPIs. This is done by **virtualising** the CPU, by running one process, the stopping it and running another, and so forth. This basic technique is known as **time sharing** of the CPU.

The counter part of this **time sharing** is **space sharing**, where a resource is divided in space. For example, disk and storage is naturally. a space shared resource.

## Constituents of a process
One obvious component that comprises a process is its **memory**. The process's instructions lie in the memory, the data the program reads and writes lie in memory as well.

Another part of a process's machine state are **registers**, many instructions explicitly read or update registers. Some special registers are the **program counter** to track the next instruction to execute, and **stack pointer** and associated **frame pointer** to manage stack for function parameters.

Finally, programs often access persistent **storage devices**, these devices likely present themselves as file descriptors.

## Process APIs
On a high level, the OS should provide these APIs in its interface for managing processes
1. **create**
2. **destroy**
3. **wait** - wait for a process to stop running
4. **misc control** - for example, suspend and resume
5. **status** - to retrieve status information about a process

**More on process creation**
The OS has to load process code and any related static data into the process's address space in the memory. Programs initially reside on **disk** or other persistent storages, in some kind of executable format.

In earlier OS, this is done eagerly, where the entirety of the process is loaded into the memory at once. In modern OS, only the critical bits are loaded in. The modern lazy loading is enabled by [[Paging and Swapping]].

Once the code and static data are loaded, some additional memory must be allocated for the program's **run-time stack** and **heap**. The OS may also do I/O initialisation tasks. On UNIX systems, each process by default has three opened file descriptors, for standard input, output, and error.

## Process States
In a simplified view, the process states can be distilled into three different states
1. running - a process is running on a processor, its instructions are being executed
2. ready - process is ready to run but OS has chosen not to run it
3. blocked - the process has performed some kind of operation that renders it not ready to run until other event takes place


## Data Structures
Like any other programs, the OS utilises some key data structure to track and manage the processes. The following is a snippet of data structure to track process, the data structure in actual OS will be much more complex

```c
struct proc {
	char *mem; 
	uint sz;
	char *kstack;
	
	enum proc_state state;
	int pid;
	struct proc *parent;
	void *chan;
	int killed;
	struct file *ofile[NOFILE];
	struct inode *cwd;
	struct context context;
	struct trapframe *tf;
}
```

## Guided Exercise
(WIP)