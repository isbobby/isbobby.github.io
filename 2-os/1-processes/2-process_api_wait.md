---
title: Waiting with wait()
parent: Processes and Process API
nav_order: 2
---
# `wait()`
Reference: [man page]([https://man7.org/linux/man-pages/man2/wait.2.html](https://man7.org/linux/man-pages/man2/wait.2.html)), [OSTEP]([https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf))

The `wait()` and `waitpid()` functions are used to wait for state changes in a child of the calling process, and obtain information about the child whose state has changed. In the meantime, the calling thread's execution is **suspended**.

The state changes could be:
1. child is terminated
2. child was stopped by a signal
3. child was resumed by a signal

### Specifying `pid`
For `waitpid(pid_t pid, int *_Nullable wstatus, int options);` - the calling thread is suspended until a child specified by *pid* argument changes state. The value of *pid* can be

1. `< -1` wait for any child process whose **process group ID** is equal to the absolute value of *pid*
2. `-1` wait for any child process
3. `0` wait for any child process whose process group ID equal to the calling thread's group ID at the point of `waitpid()` is called
4. `>0` wait for the child process whose process ID is equal to the value of *pid*

### Specifying `option`
The `options` governs the behaviour of `waitpid()`. For example, `WHOHANG` will cause the function to return immediately if no child has exited.

### Specifying `wstatus`
If `wstatus` is not `NULL`, then the `wait` functions will store status information in the `int` to which it points. We can only inspect the integer with macros specified in `man`. For example
```c
#include <sys/wait.h>

w = wait(&wstatus)
if (WIFEXITED(wstatus)) {
	printf("exited, status=%d\n", WIFEXITED(wstatus));
}
```

### Return values
`wait()` - returns process ID of the terminated child, or `-1` on failure.

`waitpid()` - returns the process ID of the child whose state has changed. If `WHOHANG` option was provided, and specified process exists, but none have changed states, then `0` is returned. `-1` is returned on failure.

### Zombie Process?
A child that terminates, but has not been waited for becomes a "zombie"!

The kernel maintains a minimal set of information about the zombie process in order to allow the parent to later perform a wait to obtain information about the child. As long as the zombie is not removed from the system via a wait, it will always consume a slot in the kernel process table.

If the parent terminates, then its "zombie" children gets adopted by `init(1)`. `init(1)` automatically performs `wait` to remove the zombies.

### Code Example
```c
#include <unistd.h>
#include <stdio.h>
#include <sys/wait.h>

#define BACKTRACE_BUFFER 100

int main() {
    printf("hello from parent (pid:%d)\n", (int) getpid());
    int rc = fork();
    char * placholder;
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
    } else if (rc == 0) {
        printf("child execute: (pid:%d)\n", (int) getpid());
        printf("enter any key to continue child process, run ps now to see active process\n");
        scanf(placholder, "%d");
    } else {
        int rc_wait = wait(NULL);
        printf("child process waited for\n");
        printf("enter any key to continue, run ps now to see active process\n");
        scanf(placholder, "%d");
        printf("parent continues in (main), parent of %d (pid: %d)\n", rc, (int) getpid());
        return 0;
    }
}
```

Combining the above with `ps`, we can examine and verify and see child processes falling into the "zombie" status. Since `wait` is called in the parent thread, it will block and client will continue executing.

When the first prompt `"enter any key to continue child process"` shows, we can run `ps`, this shows the following on my machine, this is expected as both processes are executing - child is blocked on I/O, parent is blocked on `wait()`.

```
  PID TTY           TIME CMD
67237 ttys011    0:00.01 ./wait
67238 ttys011    0:00.00 ./wait
```

After entering any key to unblock the child, the parent call to `wait` is unblocked, and running `ps` now only shows one single `./wait` process

```
  PID TTY           TIME CMD
67237 ttys011    0:00.01 ./wait
```
