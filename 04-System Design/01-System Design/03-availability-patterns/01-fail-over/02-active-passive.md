---
---

## Summary
**Active-Passive** (or Active-Standby) failover is a high-availability pattern where only one node (Active) handles traffic, while one or more nodes (Passive) remain on standby, ready to take over if the active node fails. It is simpler to implement but wastes resources on idle hardware.

## Detailed Explanation

### Definition
A Primary node processes all requests. A Secondary node monitors the Primary via "heartbeats." If the Primary stops responding, the Secondary promotes itself to Active and takes over the traffic (Failover).

### Failover Mechanisms
1.  **Heartbeats**: Periodic signals sent between nodes. Missing 3 consecutive heartbeats triggers a failover.
2.  **Virtual IP (VIP)**: A floating IP address that points to the Active node. During failover, the VIP is re-assigned to the Standby node (using protocols like VRRP/Keepalived).

### Standby Types
*   **Hot Standby**: Fully synchronized and running. Failover is instant.
*   **Warm Standby**: Running but may need to load data or warm up caches. Failover takes seconds/minutes.
*   **Cold Standby**: Turned off until needed. Failover takes minutes/hours (boot time).

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **Simplicity**: Easy to configure; no write conflicts. | **Wasted Resources**: You pay for hardware that does nothing 99% of the time. |
| **Data Consistency**: Only one writer at a time, avoiding sync conflicts. | **Failover Downtime**: There is always a small gap (seconds) while the passive node takes over. |

### Example: Postgres with Patroni
A typical HA Postgres setup uses **Patroni** to manage the cluster. Node A is Leader (Active). Node B is a Replica (Passive) streaming changes from A. If A dies, Patroni updates the configuration (via Etcd/Consul) to promote B to Leader and points the application to B.

## Interview Questions

### Q: Why would you choose Active-Passive over Active-Active?
**A:** When data consistency is more important than raw throughput, or when the complexity of resolving write conflicts is too high. Active-Passive is the standard for relational databases (SQL) because it guarantees a single source of truth for writes.

### Q: What is a "Floating IP"?
**A:** It is a virtual IP address that is not bound to a specific physical interface permanently. It can be dynamically moved from one server to another by software (like Keepalived) to mask the failure of the underlying hardware from clients.
