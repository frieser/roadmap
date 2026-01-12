---
---

## Summary
System Design is the process of defining the architecture, modules, interfaces, and data for a system to satisfy specific requirements. In the context of large-scale distributed systems, it focuses on orchestrating multiple independent components to work as a single, cohesive unit that remains scalable, reliable, and available under heavy load.

## Detailed Explanation

### What is System Design?
At its core, System Design is the bridge between a conceptual idea and a functional, scalable application. While coding focuses on implementing logic, System Design focuses on how those logic blocks (services, databases, caches) are organized and how they communicate.

In **Large-Scale Distributed Systems**, the challenge shifts from single-server performance to:
- **Horizontal Scalability**: Adding more machines rather than bigger machines.
- **Fault Tolerance**: Ensuring the system stays up even if individual nodes fail.
- **Consistency vs. Availability**: Making trade-offs based on the CAP Theorem.

### HLD vs. LLD
System Design is typically split into two layers:

| Feature | High-Level Design (HLD) | Low-Level Design (LLD) |
| :--- | :--- | :--- |
| **Focus** | Overall Architecture | Component Logic |
| **Components** | Load Balancers, Databases, Services | Classes, Methods, Data Structures |
| **Goal** | System Scalability & Flow | Code Correctness & Maintainability |
| **Diagrams** | Block Diagrams, Data Flow | Class Diagrams, Sequence Diagrams |
| **Analogy** | A city's blueprint (roads, zones) | A building's floor plan (rooms, wiring) |

### Why It Matters for Senior Engineers
For a Senior Software Engineer, System Design is often the "primary" skill evaluated for promotions and high-level roles.
1. **Managing Complexity**: Seniors are expected to design systems that won't collapse under their own weight as they grow.
2. **Trade-off Analysis**: There is no "perfect" system. Seniors must justify why they chose SQL over NoSQL, or Microservices over Monolith.
3. **Leading Teams**: A solid HLD allows multiple teams to work in parallel on different services without stepping on each other's toes.

### Core Components Overview
- **Load Balancers**: Distribute incoming traffic across multiple servers to prevent any single server from becoming a bottleneck (e.g., NGINX, HAProxy, AWS ALB).
- **Caching**: Storing frequently accessed data in memory (Redis, Memcached) to reduce latency and database load.
- **Database Sharding**: Splitting a large database into smaller, faster, more easily managed parts called shards.

### Relevance to Go (Golang) Ecosystem
Go is the language of choice for modern system design and cloud-native infrastructure for several reasons:
- **Concurrency**: Goroutines and Channels allow for high-performance concurrent processing with minimal overhead, perfect for building service meshes and high-throughput APIs.
- **Microservices**: Go's small binary size and fast startup time make it ideal for containerized microservices (Docker/Kubernetes).
- **Standard Library**: The \`net/http\` package is robust enough to build production-grade web servers without heavy frameworks.
- **Cloud-Native Heritage**: Many core system design tools (Docker, Kubernetes, Prometheus, Terraform) are written in Go.

## Go Code Example: Simple Load Balancer Pattern
This example demonstrates a basic round-robin logic often used in system design concepts.

\`\`\`go
package main

import (
	"fmt"
	"sync/atomic"
)

type Server struct {
	URL string
}

type LoadBalancer struct {
	servers []Server
	current uint64
}

func (lb *LoadBalancer) NextServer() Server {
	idx := atomic.AddUint64(&lb.current, 1)
	return lb.servers[idx%uint64(len(lb.servers))]
}

func main() {
	lb := &LoadBalancer{
		servers: []Server{
			{URL: "http://app-1.internal"},
			{URL: "http://app-2.internal"},
			{URL: "http://app-3.internal"},
		},
	}

	for i := 0; i < 5; i++ {
		server := lb.NextServer()
		fmt.Printf("Routing request %d to %s\n", i+1, server.URL)
	}
}
\`\`\`

## Interview Questions

**Q: What is the difference between Horizontal and Vertical Scaling?**
**A:** Vertical scaling (Scaling Up) means adding more power (CPU, RAM) to an existing server. Horizontal scaling (Scaling Out) means adding more servers to the pool. Horizontal scaling is preferred for large-scale systems because it provides better fault tolerance and has no upper limit.

**Q: Explain the CAP Theorem.**
**A:** CAP stands for Consistency, Availability, and Partition Tolerance. It states that in a distributed system, you can only guarantee two of the three simultaneously during a network partition. In practice, since partitions are inevitable, you usually choose between CP (Consistency) or AP (Availability).

**Q: When would you use a Cache?**
**A:** Use a cache when you have data that is read frequently but changes rarely (Read-Heavy workloads), or when database queries are computationally expensive and would slow down the user experience.
