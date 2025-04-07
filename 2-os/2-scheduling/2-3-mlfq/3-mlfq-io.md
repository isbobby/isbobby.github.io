---
title: II - Logical IO
parent: Multi-level Feedback Queue
nav_order: 3
---
# Logical IO 
We will also implement an IO device which does the following
1. accept an incoming IO event based on the event time and current system time
2. accept a IO job from CPU and execute it based on the number of cycles
3. send a ready task to the scheduler

## Synchronisation with system time
To fulfill the above requirements, we will need to be in-sync with the system clock.

First, if a job is scheduled at `t=10`, we need to know that the current system time is `10` before we can send the job to the scheduler. We can store a reference to `systemTime *Clock`, and use `systemTime.Time.Load()` to read the current system time in a concurrent-safe manner.

Second, if a job is being executed and the IO instruction requires `N` clock cycles, we need to simulate IO execution by holding the job in the IO device until `N` cycles have elapsed. This can be done by storing a reference to `clockSignal <-chan`, every `<- clockSignal` execution represents on signal from our clock, and we can advance the instruction by one cycle.

```go
type IOStream struct {
    scheduledJobs []*Job
	systemTime    *Clock
	clockSignal   <-chan interface{}
}

// to read current system time
func (s *IOStream) currentTime() {
    s.systemTime.Time.Load()
}

// to handle clock signal
func (s *IOStream) handleSignal() {
    for {
        <-s.clockSignal
        // do something
    }
}
```

## (WIP)