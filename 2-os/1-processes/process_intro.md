---
title: Processes and Process API
parent: Operating System
nav_order: 1
---
# Processes
Reference: [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/), [Github](https://github.com/remzi-arpacidusseau/ostep-homework)

Processes is one of the most fundamental abstraction OS provides. A process is defined as a **running program**.

We often need to run **multiple programs**, and processes aim to provide the illusion of many CPUs by running one process, the stopping it and running another, and so forth. This basic technique is known as time sharing (see more in the [next chapter]()).

The counter part of this time sharing is space sharing where a resource is divided in space. For example, disk and storage is naturally a space shared resource.

## Constituents of a process
One main component a process relies on is its memory. The process's instructions, the data the program reads and writes all reside in the memory.

Another component are the registers. Many instructions explicitly read or update registers. Some special registers are the **program counter** to track the next instruction to execute, and **stack pointer** and associated **frame pointer** to manage stack for function parameters.

Finally, programs often access persistent **storage devices**, these devices likely present themselves as file descriptors.

## Process APIs
On a high level, the OS should provide these APIs in its interface for managing processes
1. **create** - see [`fork`](https://isbobby.github.io/2-os/1-processes/apis/1-process_api_fork.html), technically a *copy*
2. **destroy**
3. **wait** - wait for a process to stop running, see [`wait`](https://isbobby.github.io/2-os/1-processes/apis/2-process_api_wait.html)
4. **exec** - execute a specified command or executable, see ([`exec`](https://isbobby.github.io/2-os/1-processes/apis/3-process_api_exec.html))
5. **misc control** - for example, suspend and resume
6. **status** - to retrieve status information about a process

**More on process creation**
The OS has to load process code and any related static data into the process's address space in the memory. Programs initially reside on **disk** or other persistent storages, in some kind of executable format.

In earlier OS, this was done eagerly, where the entirety of the process is loaded into the memory all at once. In modern OS, only the critical bits are loaded in. The modern lazy loading is enabled by paging and swapping.

Once the code and static data are loaded, some additional memory must be allocated for the program's run-time stack and heap. The OS may also do I/O initialisation tasks. On UNIX systems, each process by default has three opened file descriptors, for standard input, output, and error.

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

## Shell - process APIs in action
It appears odd why the action of creating a process is split into two APIs - fork and exec (instead of a single `create(executable)`).

Turns out, the separation of these APIs is essential in building the `UNIX` shell as it allows some code to be executed *after* `fork()` and *before* `exec()`. This post-fork-pre-exec code can alter the environment of the new program, enabling a variety of features.

The Shell is just a user program - it allows users to type input into it. Most of the time, the shell will figure out where in the file system the executable resides, then uses `fork()` to create a new child process to run this command, and calls `exec()` to run the command, and waits for the process to complete with `wait()` (of course this is an over-simplification, but it is a core part to a shell process).

When the child completes, the shell returns from `wait()` and prints out a prompt, ready for the next command. Take the following command as example
```
$ wc p3.c > newfile.txt
```

If we execute the above, the output from `wc` gets written into our `newfile.txt`. The shell does this by first creating a child with `fork()`, and before calling `exec()` for `wc`, closes the standard output and opens the file `newfile.txt` such that outputs are written to the file instead.

We can implement the above behaviour with the following
```c
// after fork
int rc = fork();
if (rc == 0) {
	close(STDOUT_FILENO);
	open("./output.txt", O_CREATE|O_WRONLY|O_TRUNC,S_IRWXU);
	char *myargs[3];
	execvp(myargs[0], myargs);
}
```

In the above snippet, by closing `STDOUT_FILENO`, then immediately `open`, the next call to `open()` assigns `1` to `output.txt`, as a result, any program executed after this will write its standard output to `output.txt` instead of the terminal.

## Advanced topics
The above just documents a small part of process APIs, and does not cover the rich **signal** infrastructure and system. In addition, some system researches also identified problems with the `fork + exec` approach and advocates for simpler API such as `spawn` - read widely to also take in opinions from other experts.