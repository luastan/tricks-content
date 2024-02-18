---
title: Generics
description: Using generic types in Go
badge: Go
position: 4
---

Amazing feature released with Go 1.18 that can be very usefull when working with either of the following:

- **Functions that use slices, maps or channels of any type**: Some functions usally iterate over these special types just to do something generic like calling a function for each element. This is specially interesting when working with channels, as is usual to find a lot of boilerplate code managing the necesary goroutines with `context.Context` and/or `sync.WaitGroup`.
- **General purpose data structures**: A binary tree, a linked list or something similar; basically a data structure you might want to reuse with different types.

## Basics

### Do I need to know the type?

### General purpose data structures

The boring but useful

## Functional?

Things start getting fancy here. Using channels as _streams_ of data the same way as you would use Python's generators (with `yield`) and Java's Lambda Expressions is really easy and pleasant with generic types.

```go
// SliceToChan returns a channel to send every item in a slice s
func SliceToChan[T any](s []T) <-chan T {
	stream := make(chan T)
	go func() {
		defer close(stream)
		for _, t := range s {
			stream <- t
		}
	}()
	return stream
}
```

This code tho is rather simple (and bad because it can leak goroutines), theres some gain on using generics, but not that much.
However, programming the function properly (to avoid leaking goroutines and allow channel buffer size to be specified) it's easier to see why the generic types can help to reduce boilerplate:

```go
// SliceToChan returns a channel to send every item in a slice s. Stops and
// closes de channel on ctx cancel. Channel buffer can be specified with size.
// Unspecified size or 0 results on an unbuffered channel.
func SliceToChan[T any](ctx context.Context, s []T, size ...int) <-chan T {
	chanSize := 0
	if len(size) > 0 {
		chanSize = size[0]
	}
	stream := make(chan T, chanSize)
	go func() {
		defer close(stream)
		for _, t := range s {
			select {
			case stream <- t:
			case <-ctx.Done():
				return
			}
		}
	}()
	return stream
}
```

I personally develop a lot of tools/scripts that generally iterate over collections of data and most of the time do some I/O with every element (for example making a request to every url in a file and checking the output); wich benefits a lot from using channels giving me an easy time making concurrent efficient tools instead of huge for-loops or having to save every step in a huge slice in memmory.

As an example imagine a tool that grabs a list of domains as arguments.
For every provided domain, the tool makes a request to an API that returns a JSON with some information.
That information contains a list (slice) of IPs the domain was resolving to at some point in time.
The goal is to print every IP that any of those domain ever had, but only if the IP does not belong to an specific subnet.
