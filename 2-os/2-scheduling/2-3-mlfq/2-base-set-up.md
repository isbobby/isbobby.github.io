---
title: I - System Clock & Job
parent: Multi-level Feedback Queue
nav_order: 2
---
# System Clock
To synchronise the processor and IO device, we will need a system clock to send clock signals to both components at the same time.

To achieve this, we will establish a channel between each component and the clock. The clock will maintain a list of channels. 

```go
type Clock struct {
	Time          atomic.Int64
	DelayPerCycle time.Duration
	Subscriptions []chan<- interface{}
}
```

We will propagate the signal one time, we can iterate through the `Subscriptions` array of channels and send a message in a non-blocking way. This ensures the clock doesn't get stuck if one of the consumer isn't ready to receive a clock signal.

```go
func (c *Clock) publishSignal() {
	for _, subscriber := range c.Subscriptions {
		c.nonBlockingPush(subscriber)
	}
}

func (c *Clock) nonBlockingPush(subscription chan<- interface{}) {
	select {
	case subscription <- struct{}{}:
	default:
	}
}
```

Lastly, we can introduce a `DelayPerCycle` variable to sleep the cycle for ease of debugging and observing the components' behaviours.

```go
func (c *Clock) Run(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			return
		default:
			time.Sleep(c.DelayPerCycle)
			c.publishSignal()
			c.advanceTime()
		}
	}
}

func (c *Clock) advanceTime() {
	c.Time.Add(1)
}
```

To connect a component to the clock, we just need to provide a `chan` that allows the component to receive clock signal, and add it to the `clock`'s subscription list.

```go
func init() {
    clockToIOSignal := make(chan interface{})
    clockToPSignal := make(chan interface{})
    clockSubscriptions := []chan<- interface{}{clockToIOSignal, clockToPSignal}
    clock := NewClock(time.Duration(100*time.Millisecond), clockSubscriptions)
}

func NewClock(delay time.Duration, subscriptions []chan<- interface{}) *Clock {
	c := Clock{}
	c.Time.Store(0)
	c.DelayPerCycle = delay
	c.Subscriptions = subscriptions
	return &c
}
```

# Job
We will also implement a `Job struct` to represent user jobs to be executed. It will contain a stack of instructions (represented with a slice). An instruction can be either `IO` or `CPU` instruction, each with different required clock cycles to complete.

```go
type Job struct {
	ID               int
	InstructionStack []JobInput
}

type JobInput struct {
	Cycle int
	Type  string
}
```

To simulate future inputs of jobs easily, we will also include a `ScheduleTime int`, which works together with `Clock struct`. If the current `Clock.Time` is equal to the job's `ScheduleTime`, then it can be scheduled for execution.

With reference to the concept of [job accounting](https://isbobby.github.io/2-os/2-scheduling/3-mlfq.html#job-accounting) in earlier chapter, we need to keep a record of the job's priority and time alloted remaning. This is to ensure fair sharing within the same priority.

With the added attributes, the final `Job struct` will look like
```go
type Job struct {
	ID               int
	ScheduledTime    int
	Priority         *int
	InstructionStack []JobInput
	TimeAllotment    atomic.Int32
}

type JobInput struct {
	Cycle int
	Type  string
}
```