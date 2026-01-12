---
---

## Summary
**Strong Consistency** (also known as Linearizability) guarantees that a system behaves as if it runs on a single node. Once a write is confirmed, any subsequent read operation, from any client, will return that new value. It prioritizes **Data Correctness** over Availability and Latency.

## Detailed Explanation

### Core Characteristics
1.  **Single Copy Image**: Users never see "stale" data. The data is consistent across all nodes instantly (from the client's perspective).
2.  **Synchronous Replication**: A write is not confirmed until a quorum (majority) of nodes have acknowledged it.
3.  **High Latency**: The requirement to wait for network round-trips to other nodes reduces performance.
4.  **Reduced Availability**: If a quorum of nodes cannot be reached (network partition), the system must reject writes to prevent divergence (CP in CAP).

### Implementation Protocols
*   **Two-Phase Commit (2PC)**: A strict protocol where a coordinator asks all nodes "Can you commit?" before telling them "Commit." If one says no, the transaction aborts.
*   **Consensus Algorithms (Paxos / Raft)**: Used in modern distributed systems (Etcd, Consul) to agree on a log of changes.

### Use Cases
*   **Banking**: You cannot withdraw money that you just transferred to someone else.
*   **Inventory Management**: You cannot sell the last item in stock to two different people.
*   **Distributed Locks**: Preventing two processes from modifying a resource simultaneously.

## Go Example (Conceptual)
Using a Mutex to enforce strict serial access, simulating a CP system where readers must wait for writers.

```go
package main

import (
	"fmt"
	"sync"
)

type BankAccount struct {
	mu      sync.Mutex
	balance int
}

// Write (Deposit) blocks until it completes explicitly
func (b *BankAccount) Deposit(amount int) {
	b.mu.Lock() // Acquire lock (Consensus/Lease)
	defer b.mu.Unlock()
	b.balance += amount
	fmt.Println("Deposit confirmed.")
}

// Read (GetBalance) is guaranteed to see the latest Deposit
func (b *BankAccount) GetBalance() int {
	b.mu.Lock() // Wait for any pending writes to finish
	defer b.mu.Unlock()
	return b.balance
}
```

## Interview Questions

### Q: Does Strong Consistency mean zero latency?
**A:** No, quite the opposite. It usually implies *higher* latency because the system must communicate with multiple replicas before confirming an operation. "Instant" refers to the *logical* visibility of data, not the physical speed.

### Q: Can you achieve Strong Consistency with Leader-Follower replication?
**A:** Only if you read from the Leader (or use synchronous replication to followers). If you read from an async follower, you risk seeing stale data, which breaks strong consistency.
