---
---

## Summary
The **Priority Queue** pattern ensures that high-priority requests are received and processed before lower-priority requests. In standard queues, messages are processed First-In-First-Out (FIFO). In a Priority Queue, the "head" of the queue is always the element with the highest priority, regardless of when it arrived.

## Detailed Explanation
In cloud applications, different tasks often have different urgency levels.
*   **High Priority**: Real-time user requests, payment processing.
*   **Low Priority**: Batch reporting, background archival.

If a single FIFO queue is used, a burst of low-priority tasks can block high-priority tasks, causing latency for users. The Priority Queue pattern solves this by reordering execution based on importance.

### Implementation Strategies
1.  **Single Priority Queue**: A data structure (Heap) that internally sorts items.
2.  **Multiple Physical Queues**: A common cloud pattern (e.g., in AWS SQS) is to have two distinct queues: `high_priority_queue` and `low_priority_queue`. The consumer logic always checks the `high` queue first; only if it's empty does it check the `low` queue.

### Challenges
*   **Starvation**: If high-priority messages keep arriving, low-priority messages may *never* get processed.
*   **Solution (Aging)**: Gradually increase the priority of a message the longer it sits in the queue, ensuring it eventually becomes "high priority" enough to be processed.

### Go Implementation
Go's standard library provides `container/heap` which can be used to implement a Priority Queue.

```go
package main

import (
	"container/heap"
	"fmt"
)

// Item represents a request in the queue
type Item struct {
	Value    string
	Priority int // Higher number = Higher priority
	index    int // The index of the item in the heap (needed by update)
}

// PriorityQueue implements heap.Interface and holds Items.
type PriorityQueue []*Item

func (pq PriorityQueue) Len() int { return len(pq) }

// Less decides the sort order. We want a MAX-heap (highest priority first).
func (pq PriorityQueue) Less(i, j int) bool {
	return pq[i].Priority > pq[j].Priority
}

func (pq PriorityQueue) Swap(i, j int) {
	pq[i], pq[j] = pq[j], pq[i]
	pq[i].index = i
	pq[j].index = j
}

// Push adds an item
func (pq *PriorityQueue) Push(x interface{}) {
	n := len(*pq)
	item := x.(*Item)
	item.index = n
	*pq = append(*pq, item)
}

// Pop removes the highest priority item
func (pq *PriorityQueue) Pop() interface{} {
	old := *pq
	n := len(old)
	item := old[n-1]
	item.index = -1 // for safety
	*pq = old[0 : n-1]
	return item
}

/*
Usage:
func main() {
	pq := make(PriorityQueue, 0)
	heap.Init(&pq)

	heap.Push(&pq, &Item{Value: "Background Job", Priority: 1})
	heap.Push(&pq, &Item{Value: "User Request", Priority: 5})

	// This will print "User Request" first, despite it being added second
	item := heap.Pop(&pq).(*Item)
	fmt.Printf("Processed: %s (Priority %d)\n", item.Value, item.Priority)
}
*/
```

## Interview Questions

**Q: How do you implement a Priority Queue in a distributed system (like AWS/Kafka)?**
**A:** Most distributed message brokers don't support fine-grained internal sorting.
*   **AWS SQS**: Does not support priority within a queue. You must create separate queues (`High`, `Medium`, `Low`) and have consumers poll the High queue more frequently.
*   **Redis**: Supports `Sorted Sets (ZSET)`, which is excellent for implementing a true distributed priority queue.
*   **RabbitMQ**: Has support for priority queues, but they have performance costs.

**Q: How do you solve the "Starvation" problem?**
**A:**
1.  **Aging**: Move items from Low -> High priority queue if they have waited too long.
2.  **Weighted Round Robin**: Configure consumers to not *only* read from High, but to use a ratio. E.g., Read 3 messages from High, then 1 from Low. This ensures Low always makes some progress.

**Q: Why not just use a Priority Queue for everything?**
**A:** Sorting is expensive. A FIFO queue is O(1) for insert/remove. A Priority Queue (Heap) is O(log n). In high-throughput systems, this overhead matters. Also, preserving strict order (FIFO) is often a business requirement that Priority Queues break.
