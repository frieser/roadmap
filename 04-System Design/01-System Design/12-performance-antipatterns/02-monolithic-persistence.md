---
---

# Monolithic Persistence

## Summary
Using a single, shared database for all components or microservices in an application. This creates a single point of failure, causes resource contention between unrelated domains, and forces a "one size fits all" data model that might be inefficient for specific needs.

## Detailed Development
In monolithic architectures, a single database is the norm. However, as systems evolve into microservices, keeping a shared database introduces severe anti-patterns:
- **Tight Coupling**: Changes to the schema for one service might break another.
- **Resource Contention**: A spike in log writes can slow down critical purchase transactions.
- **Inflexible Scaling**: You cannot scale the data layer independently for high-read vs high-write services.
- **Suboptimal Storage**: Trying to store blobs, logs, and relational data in the same PostgreSQL instance instead of using specialized stores.

### The Solution: Database-per-Service
Each microservice should own its own data store. Communication between services should happen via APIs (REST/gRPC) or events, never by direct DB access.

## Go-Specific Application

### 1. Independent Service Repositories
In Go, each service should define its own repository interface and data models. Even if using a "monorepo", the database connections should be isolated per service.

```go
// Each service has its own DB handle
type UserService struct {
    db *sql.DB // User-specific database
}

type OrderService struct {
    db *sql.DB // Order-specific database (could even be a different DB engine)
}
```

### 2. Polyglot Persistence
Go's ecosystem supports various data stores natively. Use the right tool for the job:
- **Relational (PostgreSQL/MySQL)**: For transactional business data.
- **Key-Value (Redis/Dragonfly)**: For session management and caching.
- **Document (MongoDB/Firestore)**: For flexible metadata.
- **Time-Series (InfluxDB/VictoriaMetrics)**: For metrics and logs.

### 3. Distributed Transactions (Sagas)
When data is split across databases, you can't use ACID transactions across services. In Go, you often implement the **Saga Pattern** using message brokers like RabbitMQ or Kafka (with libraries like `watermill`).

## Interview Preparation

### Questions
1. **What are the risks of a shared database in microservices?**
   - High coupling, difficulty in independent scaling, schema migration nightmares, and blast radius (if the DB goes down, everything goes down).

2. **How do you handle joins across microservices?**
   - **API Composition**: The calling service fetches data from multiple services and joins it in memory.
   - **CQRS**: Create a read-only view in a separate database that aggregates data from multiple services via events.

3. **What is "Polyglot Persistence"?**
   - The practice of using different database technologies for different data requirements within the same application.
