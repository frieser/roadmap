---
---

## Summary
Database Replication is the process of copying data from one database server (the Leader/Primary) to one or more other servers (the Followers/Replicas). It is used to increase data availability, improve read performance, and provide disaster recovery.

## Detailed Explanation

### Types of Replication
1.  **Single-Leader**: All writes go to one leader. Followers replicate data from the leader. Reads can be spread across all nodes. (Common in Postgres/MySQL).
2.  **Multi-Leader**: Writes can happen on multiple nodes. Good for multi-region setups, but complex to handle conflicts.
3.  **Leaderless (Dynamo-style)**: Any node can accept reads and writes. Used in NoSQL databases like Cassandra.

### Synchronous vs. Asynchronous
-   **Synchronous**: The leader waits for followers to confirm the write. Guarantees no data loss but increases latency.
-   **Asynchronous**: The leader confirms the write immediately. Lower latency but risk of data loss if the leader fails before the follower catches up (**Replication Lag**).

## Go-specific Context
In Go, you handle replication by configuring your application to send writes to the primary and reads to the replicas.

### Read-Write Splitting in GORM
GORM's `dbresolver` plugin makes this easy:
```go
import (
    "gorm.io/gorm"
    "gorm.io/plugin/dbresolver"
    "gorm.io/driver/postgres"
)

db, _ := gorm.Open(postgres.Open(primaryDSN), &gorm.Config{})

db.Use(dbresolver.Register(dbresolver.Config{
    Sources:  []gorm.Dialector{postgres.Open(primaryDSN)},
    Replicas: []gorm.Dialector{postgres.Open(replicaDSN1), postgres.Open(replicaDSN2)},
    Policy:   dbresolver.RandomPolicy{}, // Load balancing strategy
}))

// Writes automatically go to Source, Reads go to Replicas
db.Create(&user)
db.Find(&users)
```

## Interview Questions
**Q: What is 'Replication Lag'?**
**A:** It is the delay between a write happening on the leader and that write appearing on the follower. In high-traffic systems, this can lead to "read-your-own-writes" issues where a user saves data but doesn't see it immediately upon refreshing.

**Q: How do you solve the 'read-your-own-writes' problem?**
**A:** You can force critical reads (like the user's own profile after an update) to go to the primary database, while non-critical reads (like a news feed) go to replicas.

**Q: What is 'Failover'?**
**A:** Failover is the process of promoting a follower to be the new leader when the current leader fails. This is often handled by specialized software like Patroni or orchestrators like Kubernetes.
