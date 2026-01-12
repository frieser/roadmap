---
# Hexagonal Architecture (Ports and Adapters)
---

## Summary
**Hexagonal Architecture**, also known as **Ports and Adapters**, is a software design pattern introduced by **Alistair Cockburn** in 2005. It aims to create loosely coupled application components that can be easily connected to their software environment via ports and adapters. This makes components exchangeable at any level and facilitates testability.

The core idea is to isolate the **Business Logic (Domain)** from technical details such as databases, UI, or external APIs (**Infrastructure**).

---

## 1. Separation of Concerns: Domain vs Infrastructure

### **The Core (Domain/Application)**
*   Contains the **Business Rules** and **Use Cases**.
*   Does **not** depend on any external libraries or frameworks (where possible).
*   Defines **Ports** (interfaces) for communication with the outside world.
*   Is agnostic of the database, the UI, or any third-party services.

### **The Infrastructure (Periphery)**
*   Contains the technical implementation details.
*   Implements **Adapters** that connect to the core.
*   Includes databases, message brokers, web frameworks, and file systems.

---

## 2. Ports: The Entry and Exit Points

Ports are **interfaces** defined by the application core that allow communication with the outside world.

### **Primary (Driving) Ports**
*   **Purpose**: Define how the outside world interacts with the application.
*   **Direction**: Outside → Core.
*   **Examples**: `UserService`, `CreateOrderUseCase`, `PaymentGateway`.
*   The application core implements these ports.

### **Secondary (Driven) Ports**
*   **Purpose**: Define how the application interacts with external systems.
*   **Direction**: Core → Outside.
*   **Examples**: `UserRepository`, `EmailService`, `MessageBus`.
*   The application core *calls* these ports; infrastructure provides the implementation.

---

## 3. Adapters: The Bridge

Adapters are the implementation of the communication logic between the core and the outside world.

### **Primary (Driving) Adapters**
*   They *drive* the application.
*   **Examples**:
    *   **Web Adapter**: A REST Controller that calls a Primary Port.
    *   **CLI Adapter**: A terminal command that triggers a use case.
    *   **Event Adapter**: A message consumer that starts a process.

### **Secondary (Driven) Adapters**
*   They are *driven* by the application.
*   **Examples**:
    *   **DB Adapter**: A SQL implementation of a repository interface.
    *   **API Adapter**: An HTTP client calling a third-party service.
    *   **Storage Adapter**: A wrapper for AWS S3 or local disk.

---

## 4. Testing Benefits

One of the biggest advantages of Hexagonal Architecture is **superior testability**:

1.  **Unit Testing the Core**: Since the core depends only on interfaces (Secondary Ports), you can easily mock them. You don't need a real database or network connection to test business logic.
2.  **Swapping Adapters**: You can run the application with an "In-Memory" repository adapter during tests and a "PostgreSQL" adapter in production.
3.  **Independence**: Tests are faster and more reliable because they don't depend on volatile infrastructure.

---

## Go Code Example

```go
// --- CORE (Domain & Ports) ---

// User Domain Model
type User struct {
    ID   string
    Name string
}

// Secondary Port (Driven): Interface defined by the core
type UserRepository interface {
    Save(user *User) error
}

// Primary Port (Driving): The Service/Use Case
type UserService struct {
    repo UserRepository
}

func (s *UserService) RegisterUser(name string) error {
    user := &User{ID: "123", Name: name}
    // Business logic...
    return s.repo.Save(user)
}

// --- INFRASTRUCTURE (Adapters) ---

// Secondary Adapter (Driven): SQL Implementation
type SQLUserRepository struct {
    db *sql.DB
}

func (r *SQLUserRepository) Save(user *User) error {
    // SQL implementation details...
    return nil
}

// Primary Adapter (Driving): HTTP Controller
type UserHandler struct {
    service *UserService
}

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    // Parse request...
    h.service.RegisterUser("John Doe")
    // Send response...
}
```

---

## Interview Preparation Questions

1.  **How does Hexagonal Architecture differ from Layered Architecture?**
    *   In Layered Architecture, layers depend on those below them (UI → Business → Data). In Hexagonal Architecture, both UI and Data depend on the Core (Dependency Inversion).
2.  **What is the difference between a Driving and a Driven Adapter?**
    *   Driving adapters initiate the action (input), while Driven adapters are called by the core to perform an action (output).
3.  **Why is it called "Hexagonal"?**
    *   The shape represents the many sides (ports) an application can have, emphasizing that there isn't just a "top" (UI) and a "bottom" (DB).
