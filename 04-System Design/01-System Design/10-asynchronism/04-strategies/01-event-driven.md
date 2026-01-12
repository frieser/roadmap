---
---

## Summary
**Event-Driven Architecture (EDA)** is a design pattern where decoupled components communicate by producing and consuming events (state changes). It promotes loose coupling, scalability, and responsiveness.

## Detailed Explanation

### Core Concepts
*   **Event**: A significant change in state (e.g., `UserSignedUp`, `PaymentFailed`). Events are immutable.
*   **Producer**: Detects the change and publishes the event. Does not know who listens.
*   **Consumer**: Subscribes to events and reacts (e.g., Send Email, Update CRM).

### Event Sourcing
A specific pattern where the **Event Log** is the single source of truth. Instead of storing `User {Balance: 100}`, you store `[Deposited(50), Withdrawn(20), Deposited(70)]`.
*   **Pros**: Full audit trail, ability to replay history ("Time Travel").
*   **Cons**: Complexity, need for snapshots to improve read performance.

## Go Context: Channel-based Event Bus
A simple in-memory event bus.

```go
type EventBus struct {
    subscribers map[string][]chan string
}

func (b *EventBus) Subscribe(topic string) chan string {
    ch := make(chan string)
    b.subscribers[topic] = append(b.subscribers[topic], ch)
    return ch
}

func (b *EventBus) Publish(topic string, msg string) {
    for _, ch := range b.subscribers[topic] {
        go func(c chan string) { c <- msg }(ch)
    }
}
```

## Interview Questions

### Q: What is the difference between a Command and an Event?
**A:**
*   **Command**: An intent to do something (e.g., `CreateUser`). It is directed at a specific handler and can fail/be rejected.
*   **Event**: A notification that something *has happened* (e.g., `UserCreated`). It is broadcast to anyone interested and cannot be rejected (it's a fact).

### Q: What is the main challenge of Event-Driven systems?
**A:** **Eventual Consistency** and **Debugging**. Since processes happen asynchronously across distributed nodes, it's hard to trace a request's flow (requires Distributed Tracing like OpenTelemetry) or guarantee immediate data consistency.
