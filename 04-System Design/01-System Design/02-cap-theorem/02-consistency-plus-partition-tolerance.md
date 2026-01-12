---
---

## Summary
In the context of the **CAP Theorem**, a **CP (Consistency + Partition Tolerance)** system prioritizes data integrity and correctness over uptime. When a network partition occurs, the system will refuse to process requests that cannot be guaranteed to be consistent across the cluster. This often means shutting down nodes that are in the minority partition or returning errors to the client until a majority consensus (quorum) is reached.

## Detailed Explanation
CP systems choose **Consistency** and **Partition Tolerance** over **Availability**.

- **Consistency**: Every read receives the most recent write or an error. In a CP system, all nodes appear to see the same data at the same time (Linearizability).
- **Partition Tolerance**: The system continues to operate despite network failures between nodes, but it may sacrifice availability to maintain data correctness.

During a **network partition**:
1. The system detects that it cannot communicate with all replicas.
2. To prevent "split-brain" (where two different parts of the system accept different updates), the system typically uses a **quorum**-based approach (e.g., Raft or Paxos).
3. If a node cannot reach a majority of its peers, it stops accepting writes and may return errors for reads.
4. The system remains **unavailable** until the partition is resolved or a new leader is elected by the majority.

### Use Cases
*   **Banking and Finance**: Account balances must be consistent. You cannot allow two different ATMs to withdraw the same \$100 if the network between them is down.
*   **Distributed Locking**: Services like Etcd or Zookeeper are used to manage locks. If two clients think they hold the same lock due to a partition, it could lead to data corruption in the resources they protect.
*   **Inventory Management**: In high-demand scenarios (like ticket sales), you must ensure you don't sell the same seat twice.

### Error Handling in CP Systems
Since CP systems return errors during partitions, clients must be designed to handle these failures gracefully:
*   **Fail-fast**: Return an error to the user immediately, stating the service is temporarily unavailable.
*   **Retry with Exponential Backoff**: Attempt the operation again after a delay, increasing the wait time between attempts to avoid overwhelming the system as it recovers.
*   **Read-only Mode**: Some systems allow "stale reads" from minority partitions if the application can tolerate it, effectively switching to an AP-like mode for reads while keeping writes CP.

## Examples
*   **Etcd / Consul**: Distributed key-value stores used for configuration and service discovery. They use the **Raft** consensus algorithm to ensure CP.
*   **MongoDB (Default)**: While MongoDB can be tuned, its default configuration prioritizes consistency. If the primary node loses connection to the majority of secondaries, it steps down, and the cluster becomes unavailable for writes.
*   **HBase**: Built on top of HDFS, HBase is designed for consistency. If a RegionServer becomes unreachable, the data it serves is unavailable until it is recovered.
*   **Redis (Sentinel/Cluster)**: While Redis is often used for speed, in a cluster setup with strict persistence, it leans towards CP by failing writes if it can't reach a quorum.

## Go Application: Error Handling in CP
When working with a CP system like **Etcd**, the client must handle potential unavailability errors. Here is how you might handle a connection error or a context timeout in Go.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"go.etcd.io/etcd/client/v3"
)

func main() {
	// Configure Etcd client
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{"localhost:2379"},
		DialTimeout: 5 * time.Second,
	})
	if err != nil {
		log.Fatalf("Failed to connect: %v", err)
	}
	defer cli.Close()

	// Use a context with timeout to handle potential system unavailability
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	// Attempt a write operation
	_, err = cli.Put(ctx, "key", "value")
	if err != nil {
		switch err {
		case context.DeadlineExceeded:
			fmt.Println("Error: System is unavailable (Timeout). Priority: Consistency.")
		default:
			fmt.Printf("Error during Put: %v\n", err)
		}
		// In a real app, you might trigger a retry or fail-over logic here
		return
	}

	fmt.Println("Successfully wrote to CP system!")
}
```

## Related Notes
* [[01-availability-plus-partition-tolerance]]
* [[../04-availability-vs-consistency]]

## Interview Questions
*   **Q: What is the primary difference between CP and AP during a partition?**
*   **A:** In a partition, a CP system stops accepting requests that could compromise consistency (becoming unavailable), while an AP system continues to accept requests but may return inconsistent or stale data.

*   **Q: What is a "Quorum" in the context of CP systems?**
*   **A:** A quorum is the minimum number of nodes that must agree on an operation for it to be considered successful. Usually, it's defined as `(N/2) + 1`. If a partition prevents a group of nodes from reaching this number, they cannot process writes.

*   **Q: Why is Etcd considered a CP system?**
*   **A:** Etcd uses the Raft consensus algorithm, which ensures that all nodes in the cluster see the same state transition in the same order. If a leader cannot reach a majority of its followers, it will step down, and the system will stop accepting writes to ensure no divergent states are created.
