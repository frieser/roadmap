---
---

## Summary
**Abstraction** is the process of hiding complex implementation details and showing only the essential features of an object. It reduces complexity by allowing the architect to focus on *interactions* rather than *implementations*. It is the "What" without the "How".

## Detailed Explanation

### 1. Levels of Abstraction
*   **High Level**: "Send an Email".
*   **Mid Level**: "Connect to SMTP server, authenticate, transmit data".
*   **Low Level**: "Open TCP socket to port 25, write bytes".
Good architecture layers these abstractions so that high-level code never touches low-level details directly.

### 2. Leaky Abstractions
All non-trivial abstractions, to some degree, are leaky. (Joel Spolsky).
*   *Example*: ORMs abstract SQL. But if you don't understand SQL indexes, your ORM query might be slow. The abstraction "leaked" the performance reality of the underlying database.

### 3. Abstraction vs. Encapsulation
*   **Encapsulation**: Hiding *data* (information hiding) and bundling it with behavior.
*   **Abstraction**: Hiding *complexity* (implementation logic) behind a simple interface.

## Go Application (Interface Segregation)

In Go, we define abstractions using **Interfaces**. Small interfaces allow for better abstraction.

```go
// BAD: A leaky, bloated abstraction
// If I just want to Read, why do I need to know about Close?
type FileHandler interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
    Close() error
    Chmod(mode os.FileMode) error
}

// GOOD: Small, focused abstractions (from stdlib)
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Usage
// This function is very abstract. It works with Files, Network Sockets, 
// Buffers, HTTP Bodies, etc. It doesn't care about the implementation.
func CopyData(src Reader, dst Writer) error {
    // ...
}
```

## Interview Questions

**Q: How does Abstraction facilitate Refactoring?**
**A:** By coding against an abstraction (Interface), you can completely rewrite the underlying implementation (e.g., changing from a CSV file storage to a PostgreSQL database) without changing any of the code that uses that storage. The client code only knows about the `Save()` method, not how it works.

**Q: What is the "Dependency Inversion Principle" in relation to Abstraction?**
**A:** It states that high-level modules should not depend on low-level modules; both should depend on abstractions. Instead of `OrderService` importing `MySQLDriver`, both should import (or depend on) an `OrderRepository` interface.

**Q: Give an example of a "Leaky Abstraction" you've encountered.**
**A:** A common one is RPC (Remote Procedure Calls) pretending to be local function calls. The abstraction tries to hide the network. But networks have latency and failure modes that local memory calls don't. If the developer treats them identically (ignoring timeouts/retries), the abstraction fails (leaks) when the network goes down.
