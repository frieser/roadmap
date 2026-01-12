---
---

## Summary
**Clustering** is the practice of connecting multiple servers (nodes) to work together as a single system. It is used for **High Availability (HA)**, **Load Balancing**, and **Parallel Processing**. Unlike simple scaling, clustered nodes often share state or monitor each other's health ("Heartbeat").

## Detailed Explanation
### HA Configurations
1.  **Active-Active**: All nodes serve traffic. If one fails, traffic is redistributed to the remaining nodes. (Requires complex data sync).
2.  **Active-Passive**: One node serves traffic. The other sits idle (hot standby) replicating data. If Active fails, Passive takes over (Failover).

### Concepts
*   **Heartbeat**: Periodic signal between nodes to check liveness.
*   **Quorum**: The minimum number of votes required to make a decision (prevents "Split Brain" where two nodes both think they are the leader).

### Go Context
Tools like **etcd** or **Consul** (often written in Go) are used to manage cluster state and leader election.

```go
// Conceptual Leader Election using a locking mechanism (like Redis/Etcd)
/*
func tryBecomeLeader() {
    if lock("leader_key", ttl=10s) {
        startService()
        go renewLock("leader_key")
    } else {
        standby()
    }
}
*/
```

## Interview Questions
**Q: What is "Split Brain" in a cluster?**
A: When a network failure disconnects nodes, and both partitions promote themselves to Primary/Leader, leading to data corruption (both accepting conflicting writes). It is solved by requiring a Quorum (Majority) to become leader.

**Q: Difference between Clustering and Load Balancing?**
A: Load Balancing distributes traffic. Clustering makes multiple servers act as one (often sharing storage or state). You often put a Load Balancer *in front* of a Cluster.

## Diagram
```mermaid
graph TD
    subgraph Cluster
    NodeA[Node A (Active)] <-->|Heartbeat| NodeB[Node B (Passive)]
    end
    
    Storage[(Shared Storage / Sync)]
    NodeA --> Storage
    NodeB --> Storage
```
