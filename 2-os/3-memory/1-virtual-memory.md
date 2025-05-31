---
title: Memory (WIP)
parent: Operating System
nav_order: 3
---
# Introduction
1. Importance of memory virtualisation
	1. [[CPU Scheduling]] with [[Limited Direct Execution]] handles virtualisation of CPU.
	2. Memory also requires virtualisation for programs to efficiently and safely use memory.
2. Requirements of memory virtualisation
	1. transparency
	2. efficiency - enabled by hardware translation
	3. protection - programs execute as if they have private memory spaces (even though they are on the same physical memory)
3. A conceptual model of memory layout (region for instructions, stack and heap)
## Memory APIs
1. use C's `malloc` and `free` as examples, showcase how programs only interact with whatever virtual addresses provided by OS
2. common pitfalls when manually using heap memory
3. GCs
4. include examples on `free` and `pmap` as observability APIs

## [[Mechanism - Address Translation]]
