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

## Scheduling Metric - Response Time
Besides turn around time, response time is also an important metric as it measures the **perceived responsiveness** of a system.
$$ T_{response} = T_{firstrun} - T_{arrival} $$
The above schedulers can have good turnaround time, but they are insensitive to response time.


## Algorithm 4 - Round Robin
With response time in mind, we can propose a new scheduling algorithm - round robin.

The idea is simple, instead of running a task to completion, we run the job for a fixed time slice, and then switch the job out and run the next job in the run queue. This process is repeated until the entire queue is empty. For this reason, round robin scheduling is also called **time-slicing**.

> **Amortisation to reduce cost** - it is used in systems where there is a fixed cost to some action, and by performing the action less often, the total system cost is reduced. For example, if task switching requires `1ms`, and the time slice is set to `10ms`, the overhead is about 10%. However, if we increase the time slice to `100ms`, less than `1%` time is spent on task switching.

Based on the above point, the length of time slice is critical for RR. The shorter the time slice, the better performance of RR under the response time metric. But making it too short introduces much more overhead due to more frequent task switching. We cannot have a time slice that's too long, as it will render the system less responsive.

While RR optimises response time, it does not perform well for turn around time. Because turnaround time only cares about when job completes, RR is nearly pessimal, even worse than FIFO in some cases.

More generally, a fair scheduling algorithm will perform worse at turnaround time. This is a trade off as if we are being unfair and run shortest jobs first, then we optimise for response time, being fair loses this opportunity.

Now we have two categories of scheduling algorithm - SJF and STCF to optimise turnaround time, and RR to optimise response time. Next, we have two remaining assumptions - jobs do not do IO and run-time of job is known. We shall see how relaxation of these constraints affect our scheduling.

## New Problem 1 - Introducing IO
Since all programs perform IO, we must relax the assumptions that no IO takes place. This introduces one more problem for the scheduler - as a job using IO will not be using the CPU to do any work, and remains so until the IO completes. If the same task remains in the CPU, much CPU time is wasted. Hence, the scheduler should schedule another job on the CPU at the time.

One way to model the above is to use the IO windows as partition to divide a job into smaller sub-jobs. This allows for overlap, where the CPU can be used by other processes if one task is blocked on IO.

## New Problem 2 - Unknown Runtime
The above scheduling algorithms (especially SJT, STCF) assumes the scheduler know the time to run each job in advance - this is very different from reality.

Hence, this gives rise to Multi-level Feedback Queue, a scheduling mechanism which allows us to predict the future with the recent past.