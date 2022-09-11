---
title: Concurrency patterns
description: A collection of usefull snippets I use frequently while programming in go
position: 2
badge: Go
---

## Rate Limiting

Here are some ways to implement rate limiting for your Golang programs.

### Basic Rate Limiting

The simplest way is to use Go's `time.Ticker`; which is a structure with a channel that gets a value sent at a specified rate.

```go
// Instantiation of a *time.Ticker
ticker := time.NewTicker(2 * time.Second)

// Direct usage
<-ticker.C

// Usage on a Select statement
select {
case <-rl.ticker.C:
	// Do something
}
```

This works like a charm as it is.
But, in some cases you will want to dynamically change the delay between ticks; for what you would need some extra code like is shown on the following more advanced example:

### Variable Rate Limiting

The following is a the implementation of the `RateLimit` type; which is simply a Mutex around the standard library but adding a minimum delay between `Lock()` operations.
This could be very usefull when implementing rate limits such as those on scripts to avoid both getting blocked by the target and causing a DoS.

```go[rateLimit.go]
// RateLimit is a Mutex that can limits lock acquisition to a certain rate.
type RateLimit struct {
	mu       sync.Mutex
	duration time.Duration
	ticker   *time.Ticker
}

// NewRateLimit creates a new RateLimit with the given duration
func NewRateLimit(delay time.Duration) *RateLimit {
	return &RateLimit{
		duration: delay,
		ticker:   time.NewTicker(delay),
	}
}

// Lock acquires the lock. Block ends always a duration after the last call to
// the .Unlock() method
func (rl *RateLimit) Lock() {
	rl.mu.Lock()
	<-rl.ticker.C
}

// Unlock releases the lock
func (rl *RateLimit) Unlock() {
	rl.mu.Unlock()
}

// UpdateDelay changes the minimum delay between lock acquisitions. This method
// must be called with the lock acquired, otherwise it will cause race
// conditions updating the delay. If the new delay matches current delay no
// actions are performed
func (rl *RateLimit) UpdateDelay(delay time.Duration) {
	if delay == rl.duration {
		return
	}
	rl.duration = delay
	rl.ticker.Stop()
	rl.ticker = time.NewTicker(delay)
}
```

You would use it like this:

```go
rl := concurrency.NewRateLimit(2 * time.Second)
rl.Lock()

if http.StatusLocked == makeSomeRequest() {
    rl.UpdateDelay(10 * time.Second)
} else {
    rl.UpdateDelay(2 * time.Second)
}

rl.Unlock()
```

## Context

[Go's blog on Context](https://go.dev/blog/context)

## The pipeline

This is sort of a functional way of thinking about the instruction flow of the program. As an example think about Gobuster, you have a bunch of paths on a file, you have to make a request for each path and then print the URL if the path was found:

1. Read a line
2. Transform the line into a URL
3. Make the request
4. Filter the request per status codes
5. Print the URL which are left

This would be a perfect fit for Java's lambdas or for Python's `yield` operator. In Go this is really easy to implement on a paralelyzed way with channels and goroutines.
The basic idea is to have one channel in between every step, and at leas a goroutine processing the incoming data from the input channel and forwarding the results through the next channel.

### One goroutine per step

> TODO

### Unlimited goroutines per step

> TODO

### A specific number of goroutines per step

> TODO
