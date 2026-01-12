---
---

## Summary
**Event Sourcing** is a pattern where the state of an application is not stored as the "current state" in a database row, but rather as a **sequence of events** (immutable state changes) that happened over time. The current state is derived by replaying these events.

## Detailed Explanation

### Core Concept
Instead of:
*   `UPDATE orders SET status = 'SHIPPED' WHERE id = 1`

You store:
1.  `OrderCreated {ID: 1, Items: [...]}`
2.  `PaymentVerified {ID: 1}`
3.  `OrderShipped {ID: 1}`

To get the current state of Order 1, you load all events and replay them (apply them) to an empty Order object.

### Benefits
1.  **Audit Trail**: You have a 100% accurate history of *everything* that happened.
2.  **Time Travel**: You can reconstruct the state of the system as it was at any point in the past.
3.  **Debuggability**: If a bug corrupts state, you can fix the bug and replay events to restore correct state.

### Challenges
*   **Versioning**: What happens if the event structure changes? (Need migration strategies).
*   **Performance**: Replaying 10,000 events to get state is slow. Solution: **Snapshots** (save state every N events).
*   **Complexity**: Requires a shift in mindset and specialized infrastructure (Event Store).

## Go Example

```go
package eventsourcing

// Event interface
type Event interface {
	EventType() string
}

type OrderCreated struct { ID string; Amount int }
func (e OrderCreated) EventType() string { return "OrderCreated" }

type OrderShipped struct { ID string }
func (e OrderShipped) EventType() string { return "OrderShipped" }

// Aggregate
type Order struct {
	ID      string
	Amount  int
	Shipped bool
}

// Apply updates the state based on the event (Reducer logic)
func (o *Order) Apply(e Event) {
	switch ev := e.(type) {
	case OrderCreated:
		o.ID = ev.ID
		o.Amount = ev.Amount
		o.Shipped = false
	case OrderShipped:
		o.Shipped = true
	}
}

// Rehydrate reconstructs state from history
func RehydrateOrder(events []Event) *Order {
	o := &Order{}
	for _, event := range events {
		o.Apply(event)
	}
	return o
}
```

## Interview Questions

### Q: What is a Snapshot in Event Sourcing?
**A:** A Snapshot is a saved version of the Aggregate's state at a specific point in time (e.g., after event #100). When loading the aggregate, you load the latest snapshot and then only replay events that happened *after* that snapshot, significantly speeding up performance.

### Q: Can you edit or delete events?
**A:** Generally, **NO**. The Event Store should be an append-only, immutable log. If a mistake was made (e.g., "Wrong Address"), you append a *correction event* ("AddressCorrected"), you don't delete the history.
