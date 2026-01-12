#Rust
---
---

## Summary
Functions in Rust are first-class citizens defined with `fn`. They follow the "snake_case" convention. Rust distinguishes between **statements** (perform action, no return) and **expressions** (evaluate to a value). The last expression in a function block is implicitly returned.

## Detailed Explanation

### Function Definition
- **Keyword**: `fn`
- **Parameters**: Must declare types explicitly.
- **Return Type**: Declared with `-> Type`. If omitted, it returns the unit type `()`.

```rust
fn add_one(x: i32) -> i32 {
    x + 1 // No semicolon = Expression (Implicit return)
}
```

### Statements vs Expressions
- **Statement**: `let y = 6;` (Does not return a value).
- **Expression**: `x + 1`, `5`, `{ let x = 3; x + 1 }`.
- **Note**: Adding a semicolon turns an expression into a statement, discarding its value and returning `()`.

### Method Syntax
Methods are functions defined within the context of a struct, enum, or trait implementation.
- Defined inside an `impl` block.
- First parameter is always `self`, `&self`, or `&mut self`.

```rust
struct Rectangle { width: u32, height: u32 }

impl Rectangle {
    // Method (takes &self)
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Associated Function (no self, like static method)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
}
```

## Interview Questions

1. **What is the difference between a statement and an expression in Rust?**
   - Expressions evaluate to a resultant value (e.g., `5 + 5`). Statements perform some action but do not return a value (e.g., `let x = 5;`).

2. **What happens if you put a semicolon at the end of the last line in a function?**
   - It turns the expression into a statement. The function will then return the unit type `()` instead of the value, which often causes a compilation error if a return type was specified.

3. **What is an associated function?**
   - It is a function defined inside an `impl` block that does not take `self` as a parameter. It is associated with the type itself rather than an instance (similar to static methods in other languages), commonly used for constructors (e.g., `String::new()`).
