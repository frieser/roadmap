---
---

## Summary
Software Architecture is the fundamental organization of a system, defined by its components, their relationships, and the principles guiding its design and evolution. It represents the "big picture" decisions—such as architectural patterns, communication protocols, and technology stacks—that are expensive to change later. A good architecture balances functional requirements with critical quality attributes like scalability, maintainability, reliability, and security.

## Detailed Explanation

Software architecture is not just about drawing boxes and arrows; it is about making structural choices that determine how a system will behave and evolve. It serves as the blueprint for the system and the constraints within which developers work.

### Core Aspects

1.  **Structure**: How code is organized (e.g., layers, modules, microservices).
2.  **Interaction**: How components communicate (e.g., REST, gRPC, Pub/Sub).
3.  **Decisions**: Key choices about storage (SQL vs. NoSQL), deployment (Cloud vs. On-prem), and frameworks.
4.  **Trade-offs**: Every architectural decision has a cost. For example, microservices increase scalability but add operational complexity; monoliths are easier to deploy but harder to scale.

### Non-Functional Requirements (The "ilities")
Architecture is heavily driven by **Quality Attributes** (NFRs):
-   **Scalability**: Can it handle increased load? (Horizontal vs. Vertical)
-   **Reliability**: Does it function correctly under expected conditions?
-   **Availability**: Is the system up and running when needed? (99.9% uptime)
-   **Maintainability**: How easy is it to fix bugs or add new features?
-   **Security**: Is data protected in transit and at rest?

### Common Architectural Patterns
-   **Monolithic**: Single unit, shared database. Simple to start, hard to scale.
-   **Microservices**: Distributed services, independent deployment. Scalable, complex.
-   **Layered (N-Tier)**: Presentation, Business, Data layers. Separation of concerns.
-   **Event-Driven**: Components react to events. Highly decoupled and asynchronous.

### Application in Go (Golang)
Go's philosophy of simplicity and performance heavily influences its architecture.

#### 1. Clean Architecture & The Dependency Rule
In Go, we often use **Clean Architecture** or **Hexagonal Architecture** (Ports and Adapters). The key rule is that *dependencies point inward*. The core business logic (Entities/Use Cases) should not depend on external frameworks (Database, HTTP).

```go
// BAD: Domain logic depending on SQL implementation
type UserService struct {
    db *sql.DB // Direct dependency on infrastructure
}

// GOOD: Domain logic depending on an Interface (Repository Pattern)
type UserRepository interface {
    Save(user *User) error
    FindByID(id string) (*User, error)
}

type UserService struct {
    repo UserRepository // Dependency Injection via Interface
}
```

#### 2. Composition over Inheritance
Go does not have classes or inheritance. Architecture is built using **composition** (struct embedding) and **interfaces**. This leads to flatter, more flexible designs compared to deep inheritance hierarchies in Java/C#.

#### 3. Concurrency as an Architectural Primitive
Go treats concurrency as a first-class citizen. Architectures often utilize **Goroutines** and **Channels** to build high-throughput, non-blocking systems (e.g., a worker pool pattern for processing jobs).

## Interview Questions

### Q: How do you decide between a Monolithic and a Microservices architecture?
**A:** Start with a Monolith for new domains to understand boundaries and keep complexity low ("MonolithFirst"). Move to Microservices only when specific pain points arise, such as the need for independent scaling of components, separate deployment cycles for large teams, or fault isolation. Microservices introduce "distributed system tax" (network latency, consistency issues) that shouldn't be paid prematurely.

### Q: Explain the CAP theorem. How do you apply it?
**A:** The CAP theorem states a distributed data store can only guarantee two of three: **Consistency** (every read receives the most recent write), **Availability** (every request receives a response), and **Partition Tolerance** (system continues despite network drops). In distributed systems, P is mandatory. You must choose between CP (strong consistency, potential downtime) or AP (always up, eventual consistency). For example, a payment system might choose CP (accuracy over uptime), while a social media feed chooses AP.

### Q: What is the difference between Vertical Scaling and Horizontal Scaling?
**A:** **Vertical Scaling (Scaling Up)** means adding more power (CPU, RAM) to an existing server. It has a hardware limit and single point of failure. **Horizontal Scaling (Scaling Out)** means adding more machines to the pool. It allows infinite theoretical scale and high availability but requires load balancing and stateless application design.

### Q: How does Go's interface system support "Evolutionary Architecture"?
**A:** Go's interfaces are satisfied implicitly. This allows us to define small, focused interfaces where we need them (Consumer side) rather than where the type is defined. This makes it easy to swap out implementations (e.g., replacing a real StripeClient with a MockClient for testing, or migrating from MySQL to Postgres) without changing the business logic that consumes the interface.
