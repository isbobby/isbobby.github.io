---
title: Scheduling Strategies
parent: CPU Scheduling
nav_order: 2
has_toc: false
---
# Introduction
Scheduling predates computer systems, early approaches were taken from fields like operation management.

We want to develop a basic framework for thinking about scheduling policies, including key assumptions and metrics to track.

## Workload Assumptions
Here, we represent all processes running as **workload**. We start with the most forgiving assumptions for now, and slowly relax them as we go. For now, we make these simple yet unrealistic assumptions
1. each job runs for the same amount of time
2. all jobs arrive at the same time
3. each job runs to completion once started
4. all job only uses the CPU, no I/O operations
5. we know the run-time of each job before hand

## Scheduling Metrics
We want a set of metrics to compare different scheduling policies - a **scheduling metric**. Like the simple assumptions, we start with just a simple metric - the **turnaround time**. This refers to the time at which the job completes, minus the time at which the job arrived in the system.

$$T_{turnaround}=T_{completion} - T_{arrival}$$

This turnaround time is a **performance** metric. Another metric of interest for scheduler is **fairness**. Performance and fairness are often at odds in scheduling.

## Algorithm 1 - FIFO
The simplest scheduling algorithm, simply schedule whichever task is read from input first. This can have very bad turnaround time if a long running task blocks the queue.

See [FIFO Scheduling](https://isbobby.github.io/2-os/2-scheduling/2-1-scheduling-strats/2-fifo.html) for more explanation and implementation.

## Algorithm 2 - SJF
The above example showcased a single large workload can degrade performance in a FIFO scheduler. If we assume the run-time of each workload before hand, we can schedule to run the shortest jobs first. For the above example, we will run the two 10s tasks, and lastly the 100s task. This gives a much quicker turnaround of $$sum(10+20+120)/3=50s$$, a more than 2x improvement of the previous FIFO. In fact, if we assume all jobs arrive at the same time and their runtime is known, SJF is the optimal scheduling policy for turnaround time.

However, the optimisation this approach brings also depends on the assumption that all workloads arrive at the same time, and we can re-order the execution by starting the shortest workload first. When we relax this assumption, the following scenario also leads to a convoy problem in SJF.

Instead of all 3 tasks arriving at the same time, the largest workload may arrive first. Since no other tasks have arrived, it is scheduled to execute.

After 10 seconds of execution, two other 10s tasks arrive. They can't be executed as the SJF scheduler is non-preemptive. This leads to an increase in the average turnaround time to $$sum(100+110-10+120-10)=103.33s$$.

## Algorithm 3 - STCF
The above concern can be addressed with the help of context switching and preemptive scheduling. Once the shorter tasks arrive, it can preempt the currently running job to run another job. The job to schedule will have the **shortest time to complete**, hence the name of this scheduling policy - shortest time to complete first.

With this, the above problem results in a shorter turnaround time to $$sum(120+20-10+30-10)=50s$$.
