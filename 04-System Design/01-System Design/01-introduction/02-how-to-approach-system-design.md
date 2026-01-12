---
---

## Summary
Approaching a system design problem requires a structured framework to ensure all critical aspects—from user requirements to scalability bottlenecks—are addressed systematically. This approach is essential for both real-world architectural planning and technical interviews.

## Detailed Explanation

### The System Design Framework (Step-by-Step)

#### 1. Understand the Problem & Scope
Before diving into components, clarify the requirements.
- **Functional Requirements**: What should the system do? (e.g., "User can post a tweet", "User can follow others").
- **Non-Functional Requirements**: How should it perform? (e.g., "Highly available", "Low latency", "Scalable to 100M users").
- **Out of Scope**: What will we NOT build? (Prevents scope creep).

#### 2. Back-of-the-Envelope Estimation
Estimate the scale to inform technology choices.
- **Throughput**: Requests per second (RPS).
- **Storage**: How much data will we store over 5 years?
- **Bandwidth**: Data transfer rates.
- *Tip*: If you need 100k+ RPS, you definitely need a Load Balancer and likely Sharding.

#### 3. API Design
Define the contract between the client and the server.
- **Endpoints**: `POST /v1/tweet`, `GET /v1/feed`.
- **Parameters**: `user_id`, `content`, `timestamp`.
- **Response**: JSON structure.

#### 4. Database Schema
Decide how data will be stored and related.
- **SQL vs. NoSQL**: Choose based on data structure and consistency needs.
- **Schema**: Tables/Collections, Primary Keys, Indexes.

#### 5. High-Level Design (HLD)
Sketch the core components and data flow.
- **Client → Load Balancer → Web Servers → Database**.
- Add a **Cache** for performance and a **Message Queue** for asynchronous tasks.

#### 6. Detailed Design (LLD & Deep Dive)
Focus on the most challenging parts of the system.
- How do we generate unique IDs? (e.g., Snowflake ID).
- How do we handle "Hot Keys" in the cache?
- How do we ensure data consistency across shards?

#### 7. Identify & Resolve Bottlenecks
Review the design for weaknesses.
- **Single Point of Failure (SPOF)**: What happens if the Load Balancer dies?
- **Monitoring/Logging**: How do we know the system is healthy?
- **Security**: Authentication, Rate Limiting.

### Go Application: Scalable Worker Pattern
In Go, approaching a design often involves using the worker pool pattern to handle high-concurrency tasks efficiently.

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for j := range jobs {
		fmt.Printf("worker:%d processing job:%d\n", id, j)
		results <- j * 2
	}
}

func main() {
	const numJobs = 5
	const numWorkers = 3

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)
	var wg sync.WaitGroup

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		wg.Add(1)
		go worker(w, jobs, results, &wg)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Wait and collect results
	wg.Wait()
	close(results)

	for r := range results {
		fmt.Println("Result:", r)
	}
}
```

## Interview Questions

**Q: How do you handle a system that is experiencing high write latency?**
**A:** I would first check if the database is the bottleneck. If so, I could implement **Database Sharding** to distribute writes, or use a **Message Queue** (like Kafka or RabbitMQ) to buffer writes and process them asynchronously (Write-Back pattern).

**Q: What is the "Master-Slave" (Leader-Follower) replication?**
**A:** It is a database pattern where one node (Master) handles all writes, and one or more nodes (Slaves) replicate data from the Master to handle read requests. this improves read scalability but introduces complexity in handling Master failures.

**Q: Why is "Requirements Clarification" the most important step?**
**A:** Without it, you might design a system for 10 users that needs to support 10 million, or over-engineer a simple internal tool with a complex microservices architecture. It sets the constraints for all subsequent decisions.
