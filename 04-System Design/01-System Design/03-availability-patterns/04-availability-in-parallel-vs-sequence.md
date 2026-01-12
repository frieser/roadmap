---
---

## Summary
The availability of a composite system depends on how its components are arranged. Components in **Sequence** (dependencies) reduce overall availability, while components in **Parallel** (redundancy) increase overall availability.

## Detailed Explanation

### Sequence (Serial) Availability
When Component A depends on Component B (A -> B), both must work for the system to work.
*   **Formula**: $Availability = A \times B$
*   **Implication**: If you chain 3 services with 99% availability ($0.99 \times 0.99 \times 0.99$), the total availability drops to **97%**.
*   **Lesson**: Microservices architectures with deep dependency chains are inherently less available than monoliths unless each service is highly redundant.

### Parallel (Redundant) Availability
When Component A and Component B do the same job (Load Balanced), the system works if *at least one* works.
*   **Formula**: $Availability = 1 - (1 - A) \times (1 - B)$
*   **Implication**: Two servers with 90% availability in parallel provide **99%** total availability ($1 - (0.1 \times 0.1) = 0.99$).
*   **Lesson**: This is why we run multiple instances of every service behind a load balancer.

### The Microservices Trade-off
Microservices often put components in **Sequence** (Service A calls B calls C). To counter the drop in availability, each individual service must be deployed in **Parallel** (Cluster of A, Cluster of B).

## Interview Questions

### Q: If I have a service with 99% uptime, how many instances do I need to reach 99.99%?
**A:**
*   1 Instance: 99% (0.99) -> Failure chance 0.01
*   2 Instances: $1 - (0.01 \times 0.01) = 99.99\%$
*   **Answer**: You need 2 independent instances running in parallel.

### Q: Why does adding a cache sometimes reduce availability?
**A:** If the cache is a hard dependency (Sequence) and the application fails when the cache is down, you have added another point of failure. If the cache is optional (fallback to DB), it acts in Parallel for performance, not availability.
