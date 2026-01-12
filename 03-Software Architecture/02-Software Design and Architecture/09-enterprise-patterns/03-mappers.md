---
---

# Mapper Pattern

## Summary
The **Mapper Pattern** (often specifically the Data Mapper) is a layer of mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself. In modern application development, it also refers to components responsible for transforming one object type (e.g., Domain Entity) into another (e.g., DTO).

## Detailed Explanation

### Purpose and Definition
The primary intent of a Mapper is to handle the complexity of data transformation in a single, reusable place. It ensures that neither the source nor the destination object needs to know about the other's internal structure.
*   **Decoupling**: The business logic (Domain) remains pure and doesn't "leak" details about how data is presented (DTO) or stored (Entity).
*   **Consistency**: Centralizes transformation logic, making it easier to update and test.

### Implementation in Go (Golang)

In Go, mappers are usually implemented as:
1.  **Conversion Methods**: Methods on the struct itself (e.g., `func (u User) ToDTO()`).
2.  **Mapper Functions**: Package-level functions in a `mapper` or `converter` package.
3.  **Interface-based Mappers**: For more complex scenarios where multiple mapping strategies are needed.

#### Example: Manual Mapper Function
```go
package mapper

import (
    "myproject/internal/domain"
    "myproject/internal/dto"
)

func UserToDTO(user *domain.User) *dto.UserResponse {
    if user == nil {
        return nil
    }
    return &dto.UserResponse{
        ID:       user.ID,
        FullName: user.FirstName + " " + user.LastName,
        Email:    user.Email,
    }
}
```

### Manual vs Automated Mapping

| Strategy | Performance | Type Safety | Maintenance |
| :--- | :--- | :--- | :--- |
| **Manual** | Excellent | High (Compile-time) | High (More code) |
| **Reflection** | Poor | Low (Runtime) | Low (Less code) |
| **CodeGen** | Excellent | High (Compile-time) | Medium (Build step) |

## Pros and Cons in Enterprise Applications

### Pros
*   **Separation of Concerns**: Business logic doesn't care about JSON tags or database column names.
*   **Testability**: Mapping logic can be unit-tested in isolation.
*   **Data Aggregation**: Mappers can combine data from multiple domain entities into a single DTO.

### Cons
*   **Code Duplication**: Often leads to "Pass-through" code where fields are simply copied one-to-one.
*   **Maintenance Burden**: Refactoring a domain field requires updating mappers.

## Interview Questions
*   **Q: What is the difference between a Data Mapper and a Row Data Gateway?**
*   **A:** A Data Mapper is a separate component that maps between objects and the DB, keeping the objects unaware of the DB. A Row Data Gateway is an object that acts as a gateway to a single record in the DB (usually the object itself knows how to save).
*   **Q: When is automated mapping (reflection) acceptable in Go?**
*   **A:** In internal tools or low-traffic services where developer productivity is prioritized over extreme performance.
*   **Q: How do you handle nested objects in mappers?**
*   **A:** By recursively calling mappers for the child objects, ensuring nil checks at each level to prevent panics.

