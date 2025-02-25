---
title: Introduction to Scheduling
parent: CPU Scheduling
nav_order: 2
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
The most basic scheduler algorithm, we execute workload by the order of their arrival. It works well given our simple assumptions.

Supposed we have three jobs arriving at the same time $$t=0$$, and each job takes 10 seconds to complete, the average turnaround time would be just $$sum(10+20+30)/3=20s$$.

Now, if we relax assumption 1, where jobs have different runtime, and if job A runs for 100s, the average turnaround becomes $$sum(100+110+120)/3=110s$$. Although job B and C have the same runtime, their turnaround gets degraded. This is referred to as the **convoy effect**, where a number of shorter workloads gets queued behind a large workload.


