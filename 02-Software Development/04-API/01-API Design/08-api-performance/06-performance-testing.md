# Performance Testing

## Summary
Performance Testing determines how a system performs in terms of responsiveness and stability under a particular workload. It is crucial for validating reliability before deployment. Types include **Load Testing** (normal conditions), **Stress Testing** (breaking point), **Soak Testing** (endurance), and **Spike Testing** (sudden bursts). Popular tools include **k6**, **JMeter**, and Go's built-in benchmarking.

## Detailed Explanation

### Types of Performance Tests
1.  **Load Testing**: Verify the system can handle the expected number of concurrent users/requests with acceptable response times.
2.  **Stress Testing**: Push the system beyond normal limits to find the breaking point (e.g., when does the DB crash?) and ensure graceful recovery.
3.  **Soak (Endurance) Testing**: Run a sustained load for a long period (e.g., 24h) to detect memory leaks or resource exhaustion.
4.  **Spike Testing**: Suddern large bursts of traffic to test autoscaling and throttling configurations.

### Go Benchmarks (`testing` package)
Go has excellent built-in support for micro-benchmarks.

```go
package main

import (
	"fmt"
	"testing"
)

// Function to test
func CalculatePrime(n int) int {
	// ... logic ...
	return n
}

// Benchmark
func BenchmarkCalculatePrime(b *testing.B) {
	for i := 0; i < b.N; i++ {
		CalculatePrime(100)
	}
}
```
Run with: `go test -bench=. -benchmem`

### Load Testing with k6
**k6** is a modern load testing tool written in Go, scriptable in JavaScript.

*Example k6 script (`script.js`)*:
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30s', target: 20 }, // Ramp up to 20 users
    { duration: '1m', target: 20 },  // Stay at 20 users
    { duration: '10s', target: 0 },  // Ramp down
  ],
};

export default function () {
  let res = http.get('http://localhost:8080/api/users');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}
```

### Simple Go Load Generator
For simple internal tests, you can write a load generator in Go using goroutines.

```go
package main

import (
	"fmt"
	"net/http"
	"sync"
	"time"
)

func main() {
	url := "http://localhost:8080"
	requests := 1000
	concurrency := 10

	var wg sync.WaitGroup
	start := time.Now()

	sem := make(chan bool, concurrency)

	for i := 0; i < requests; i++ {
		wg.Add(1)
		sem <- true // Acquire token
		go func() {
			defer wg.Done()
			defer func() { <-sem }() // Release token
			
			resp, err := http.Get(url)
			if err != nil {
				fmt.Printf("Error: %v\n", err)
				return
			}
			resp.Body.Close()
		}()
	}

	wg.Wait()
	duration := time.Since(start)
	fmt.Printf("Finished %d requests in %v (RPS: %.2f)\n", requests, duration, float64(requests)/duration.Seconds())
}
```

## Interview Questions

**Q: What is the difference between Load Testing and Stress Testing?**
**A:** Load testing verifies performance under *anticipated* peak load (e.g., "Can we handle 1000 users?"). Stress testing pushes *beyond* the limits (e.g., "At what point does the server crash? 5000 users?"). Stress testing is about finding the point of failure and ensuring data integrity during crashes.

**Q: Why is Soak Testing important?**
**A:** Some bugs only appear over time. A memory leak might lose 1MB per hour, which is invisible in a 10-minute load test but crashes the server after 3 days. Soak testing helps identify these long-term stability issues.

**Q: How do you interpret "Requests Per Second" (RPS) vs "Concurrent Users"?**
**A:** They are related but different. 1000 Concurrent Users clicking a button once every 10 seconds results in only 100 RPS. 100 Concurrent Users clicking every 1 second results in 100 RPS. RPS is the measure of server throughput; Concurrent Users is a measure of client-side load simulation.
