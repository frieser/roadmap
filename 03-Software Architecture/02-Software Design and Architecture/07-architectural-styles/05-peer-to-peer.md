---
---

# Peer-to-Peer (P2P) Architecture

## Summary
Peer-to-Peer (P2P) is a decentralized architectural style where tasks and workloads are partitioned between peers. Peers are equally privileged, equipotent participants in the application. They form a self-organizing network of nodes that function simultaneously as both "clients" and "servers" to the other nodes on the network.

## Detailed Explanation

### 1. Types of P2P Networks

*   **Pure P2P**: No central server exists. Any peer can be removed without loss of network functionality. (e.g., Gnutella, Bitcoin).
    *   *Challenge*: Discovery. How do I find the first peer?
*   **Hybrid P2P**: Uses a central server for indexing/discovery, but data transfer happens directly between peers. (e.g., Original Napster, BitTorrent trackers).
    *   *Challenge*: The index server is a single point of failure.

### 2. Key Concepts

*   **Overlay Network**: A logical network built on top of the physical network (IP). It maps logical IDs (keys) to physical nodes.
*   **DHT (Distributed Hash Table)**: A decentralized way to look up which node holds a specific piece of data. Used in Kademlia (BitTorrent, IPFS).
*   **Gossip Protocol**: Nodes periodically exchange state information with random neighbors to propagate updates across the cluster (used in Cassandra, DynamoDB).

### 3. Pros & Cons

| Feature | Description |
| :--- | :--- |
| **Resilience** | No single point of failure. High fault tolerance. |
| **Scalability** | Theoretical infinite scalability. Adding a node adds resources (CPU/Storage/Bandwidth) to the network. |
| **Cost** | Low infrastructure cost (utilizes user resources). |
| **Security** | Hard to trust peers. Sybil attacks (one user faking multiple identities) are a threat. |
| **Complexity** | High. Requires complex algorithms for consistency, routing, and discovery. |

## Real-World Examples
*   **BitTorrent**: File sharing protocol.
*   **Bitcoin / Ethereum**: Blockchain networks (distributed ledgers).
*   **IPFS (InterPlanetary File System)**: Decentralized storage web.
*   **WebRTC**: Browser-to-browser real-time communication.

## Go Implementation Example

Go is the language of choice for modern P2P (IPFS, Ethereum, Libp2p are written in Go).

### Conceptual Node (Discovery)

```go
package main

import (
	"fmt"
	"net"
)

// Peer represents a node in the network
type Peer struct {
	Address string
}

// Simple discovery: Connect to a known seed node
func connectToSeed(seedAddr string) {
	conn, err := net.Dial("tcp", seedAddr)
	if err != nil {
		fmt.Println("Failed to connect to seed:", err)
		return
	}
	defer conn.Close()

	// Handshake: Exchange peer lists
	fmt.Fprintf(conn, "GET_PEERS\n")
	// ... logic to read response and update local peer table
}

func main() {
	// In a real P2P app, we would start a listener AND try to dial others
	go func() {
		ln, _ := net.Listen("tcp", ":3000")
		for {
			conn, _ := ln.Accept()
			fmt.Println("New peer connected:", conn.RemoteAddr())
		}
	}()

	// Bootstrap
	connectToSeed("seed.p2p-network.org:3000")
}
```

## Interview Questions

**Q: What is a Sybil Attack?**
**A:** An attack where a single adversary controls multiple nodes (identities) on the network to influence the system (e.g., outvoting honest nodes in a consensus mechanism). It is typically mitigated by Proof-of-Work (Bitcoin) or Proof-of-Stake.

**Q: How does a DHT (Distributed Hash Table) work?**
**A:** A DHT provides a lookup service similar to a hash table: `(key, value)`. Each node is responsible for a range of keys. When you want to find the value for key `K`, the algorithm (like Kademlia) routes your request to the node closest to `K` (mathematically) in `O(log n)` steps.

**Q: Why is P2P preferred for large file distribution (like Linux ISOs)?**
**A:** It eliminates the bottleneck of a central server. As more users download the file, they also upload pieces of it to others. The network's bandwidth capacity grows linearly with the number of users.
