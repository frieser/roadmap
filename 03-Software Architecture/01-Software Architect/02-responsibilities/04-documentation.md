---
---

## Summary
Documentation is the artifact that communicates the architectural vision, decisions, and constraints to the development team and future maintainers. Good architectural documentation is "living," concise, and sits close to the code to avoid becoming obsolete.

## Detailed Explanation

For a deep dive into Documentation as Code, C4 models, and specific tooling, see the [[01-documentation|Technical Skills: Documentation]] note.

The goal of documentation is to transfer knowledge and prevent "Tribal Knowledge" silos.

### Key Concepts

1.  **The C4 Model**: A standard for visualizing software architecture at different levels of zoom.
    *   **Context**: System + Users + External Systems (Big Picture).
    *   **Containers**: Applications, Data Stores, Microservices (High Level Tech).
    *   **Components**: Internal structure of a Container (Controllers, Services).
    *   **Code**: UML Class diagrams (Detailed Implementation - rarely needed).

2.  **Architecture Decision Records (ADR)**: A lightweight template to capture important architectural choices.
    *   Records the Context, Decision, Status (Proposed/Accepted), and Consequences (Trade-offs).
    *   Stored in the git repository (e.g., `/docs/adr/001-use-postgres.md`).

### Application in Go (Golang)

Go culture emphasizes documentation as part of the code.

#### 1. GoDoc
The standard tool `godoc` parses comments in the source code to generate HTML documentation.
*   *Best Practice*: Start comments with the function/type name.
    ```go
    // User represents a customer in the system.
    type User struct { ... }
    ```

#### 2. API Documentation (Swagger/OpenAPI)
For HTTP APIs, architects often enforce the use of tools like `swaggo/swag` to generate OpenAPI specs from annotations.

```go
// @Summary      Create a new user
// @Description  Register a new user with the provided details
// @Tags         users
// @Accept       json
// @Produce      json
// @Success      200  {object}  User
// @Router       /users [post]
func CreateUser(c *gin.Context) { ... }
```

#### 3. C4 as Code
Using tools like **Structurizr**, you can define your C4 model in code (or even Go DSLs), allowing you to version control your diagrams.

## Interview Questions

### Q: Why do you use ADRs (Architecture Decision Records)?
**A:** I use ADRs to capture the "Why" behind a decision, not just the "What". Software projects last for years, and team members change. An ADR prevents us from re-litigating settled decisions ("Why didn't we use MongoDB?") by showing the context and trade-offs considered at the time (e.g., "We chose Postgres because we needed strong ACID compliance for financial transactions, which MongoDB lacked at the time").

### Q: How do you prevent documentation from becoming outdated?
**A:** I follow the "Docs as Code" philosophy. Documentation lives in the same Git repository as the source code. I treat documentation bugs (incorrect info) with the same severity as code bugs. I also automate what I can (e.g., generating API docs from code annotations using Swagger) so the implementation and documentation can't drift apart.

### Q: Explain the C4 Model levels.
**A:** C4 stands for Context, Containers, Components, and Code.
-   **Context**: Non-technical view for business stakeholders.
-   **Containers**: High-level tech view (WebApp, MobileApp, Database) for Ops/Devs.
-   **Components**: Low-level view (Controller, Service) for Developers.
-   **Code**: Implementation details (Classes) - usually generated or skipped.
