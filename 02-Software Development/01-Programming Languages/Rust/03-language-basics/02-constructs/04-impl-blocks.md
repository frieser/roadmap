#Rust
---
---

## Summary
`impl` (Implementation) blocks are where you define functions and methods associated with a specific type (struct or enum). This separates data definition (`struct`) from behavior definition (`impl`). You can have multiple `impl` blocks for the same type.

## Detailed Explanation

### Defining Methods
Methods are functions that operate on an instance of the type. The first parameter is always `self`.
- `&self`: Immutable borrow (most common). Reads data.
- `&mut self`: Mutable borrow. Modifies data.
- `self`: Takes ownership. Consumes the instance (rare, often for transformation).

```rust
struct Rectangle { width: u32, height: u32 }

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

### Associated Functions
Functions that do not take `self` are called associated functions. They act like static methods and are called using the `::` syntax.
- Typically used for constructors.

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

// Usage
let sq = Rectangle::square(3);
```

### Multiple Impl Blocks
Rust allows separating methods into multiple `impl` blocks. This is useful for organization or when conditional compilation (features) is involved.
```rust
impl Rectangle {
    fn area(&self) -> u32 { ... }
}

impl Rectangle {
    fn perimeter(&self) -> u32 { ... }
}
```

## Interview Questions

1. **Why does Rust separate `struct` definitions from `impl` blocks?**
   - It separates data from behavior. This aligns with the "Data-Oriented Design" philosophy and allows implementing traits for types defined in other crates (the Orphan Rule context), keeping definitions clean.

2. **What is the difference between `fn foo(&self)` and `fn foo(self)`?**
   - `&self` borrows the instance immutably; the caller retains ownership. `self` takes ownership of the instance; the instance is moved into the method and dropped (or returned) at the end, meaning the caller cannot use it afterwards.

3. **Can you add methods to types you didn't define (like `String` or `i32`)?**
   - Not directly via an inherent `impl` block. However, you can define a **Trait** with the method and implement that Trait for the external type (extension trait pattern).
