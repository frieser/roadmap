---
---

## Summary
**Program Against Abstractions, Not Concretions** (also known as Dependency Inversion) is the key to loosely coupled architecture. Client code should interact with interfaces (Abstractions), not specific classes (Concretions). This allows implementations to be swapped without affecting the client.

## Detailed Explanation

### 1. The Why
*   **Flexibility**: You can switch from `MySQL` to `PostgreSQL` easily if your logic depends on a `Repository` interface.
*   **Testability**: You can swap a real Service for a `MockService` during unit tests.
*   **Parallel Development**: One team works on the Logic, another on the Database, agreeing only on the Interface contract.

### 2. The "New" Operator is Glue
Every time you call `new MyClass()` (or `&MyStruct{}` in Go) inside a business logic function, you are gluing that function to that specific implementation. 
*   **Solution**: Dependency Injection. Ask for the object to be passed in, rather than creating it yourself.

## Go Application (Interfaces)

### Violation (Concrete Dependency)
```go
type PDFGenerator struct{}
func (p PDFGenerator) Generate() []byte { return []byte("pdf") }

type ReportService struct{}

func (r ReportService) CreateReport() {
    // Hard dependency on PDF. Cannot swap for CSV. Cannot mock.
    gen := PDFGenerator{} 
    data := gen.Generate()
    // ...
}
```

### Correction (Abstraction)
```go
// 1. Define the Abstraction
type Generator interface {
    Generate() []byte
}

// 2. Depend on the Abstraction
type ReportService struct {
    Gen Generator // Injected
}

func (r ReportService) CreateReport() {
    data := r.Gen.Generate() // Polymorphic call
    // ...
}

// Usage
svc := ReportService{Gen: PDFGenerator{}}
```

## Interview Questions

**Q: Is it okay to depend on concrete types from the Standard Library (like `string` or `time.Time`)?**
**A:** Yes. These are "Stable Concretions." They are unlikely to change in breaking ways. We abstract volatile things (Databases, External APIs, Complex Algorithms), not fundamental language primitives.

**Q: What is the cost of programming against abstractions?**
**A:** 
1.  **Indirection**: It's harder to navigate code ("Go to Definition" takes you to the interface, not the code).
2.  **Complexity**: More files, more types.
3.  **Performance**: Slight overhead for dynamic dispatch (negligible in most business apps).
Use it at architectural boundaries, not necessarily for every single helper function.
