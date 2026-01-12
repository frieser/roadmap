---
---

## Summary
Load balancing is a critical technique in system design used to distribute incoming network traffic or application load across multiple servers. It ensures high availability, reliability, and scalability by preventing any single server from becoming a bottleneck. Algorithms are generally categorized into **Static** (fixed distribution logic) and **Dynamic** (load-aware distribution).

## Detailed Explanation

### Static Load Balancing Algorithms
Static algorithms distribute traffic based on a fixed rule without considering the real-time state or performance of the servers.

#### 1. Round Robin (RR)
Requests are distributed sequentially across the list of available servers in a cyclic manner.
*   **Best for**: Servers with identical hardware and tasks with similar processing times.
*   **Pros**: Extremely simple to implement; no overhead for monitoring server state.
*   **Cons**: Can lead to imbalances if some requests take significantly longer than others or if servers have different capacities.

#### 2. Weighted Round Robin (WRR)
An extension of Round Robin where each server is assigned a weight based on its capacity. Servers with higher weights receive more requests.
*   **Best for**: Heterogeneous server clusters (e.g., mixing powerful high-RAM servers with older hardware).
*   **Pros**: Better resource utilization than plain RR.
*   **Cons**: Still static; does not account for real-time load spikes or server health degradation.

#### 3. IP Hash
Uses a hash function on the client's IP address to determine which server receives the request.
*   **Best for**: Session persistence ("sticky sessions") in applications that don't share state between servers.
*   **Pros**: Ensures a specific client always hits the same backend without needing to store session state in the balancer.
*   **Cons**: Can lead to uneven distribution if many clients are behind the same NAT/Proxy (sharing one IP).

### Dynamic Load Balancing Algorithms
Dynamic algorithms monitor the health and performance of backends to make real-time routing decisions.

#### 1. Least Connections
Directs traffic to the server with the fewest active connections.
*   **Best for**: Applications with long-lived connections (e.g., WebSockets, SQL) or where processing time varies widely.
*   **Pros**: Prevents overloading any single server with too many concurrent tasks.
*   **Cons**: Requires the load balancer to track the state of every connection, adding overhead.

#### 2. Least Response Time
Routes requests to the server with the lowest number of active connections and the fastest average response time (TTFB - Time to First Byte).
*   **Best for**: Performance-critical applications where latency is the primary metric.
*   **Pros**: Automatically avoids slow or struggling servers.
*   **Cons**: Complex to implement and requires continuous active monitoring/probing.

### Consistent Hashing
Unlike simple modulo hashing ($hash(key) \pmod n$), **Consistent Hashing** maps both servers and keys to a "hash ring". When a server is added or removed, only $K/n$ keys need to be remapped (where $K$ is the number of keys and $n$ is the number of servers).
*   **Use Case**: Distributed caching (Memcached, Redis) and distributed databases (Cassandra, DynamoDB).
*   **Key Benefit**: Minimizes "churn" and cache misses during scaling events.

### Algorithm Comparison

| Algorithm | Type | Complexity | Best Use Case |
| :--- | :--- | :--- | :--- |
| **Round Robin** | Static | Low | Uniform server farms |
| **Weighted RR** | Static | Low | Mixed hardware capacity |
| **IP Hash** | Static | Medium | Session stickiness |
| **Least Connections** | Dynamic | Medium | Variable request duration |
| **Least Response Time**| Dynamic | High | Latency-sensitive apps |
| **Consistent Hashing**| Dynamic | High | Distributed Caching |

## Go Application

In Go, load balancers are often implemented using the `net/http/httputil` package for reverse proxies or within gRPC's internal balancer. For performance, it is crucial to use `sync/atomic` for counters to avoid mutex contention.

### Weighted Round Robin Implementation
Below is a simple thread-safe implementation of a Weighted Round Robin balancer.

```go
package main

import (
	"fmt"
	"sync/atomic"
)

type Server struct {
	Name   string
	Weight int
}

type WRRBalancer struct {
	servers []*Server
	index   uint64
}

func NewWRRBalancer(servers []*Server) *WRRBalancer {
	// Expand servers based on weight for simple WRR
	// In production, use "Smooth WRR" (Nginx-style) to avoid bursts
	var weightedList []*Server
	for _, s := range servers {
		for i := 0; i < s.Weight; i++ {
			weightedList = append(weightedList, s)
		}
	}
	return &WRRBalancer{servers: weightedList}
}

func (lb *WRRBalancer) Next() *Server {
	n := atomic.AddUint64(&lb.index, 1)
	return lb.servers[(n-1)%uint64(len(lb.servers))]
}

func main() {
	servers := []*Server{
		{Name: "Server-A", Weight: 3}, // Powerful
		{Name: "Server-B", Weight: 1}, // Weak
	}

	lb := NewWRRBalancer(servers)

	for i := 0; i < 8; i++ {
		fmt.Printf("Request %d -> %s\n", i+1, lb.Next().Name)
	}
}
```

## Interview Questions

**Q: What is the difference between Layer 4 (L4) and Layer 7 (L7) load balancing?**
**A:** L4 operates at the transport layer (TCP/UDP), routing based on IP and port. It's fast but "blind" to application data. L7 operates at the application layer (HTTP/HTTPS), allowing routing based on URLs, cookies, or headers. It's more flexible but requires more CPU to inspect packets.

**Q: How do you handle "Session Stickiness" if the load balancer is stateless?**
**A:** You use **IP Hashing** or **Cookie Insertion**. With Cookie Insertion, the LB adds a specific cookie to the first response; subsequent requests from the client include this cookie, allowing the LB to route them to the same backend.

**Q: Why is "Round Robin" a bad choice for a cluster with mixed hardware?**
**A:** Because it treats all servers as equal. A weak server will receive the same number of requests as a powerful one, causing the weak server to become a bottleneck while the powerful one remains underutilized. **Weighted Round Robin** is the preferred alternative.

**Q: Explain how Consistent Hashing solves the "Server Addition" problem in a cache.**
**A:** In standard hashing ($ID \pmod N$), adding a server changes $N$, causing almost all keys to map to different servers, resulting in a massive cache miss storm. Consistent Hashing only affects the keys immediately "behind" the new server on the hash ring, keeping most of the cache valid.
