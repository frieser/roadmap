---
---

## Summary
Object-Oriented Programming (OOP) is a programming paradigm based on the concept of "objects," which can contain data (fields) and code (methods). While traditional OOP languages like Java or C++ use classes and inheritance, **Go (Golang)** takes a distinct approach by favoring **composition over inheritance** and using interfaces for polymorphism. Go does not have classes; instead, it uses structs, methods, and interfaces to achieve OOP goals in a more flexible and decoupled manner.

## Core OOP Concepts

### 1. Class and Object
*   **Class**: A blueprint or template for creating objects. It defines the properties (data) and behaviors (methods) that its objects will have.
*   **Object**: A specific instance of a class. It contains actual data and can perform the actions defined by its class.

### 2. Encapsulation
Encapsulation is the bundling of data and the methods that operate on that data into a single unit (the object). It also involves **data hiding**, where the internal state of an object is protected from direct access by external code, usually via access modifiers (e.g., private, protected, public).

### 3. Abstraction
Abstraction is the process of hiding complex implementation details and showing only the necessary features of an object. It allows developers to work with high-level interfaces without worrying about how things work under the hood.

### 4. Inheritance
Inheritance allows a new class (subclass) to acquire the properties and methods of an existing class (superclass). This promotes code reusability.

### 5. Polymorphism
Polymorphism (meaning "many forms") allows objects of different types to be treated as objects of a common supertype. The most common form is **method overriding**, where a subclass provides a specific implementation of a method defined in its parent.

---

## OOP in Go (Golang)

Go is often described as "lightweight" OOP. It achieves the core principles without the complexity of class hierarchies.

### 1. No Classes, only Structs
In Go, we define data using `struct`. There is no `class` keyword.
```go
type User struct {
    Name  string
    Email string
}
```

### 2. Methods (Receiver Functions)
Behaviors are added to structs using **methods**. A method is just a function with a special **receiver** argument.
```go
func (u User) Greet() string {
    return "Hello, my name is " + u.Name
}
```

### 3. Encapsulation via Exporting
Go uses a simple rule for visibility:
*   **Exported (Public)**: Starts with an **Uppercase** letter (e.g., `User`, `GetName`). Visible outside the package.
*   **Unexported (Private)**: Starts with a **Lowercase** letter (e.g., `user`, `password`). Visible only within the package.

### 4. Composition instead of Inheritance (Embedding)
Go uses **Struct Embedding** to achieve code reuse. Instead of saying "A Manager *is an* Employee" (Inheritance), Go says "A Manager *contains an* Employee" (Composition).
```go
type Employee struct {
    ID   int
    Name string
}

type Manager struct {
    Employee // Anonymous field (Embedding)
    Level    int
}
```
The `Manager` struct now has direct access to the `ID` and `Name` fields and any methods of `Employee`.

### 5. Interfaces and Polymorphism
Polymorphism in Go is achieved through **Interfaces**. An interface is satisfied **implicitly**. If a type implements all the methods of an interface, it is considered to implement that interface—no `implements` keyword required.

---

## Code Example: OOP in Go

This example demonstrates encapsulation, composition (embedding), and polymorphism (interfaces).

```go
package main

import (
	"fmt"
	"math"
)

// Abstraction: Define a Shape behavior
type Shape interface {
	Area() float64
}

// Encapsulation: Circle struct (Exported) with private fields (would be lowercase in real package)
type Circle struct {
	radius float64
}

// Constructor-like function
func NewCircle(r float64) Circle {
	return Circle{radius: r}
}

// Method for Circle
func (c Circle) Area() float64 {
	return math.Pi * c.radius * c.radius
}

// Composition: Cylinder "embeds" Circle
type Cylinder struct {
	Circle // Embedding Circle
	height float64
}

func NewCylinder(r, h float64) Cylinder {
	return Cylinder{Circle: NewCircle(r), height: h}
}

// Method for Cylinder (Overriding logic, though not technically overriding in the Java sense)
func (cy Cylinder) Area() float64 {
	// 2*π*r*h + 2*π*r²
	return 2*math.Pi*cy.Circle.radius*cy.height + 2*cy.Circle.Area()
}

// Polymorphism: Accepts any Shape
func PrintArea(s Shape) {
	fmt.Printf("Area: %.2f\n", s.Area())
}

func main() {
	c := NewCircle(5)
	cy := NewCylinder(5, 10)

	PrintArea(c)  // Polymorphism
	PrintArea(cy) // Polymorphism (Cylinder implements Area())
	
	// Accessing embedded field
	fmt.Println("Cylinder base radius:", cy.Circle.radius)
}
```

---

## Interview Questions

**Q: How does Go handle inheritance if it doesn't have the `extends` keyword?**
**A:** Go uses **composition** through **struct embedding**. By embedding one struct into another, the outer struct gains the fields and methods of the inner struct. This provides code reuse without the rigid hierarchy and "fragile base class" problems of traditional inheritance.

**Q: What is the difference between a method and a function in Go?**
**A:** A function is a standalone block of code, while a **method** is a function with a **receiver**. The receiver allows the method to access and modify the data within a specific instance of a type (struct). Methods are defined as `func (receiver Type) Name()`.

**Q: How is encapsulation achieved in Go?**
**A:** Encapsulation is achieved through **visibility rules** based on capitalization. Identifiers starting with an uppercase letter are **exported** (public), while those starting with a lowercase letter are **unexported** (private to the package).

**Q: What does it mean that Go interfaces are "satisfied implicitly"?**
**A:** In languages like Java, a class must explicitly declare `implements InterfaceName`. In Go, if a type defines all the methods required by an interface, it **automatically** implements that interface. This promotes decoupling and allows you to define interfaces for types you don't even own (e.g., from the standard library).

**Q: Why does Go prefer composition over inheritance?**
**A:** Composition is generally considered more flexible and less error-prone. It avoids deep inheritance hierarchies where a change in a top-level class can have unpredictable effects on many subclasses. It also encourages "flat" designs and better decoupling through interfaces.
