---
---

## Summary
**CQRS (Command Query Responsibility Segregation)** is an architectural pattern that separates the models for reading and writing data. Unlike traditional CRUD where the same data model is used for both operations, CQRS typically uses a **Command Model** to update the system (write) and a separate **Query Model** to read the system state.

## Detailed Explanation
In complex systems, the read and write workloads often have vastly different performance, scale, and consistency requirements.
*   **Write Side (Command)**: Focuses on complex business logic, validation, and data consistency. It handles "Commands" (e.g., `PlaceOrder`, `UpdateCustomer`).
*   **Read Side (Query)**: Focuses on fast data retrieval. It handles "Queries" (e.g., `GetOrderById`, `ListRecentOrders`).

### Benefits
1.  **Independent Scaling**: You can scale read replicas independently of write nodes (useful for read-heavy systems).
2.  **Optimized Schemas**: The read database can be denormalized (e.g., pre-joined tables in SQL or a document in MongoDB) for speed, while the write database remains normalized (3NF) for integrity.
3.  **Security**: You can apply different security permissions to read vs. write operations.

### Go Implementation
In Go, CQRS is often implemented by defining separate interfaces or structs for Commands and Queries.

```go
package main

import (
	"context"
	"errors"
	"fmt"
)

// --- Command Side ---

type CreateOrderCommand struct {
	CustomerID string
	Amount     float64
}

type OrderCommandHandler struct {
	// dependencies like Repository, EventBus
}

func (h *OrderCommandHandler) Handle(ctx context.Context, cmd CreateOrderCommand) error {
	if cmd.Amount <= 0 {
		return errors.New("amount must be positive")
	}
	// Logic: Save to WriteDB, Publish Event
	fmt.Printf("Command: Order created for Customer %s with amount %.2f\n", cmd.CustomerID, cmd.Amount)
	return nil
}

// --- Query Side ---

type OrderQueryModel struct {
	OrderID    string
	Total      float64
	Status     string
}

type GetOrderQuery struct {
	OrderID string
}

type OrderQueryHandler struct {
	// dependencies like ReadOnlyDB (e.g., Redis, Elasticsearch)
}

func (h *OrderQueryHandler) Handle(ctx context.Context, query GetOrderQuery) (*OrderQueryModel, error) {
	// Logic: Fetch from ReadDB (fast, denormalized)
	fmt.Printf("Query: Fetching order %s\n", query.OrderID)
	return &OrderQueryModel{OrderID: query.OrderID, Total: 100.00, Status: "SHIPPED"}, nil
}

/*
Usage:
func main() {
    // Wiring it up
    cmdHandler := &OrderCommandHandler{}
    queryHandler := &OrderQueryHandler{}
    
    // Write
    cmdHandler.Handle(context.Background(), CreateOrderCommand{CustomerID: "123", Amount: 99.99})
    
    // Read
    queryHandler.Handle(context.Background(), GetOrderQuery{OrderID: "555"})
}
*/
```

## Interview Questions

**Q: What is the relationship between CQS (Command Query Separation) and CQRS?**
**A:** 
*   **CQS** is a principle at the class/method level (a method should either return a value OR change state, not both).
*   **CQRS** applies this principle at the architectural level (separate subsystems/databases for reads and writes). CQRS is essentially "CQS at scale."

**Q: How do you handle "Read Your Own Writes" consistency issues?**
**A:** Since the Read DB is often eventually consistent (updated asynchronously), a user might create an order and not see it immediately in the list. Solutions:
1.  **Optimistic UI**: The frontend assumes success and adds the item to the list locally.
2.  **Write-Side Read**: For specific critical flows (like the "Success" page), read from the primary Write DB or cache.
3.  **Version Tokens**: The client sends the version of the write it just performed, and the Read API waits until its projection catches up to that version before returning.

**Q: When is CQRS an Anti-Pattern?**
**A:** For simple CRUD applications. If your Read model is just a 1:1 mapping of your Write model, CQRS adds massive accidental complexity (synchronization code, extra databases) with zero benefit. Use it only when the read/write access patterns or logic diverge significantly.
