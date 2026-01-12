# CQRS and Eventual Consistency

## Summary
Command Query Responsibility Segregation (CQRS) is an architectural pattern that separates the models for reading and writing data. It addresses the mismatch between the domain's transactional requirements (complex validation, strong consistency) and its query requirements (high performance, diverse views). In distributed systems, this often leads to **Eventual Consistency**, where read models are updated asynchronously via Event Sourcing or messaging.

## Detailed Explanation

### 1. Separation of Read and Write Models

Traditional CRUD architectures use a single data model for both operations, which can lead to complex and inefficient schemas. CQRS splits this into:

#### The Command Side (Write Model)
*   **Responsibility**: Validates business rules, enforces invariants, and accepts/rejects commands.
*   **Characteristics**: Normalized (3NF), optimized for transactional integrity, behavior-rich.
*   **Consistency**: Strong consistency (ACID) within the aggregate boundary.

#### The Query Side (Read Model)
*   **Responsibility**: Returns data to the UI/API.
*   **Characteristics**: Denormalized (often materialized views), optimized for read performance (e.g., avoiding joins).
*   **Consistency**: Eventually consistent (BASE).

### 2. Event Sourcing Integration

While CQRS can be implemented with just two databases (e.g., SQL for writes, ElasticSearch for reads), it is most powerful when paired with **Event Sourcing**.
*   **Source of Truth**: The sequence of immutable events (`OrderCreated`, `AddressUpdated`) is the primary record.
*   **Projections**: Background processes ("Projectors") consume these events to update the Read Models.
*   **Replayability**: New read models (e.g., a new report) can be built by replaying the entire event history.

### 3. Eventual Consistency & CAP Theorem

In a distributed environment (Microservices), CQRS implies a choice in the **CAP Theorem**:
*   The Write side favors **Consistency (CP)** to ensure data integrity.
*   The Read side favors **Availability (AP)** to ensure the system is always responsive, even if data is slightly stale.
*   **Handling Lag**: The time between a command being accepted and the read model being updated is the "consistency window."

### 4. Trade-offs

| Feature | Monolith / CRUD | CQRS |
| :--- | :--- | :--- |
| **Complexity** | Low | **High** (requires messaging, error handling for async updates) |
| **Scalability** | Limited by single DB | **High** (Read and Write scale independently) |
| **Security** | Single point of control | Granular control (e.g., different permissions for R/W) |
| **Performance** | Reads can be slow (joins) | **Instant Reads** (pre-calculated views) |

## Go Implementation (Command Handler Pattern)

```go
package main

import "fmt"

// Command: Intent to change state
type CreateOrderCommand struct {
	OrderID   string
	UserID    string
	Total     float64
}

// CommandHandler: Business logic
type OrderHandler struct {
	eventStore EventStore
}

func (h *OrderHandler) Handle(cmd CreateOrderCommand) error {
	// 1. Validate
	if cmd.Total <= 0 {
		return fmt.Errorf("invalid total")
	}

	// 2. Create Event
	event := OrderCreatedEvent{
		ID:    cmd.OrderID,
		Total: cmd.Total,
	}

	// 3. Persist Event (Source of Truth)
	return h.eventStore.Append(event)
}

// EventStore interface
type EventStore interface {
	Append(event interface{}) error
}

// ... Projectors would listen to these events to update the Read DB ...
```

## Interview Questions

*   **Q: What is the main difference between CQRS and CQS?**
    *   **A:** CQS (Command Query Separation) is a code-level principle (methods should either return data or change state, not both). CQRS is an architectural pattern that applies CQS to the entire system, separating the data models and often the infrastructure for reads and writes.
*   **Q: How do you handle a user who expects to see their changes immediately in an Eventually Consistent system?**
    *   **A:** 
        1.  **Optimistic UI**: Update the UI immediately assuming success.
        2.  **Read-Your-Own-Writes**: Route the user's specific read requests to the Write DB (or a synchronous replica) for a short window.
        3.  **Client-side Polling**: Have the client poll the read API until the specific version/event ID appears.
*   **Q: When is CQRS overkill?**
    *   **A:** For simple CRUD applications, systems with low concurrency, or domains where the read and write models are nearly identical. The complexity of maintaining two models and a synchronization mechanism is not justified.
