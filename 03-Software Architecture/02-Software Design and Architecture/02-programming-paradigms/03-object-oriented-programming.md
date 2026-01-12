---
---

## Summary
**Object-Oriented Programming (OOP)** is a programming paradigm based on the concept of "objects," which can contain data and code. It is one of the most widely used paradigms in software engineering, providing the foundation for many design patterns and architectural principles.

## Core Pillars of OOP
1.  **Encapsulation**: Bundling data and methods that operate on that data within a single unit (class/struct) and restricting direct access to some of the object's components.
2.  **Abstraction**: Hiding complex implementation details and showing only the necessary features of an object.
3.  **Inheritance**: The mechanism of basing an object or class upon another object or class, retaining similar implementation. (Note: Modern design favors **Composition** instead).
4.  **Polymorphism**: The ability of different types to be treated as a common type, usually through interfaces.

## Relevance to Architecture
For a deeper dive into how OOP applies to **Software Architecture**, specifically regarding Microservices, Design Patterns, and Domain Modeling, see:
*   [[Work/Search/Roadmap/03-Software Architecture/01-Software Architect/07-patterns-design-principles/02-oop.md|OOP for Software Architects]]
*   [[Work/Search/Roadmap/03-Software Architecture/02-Software Design and Architecture/04-design-principles/07-composition-over-inheritance.md|Composition over Inheritance]]

## Go Implementation
Go implements OOP principles differently than traditional class-based languages:
*   **Encapsulation**: Achieved via exported (Upper case) and unexported (lower case) identifiers at the package level.
*   **Abstraction & Polymorphism**: Achieved via **Interfaces**.
*   **Inheritance substitute**: Achieved via **Struct Embedding** (Composition).

```go
type Shaper interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}
```
