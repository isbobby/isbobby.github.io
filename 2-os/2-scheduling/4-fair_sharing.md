---
title: Proportional Fair Sharing Scheduling
parent: CPU Scheduling
nav_order: 4
has_toc: false
---
# Introduction
The **[Multi-level Feedback Queue](https://isbobby.github.io/2-os/2-scheduling/3-mlfq.html)** dynamically adjusts to system behaviour, prioritising tasks that demand high responsiveness. For instance, I/O-bound tasks _(e.g., keyboard input handlers)_ retain higher priority since they frequently yield the CPU. 

However, CPU-bound tasks _(e.g., video rendering)_ risk **priority demotion** when they exhaust their time quantum. In workloads dominated by I/O activity, this can lead to **starvation of CPU-intensive processes**, as they may languish in lower-priority queues.

To prevent starvation, some schedulers enforce proportional-share fairness, guaranteeing each task receives its allocated CPU share, even under contention with higher priority tasks. Two well-known examples are lottery scheduling and Linux's Completely Fair Scheduler.

## Lottery Scheduling
Lottery based scheduling enforce fairness through the use of "tickets" and a randomised selection process.

Lottery scheduling allocates tickets to each task proportionally to its CPU time share. The scheduler then randomly selects a winning ticket from this range, and the task holding that ticket is scheduled.

To implement a basic lottery scheduler requires the OS to maintain a range of tickets, and a table which persists the mapping between ticket ranges and the corresponding task. When the scheduler is executed, it will generate a random number and schedule the task with matching ticket range.
### Improvement - prevent misallocation of resources
The above implementation can be exploited by a task requesting a very large number of tickets, which drastically reduces the scheduling probabilities of other tasks. This problem can be prevented by implementing a hierarchical ticketing system with exchange rate and limits.

This solution does not grant tasks direct access to the same global ticket pool. Instead
1. the global ticket pool is broken down into different local pools (per group / per user)
2. each task requests for local tickets, which are converted back to the global ticket
3. this prevents any single task from monopolising the global ticket pool

### Improvement - ticket transfer
During execution, some tasks may wish to request for higher priority. With the ticketing system, this can be achieved by ticket transfer or inflation, where a task gives up some of its tickets to a higher priority task, or it inflates its own ticket count.

### Pros & Cons of relying on randomness
Lottery scheduling relies on random number generation for task selection, which makes implementation very straight forward.

However, it does have the following drawbacks
1. Random selection will only approach proportional fairness when there are sufficient iterations, and may be unfair in a shorter span
2. Random selection is unpredictable
3. Random selection only approximates proportional fairness, making it unsuitable for use cases requiring strict guarantees
