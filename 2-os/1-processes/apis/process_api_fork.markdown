---
title: Process Creation with fork()
parent: Processes and Process API
nav_order: 1
---
# `fork()`
References: [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf), [man fork(2)](https://man7.org/linux/man-pages/man2/fork.2.html), [code](https://github.com/isbobby/system-programming/tree/main/c/os/process).

**`fork()`** creates a child process. The calling process is the **parent process**.

It's easier to understand `fork()` with the following snippet
```c
#include <unistd.h>
#include <stdio.h>

int main() {
    printf("hello (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
    } else if (rc == 0) {
        printf("child execute: (pid:%d)\n", (int) getpid());
    } else {
        printf("parent continues in (main), parent of %d (pid: %d)\n", rc, (int) getpid());
    }
    return 0;
}
```

The process first prints out its own **PID**, the **PID** is used to issue command related to the process, such as stopping.

Next, the `fork()` is called. The process being created by `fork()` is an (almost) exact copy of the calling process. To the OS, it appears there are two copies of the same program running, and both are about to return from the `fork()` system call.

The return code of `fork()` helps us to identify the parent and child process. The parent receives the `pid` of the child process created, whereas the child receives a `0`.

When the child process is created, there are now two active (ready) processes waiting to be scheduled. Hence, either process can run and the output for the prints will not be deterministic.

We can embed a `trace` function to see the memory address of the instruction
```c
#include <unistd.h>
#include <stdio.h>
#include <execinfo.h>

#define BACKTRACE_BUFFER 100

void trace() {
    int nptrs;
    void *buffer[BACKTRACE_BUFFER];
    nptrs = backtrace(buffer, BACKTRACE_BUFFER);
    
    char **strings;
    strings = backtrace_symbols(buffer, nptrs);

    for (size_t j = 0; j < nptrs; j++) {
        printf("%s\n", strings[j]);
    }
} 

int main() {
    printf("hello from parent (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
    } else if (rc == 0) {
        printf("\n======== child log trace ========\n");
        printf("child execute: (pid:%d)\n", (int) getpid());
        trace();
    } else {
        printf("\n======== parent log trace ========\n");
        printf("parent continues in (main), parent of %d (pid: %d)\n", rc, (int) getpid());
        trace();
    }
    return 0;
}
```

As it turns out (on my personal computer), the address for `main` is different, but **same** for `trace` function. This is likely due to address space layout randomisation. 

```
bobby@Bobbys-MacBook-Pro process % ./fork
hello from parent (pid:63928)

======== parent log trace ========
parent continues in (main), parent of 63929 (pid: 63928)
0   fork                                0x0000000102257ce8 trace + 48
1   fork                                0x0000000102257e70 main + 232
2   dyld                                0x000000018fb7c274 start + 2840

======== child log trace ========
child execute: (pid:63929)
0   fork                                0x0000000102257ce8 trace + 48
1   fork                                0x0000000102257e2c main + 164
2   dyld                                0x000000018fb7c274 start + 2840
```
