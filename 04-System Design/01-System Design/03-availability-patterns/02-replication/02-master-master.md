---
---

## Summary
**Multi-Leader** (Master-Master) replication allows multiple nodes to accept **Write** requests simultaneously. This increases write scalability and fault tolerance but introduces significant complexity in resolving data conflicts when two leaders modify the same record.

## Detailed Explanation

### Use Cases
*   **Multi-Datacenter**: A Leader in the US and a Leader in the EU allow local users to write with low latency. The leaders sync with each other asynchronously.
*   **Offline Clients**: Applications (like Calendar or Notes) where every device is a "Leader" that syncs when online.

### Conflict Resolution
Since writes happen concurrently on different nodes, conflicts are inevitable.
1.  **Last Write Wins (LWW)**: The write with the latest timestamp overwrites others. (Prone to clock skew issues).
2.  **Conflict-free Replicated Data Types (CRDTs)**: Mathematical data structures (like Counters or Sets) that merge automatically without conflict.
3.  **On-Read Resolution**: The database stores both conflicting versions, and the application (or user) decides which one to keep when reading.

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **Write Scalability**: Writes are distributed across nodes. | **Conflict Hell**: Merging data from two divergent sources is hard. |
| **Local Latency**: Users write to the nearest node. | **Consistency**: Achieving strong consistency is nearly impossible. |
| **Resilience**: If one Leader fails, others continue accepting writes. | **Complexity**: Setup and maintenance are significantly harder than Master-Slave. |

## Interview Questions

### Q: When would you use Master-Master over Master-Slave?
**A:** Only when you absolutely need **multi-region write availability** (e.g., a global app where US and EU users both need fast writes). For 99% of apps, Master-Slave is preferred because the complexity of conflict resolution in Master-Master often outweighs the benefits.

### Q: What is the "Split-Brain" problem in Master-Master?
**A:** If the network link between two leaders is cut, both will continue accepting writes independently. When the link returns, the two databases might have totally different states for the same records, requiring a complex merge or data loss.
