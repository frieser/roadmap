---
---

## Summary
**Reactive Programming** is a declarative programming paradigm concerned with data streams and the propagation of change. In a reactive system, components react to data as it arrives (streams), rather than requesting data (polling). It is essential for building highly responsive, resilient, and scalable systems, often summarized by the **Reactive Manifesto**.

## Detailed Explanation

### 1. The Reactive Manifesto
Reactive systems are defined by four traits:
1.  **Responsive**: The system responds in a timely manner if at all possible (Latency is king).
2.  **Resilient**: The system stays responsive in the face of failure (Isolation, Replication).
3.  **Elastic**: The system stays responsive under varying workload (Scaling up/down).
4.  **Message Driven**: Reactive systems rely on asynchronous message-passing to establish a boundary between components.

### 2. Core Concepts
*   **Streams**: A sequence of data elements made available over time. Anything can be a stream: variables, user inputs, properties, caches, data structures, etc.
*   **Observer Pattern**: Reactive programming is essentially the Observer pattern done right. Subscribers listen to Publishers.
*   **Backpressure**: A critical mechanism where the consumer signals the producer to slow down if it's overwhelmed. Without backpressure, a fast producer can crash a slow consumer (OOM).
*   **Functional Transformation**: Streams are manipulated using functional operators like `map`, `filter`, `reduce`, `merge`.

### 3. Reactive vs. Imperative
*   **Imperative**: `b = c + d`. If `c` changes later, `b` remains the old value. You must re-execute the statement.
*   **Reactive**: `b` is bound to the stream of `c` and `d`. Whenever `c` changes, `b` updates automatically.

---

## Go Application (Channels & Streams)

Go is naturally reactive due to Channels and Goroutines.

```go
package main

import (
	"fmt"
	"time"
)

// Producer: Generates a stream of integers
func sourceStream() <-chan int {
	out := make(chan int)
	go func() {
		for i := 1; i <= 5; i++ {
			out <- i
			time.Sleep(100 * time.Millisecond) // Simulate work
		}
		close(out)
	}()
	return out
}

// Operator: Maps (transforms) the stream
func mapStream(in <-chan int, fn func(int) int) <-chan int {
	out := make(chan int)
	go func() {
		for n := range in {
			out <- fn(n)
		}
		close(out)
	}()
	return out
}

func main() {
	// 1. Source Stream
	nums := sourceStream()

	// 2. Functional Transformation (Reactive Chain)
	// square = nums.map(x -> x * x)
	squared := mapStream(nums, func(x int) int {
		return x * x
	})

	// 3. Subscriber (Consumer)
	for res := range squared {
		fmt.Printf("Received: %d\n", res)
	}
}
```

### Backpressure in Go
In Go, **unbuffered channels** provide natural backpressure. If the receiver isn't ready, the sender blocks.

```go
// Buffered channel with size 0 (or small size) acts as a flow control
stream := make(chan Data) 

// If consumer is slow, this line blocks, effectively slowing down the producer
stream <- data 
```

---

## Interview Questions

**Q: What is Backpressure and why is it important?**
**A:** Backpressure is a feedback mechanism where a consumer communicates its processing capacity to the producer. It prevents system overload. If a fast producer sends 1000 msg/sec to a consumer that can only handle 100 msg/sec, without backpressure, the consumer's buffer will overflow, causing a crash.

**Q: How does Reactive Programming differ from standard Event-Driven Programming?**
**A:** While related, Event-Driven focuses on *individual events* (like a button click), whereas Reactive Programming focuses on *streams of data* and the *propagation of change* through functional operators. Reactive libraries (RxJava, RxJS) provide powerful tools to combine, filter, and transform these streams that standard event listeners lack.

**Q: When should you use Reactive Programming?**
**A:** It shines in scenarios with high concurrency, high latency (network calls), and "Push-based" data (WebSockets, IoT sensors, UI events). It is overkill for simple, synchronous CRUD applications.
