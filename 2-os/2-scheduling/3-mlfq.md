---
title: Multi-level Feedback Queue
parent: CPU Scheduling
nav_order: 3
has_toc: false
---
# Introduction
As discussed in [Basic CPU Scheduling](https://isbobby.github.io/2-os/2-scheduling/2-scheduling_strategies.html), occurrence of IO and the length of execution of a job is not known to the scheduler. Although round-robin does well in achieving good response time, it has worse turn around time.

We want a scheduling mechanism to optimise both response and turn around time - this gives rise to the multi-level feedback queue (MLFQ).

## Basic MLFQ Rules
Unlike the scheduling strategies described previously, MLFQ will track jobs using multiple distinct queues. These queues represent jobs of different priority. A job can only be on one queue at a time, and more than one job can exist on a queue.

For jobs on the same queue, they are considered to have the same priority. For jobs within the same queue, we will just apply round-robin scheduling.

This sets the first basic rules of MLFQ
1. if `priority(A) > priority(B)`, run A only
2. if `priority(A) == priority(C)`, do `round_robin(A,C)`

The key to MLFQ is that it adjusts the priority of a job dynamically throughout its execution. How the MLFQ adjusts the job priority uses some heuristics:
1. if a job is switched out from IO, it is likely to be an interactive job
2. if a job's time slice expires, it is likely a compute intensive and long running job
We will use these heuristics to infer characteristics of a job, and change their priority accordingly.

## Changing Priorities
Extending the basic rules, we will add the following behaviours to MLFQ
1. when a job first joins MLFQ, it is assigned the highest priority
2. if a job uses up its time slice, it's likely compute intensive and long running, we want to decrease its priority
3. if a job gives up CPU time due to IO, it's likely going to be an interactive job, we want to keep it where it is.

However, this solution is incomplete as it exposes a problem of **starvation** - where if the system has too many interactive jobs, a long running job can never get executed.

Additionally, this also exposes the MLFQ to the risk of being **gamed**, where malicious program can invoke IO call right before its time slice expires and stay in the highest priority forever.

Finally, a program can change behaviour overtime - what if a CPU intensive job enters a phase of interactivity?
## Priority Boost
A solution against **starvation** would be to do a priority boost - shifting all jobs into the highest priority queue after some time.

This mechanism solves two problems at once, first, CPU intensive job will not starve even if there are many other interactive jobs. Second, when a long running CPU intensive job enters an interactive phase, it will be able to be executed in a high priority until its interactive phase is over.
## Job Accounting
The reason why the current MLFQ can be gamed is due to the rule where a job stays in the same priority if it's swapped out due to an IO event.

To address this, MLFQ will have to implement an accounting mechanism at each queue level, such that it is able to track the **overall allotment time** a job has spent in current priority. With this accounting, we can modify our rule to demote a job once its time allotment in the current level is up, regardless of how many times it gives up the CPU due to IO.

Hence, we will change the following rules
```
if a job uses up its time slice, it's likely compute intensive and long 
running, we want to decrease its priority

if a job gives up CPU time due to IO, it's likely going to be an interactive 
job, we want to keep it where it is.
```

to just the following
```
if a job uses up its time slice on its current level, we want to decrease 
its priority regardless how many times it gives up on CPU
```
## MLFQ Parameters and Tuning
The above approaches introduced some important variables to govern MLFQ behaviour.
1. Number of queues
2. Time interval to reset priority
3. Time allotment at each level

Configuring these values dynamically during execution is challenging, some MLFQ uses a table to define the above values, some OS also uses formulas to deduce these variables (see decay-usage algorithm).

In practice, most MLFQ reserves the highest priority for OS jobs, and a user task can never obtain the highest priority. Some systems allow user advice to set priorities, see more on command `nice` which allows user to increase and decrease a job priority.

## Summary
MLFQ is a smart mechanism which performs prioritisation without any prior knowledge of jobs, it is governed by the following rules

1. if `priority(A) > priority(B)`, run A only
2. if `priority(A) == priority(C)`, do `round_robin(A,C)`
3. when a job first joins MLFQ, it is assigned the highest priority
4. if a job uses up its time slice on its current level, we want to decrease its priority
5. after some time interval, move every job to the highest priority queue (reshuffle the system)

## Implementation in Go
We will convert the above concepts into a list of requirements and implement our own MLFQ. An initial design is proposed and documented in the [nested pages](https://isbobby.github.io/2-os/2-scheduling/2-3-mlfq/1-mlfq-design.html).

The source code is also available in this [Github repository directory](https://github.com/isbobby/system-programming/tree/main/go/os/scheduling/mlfq).
