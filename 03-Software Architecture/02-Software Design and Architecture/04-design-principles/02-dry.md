---
---

## Summary
**DRY (Don't Repeat Yourself)** states that "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system." It is often misunderstood as simply "don't type the same code twice." However, its true goal is to eliminate **duplication of business logic** to prevent inconsistency and reduce maintenance burden.

## Detailed Explanation

### 1. Logic vs. Code Duplication
*   **Logic Duplication (The Real Enemy)**: Implementing the same business rule in two places (e.g., tax calculation in both Frontend and Backend). If the rule changes, you must update both.
*   **Accidental Duplication (Acceptable)**: Two blocks of code look identical but serve different purposes. Merging them creates unnecessary coupling (Violating SRP).

### 2. The Rule of Three
"Two instances of duplication might be a coincidence; wait for the third to refactor." Prematurely applying DRY leads to complex, abstract code that is hard to read and hard to separate later (WET - Write Everything Twice).

### 3. DRY in Architecture
*   **Single Source of Truth**: Data normalization in Databases.
*   **Shared Libraries**: Common utilities (logging, auth) should be centralized.
*   **Infrastructure as Code**: Using modules in Terraform instead of copy-pasting resource definitions.

## Go Application

### Violation
Duplicating validation logic. If the rule "Age > 18" changes to "Age > 21", we might miss one spot.

```go
func RegisterUser(u User) error {
    if u.Age < 18 { // Duplication 1
        return errors.New("too young")
    }
    // save...
}

func UpdateProfile(u User) error {
    if u.Age < 18 { // Duplication 2
        return errors.New("too young")
    }
    // update...
}
```

### Correction
Centralize the "knowledge" of what constitutes a valid user.

```go
// User knows how to validate itself (Single source of truth)
func (u User) IsAdult() bool {
    return u.Age >= 18
}

func RegisterUser(u User) error {
    if !u.IsAdult() {
        return errors.New("too young")
    }
    // ...
}
```

## Interview Questions

**Q: Can DRY be harmful?**
**A:** Yes. Over-applying DRY leads to "Premature Abstraction." If you create a shared function for two pieces of code that just *happen* to look alike but evolve differently, you've introduced tight coupling. You then add boolean flags to the shared function to handle the differences, creating spaghetti code.

**Q: How does Microservices architecture challenge DRY?**
**A:** Microservices prefer "Independence" over "DRY." Sharing a common library for domain logic between services creates a "Distributed Monolith" where changing the library requires redeploying all services. In Microservices, it is often better to duplicate small logic or DTO definitions to maintain service autonomy.
