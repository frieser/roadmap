---
---

# Distributed Architecture

## Summary
Distributed Architecture refers to a system where components are located on different networked computers, which communicate and coordinate their actions by passing messages. The goal is to appear to users as a single coherent system while maximizing scalability, availability, and fault tolerance.

## Detailed Explanation

### 1. The 8 Fallacies of Distributed Computing
Designers often fail by assuming these are true (when they are not):
1.  The network is reliable.
2.  Latency is zero.
3.  Bandwidth is infinite.
4.  The network is secure.
5.  Topology doesn't change.
6.  There is one administrator.
7.  Transport cost is zero.
8.  The network is homogeneous.

### 2. Theoretical Foundations

#### CAP Theorem
A distributed data store can only provide **two** of the following three guarantees:
*   **Consistency**: Every read receives the most recent write or an error.
*   **Availability**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
*   **Partition Tolerance**: The system continues to operate despite an arbitrary number of messages being dropped/delayed by the network.
*   *Reality*: In a distributed system, **P** is unavoidable. You must choose between **CP** (Consistency) or **AP** (Availability) during a partition.

#### PACELC Theorem
An extension of CAP. It states that *even when the system is running normally* (Else - no Partition), one has to choose between **Latency (L)** and **Consistency (C)**.

### 3. Consistency Models
*   **Strong Consistency**: All nodes see the same data at the same time. (Hard, slow).
*   **Eventual Consistency**: If no new updates are made, eventually all accesses will return the last updated value. (Standard for DNS, Cassandra).
*   **Causal Consistency**: Writes that are causally related must be seen by all processes in the same order.

## Real-World Examples
*   **Google Spanner**: A CP database that uses atomic clocks (TrueTime) to achieve external consistency.
*   **Amazon Dynamo**: An AP key-value store emphasizing availability.
*   **Kubernetes (etcd)**: Uses Raft consensus for strong consistency of cluster state.

## Go Implementation Example

Go is famous for distributed systems code (Docker, K8s, CockroachDB).

### Conceptual Consensus (Raft Leader Election)

```go
// Using Hashicorp's Raft library (conceptual)
package main

import (
	"log"
	"time"
	"github.com/hashicorp/raft"
)

func main() {
	// Setup Raft config
	config := raft.DefaultConfig()
	config.LocalID = raft.ServerID("node-1")

	// Create Raft node
	r, err := raft.NewRaft(config, fsm, logStore, stableStore, snapshotStore, transport)
	if err != nil {
		log.Fatal(err)
	}

	// Wait for leader election
	for {
		if r.State() == raft.Leader {
			log.Println("I am the Leader! I can accept writes.")
		} else {
			log.Println("Following...")
		}
		time.Sleep(5 * time.Second)
	}
}
```

## Interview Questions

**Q: What is the difference between Vertical Scaling and Horizontal Scaling?**
**A:**
*   **Vertical (Scale Up)**: Adding more CPU/RAM to a single machine. Easy but has a hardware limit and is a single point of failure.
*   **Horizontal (Scale Out)**: Adding more machines to the pool. Complex (requires load balancing, data partitioning) but offers theoretically infinite scale.

**Q: Explain "Sharding".**
**A:** Sharding is a method of horizontal partitioning where a database is split into smaller chunks (shards) based on a key (e.g., UserID 0-1000 go to Server A, 1001-2000 go to Server B). It reduces the load on a single server but makes cross-shard queries difficult.

**Q: What is a "Split-Brain" scenario?**
**A:** A network partition causes two parts of a cluster to lose contact. Both parts believe the other is dead and elect their own "Leader". If both accept writes, data corruption occurs. Distributed systems use Quorums (majority vote) to prevent this—the minority side shuts down.
