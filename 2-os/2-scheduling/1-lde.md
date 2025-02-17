---
title: Limited Direct Execution
parent: CPU Scheduling
nav_order: 1
---
# Introduction
Before diving deeper into this topic, we need to be familiar with how a process gets executed on the CPU, and what are the problems when we attempt to implement **time sharing** between different processes.

If we assume there's only one process, a naive technique would be just a *direct execution*, where we run the program directly on the CPU. The OS may perform the following
1. create entry for process list
2. allocate memory and load program into memory
3. set up stacks
4. clear CPU registers
5. execute **`main()`**

the control will be handed to the program, which now runs on the CPU until it executes **`return`** from **`main()`**.

Then, the OS will free the memory used and remove it from the process list.

This simple approach gives rise to a few problems
1. how can the OS make sure the program doesn't do anything that we want it to do, while still running it efficiently?
2. how can the OS stop the running process halfway, and switch to another process?

The answers to the above questions help us identify what's needed to virtualise the CPU, and while developing these techniques, we can see where does the "limit" in limited direct execution comes from.

(TBC)