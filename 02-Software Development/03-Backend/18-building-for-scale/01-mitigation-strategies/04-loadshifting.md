---
---

## Summary
Load Shifting is a scalability strategy that involves moving workload processing to a different time (temporal shifting) or a different location (spatial shifting) to avoid peak congestion. By flattening traffic spikes, it reduces the need for expensive over-provisioning of infrastructure.

## Detailed Explanation

### Temporal Shifting (Time)
Moving non-critical tasks from peak hours to off-peak hours.
*   **Peak Shaving**: If users upload videos during the day, don't transcode them immediately. Queue them and process them at night when traffic is low.
*   **Examples**: Daily backups, generating analytics reports, batch email notifications.

### Spatial Shifting (Location)
Routing traffic to a different data center or region.
*   **Follow the Sun**: As traffic wakes up in New York, route extra load to London (where it's evening) or California (where it's early morning) if latency permits.
*   **Overflow**: If `us-east-1` is at 100% capacity, route overflow traffic to `us-east-2`.

### Key Benefits
*   **Cost Efficiency**: You don't need to provision enough servers to handle the absolute maximum peak; you only need enough for the average load.
*   **Reliability**: Prevents system overload during unexpected spikes by deferring work.

## Go-Specific Context/Examples

In Go, Load Shifting is typically implemented using **Worker Pools** and **Message Queues** (like RabbitMQ, SQS, or simple in-memory channels).

### Example: Asynchronous Job Processing (Temporal Shift)

Instead of processing a heavy task during the HTTP request, we push it to a queue to be processed by a worker later.

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

// Job represents a heavy task
type Job struct {
	ID   int
	Data string
}

// Global buffered channel acting as a queue
// If the buffer is large, it allows absorbing spikes
var jobQueue = make(chan Job, 100)

func worker(id int) {
	for job := range jobQueue {
		fmt.Printf("Worker %d: Processing job %d (Load Shifted)...\n", id, job.ID)
		time.Sleep(2 * time.Second) // Simulate heavy work
		fmt.Printf("Worker %d: Job %d done.\n", id, job.ID)
	}
}

func handler(w http.ResponseWriter, r *http.Request) {
	// Instead of doing work here, we create a job
	// This returns immediately to the user (Low Latency)
	job := Job{ID: int(time.Now().Unix()), Data: "User Upload"}
	
	select {
	case jobQueue <- job:
		w.WriteHeader(http.StatusAccepted) // 202 Accepted
		w.Write([]byte("Request accepted, processing in background"))
	default:
		// Optional: Reject if queue is totally full (Backpressure)
		http.Error(w, "System overloaded, try later", http.StatusServiceUnavailable)
	}
}

func main() {
	// Start a fixed pool of 3 workers
	// Even if 100 requests come in at once, only 3 run concurrently.
	// The rest wait in the queue (Time Shifting).
	for i := 1; i <= 3; i++ {
		go worker(i)
	}

	http.HandleFunc("/upload", handler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: What is the trade-off of Load Shifting?**
**A:** The main trade-off is **Consistency/Freshness**. By delaying processing, the user might not see the result immediately (Eventual Consistency). For example, if you upload a video, it might say "Processing..." for 10 minutes instead of being ready instantly.

**Q: How does Load Shifting differ from Auto-scaling?**
**A:** Auto-scaling reacts to load by **adding more resources** (spending money) to handle the spike now. Load shifting reacts to load by **delaying the work** (saving money) to handle it later with existing resources.

**Q: Can you use Load Shifting for real-time payments?**
**A:** Generally, no. Payments usually require strong consistency and immediate confirmation (Synchronous). However, generating the PDF receipt or sending the confirmation email can definitely be load-shifted to a background queue.
