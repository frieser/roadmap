---
---

## Summary
The **CAP Theorem** states that in a distributed system, you can only provide two of the three guarantees: **Consistency** (every read receives the most recent write), **Availability** (every request receives a response), and **Partition Tolerance** (system continues to work despite network failures). Since network partitions are inevitable, you must choose between **CP** (Consistency/Data correctness) and **AP** (Availability/Uptime).

## Detailed Explanation

### The Three Pillars
1.  **Consistency (Linearizability)**: All nodes see the same data at the same time. A read is guaranteed to return the most recent write or an error.
2.  **Availability**: Every request gets a non-error response, without the guarantee that it contains the most recent write.
3.  **Partition Tolerance**: The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.

### The Trade-off (PACELC)
Since P (Partition Tolerance) is mandatory in distributed systems (networks fail), the real choice is:
*   **CP (Consistency > Availability)**: If the network splits, stop accepting writes to prevent data divergence.
    *   *Example*: Banking, Inventory Management.
*   **AP (Availability > Consistency)**: If the network splits, keep accepting writes on both sides, even if they conflict. Reconcile later (Eventual Consistency).
    *   *Example*: Social Media Feeds, Shopping Carts.

### Consistency Models
1.  **Strong Consistency**: Data is instantly replicated. Highest latency.
2.  **Eventual Consistency**: Data will replicate *eventually*. Fastest performance, lowest consistency.
3.  **Causal Consistency**: Preserves the order of causally related events (replies follow comments), but unrelated events can be out of order.

## High Availability Techniques
*   **Replication**: Storing copies of data on multiple nodes.
*   **Failover**: Automatically switching to a standby server if the primary fails.
    *   *Active-Passive*: One writes, others wait.
    *   *Active-Active*: All accept writes (complex conflict resolution).

## Go Example (Conceptual)

### Eventual Consistency Simulation
Simulating a "Gossip Protocol" where nodes update independently.

```go
type Node struct {
    Data string
    mu   sync.Mutex
}

func (n *Node) Update(val string) {
    n.mu.Lock()
    defer n.mu.Unlock()
    n.Data = val
    // Async replication (Eventual Consistency)
    go n.BroadcastToPeers(val) 
}

func (n *Node) Read() string {
    n.mu.Lock()
    defer n.mu.Unlock()
    return n.Data // Might return old data if replication hasn't arrived
}
```

## Interview Questions

### Q: Can you have a CA system?
**A:** In a distributed system, no. A CA system implies that partitions (network failures) never happen, which is impossible in the real world. CA is only possible in a monolithic system running on a single machine.

### Q: How do you handle consistency in an AP system?
**A:** You accept **Eventual Consistency**. You use techniques like **Read Repair** (fix data when reading it), **Vector Clocks** (detect conflicts), or **Last-Write-Wins** (simple but risky) to reconcile data differences once the partition heals.
