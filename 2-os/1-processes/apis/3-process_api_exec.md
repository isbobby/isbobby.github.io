---
title: Execute with exec()
parent: Processes and Process API
nav_order: 3
---
# `exec()`
References: [`execve(2)`](https://man7.org/linux/man-pages/man2/execve.2.html), [`exec(3)`](https://man7.org/linux/man-pages/man3/exec.3.html), [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf)

The family of `exec()` calls is useful when we want to run a program that's different from the calling program.

For example, when we call `fork()`, we only keep running copies of the same program. When we want to run a different program, we will need to use `exec()`.

The `exec()` function loads the code and static data from an executable, and overwrites its current code segment with it. Then, the heap and stack and other parts of the memory space of the program are re-initialised. Thus, no new process is created, rather, the currently running program is transformed into a different running program.

The `exec()` function **only returns if there's an error**, we can do
```c
execlp("./child", NULL);
perror("execlp");
exit(EXIT_FAILURE);
```

**Child program** - need to compile to output the executable binary
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("Hello from child\n");
    exit(EXIT_SUCCESS);
}
```

**Parent program**
```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid;
    pid = fork();
    switch (pid)
    {
    case -1:
        perror("fork fail");
        exit(EXIT_FAILURE);
        break;
    case 0:
        execlp("./child", NULL);
        perror("execlp"); // exec returns only on error
        exit(EXIT_FAILURE);
    default:
        puts("parent executed and exited");
        break;
        _exit(EXIT_SUCCESS);
    }
}
```
