---
---

# Migration Strategies

## Summary
Migration strategies are patterns used to move from a legacy system (often a monolith) to a new architecture (like microservices) or to roll out new versions of a service with minimal risk and downtime. Key strategies include the **Strangler Fig Pattern** for incremental replacement, **Blue/Green Deployment** for zero-downtime cutover, **Canary Releases** for gradual rollout based on blast radius, and **Shadow Traffic** for testing production-grade load without impacting users.

## Detailed Explanation

### 1. Strangler Fig Pattern
Named after the strangler fig tree that grows around another tree until it eventually replaces it. This pattern involves wrapping a new system around the edges of an old system.
*   **Mechanism**: A proxy or "strangler facade" intercepts requests. It routes specific features to the new service while keeping the rest on the legacy system.
*   **Why use it?**: Reduces risk by avoiding "big bang" migrations. It allows for incremental delivery and feedback.
*   **Go Application**: Implementing an API Gateway or Reverse Proxy in Go using `httputil.ReverseProxy` to route traffic between legacy and new services.

### 2. Blue/Green Deployment
A technique that reduces downtime and risk by running two identical production environments, only one of which serves live traffic.
*   **Mechanism**: Environment **Blue** is live; **Green** is the new version. Once Green is tested, the load balancer switches all traffic to Green.
*   **Rollback**: If an issue is found, switching back to Blue is instantaneous.
*   **Challenges**: Database schema synchronization is the primary hurdle.

### 3. Canary Releases
Gradual rollout of a new version to a small subset of users before making it available to everyone.
*   **Mechanism**: 1% of traffic goes to the "Canary" version. Metrics (error rates, latency) are monitored. If stable, the percentage increases (5%, 25%, 100%).
*   **Why use it?**: Limits the "blast radius" of potential bugs.

### 4. Shadow Traffic (Dark Launching)
Sending a copy of live production traffic to a new version of the service without returning the result to the user.
*   **Mechanism**: The user receives the response from the legacy service. The new service processes the request, and its output/performance is compared against the legacy one.
*   **Why use it?**: Validates performance and correctness under real load without any risk to the user experience.

## Go Context
Go is particularly suited for implementing these strategies due to its excellent networking primitives:
*   **Middleware**: Go's `http.Handler` interface makes it easy to write "Strangler" proxies or traffic splitters.
*   **Efficiency**: A Go-based proxy can handle thousands of concurrent "shadowed" requests with minimal overhead using Goroutines.
*   **Example: Simple Traffic Splitter (Canary/Shadow)**
    ```go
    func TrafficSplitter(canary http.Handler, stable http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            if rand.Float64() < 0.10 { // 10% Canary
                canary.ServeHTTP(w, r)
            } else {
                stable.ServeHTTP(w, r)
            }
        })
    }
    ```

## Interview Questions
**Q: How would you migrate a legacy monolith to microservices?**
**A:** I would use the **Strangler Fig Pattern**. First, identify a single, low-risk bounded context. Create a microservice for it. Use an API Gateway to route traffic for that specific feature to the new service while the rest remains in the monolith. Repeat this process incrementally until the monolith is no longer used.

**Q: What are the risks of Shadow Traffic?**
**A:** The main risk is **Side Effects**. If the shadowed request triggers a database write, an email, or a payment, it will happen twice. You must ensure the shadowed service is in a "dry run" mode or uses mock sinks for external side effects.

**Q: Explain the difference between Canary and Blue/Green.**
**A:** Blue/Green is an "all-or-nothing" switch between two environments (though easily reversible). Canary is a gradual rollout to a small percentage of users to verify stability before a full cutover.
