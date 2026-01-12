---
---

# Queue

## Abstract
A **Queue** is a linear data structure that follows the **FIFO** (First In, First Out) principle. The first element added is the first one to be removed, similar to a line of people waiting at a checkout counter. This ordering is essential for scenarios where order of processing must be preserved, such as task scheduling, data buffering, and breadth-first traversals.

## Development

### Core Concept
A queue is defined by two primary operations:
1.  **Enqueue**: Add an element to the **back** (tail) of the queue.
2.  **Dequeue**: Remove an element from the **front** (head) of the queue.

**Visual Representation**:
```text
      Front (Remove)                         Back (Add)
      +-----+     +-----+     +-----+     +-----+
<---- | [A] | <---| [B] | <---| [C] | <---| [D] | <---- Enqueue(E)
      +-----+     +-----+     +-----+     +-----+
```

### Time Complexity

| Operation | Complexity | Description |
|-----------|------------|-------------|
| **Enqueue**| **O(1)**   | Adding to the tail. |
| **Dequeue**| **O(1)**   | Removing from the head. |
| **Peek**   | **O(1)**   | Inspecting the head without removing. |
| **Search** | O(n)       | Linear scan. |

### Common Implementations
1.  **Slice-based**: Uses a dynamic array. Fast access, but careful management is needed to avoid memory leaks (shifting the view doesn't free the underlying array).
2.  **Linked List-based**: Truly dynamic. No resizing or memory drift issues, but higher per-node overhead.
3.  **Ring Buffer (Circular Queue)**: Fixed-size array where the "next" pointer wraps around to the beginning. efficient for buffering.

## Code Examples (Go)

### 1. Slice-based Queue (Basic)
The simplest implementation uses `append` for enqueue and slicing for dequeue.

```go
package main

import "fmt"

func main() {
	var queue []string

	// 1. Enqueue
	queue = append(queue, "First")
	queue = append(queue, "Second")
	queue = append(queue, "Third")

	// 2. Dequeue
	if len(queue) > 0 {
		// Get Front
		front := queue[0]
		// Slice off the front
		queue = queue[1:] 
		fmt.Printf("Dequeued: %s\n", front) // First
	}

	fmt.Println("Queue:", queue) // [Second Third]
}
```
*Note: This basic method causes "Memory Drift". The underlying array grows but the start index keeps moving forward, potentially holding reference to unused memory at the beginning.*

### 2. Channel-based Queue (Concurrent)
Go's unique feature. Channels are essentially thread-safe (goroutine-safe) FIFO queues.

```go
package main

import "fmt"

func main() {
	// Buffered channel acting as a queue with capacity 3
	queue := make(chan int, 3)

	// Enqueue (Non-blocking if buffer not full)
	queue <- 10
	queue <- 20
	queue <- 30

	// Dequeue
	val := <-queue
	fmt.Println("Dequeued:", val) // 10
	
	// Close when done
	close(queue)
}
```

### 3. Linked List Queue (`container/list`)
Using the standard library avoids the memory drift of slices and the fixed size of channels.

```go
package main

import (
	"container/list"
	"fmt"
)

func main() {
	queue := list.New()

	// Enqueue
	queue.PushBack("Job 1")
	queue.PushBack("Job 2")

	// Dequeue
	front := queue.Front()
	if front != nil {
		val := queue.Remove(front)
		fmt.Println("Processed:", val) // Job 1
	}
}
```

## Go Application & Ecosystem

### Slice "Memory Drift" & Fix
In long-running applications, using `queue = queue[1:]` repeatedly will cause the underlying array to keep growing indefinitely until a reallocation happens.
**Fix**: Periodically reset the slice if it has too much empty space at the front, or use a **Circular Queue**.

```go
// Resetting a slice queue to reclaim space
if len(queue) < cap(queue)/2 {
    // Compact: Move elements to start of a new backing array
    newQ := make([]int, len(queue))
    copy(newQ, queue)
    queue = newQ
}
```

### Use Cases
1.  **Breadth-First Search (BFS)**: Graph traversal algorithms (e.g., finding shortest path) require a queue to explore neighbors layer by layer.
2.  **Job Scheduling**: Task queues (like Sidekiq/Celery, or internal Go worker pools) use queues to buffer tasks for workers.
3.  **IO Buffering**: `bufio.Reader` and network stacks use circular buffers to handle streaming data.

## Interview Preparation

### Common Questions

1.  **Implement a Queue using Stacks.**
    *   **Question**: Construct a FIFO queue using only two LIFO stacks.
    *   **Answer**: Use two stacks: `input` and `output`. Always push to `input`. When dequeueing, if `output` is empty, pop **all** elements from `input` and push them to `output` (reversing their order). Then pop from `output`. This gives amortized O(1).

2.  **Design a Circular Queue (Ring Buffer).**
    *   **Question**: Implement a fixed-size queue that wraps around.
    *   **Answer**: Maintain an array and two pointers: `head` and `tail`.
        *   Enqueue: `arr[tail] = val; tail = (tail + 1) % capacity`
        *   Dequeue: `val = arr[head]; head = (head + 1) % capacity`
        *   Full/Empty checks rely on comparing `head` and `tail` (often keeping a `count` variable is simpler).

3.  **Blocking vs Non-Blocking Queue in Go.**
    *   **Answer**: A buffered channel is a blocking queue (send blocks if full, receive blocks if empty). To make it non-blocking, use `select` with a `default` case.

4.  **Generate binary numbers from 1 to N.**
    *   **Question**: Print binary 1 to N using a queue.
    *   **Answer**: Start with queue `["1"]`. Loop N times: Dequeue `s`. Print `s`. Enqueue `s + "0"`. Enqueue `s + "1"`.
