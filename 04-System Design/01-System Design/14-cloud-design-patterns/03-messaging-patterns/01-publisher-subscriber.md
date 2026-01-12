---
---

## Summary
The **Publisher-Subscriber (Pub-Sub)** pattern is a messaging pattern that decouples senders (publishers) from receivers (subscribers). Publishers categorize messages into classes (topics) without knowledge of which subscribers, if any, there may be. Similarly, subscribers express interest in one or more classes and only receive messages that are of interest, without knowledge of which publishers are sending them.

## Detailed Explanation
In complex distributed systems, point-to-point communication (Service A calls Service B) creates tight coupling. If Service B changes its IP or API, Service A breaks.

The Pub-Sub pattern introduces an intermediary (Message Broker or Event Bus) to solve this:
1.  **Decoupling in Space**: Publishers and Subscribers don't need to know each other's network locations.
2.  **Decoupling in Time**: They don't need to be active at the same time (if the broker is durable).
3.  **Decoupling in Synchronization**: Publishers don't block while subscribers process messages.

### Key Concepts
*   **Topic**: A named channel to which messages are published.
*   **Fan-Out**: One message sent to a topic is copied and delivered to *all* subscribers of that topic.
*   **At-Least-Once Delivery**: The standard guarantee for cloud Pub/Sub systems (AWS SNS, Google Cloud Pub/Sub), meaning a message is guaranteed to be delivered but might arrive multiple times.

### Go Implementation
A local, in-memory Pub-Sub implementation in Go using Channels and Mutexes.

```go
package main

import (
	"fmt"
	"sync"
)

// Subscriber is a channel that receives messages
type Subscriber chan interface{}

// PubSub Broker structure
type PubSub struct {
	mu            sync.RWMutex
	subscriptions map[string][]Subscriber
	closed        bool
}

func NewPubSub() *PubSub {
	return &PubSub{
		subscriptions: make(map[string][]Subscriber),
	}
}

// Subscribe creates a new channel for a topic and registers it
func (ps *PubSub) Subscribe(topic string) Subscriber {
	ps.mu.Lock()
	defer ps.mu.Unlock()

	ch := make(Subscriber, 10) // Buffered channel
	ps.subscriptions[topic] = append(ps.subscriptions[topic], ch)
	return ch
}

// Publish sends a message to all subscribers of a topic
func (ps *PubSub) Publish(topic string, msg interface{}) {
	ps.mu.RLock()
	defer ps.mu.RUnlock()

	if ps.closed {
		return
	}

	for _, sub := range ps.subscriptions[topic] {
		// Non-blocking send to prevent slow subscribers from blocking publisher
		select {
		case sub <- msg:
		default:
			fmt.Println("Warning: Subscriber buffer full, dropping message")
		}
	}
}

func (ps *PubSub) Close() {
	ps.mu.Lock()
	defer ps.mu.Unlock()

	if !ps.closed {
		ps.closed = true
		for _, subs := range ps.subscriptions {
			for _, sub := range subs {
				close(sub)
			}
		}
	}
}
```

## Interview Questions

**Q: What is the difference between a Message Queue and Pub-Sub?**
**A:** 
*   **Message Queue (Point-to-Point)**: A message is processed by **only one** consumer (Load Balancing pattern). Useful for distributing heavy workloads.
*   **Pub-Sub (Broadcast)**: A message is processed by **all** active subscribers (Fan-out pattern). Useful for notifications, event logging, and updating caches.

**Q: How do you handle duplicate messages in a Pub-Sub system?**
**A:** Since most distributed Pub-Sub systems guarantee "At-Least-Once" delivery, duplicates can happen (e.g., due to network ack failures). Subscribers must be **Idempotent**. This is often achieved by tracking a unique `MessageID` in a database or Redis cache and discarding messages that have already been processed.

**Q: What is the "Fan-Out" pattern?**
**A:** Fan-Out occurs when a message published to a single topic is distributed to multiple distinct queues or services. For example, a "UserSignup" event might be fanned out to:
1.  Email Service (send welcome email)
2.  Analytics Service (log signup)
3.  Fraud Service (check IP)
All three services process the same event in parallel.
