---
---

## Summary
The **CAP Theorem** (Brewer's Theorem) states that a distributed data store can only provide **two** of the following three guarantees: **Consistency** (every read receives the most recent write), **Availability** (every request receives a response), and **Partition Tolerance** (system continues to operate despite network failures). In reality, Partition Tolerance (P) is unavoidable in distributed systems, so the choice is usually **CP vs AP**.

## Detailed Explanation
### The Three Guarantees
1.  **Consistency (C)**: Linearizability. All nodes see the same data at the same time.
2.  **Availability (A)**: Every request gets a non-error response, regardless of the state of individual nodes.
3.  **Partition Tolerance (P)**: The system continues to work even if communication between nodes is lost (network partition).

### The Trade-off (CP vs AP)
Since networks *will* fail (P is a given), you must choose what to do when nodes cannot talk to each other:
*   **CP (Consistency + Partition Tolerance)**: Stop accepting writes to avoid data divergence. "Better to be down than wrong." (e.g., MongoDB, HBase, Banking).
*   **AP (Availability + Partition Tolerance)**: Accept writes on both sides of the partition, even if they conflict. "Better to remain online than be perfect." (e.g., Cassandra, DynamoDB, Social Media).
*   **CA (Consistency + Availability)**: Only possible if partitions *never* happen (i.e., single node/monolith). Not applicable to distributed systems.

### Go Context
Go developers building distributed services must decide:
*   Do I return an error (`500`) when the DB master is unreachable (CP)?
*   Do I return stale data from a local cache/replica (AP)?

## Interview Questions
**Q: Why can't you have all three (C, A, and P)?**
A: If a partition occurs (P), Node 1 cannot talk to Node 2. If you allow a write to Node 1, Node 2 doesn't know about it.
*   If you let Node 2 serve the old data, you have Availability (A) but lost Consistency (C).
*   If you stop Node 2 from serving data until it talks to Node 1, you have Consistency (C) but lost Availability (A).

**Q: Is a traditional SQL database CA?**
A: In a single-node setup, yes. But once you introduce clustering/replication, it usually defaults to CP (Master-Slave) or requires configuration to become AP.

## Diagram
```mermaid
graph TD
    C[Consistency]
    A[Availability]
    P[Partition Tolerance]
    
    C --- P
    A --- P
    C -.- A
    
    note[Pick 2 lines (CP or AP)\nCA is not real in distributed systems]
```
