---
---

# Cloud Design Patterns: Data Management

Data management patterns address how to handle data consistency, performance, and storage in distributed systems.

## Summary

*   **CQRS (Command and Query Responsibility Segregation)**: Segregating operations that read data from operations that update data.
*   **Event Sourcing**: Storing the state of a system as a sequence of events.
*   **Sharding**: Dividing a data store into a set of horizontal partitions or shards.

---

## Go Implementation: CQRS (Simplified)

Separating the "Write" model from the "Read" model using Interfaces in Go.

```go
package main

import "fmt"

// --- Command Side (Write) ---
type CommandHandler struct {
	store map[string]string
}

func (c *CommandHandler) CreateUser(id string, name string) {
	fmt.Printf("Command: Creating User %s\n", name)
	c.store[id] = name
	// In real CQRS, this would publish an event like "UserCreated"
}

// --- Query Side (Read) ---
type QueryHandler struct {
	// In real CQRS, this might be a completely different DB (e.g., ElasticSearch vs Postgres)
	// synced via events.
	readReplica map[string]string 
}

func (q *QueryHandler) GetUser(id string) string {
	fmt.Printf("Query: Reading User %s\n", id)
	return q.readReplica[id]
}

func main() {
	// Shared storage for demo purposes
	data := make(map[string]string)
	
	commands := &CommandHandler{store: data}
	queries := &QueryHandler{readReplica: data}

	// Write
	commands.CreateUser("1", "Alice")

	// Read
	user := queries.GetUser("1")
	fmt.Println("Result:", user)
}
```

## Interview Questions

**Q: When should you use CQRS?**
**A:** Use CQRS when there is a large imbalance between the number of reads and writes, or when the "Read" model needs to be denormalized for performance (e.g., complex joins are pre-calculated). It adds significant complexity, so don't use it for simple CRUD apps.

**Q: What is the main challenge of Sharding?**
**A:** **Resharding**. If your shards become unbalanced (e.g., one shard gets all the traffic), moving data to new shards while the system is live is extremely difficult and risky. Choosing the correct Shard Key (Partition Key) upfront is critical.
