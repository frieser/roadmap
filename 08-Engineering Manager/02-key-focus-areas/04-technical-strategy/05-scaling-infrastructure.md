# Scaling Infrastructure

## Summary
Scaling Infrastructure is the practice of expanding system capacity to handle increased load (users, data, traffic) without sacrificing performance or reliability. It involves strategies like vertical scaling (bigger machines) vs. horizontal scaling (more machines), database sharding, caching, and load balancing.

## Detailed Explanation
Scalability is not just about "handling more traffic"; it's about handling it *cost-effectively* and *reliably*.

### Scaling Dimensions
1.  **Vertical Scaling (Scale Up)**: Adding CPU/RAM to a single server. Easiest, but has a hard ceiling and is expensive.
2.  **Horizontal Scaling (Scale Out)**: Adding more servers. Infinite theoretical scale, but requires stateless application design.

### Key Strategies
- **Load Balancing**: Distributing traffic across healthy instances (Nginx, AWS ALB).
- **Caching**: Storing frequently accessed data in memory (Redis, Memcached) to reduce DB load. The "freshest" data is the most expensive to fetch.
- **Database Scaling**:
    *   *Read Replicas*: Offloading reads from the primary.
    *   *Sharding*: Splitting data across multiple servers by key (e.g., UserID).
- **Asynchronous Processing**: Using message queues (Kafka, SQS) to decouple heavy tasks (e.g., sending emails, processing video) from the user request loop.

### The "Cattle vs. Pets" Concept
Treat servers like cattle (numbered, replaceable, automated) rather than pets (named, hand-nursed, unique). This is essential for horizontal scaling.

## Go Code Example
This example demonstrates a simple **Load Balancer** using the Round-Robin algorithm. It simulates distributing incoming requests across a pool of backend servers.

```go
package main

import (
	"fmt"
	"sync"
)

// Server represents a backend instance
type Server struct {
	URL     string
	Alive   bool
	RMutex  sync.RWMutex
}

// LoadBalancer holds the pool of servers
type LoadBalancer struct {
	Servers []*Server
	current int // Index for Round Robin
	mutex   sync.Mutex
}

func NewLoadBalancer(urls []string) *LoadBalancer {
	var servers []*Server
	for _, url := range urls {
		servers = append(servers, &Server{URL: url, Alive: true})
	}
	return &LoadBalancer{Servers: servers}
}

// NextServer returns the next available server using Round Robin
func (lb *LoadBalancer) NextServer() *Server {
	lb.mutex.Lock()
	defer lb.mutex.Unlock()

	// Simple loop to find next alive server
	cycleCount := 0
	for cycleCount < len(lb.Servers) {
		server := lb.Servers[lb.current]
		lb.current = (lb.current + 1) % len(lb.Servers)
		
		if server.Alive {
			return server
		}
		cycleCount++
	}
	return nil
}

func main() {
	lb := NewLoadBalancer([]string{
		"http://server-1",
		"http://server-2",
		"http://server-3",
	})

	fmt.Println("--- Simulating Traffic ---")
	
	// Simulate 5 requests
	for i := 1; i <= 5; i++ {
		target := lb.NextServer()
		if target != nil {
			fmt.Printf("Request %d -> routed to %s\n", i, target.URL)
		} else {
			fmt.Println("503 Service Unavailable")
		}
	}

	// Simulate Server 2 going down
	fmt.Println("\n[!] Server-2 crashes")
	lb.Servers[1].Alive = false

	// Simulate 4 more requests
	for i := 6; i <= 9; i++ {
		target := lb.NextServer()
		if target != nil {
			fmt.Printf("Request %d -> routed to %s\n", i, target.URL)
		}
	}
}
```

## Interview Questions
1.  **What is the difference between vertical and horizontal scaling? When do you use which?**
    *   *Focus*: Vertical = easy fix for DBs initially; Horizontal = long term solution for stateless apps.
2.  **Explain the "Thundering Herd" problem and how you prevent it.**
    *   *Focus*: Many clients retrying simultaneously after an outage; fix with exponential backoff and jitter.
3.  **How do you scale a relational database that has reached its write capacity limits?**
    *   *Focus*: Sharding (partitioning), vertical scaling first, or migrating to NoSQL if relational integrity isn't strictly required.
4.  **What role does caching play in scaling? What are the risks?**
    *   *Focus*: Cache invalidation is hard, stale data, cache stampedes.
