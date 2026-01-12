---
---

## Summary
**CQRS (Command Query Responsibility Segregation)** is a pattern that separates the models for **reading** data (Queries) from the models for **writing** data (Commands). This separation allows each side to be optimized independently, often leading to performance gains and scalability in complex systems.

## Detailed Explanation

### The Problem with Single Model
In traditional CRUD (Create, Read, Update, Delete), a single model acts as both the data storage and the data view.
*   **Complex joins**: Reads often require complex queries that don't match the normalized write schema.
*   **Contention**: Locking rows for updates can slow down reads.
*   **Scaling**: You often need to scale reads (1000x) more than writes (1x), but a single model forces you to scale both together.

### The CQRS Solution
1.  **Command Side (Write)**: Focuses on business logic and validation. Optimized for transactional integrity.
2.  **Query Side (Read)**: Focuses on fetching data fast. Can use denormalized views, materialized views, or even a completely different database (e.g., ElasticSearch).

### Relationship with Event Sourcing
CQRS is often used with Event Sourcing. The Write side generates events, which are then consumed by the Read side to update separate "Read Models" (Projections).

## Go Example structure

```go
package cqrs

import "context"

// --- COMMAND SIDE ---

type CreateOrderCommand struct {
	UserID string
	Items  []string
}

type OrderCommandHandler struct {
	repo OrderRepository // Write-optimized repository
}

func (h *OrderCommandHandler) Handle(ctx context.Context, cmd CreateOrderCommand) error {
	// 1. Validate
	if len(cmd.Items) == 0 {
		return errors.New("order must have items")
	}
	// 2. Execute Domain Logic
	order := domain.NewOrder(cmd.UserID, cmd.Items)
	
	// 3. Persist
	return h.repo.Save(ctx, order)
}

// --- QUERY SIDE ---

type OrderQueryService struct {
	db *sql.DB // Read-optimized DB (maybe with Read Replicas)
}

type OrderSummaryDTO struct {
	OrderID    string
	ItemCount  int
	TotalPrice float64
}

// GetOrderSummary fetches a specific view optimized for the UI
func (q *OrderQueryService) GetOrderSummary(ctx context.Context, orderID string) (*OrderSummaryDTO, error) {
	// Simple SELECT, no domain logic, maybe joining denormalized tables
	row := q.db.QueryRow("SELECT id, item_count, total FROM order_summaries WHERE id=?", orderID)
	// ... scan and return DTO
}
```

## Interview Questions

### Q: When should you use CQRS?
**A:** Use it when there is a high mismatch between the read and write models, or when the read/write load is extremely asymmetrical. Don't use it for simple CRUD apps; it adds unnecessary complexity (2 models to maintain).

### Q: What is the main trade-off of CQRS?
**A:** **Consistency**. If the Read model is updated asynchronously (especially with Event Sourcing), the system becomes **Eventually Consistent**. Users might create an item and not see it immediately in the list.
