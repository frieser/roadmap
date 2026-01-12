---
---

# Publish-Subscribe (Pub/Sub)

## Summary
The Publish-Subscribe (Pub/Sub) pattern is a messaging pattern where senders (publishers) do not program the messages to be sent directly to specific receivers (subscribers). Instead, messages are characterized into classes (topics) without knowledge of which subscribers, if any, there may be. It provides high decoupling and scalability for distributed systems.

## Detailed Explanation

### 1. Core Mechanics

*   **Publisher**: Emits messages to a specific "Topic" or "Channel". It has no knowledge of who will receive it.
*   **Subscriber**: Expresses interest in one or more topics. It receives only messages from those topics.
*   **Broker (optional but common)**: An intermediary service (like Google Pub/Sub, Redis Pub/Sub) that manages the routing of messages from publishers to subscribers.

### 2. Pub/Sub vs. Observer Pattern
*   **Observer Pattern**: Typically synchronous and in-memory (e.g., UI event listeners). The Subject knows its Observers (holds a list of references).
*   **Pub/Sub**: typically asynchronous and distributed. The Publisher and Subscriber do not know each other; they only know the *Event Channel* or *Topic*.

### 3. Pub/Sub vs. Message Queues (Point-to-Point)
*   **Message Queue**: 1-to-1. A message is processed by *one* consumer. (Load Balancing).
*   **Pub/Sub**: 1-to-Many. A message is processed by *all* subscribers to that topic. (Broadcast).

### 4. Filtering Methods
*   **Topic-Based**: Subscribers listen to a specific named channel (e.g., `logs.errors`).
*   **Content-Based**: Subscribers define a query (e.g., `severity > 5`), and the broker filters messages based on payload. This is more complex to implement.

## Real-World Examples
*   **Redis Pub/Sub**: Fast, fire-and-forget messaging.
*   **Google Cloud Pub/Sub**: Global scale messaging.
*   **MQTT**: Lightweight Pub/Sub for IoT devices.

## Go Implementation Example

A simple in-memory Pub/Sub using Go channels.

```go
package main

import (
	"fmt"
	"sync"
)

type PubSub struct {
	mu     sync.RWMutex
	topics map[string][]chan string
}

func (ps *PubSub) Subscribe(topic string) <-chan string {
	ps.mu.Lock()
	defer ps.mu.Unlock()
	ch := make(chan string, 10)
	ps.topics[topic] = append(ps.topics[topic], ch)
	return ch
}

func (ps *PubSub) Publish(topic string, msg string) {
	ps.mu.RLock()
	defer ps.mu.RUnlock()
	if subscribers, found := ps.topics[topic]; found {
		for _, ch := range subscribers {
			// Non-blocking send to avoid hanging if subscriber is slow
			select {
			case ch <- msg:
			default:
				fmt.Println("Warning: Subscriber buffer full, dropping message")
			}
		}
	}
}

func main() {
	ps := &PubSub{topics: make(map[string][]chan string)}

	// Subscriber 1
	sub1 := ps.Subscribe("news")
	go func() {
		for msg := range sub1 {
			fmt.Println("[Sub1] Received:", msg)
		}
	}()

	// Subscriber 2
	sub2 := ps.Subscribe("news")
	go func() {
		for msg := range sub2 {
			fmt.Println("[Sub2] Received:", msg)
		}
	}()

	ps.Publish("news", "Breaking News: Go is awesome!")
	// Output: Both subscribers receive the message
}
```

## Interview Questions

**Q: What happens if a subscriber is offline when a message is published?**
**A:** In a basic Pub/Sub (like Redis), the message is lost (Fire-and-Forget). In a durable Pub/Sub (like Kafka or Google Pub/Sub), the system retains the message for a configurable retention period until the subscriber acknowledges it.

**Q: What is the "Thundering Herd" problem in Pub/Sub?**
**A:** If a massive number of subscribers wake up simultaneously to process a message (or reconnect to the broker), it can overwhelm the system. This is mitigated by exponential backoff and jitter.

**Q: When would you choose Point-to-Point (Queue) over Pub/Sub?**
**A:** When you want to distribute the workload (work queues). If you have 5 workers and 100 jobs, you want each job processed once, not 5 times. Use Pub/Sub when you need to notify multiple systems about the same event.
