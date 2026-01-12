---
---

## Summary
The **Integrated Architecture Framework (IAF)** is a comprehensive enterprise architecture framework developed by **Capgemini** in 1993. It provides a structured, multi-dimensional approach to aligning business strategy with IT infrastructure. By utilizing a matrix of **aspect areas** and **abstraction levels**, IAF ensures that architecture decisions are grounded in business context and traceable from high-level goals to physical implementation.

## Detailed Explanation

### Origin and Purpose
IAF was created by Capgemini architects to address the complexity of large-scale IT transformations. It was heavily influenced by the **Zachman Framework** but evolved into a more practical, "product-centric" framework. Many of its core concepts were eventually integrated into **TOGAF 9**, making it a foundational framework for modern enterprise architecture.

The primary purpose of IAF is to ensure a **holistic view** of the enterprise, bridging the gap between business requirements and technical execution.

### The IAF Structure (4x4 Matrix)
The framework is organized into a matrix that cross-references **Aspect Areas** (the "What") with **Abstraction Levels** (the "How deep").

#### 1. Abstraction Levels (The "Horizontal" View)
These levels represent the progression from business vision to technical reality:
*   **Contextual (WHY):** Defines the business drivers, scope, constraints, and vision. It answers why the architecture is needed.
*   **Conceptual (WHAT):** Defines the essential capabilities and requirements without technical bias. It focuses on "what" needs to be done.
*   **Logical (HOW):** Describes the solution structure and patterns. It is platform-independent and focuses on "how" components interact.
*   **Physical (WITH WHAT):** Specifies the actual technologies, products, and configurations. It answers "with what" the solution is built.

#### 2. Aspect Areas (The "Vertical" View)
These areas define the specific domains of the architecture:
*   **Business:** Processes, organizational structure, people, and culture.
*   **Information:** Data architecture, data management, and information flows.
*   **Information Systems:** Application landscape and software components.
*   **Technology Infrastructure:** Hardware, networks, and runtime environments.

#### Cross-cutting Aspects (Perspectives)
In addition to the core 4x4 matrix, IAF includes perspectives that span all areas:
*   **Security:** Risk management and protection across all levels.
*   **Governance:** Compliance, standards, and decision-making processes.
*   **Sustainability (V6):** Impact on environmental and social goals.

### Visualization: The IAF Matrix
```mermaid
grid-layout
    title Integrated Architecture Framework Matrix
    columns 5
    | Level / Aspect | Business | Information | Information Systems | Technology |
    | Contextual (Why) | Business Vision | Info Principles | IS Strategy | Tech Principles |
    | Conceptual (What) | Business Services | Info Entities | IS Capabilities | Tech Services |
    | Logical (How) | Process Models | Data Models | Application Logic | Network Topologies |
    | Physical (With What) | Org Units | Databases | Software Packages | Servers/Cloud |
```

### Comparison: IAF vs. TOGAF
While both are prominent enterprise architecture frameworks, they serve different primary focuses:

| Feature | IAF (Capgemini) | TOGAF (The Open Group) |
| :--- | :--- | :--- |
| **Type** | Product-centric / Matrix-driven | Process-centric / ADM-driven |
| **Focus** | "What artifacts do I need to produce?" | "How do I execute the architecture process?" |
| **Structure** | Fixed 4x4 matrix of aspects/levels | Flexible phases (A-H) in the ADM |
| **Relationship** | Heavily influenced TOGAF 9 | The global "Gold Standard" for EA process |

## Interview Questions

**Q: What are the four levels of abstraction in IAF and what do they represent?**
**A:** The four levels are **Contextual** (Why - drivers and scope), **Conceptual** (What - services and capabilities), **Logical** (How - structure and patterns), and **Physical** (With What - technology and implementation).

**Q: How does IAF handle cross-cutting concerns like Security?**
**A:** IAF treats **Security** and **Governance** as layers or "perspectives" that intersect with all Aspect Areas and Abstraction Levels, ensuring they are addressed from the business vision down to the physical hardware.

**Q: In an interview for a Software Architect role, why would you choose IAF over TOGAF for a specific project?**
**A:** I would choose IAF if the project requires a highly structured, artifact-driven approach to ensure traceability between business goals and technical decisions. IAF is particularly strong at "filling the matrix" to ensure no architectural gaps exist, whereas TOGAF is better for defining the overall organizational lifecycle of architecture.

**Q: What is the relationship between IAF and the Zachman Framework?**
**A:** IAF is considered an evolution of the Zachman Framework. While Zachman provides a comprehensive "taxonomy" of architecture, IAF is more specialized for IT transformations and provides a more practical path from conceptual design to implementation.

## Go Application Context
While IAF is an Enterprise Architecture framework, its principles can be applied to Go project structures to ensure clean separation of concerns:

*   **Logical Level (The "How"):** Defined via Go interfaces and domain services.
*   **Physical Level (The "With What"):** Concrete implementations (e.g., PostgreSQL repository, AWS S3 uploader).

```go
// Logical Level: The "How" (Business Logic/Interface)
type OrderService interface {
    Process(order Order) error
}

// Physical Level: The "With What" (Implementation Detail)
type PostgresOrderRepository struct {
    db *sql.DB
}

func (r *PostgresOrderRepository) Save(order Order) error {
    // Concrete implementation using PostgreSQL
    return nil
}
```
