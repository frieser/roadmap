#Rust
---
---

## Summary
Traits are Rust's way of defining shared behavior, similar to interfaces in Java or abstract base classes in C++. They define a set of methods that a type must implement. Traits are the foundation of Rust's polymorphism and generic programming constraints.

## Detailed Explanation

### Defining a Trait
A trait defines method signatures.
```rust
pub trait Summary {
    fn summarize(&self) -> String;
    
    // Default implementation
    fn summarize_author(&self) -> String {
        format!("(Read more...)")
    }
}
```

### Implementing a Trait
You implement a trait for a specific type using `impl Trait for Type`.
```rust
struct Tweet {
    username: String,
    content: String,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

### Derivable Traits
Rust provides the `derive` attribute to automatically implement common traits if all fields of the struct support them.
- `Debug`: Format using `{:?}`.
- `Clone`, `Copy`: Duplicate values.
- `PartialEq`, `Eq`: Equality comparison.

```rust
#[derive(Debug, Clone, PartialEq)]
struct User {
    id: u32,
    username: String,
}
```

### Trait Bounds
Traits are primarily used to constrain Generics.
```rust
// Only accepts types that implement Summary
fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

## Interview Questions

1. **What is a Trait in Rust?**
   - A Trait is a collection of method signatures that define a set of behaviors. Types implement traits to guarantee they provide those behaviors. They are analogous to interfaces in other languages.

2. **What is a "Default Implementation" in a trait?**
   - A trait can provide a default body for a method. Types implementing the trait can choose to use the default or override it with their own implementation.

3. **What does `#[derive(Debug)]` do?**
   - It effectively writes the `impl Debug for MyType { ... }` code for you, allowing instances of your type to be printed using the `{:?}` formatter for debugging purposes.
