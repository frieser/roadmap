---
---

# Monolithic Architecture

## Summary
Monolithic Architecture involves building an application as a single, unified unit. All functional components (UI, Business Logic, Data Access) are packaged and deployed together. While often criticized in the microservices era, the monolith remains the most effective architecture for starting new projects due to its simplicity and low operational overhead.

## Detailed Explanation

### 1. Types of Monoliths

*   **Single Process Monolith**: The entire code runs in one process. If one part crashes (e.g., a memory leak in the reporting module), the entire application goes down.
*   **Modular Monolith**: The code is structured into distinct modules (folders/packages) with clear boundaries. Calls between modules are in-process function calls. This is a highly recommended pattern (e.g., Shopify).
*   **Distributed Monolith (Anti-Pattern)**: A system that *looks* like microservices (deployed separately) but is tightly coupled (sharing databases, synchronous dependencies). It has the worst of both worlds.

### 2. Characteristics
*   **Unified Codebase**: One Git repository (usually).
*   **Unified Deployment**: The entire app is built into a single binary or WAR file.
*   **Shared Database**: Typically uses a single large database for all domains.

### 3. Pros & Cons

| Feature | Description |
| :--- | :--- |
| **Simplicity** | Easy to develop, test, and debug (IDE support is perfect). |
| **Performance** | Communication is in-memory (function calls), which is orders of magnitude faster than network calls (RPC/HTTP). |
| **Deployment** | Simple. One pipeline, one artifact. |
| **Scalability** | "Scale Cube" limit: Can only scale via X-axis (cloning the whole app). Hard to scale specific bottlenecks (e.g., video processing). |
| **Coupling** | Changes in one module can inadvertently break others. |
| **Tech Stack** | Locked into one technology (e.g., can't write one part in Rust and another in Node.js). |

## Real-World Examples
*   **Stack Overflow**: Runs largely as a monolith on .NET.
*   **Shopify**: Massive Ruby on Rails modular monolith.
*   **GitHub**: Started as a Rails monolith.

## Go Implementation Example

A Go monolith typically has all routes registered in `main.go` or a centralized router.

```go
package main

import (
	"net/http"
	"monolith/auth"
	"monolith/payment"
	"monolith/shipping"
)

func main() {
	mux := http.NewServeMux()

	// All modules are imported and used in the same binary
	mux.HandleFunc("/login", auth.LoginHandler)
	mux.HandleFunc("/pay", payment.ProcessHandler)
	mux.HandleFunc("/ship", shipping.CreateLabelHandler)

	// Single database connection shared across modules
	db := ConnectDB()
	auth.SetDB(db)
	payment.SetDB(db)

	http.ListenAndServe(":8080", mux)
}
```

## Interview Questions

**Q: When should you migrate from Monolith to Microservices?**
**A:** Only when the organizational complexity (team size) or scaling requirements (performance bottlenecks) outweigh the productivity benefits of the monolith. As Martin Fowler says: "Don't start with a microservice."

**Q: What is a "Modular Monolith"?**
**A:** It is a strategy to organize a monolith code into strictly separated modules (domains) that interact only via public APIs (interfaces), even though they run in the same process. This enforces boundaries and makes a future split into microservices easier.

**Q: How do you scale a Monolith?**
**A:** primarily via **Horizontal Scaling** (running multiple copies of the monolith behind a load balancer). You can also use **Vertical Scaling** (bigger hardware) or offload specific heavy tasks to background workers (Queues), keeping the main monolith responsive.
