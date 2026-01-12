---
---

## Summary
Solution Architecture bridges the gap between business problems and technical implementation. It focuses on designing a complete solution by integrating multiple applications, services, and infrastructure components to meet specific business requirements and functional needs.

## Detailed Explanation

While Application Architecture looks *inside* a service, Solution Architecture looks *across* services. It is the "Broad" view.

### Key Responsibilities
1.  **System Integration**: Defining how different systems (e.g., Web App, Payment Gateway, Legacy ERP, CRM) talk to each other.
2.  **Technology Stack Selection**: Choosing the right set of technologies (Languages, DBs, Cloud Services) for the specific problem.
3.  **Non-Functional Requirements (NFRs)**: Ensuring the solution meets constraints like budget, timeline, security, and compliance.
4.  **Transition Planning**: Designing the roadmap to move from the current state to the future state.

### Core Patterns
-   **Service-Oriented Architecture (SOA)**: Services communicating via an Enterprise Service Bus (ESB).
-   **Microservices**: Decentralized, lightweight services communicating via APIs.
-   **Event-Driven Architecture (EDA)**: Systems reacting to state changes via message brokers.

### Application in Go (Golang)

Go is a dominant language in Solution Architecture for cloud-native and high-performance systems.

#### 1. Communication Protocols
Solution architects using Go often choose:
-   **gRPC**: For high-performance internal service-to-service communication (low latency, strongly typed contracts via Protobuf).
-   **REST/JSON**: For public-facing APIs or simple integrations.

#### 2. Event-Driven Integration
Go's concurrency model makes it excellent for consuming and processing events from brokers like **Kafka**, **RabbitMQ**, or **NATS**.
*   *Example*: A Solution Architect designs a system where a Go "Ingestion Service" accepts high-volume writes, pushes them to NATS JetStream, and multiple Go "Worker Services" process them asynchronously.

#### 3. Orchestration
Solution architects design how Go services run on infrastructure, typically **Kubernetes**. They define the sidecar patterns (e.g., Envoy) for service mesh implementation to handle retry logic, tracing, and mTLS between Go services.

```go
// Example: Solution Integration using gRPC in Go
// The solution defines a strict contract (Protobuf) for interaction between
// the Order Service and the Inventory Service.

// Proto definition (Abstract Contract)
// service Inventory {
//   rpc CheckStock (StockRequest) returns (StockResponse);
// }

// Go Implementation (Client side in Order Service)
conn, _ := grpc.Dial("inventory-service:50051", grpc.WithInsecure())
client := pb.NewInventoryClient(conn)
resp, err := client.CheckStock(ctx, &pb.StockRequest{ProductId: "123"})
```

## Interview Questions

### Q: When would you choose gRPC over REST for a solution?
**A:** I would choose gRPC for internal service-to-service communication where performance (binary protobuf serialization is smaller/faster than JSON) and developer experience (auto-generated strongly-typed clients) are priorities. I would stick to REST for public-facing APIs where broad compatibility (browser support, ease of debugging with curl) is more important.

### Q: How do you handle Distributed Transactions across multiple services?
**A:** Distributed transactions (ACID across services) are difficult. We avoid 2-Phase Commit (2PC) due to locking and performance issues. Instead, we use the **Saga Pattern**: a sequence of local transactions. If one fails, we execute "compensating transactions" to undo the previous steps.
*   *Example*: Service A charges card -> Service B reserves stock (Failed) -> Service A refunds card (Compensation).

### Q: How do you ensure data consistency in an Event-Driven solution?
**A:** We aim for **Eventual Consistency**. We use patterns like the **Transactional Outbox**: The service saves the state change and the event to be published in the same local database transaction. A separate process then reads the event from the DB and pushes it to the message broker, ensuring no events are lost even if the broker is down momentarily.
