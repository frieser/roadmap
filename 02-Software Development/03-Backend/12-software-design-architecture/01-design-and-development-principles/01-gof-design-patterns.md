---
---

## Summary
GoF (Gang of Four) design patterns are classic solutions to common software design problems. In Go, these patterns are implemented leveraging the language's unique features like interfaces, composition over inheritance, and implicit satisfaction of interfaces, often resulting in simpler and more idiomatic code than in traditional OOP languages.

## Detailed Explanation

The 23 GoF patterns are divided into three categories: Creational, Structural, and Behavioral. While some patterns (like inheritance-based ones) are less relevant in Go, others are fundamental to writing robust Go applications.

### Creational Patterns
These patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.
*   **Singleton**: Ensures a class has only one instance and provides a global point of access to it. In Go, this is often implemented using `sync.Once`.
*   **Factory Method**: Defines an interface for creating an object, but let subclasses decide which class to instantiate. In Go, "Factory" often refers to functions that return interface types.
*   **Builder**: Separates the construction of a complex object from its representation. Useful for objects with many optional parameters.

### Structural Patterns
These patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.
*   **Adapter**: Allows incompatible interfaces to work together.
*   **Composite**: Lets you compose objects into tree structures and then work with these structures as if they were individual objects.
*   **Decorator**: Allows behavior to be added to an individual object, dynamically, without affecting the behavior of other objects from the same class.

### Behavioral Patterns
These patterns are concerned with algorithms and the assignment of responsibilities between objects.
*   **Strategy**: Defines a family of algorithms, encapsulates each one, and makes them interchangeable.
*   **Observer**: Defines a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.
*   **Command**: Turns a request into a stand-alone object that contains all information about the request.

## Go-specific Context and Examples

Go does not have classes or inheritance. Instead, it uses structs and interfaces. This leads to a more "compositional" approach to design patterns.

### Singleton Pattern in Go
The most idiomatic way to implement a Singleton in Go is using `sync.Once` to ensure thread-safe initialization.

```go
package singleton

import (
	"sync"
)

type database struct {
	connection string
}

var (
	instance *database
	once     sync.Once
)

// GetInstance returns the singleton database instance
func GetInstance() *database {
	once.Do(func() {
		instance = &database{connection: "connected"}
	})
	return instance
}
```

### Strategy Pattern in Go
Interfaces make the Strategy pattern very natural in Go.

```go
package strategy

import "fmt"

// PaymentStrategy defines the interface for different payment methods
type PaymentStrategy interface {
	Pay(amount float64)
}

// CreditCard implements PaymentStrategy
type CreditCard struct {
	Name string
}

func (c *CreditCard) Pay(amount float64) {
	fmt.Printf("Paid %.2f using Credit Card: %s\n", amount, c.Name)
}

// PayPal implements PaymentStrategy
type PayPal struct {
	Email string
}

func (p *PayPal) Pay(amount float64) {
	fmt.Printf("Paid %.2f using PayPal: %s\n", amount, p.Email)
}

// ShoppingCart uses a strategy
type ShoppingCart struct {
	PaymentMethod PaymentStrategy
}

func (s *ShoppingCart) Checkout(amount float64) {
	s.PaymentMethod.Pay(amount)
}
```

### Factory Pattern (Functional Approach)
In Go, factories are usually simple functions returning an interface.

```go
package factory

type Storage interface {
	Save(data string)
}

type diskStorage struct{}
func (d *diskStorage) Save(data string) {}

type memoryStorage struct{}
func (m *memoryStorage) Save(data string) {}

func NewStorage(storageType string) Storage {
	if storageType == "disk" {
		return &diskStorage{}
	}
	return &memoryStorage{}
}
```

## Interview Questions

**Q: Why is the Singleton pattern controversial, and how do you implement it safely in Go?**
**A:** Singleton is often considered an anti-pattern because it introduces global state, making testing difficult (coupling). In Go, it's implemented safely using `sync.Once` to guarantee that the initialization logic runs exactly once, even with multiple goroutines.

**Q: How does Go's lack of inheritance affect the implementation of GoF patterns?**
**A:** Go uses composition and interfaces instead of inheritance. Patterns that rely heavily on class hierarchies (like Template Method) are often replaced by embedding or functional approaches (passing functions as arguments).

**Q: Can you explain the Decorator pattern in the context of Go's standard library?**
**A:** A classic example is `io.Reader`. You can wrap an `os.File` (which implements `io.Reader`) with a `bufio.Reader`, which adds buffering behavior while still satisfying the `io.Reader` interface. This is a powerful way to compose functionality.
