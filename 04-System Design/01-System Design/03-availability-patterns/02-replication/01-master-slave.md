---
---

## Summary
**Leader-Follower** (Master-Slave) is the most common replication pattern. One node (Leader) handles all **Write** operations and propagates changes to one or more nodes (Followers), which handle **Read** operations. This scales reads horizontally but leaves writes as a bottleneck.

## Detailed Explanation

### Data Flow
1.  Client sends `INSERT/UPDATE` to the **Leader**.
2.  Leader writes to its local storage.
3.  Leader sends the data change (replication log) to **Followers**.
4.  Followers update their local storage.
5.  Clients send `SELECT` (Read) requests to any **Follower**.

### Replication Types
*   **Synchronous**: Leader waits for Followers to acknowledge the write.
    *   *Pro*: Strong Consistency.
    *   *Con*: Slow writes; one slow follower halts the system.
*   **Asynchronous**: Leader returns "Success" immediately; Followers update in the background.
    *   *Pro*: Fast writes.
    *   *Con*: **Replication Lag** (Eventual Consistency). If Leader dies before syncing, data is lost.

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **Read Scalability**: Add more followers to handle more traffic. | **Write Bottleneck**: Only one node can accept writes. |
| **Simplicity**: Single source of truth for data modification. | **Replication Lag**: Followers might serve stale data. |
| **Backup**: Followers act as hot standbys for failover. | **SPOF**: If Leader dies, downtime occurs until a new one is elected. |

## Go Example (Concept)
Using separate database handles for Read and Write paths.

```go
type DB struct {
    Leader *sql.DB
    Followers []*sql.DB
}

func (db *DB) Write(query string) {
    db.Leader.Exec(query) // Always to Leader
}

func (db *DB) Read(query string) {
    // Round-robin load balancing for reads
    target := db.Followers[rand.Intn(len(db.Followers))]
    target.Query(query)
}
```

## Interview Questions

### Q: How do you handle "Replication Lag"?
**A:** For critical reads (e.g., a user viewing their own profile immediately after editing it), implement "Read-Your-Writes" consistency. This means reading that specific user's data from the Leader for a short window after a write, while sending other traffic to Followers.

### Q: What happens if the Leader fails?
**A:** A failover process begins. The system detects the failure (heartbeats), a consensus algorithm (like Raft) elects the most up-to-date Follower as the new Leader, and clients are redirected to the new Leader.
