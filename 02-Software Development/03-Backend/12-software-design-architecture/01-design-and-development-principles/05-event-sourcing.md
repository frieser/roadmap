---
---

## Summary
Event Sourcing is an architectural pattern where state changes are stored as a sequence of events rather than just the current state. The current state of an application is derived by replaying these events. In Go, this is often implemented using a central "Event Store" and is frequently paired with CQRS (Command Query Responsibility Segregation).

## Detailed Explanation

### Core Concepts
*   **Events**: Immutable records of something that happened in the past (e.g., `OrderPlaced`, `ItemAdded`).
*   **Event Store**: An append-only database specifically designed to store events.
*   **Replay**: The process of re-applying events from the store to reconstruct the current state of an Aggregate.
*   **Snapshots**: Optimization technique where the state is saved at a specific point in time to avoid replaying thousands of events.

### Advantages
*   **Complete Audit Trail**: You have a perfect history of everything that happened in the system.
*   **Time Travel**: You can reconstruct the state of the system at any point in history.
*   **Scalability**: Writes are very fast (append-only), and reads can be scaled independently using projections.

## Go-specific Context and Examples

Implementing Event Sourcing in Go involves defining events as structs and an Event Store interface.

### Defining Events
```go
package events

import "time"

type Event interface {
	EventType() string
}

type UserRegistered struct {
	UserID   string
	Username string
	At       time.Time
}

func (e UserRegistered) EventType() string { return "UserRegistered" }

type EmailChanged struct {
	NewEmail string
	At       time.Time
}

func (e EmailChanged) EventType() string { return "EmailChanged" }
```

### Aggregate Root with Event Replay
```go
package domain

type UserAggregate struct {
	ID       string
	Username string
	Email    string
	Version  int
}

func (u *UserAggregate) Apply(event Event) {
	switch e := event.(type) {
	case UserRegistered:
		u.ID = e.UserID
		u.Username = e.Username
	case EmailChanged:
		u.Email = e.NewEmail
	}
	u.Version++
}
```

### Event Store Interface
```go
package persistence

import "context"

type EventStore interface {
	Save(ctx context.Context, aggregateID string, events []Event, expectedVersion int) error
	Load(ctx context.Context, aggregateID string) ([]Event, error)
}
```

## Interview Questions

**Q: What is the main difference between traditional state-based storage and Event Sourcing?**
**A:** Traditional storage only keeps the latest "snapshot" of data (the current state). Event Sourcing stores every change as an immutable event. The current state is a "projection" of all historical events.

**Q: How do you handle read performance in an Event Sourcing system?**
**A:** By using CQRS. You separate the Write side (Event Store) from the Read side. The Read side uses "Projections" (read models) that are updated as events are published, allowing for highly optimized queries.

**Q: What are the challenges of Event Sourcing?**
**A:** Increased complexity, event versioning (handling changes in event schemas over time), and the eventual consistency model between the write and read sides.
