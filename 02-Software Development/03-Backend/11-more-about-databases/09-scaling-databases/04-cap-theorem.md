---
---

## Summary
The CAP Theorem (Brewer's Theorem) states that in a distributed data store, it is impossible to simultaneously provide more than two out of three guarantees: **Consistency, Availability, and Partition Tolerance**.

## Detailed Explanation

### The Three Guarantees
1.  **Consistency (C)**: Every read receives the most recent write or an error. (All nodes see the same data at the same time).
2.  **Availability (A)**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
3.  **Partition Tolerance (P)**: The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.

### The Trade-off
Since network partitions (P) are inevitable in distributed systems, we must choose between:
-   **CP (Consistency + Partition Tolerance)**: The system returns an error if it cannot guarantee consistency during a partition. (e.g., MongoDB, Redis, HBase).
-   **AP (Availability + Partition Tolerance)**: The system returns the best available data, even if it might be stale. (e.g., Cassandra, CouchDB, DynamoDB).

### Why 'CA' doesn't really exist?
In a world with networks, partitions *will* happen. If you choose CA, your system will fail completely if the network goes down. Most "CA" systems are actually single-node databases.

## Go-specific Context
When building distributed microservices in Go, you must decide which side of the CAP theorem your service falls on.

### CAP in Go Microservices
-   **Using Postgres (CP focus)**: If you need high consistency for financial transactions.
-   **Using Cassandra (AP focus)**: If you need a global system that must never go down, even if some users see slightly older data (e.g., social media likes).

### Dealing with Eventual Consistency (AP)
If you choose AP, your Go code must be able to handle "stale" data.
```go
// In an AP system, this might return an older version of the user
user, _ := cassandraClient.Get(ctx, "user:123")
```

## Interview Questions
**Q: What happens to a CP system during a network partition?**
**A:** The system will stop accepting writes and potentially reads until the partition is resolved, to ensure that no inconsistent data is recorded or served. It prioritizes truth over availability.

**Q: What is 'Eventual Consistency'?**
**A:** A consistency model used in AP systems. It guarantees that if no new updates are made to a data item, eventually all accesses to that item will return the last updated value.

**Q: Which two properties do traditional SQL databases like MySQL usually provide?**
**A:** In a single-node setup, they provide **CA** (Consistency and Availability). However, when you introduce replication and clustering, they must choose between CP and AP during a partition.
