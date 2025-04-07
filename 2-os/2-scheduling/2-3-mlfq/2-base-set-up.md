---
title: I - System Clock
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
