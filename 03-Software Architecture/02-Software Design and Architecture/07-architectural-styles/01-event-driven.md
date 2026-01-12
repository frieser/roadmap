---
---

# Event-Driven Architecture (EDA)

## Summary
Event-Driven Architecture (EDA) is a software architectural pattern where decoupled applications and services communicate by asynchronously producing, detecting, and consuming events. Unlike request/response models (like HTTP), EDA allows services to react to changes in state (events) in near real-time without direct dependencies between producers and consumers. This style is critical for modern microservices, serverless applications, and reactive systems.

## Detailed Explanation

### 1. Core Concepts

EDA revolves around the concept of an **Event**—a significant change in state (e.g., "OrderPlaced", "PaymentFailed").

*   **Event Producer**: The component that detects the state change and emits the event. It does not know who listens to it.
*   **Event Consumer (Sink)**: The component that listens for events and reacts to them (e.g., updating a database, sending an email).
*   **Event Channel/Router**: The middleware (Message Broker) that transmits events from producers to consumers.

### 2. Common Topologies

#### A. Broker Topology (Choreography)
There is no central coordinator. Producers publish events to a broker (e.g., RabbitMQ, Kafka), and consumers subscribe to topics.
*   **Flow**: Service A emits "OrderCreated". Service B and Service C listen for it and act independently.
*   **Best for**: High scalability, loose coupling.

#### B. Mediator Topology (Orchestration)
A central mediator (Orchestrator) receives events and directs the logic of which steps happen next.
*   **Flow**: Service A emits "OrderCreated" to the Mediator. Mediator commands Service B to "ProcessPayment" and Service C to "SendEmail".
*   **Best for**: Complex workflows where sequence matters (e.g., Sagas).

### 3. Key Patterns within EDA
*   **Event Notification**: "Something happened". The event payload is minimal (often just an ID). The consumer calls back to fetch details.
*   **Event-Carried State Transfer**: The event contains all changed data. The consumer updates its local cache without calling back.
*   **Event Sourcing**: The state of the system is stored as a sequence of events, not just the current snapshot.

### 4. Pros & Cons

| Feature | Description |
| :--- | :--- |
| **Decoupling** | Producers and consumers don't need to know about each other. |
| **Scalability** | Components can scale independently based on event volume. |
| **Resilience** | If a consumer is down, events persist in the broker until it recovers. |
| **Complexity** | Harder to debug and trace flow compared to monolithic/synchronous calls. |
| **Consistency** | Relies on **Eventual Consistency**. Immediate data consistency is hard to guarantee. |

## Real-World Examples
*   **Apache Kafka**: High-throughput event streaming.
*   **RabbitMQ / ActiveMQ**: Traditional message queuing.
*   **AWS EventBridge / SNS / SQS**: Cloud-native event routing.
*   **Redis Streams**: Lightweight streaming.

## Go Implementation Example

Go is excellent for EDA due to its concurrency primitives (Channels).

### Simple In-Memory Event Bus

```go
package main

import (
	"fmt"
	"time"
)

// Event represents a state change
type Event struct {
	Name    string
	Payload interface{}
}

// EventBus dispatches events to subscribers
type EventBus struct {
	subscribers map[string][]chan Event
}

func (eb *EventBus) Subscribe(topic string) <-chan Event {
	ch := make(chan Event)
	eb.subscribers[topic] = append(eb.subscribers[topic], ch)
	return ch
}

func (eb *EventBus) Publish(topic string, data interface{}) {
	if chans, found := eb.subscribers[topic]; found {
		for _, ch := range chans {
			// Non-blocking send
			go func(c chan Event) {
				c <- Event{Name: topic, Payload: data}
			}(ch)
		}
	}
}

func main() {
	bus := &EventBus{subscribers: make(map[string][]chan Event)}

	// Consumer: Billing Service
	billingCh := bus.Subscribe("order_created")
	go func() {
		for e := range billingCh {
			fmt.Printf("[Billing] Processing payment for order: %v\n", e.Payload)
		}
	}()

	// Producer: Order Service
	fmt.Println("Creating Order...")
	bus.Publish("order_created", map[string]int{"order_id": 101, "amount": 50})

	time.Sleep(time.Second)
}
```

## Interview Questions

**Q: What is the difference between Message Queues and Event Streams?**
**A:** A **Message Queue** (e.g., RabbitMQ) typically removes the message once it is processed by a consumer (destructive). An **Event Stream** (e.g., Kafka) retains the history of events (log-based), allowing multiple consumers to read the same events at different times or replay history.

**Q: How do you handle "Eventual Consistency" in EDA?**
**A:** Since updates don't happen atomically across services, the system may be temporarily inconsistent. We handle this by designing idempotent consumers (handling the same event twice safely), using Sagas for distributed transactions, and accepting that the UI might need to poll or use WebSockets for final status updates.

**Q: What is the "Dead Letter Queue" (DLQ)?**
**A:** A DLQ is a storage queue for events that could not be processed successfully after a certain number of retries. It prevents bad events from blocking the system and allows developers to inspect/fix the root cause later.
