---
---

## Summary
**Event Sourcing** is a persistence pattern where the state of an application is determined by a sequence of events. Instead of storing just the *current state* of data (e.g., `OrderStatus: "Shipped"`), you store a log of *everything that happened* (e.g., `OrderCreated`, `PaymentVerified`, `OrderShipped`). The current state is derived by replaying these events.

## Detailed Explanation
In traditional databases, updating a record overwrites the previous state—history is lost. Event Sourcing flips this:
1.  **Immutable Events**: Changes are stored as immutable events in an **Event Store** (an append-only log).
2.  **Deriving State**: To get the current state of an object (Aggregate), the system reads all events for that ID and applies them sequentially.
3.  **Audit Trail**: You have a perfect, unchangeable history of every business action ever taken.
4.  **Time Travel**: You can reconstruct the state of the system as it was at any point in the past.

### Comparison
*   **Traditional**: `UPDATE accounts SET balance = 150 WHERE id = 1;` (Old value 100 is lost).
*   **Event Sourcing**: `INSERT INTO events (type, amount) VALUES ('Deposited', 50);` (Current balance is calculated as 100 + 50).

### Go Implementation
Implementing an **Aggregate Root** that applies events to rebuild its state.

```go
package main

import (
	"fmt"
	"time"
)

// Event Interface
type Event interface {
	EventType() string
}

// Concrete Events
type AccountOpened struct { Owner string }
func (e AccountOpened) EventType() string { return "AccountOpened" }

type MoneyDeposited struct { Amount int }
func (e MoneyDeposited) EventType() string { return "MoneyDeposited" }

type MoneyWithdrawn struct { Amount int }
func (e MoneyWithdrawn) EventType() string { return "MoneyWithdrawn" }

// Aggregate
type BankAccount struct {
	ID      string
	Owner   string
	Balance int
	Version int
}

// Apply updates the state based on the event (Transition Logic)
func (a *BankAccount) Apply(e Event) {
	switch ev := e.(type) {
	case AccountOpened:
		a.Owner = ev.Owner
	case MoneyDeposited:
		a.Balance += ev.Amount
	case MoneyWithdrawn:
		a.Balance -= ev.Amount
	}
	a.Version++
}

// Rehydrate builds the current state from history
func NewBankAccountFromHistory(events []Event) *BankAccount {
	acc := &BankAccount{}
	for _, e := range events {
		acc.Apply(e)
	}
	return acc
}

/*
Usage:
func main() {
    // 1. Load history from Event Store (DB)
    history := []Event{
        AccountOpened{Owner: "Alice"},
        MoneyDeposited{Amount: 100},
        MoneyWithdrawn{Amount: 20},
    }

    // 2. Rehydrate state
    account := NewBankAccountFromHistory(history)

    fmt.Printf("Account Owner: %s, Balance: %d\n", account.Owner, account.Balance)
    // Output: Account Owner: Alice, Balance: 80
}
*/
```

## Interview Questions

**Q: What is Snapshotting and why is it necessary?**
**A:** If an aggregate (e.g., a Bank Account) has millions of events, replaying them all every time you load the object is too slow. A **Snapshot** is a saved copy of the current state at a specific version (e.g., every 1000 events). To load, the system grabs the latest snapshot and then only plays the events that occurred *after* that snapshot.

**Q: How do you handle Schema Evolution (changing event structure)?**
**A:** Since events are immutable, you can't just "alter table". Strategies include:
1.  **Upcasting**: When loading events, a middleware converts old event versions (v1) into the new structure (v2) on the fly.
2.  **Flexible Serialization**: Use JSON serialization in Go that tolerates missing fields or has default values.

**Q: Can you delete data in Event Sourcing (GDPR)?**
**A:** "Crypto-shredding" is the standard pattern. You encrypt the sensitive data in the event payload with a per-user key. To "delete" the data, you delete the key. The events remain (preserving the log integrity), but the sensitive data inside them becomes unreadable garbage.
