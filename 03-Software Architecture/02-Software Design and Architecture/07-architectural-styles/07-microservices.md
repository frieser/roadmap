---
---

# Microservices Architecture

## Summary
Microservices Architecture is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms (usually HTTP APIs or Messaging). Each service is built around a specific business capability (Bounded Context) and is independently deployable by fully automated deployment machinery.

## Detailed Explanation

### 1. Key Principles
*   **Single Responsibility**: Do one thing and do it well.
*   **Database per Service**: Each service owns its private database. Other services must use the API to access data. This prevents tight coupling at the DB layer.
*   **Independently Deployable**: A change in the "Billing" service should not require redeploying the "User" service.
*   **Smart Endpoints, Dumb Pipes**: Logic lives in the services, not the middleware (contrast with ESB in SOA).

### 2. Core Components
*   **API Gateway**: The single entry point for clients. Routes requests to appropriate backend services.
*   **Service Registry (Discovery)**: A phonebook for services (e.g., Consul, Etcd, Kubernetes DNS).
*   **Circuit Breaker**: Prevents cascading failures when a downstream service is down.

### 3. Pros & Cons

| Feature | Description |
| :--- | :--- |
| **Scalability** | Precision scaling. Scale only the service that needs it (e.g., Video Encoder). |
| **Agility** | Small teams can own services and deploy frequently without coordinating with everyone. |
| **Tech Diversity** | Can use the best tool for the job (e.g., Python for ML service, Go for high-concurrency). |
| **Complexity** | Distributed systems are hard. Network latency, partial failures, distributed tracing. |
| **Data Consistency** | No global transactions (ACID). Must use Sagas and Eventual Consistency. |
| **Ops Overhead** | Requires mature DevOps (CI/CD, Monitoring, Logging) to manage 100+ services. |

## Real-World Examples
*   **Netflix**: 1,000+ microservices.
*   **Uber**: 4,000+ microservices (migrating some back to "Macroservices").
*   **Amazon**: The pioneer of "Two-Pizza Teams".

## Go Implementation Example

In a microservices world, Go is king due to its small footprint and fast startup.

### Service A (Caller) calling Service B

```go
package main

import (
	"encoding/json"
	"net/http"
	"time"
)

// User Service (Caller)
func getUserOrders(w http.ResponseWriter, r *http.Request) {
	// Call Order Service (Service B) via HTTP
	client := http.Client{Timeout: 2 * time.Second}
	resp, err := client.Get("http://order-service/api/orders/user/123")
	
	if err != nil {
		http.Error(w, "Order Service Unavailable", 503)
		return
	}
	defer resp.Body.Close()

	// Forward response
	var orders []string
	json.NewDecoder(resp.Body).Decode(&orders)
	json.NewEncoder(w).Encode(orders)
}

func main() {
	http.HandleFunc("/user/orders", getUserOrders)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: How do you handle transactions across multiple microservices?**
**A:** Distributed transactions are handled using the **Saga Pattern**.
*   **Choreography**: Service A emits an event -> Service B listens and updates -> Service C listens...
*   **Orchestration**: A central Coordinator service tells A, B, and C what to do.
If a step fails, "Compensating Transactions" (undo actions) are triggered to roll back changes.

**Q: What is the "Strangler Fig" pattern?**
**A:** A strategy to migrate a Monolith to Microservices. You build a new microservice for a specific feature, then route traffic for that feature to the new service (via a proxy/gateway), slowly "strangling" the monolith until it disappears.

**Q: Why is "Database per Service" hard?**
**A:** It makes joins impossible. To aggregate data (e.g., "Show User details with their Last Order"), you must either:
1.  **API Composition**: Call User Service, then Call Order Service, and merge in code.
2.  **CQRS (Command Query Responsibility Segregation)**: Maintain a separate Read Model database that contains joined data, updated via events.
