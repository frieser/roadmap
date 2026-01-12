---
---

## Summary
**Active-Active** failover is a high-availability architecture where all nodes in a cluster are simultaneously active and process incoming traffic. It utilizes resources efficiently and eliminates idle hardware, but requires complex synchronization to ensure data consistency across nodes.

## Detailed Explanation

### Definition
In an Active-Active setup, load balancers distribute traffic across all available nodes. There is no concept of a "primary" or "standby" node; every node is a primary.

### Key Characteristics
1.  **Traffic Distribution**: Uses a Global Load Balancer (GSLB) or DNS Round Robin to route requests to all nodes.
2.  **Resource Utilization**: Since all nodes are active, you get 100% value from your hardware investment.
3.  **Scalability**: Adding a new node immediately increases the cluster's total capacity.
4.  **Complexity**: The biggest challenge is **state management**. If User A logs in on Node 1, Node 2 must know about that session immediately. This usually requires a distributed cache (Redis) or sticky sessions.

### Pros and Cons
| Pros | Cons |
| :--- | :--- |
| **High Throughput**: Combined capacity of all nodes. | **Complex Consistency**: Preventing conflicting writes (e.g., "split-brain") is hard. |
| **Fast Failover**: No "promotion" delay; traffic is just re-routed. | **Testing**: Debugging synchronization issues is difficult. |
| **Redundancy**: No single point of failure. | **Cost**: High operational overhead to maintain sync. |

### Example: Multi-Master Database
In a multi-master Postgres setup (e.g., using Bukardo or BDR), writes can happen on Server A or Server B. If both servers update the same record at the same time, the system must detect and resolve the conflict (e.g., Last-Write-Wins).

## Interview Questions

### Q: How do you handle user sessions in an Active-Active web cluster?
**A:** You cannot store sessions in the web server's memory. You must use an external, distributed session store (like a Redis Cluster) that all nodes can access. Alternatively, you can use "Sticky Sessions" at the load balancer, but that creates uneven load distribution.

### Q: What is the risk of "Split-Brain" in Active-Active?
**A:** If the network link between two active nodes breaks, both might think they are the only survivor and accept conflicting writes. When the network heals, merging the data is difficult or impossible without data loss.
