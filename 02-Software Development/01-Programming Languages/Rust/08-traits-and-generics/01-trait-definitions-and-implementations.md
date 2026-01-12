#Rust
---
---

## Summary
Traits in Rust define shared behavior across different types, similar to **interfaces** in other languages but more powerful. They allow you to abstract over behavior, ensuring that different types can be used interchangeably if they implement the same trait. Rust uses **Static Dispatch** (generics) by default for zero-cost abstractions, but supports **Dynamic Dispatch** (Trait Objects) when flexibility is needed at runtime.

## Detailed Explanation

### 1. Defining and Implementing Traits
A trait defines a set of method signatures that implementing types must provide.
- **Definition**: Use the `trait` keyword.
- **Implementation**: Use `impl TraitName for TypeName`.
- **Default Implementations**: Traits can provide default code for methods, which implementors can override or use as-is.

### 2. Derivable Traits
Rust provides the `#[derive(...)]` attribute to automatically generate implementations for common traits.
- **Common Derives**: `Debug`, `Clone`, `Copy`, `PartialEq`, `Eq`, `Default`.
- This reduces boilerplate for standard behavior.

### 3. Static vs. Dynamic Dispatch
- **Static Dispatch (`impl Trait`)**: The compiler generates specialized code for each concrete type (Monomorphization). Extremely fast, but increases binary size.
- **Dynamic Dispatch (`dyn Trait`)**: Uses a vtable (virtual method table) at runtime. Slower and prevents some compiler optimizations, but allows collections of heterogeneous types (e.g., `Vec<Box<dyn Animal>>`).

### 4. The Orphan Rule (Coherence)
To ensure consistency, you can only implement a trait for a type if:
1. You own the **Trait**, OR
2. You own the **Type**.
You *cannot* implement an external trait for an external type (e.g., implementing `Display` for `Vec<T>`). This prevents conflicting implementations from unrelated crates.

## Rust Application

### Defining and Implementing a Trait

```rust
// 1. Define the trait
pub trait Summary {
    fn summarize(&self) -> String;
    
    // Default implementation
    fn summarize_author(&self) -> String {
        format!("(Read more from {}...)", self.summarize())
    }
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
}

// 2. Implement the trait
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

### Trait Objects vs Generics

```rust
// Static Dispatch (Generics) - Zero runtime cost
// Compiler generates versions for Tweet AND NewsArticle
fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// Dynamic Dispatch (Trait Object) - Runtime vtable lookup
// Allows passing any type implementing Summary at runtime
fn notify_dynamic(item: &dyn Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

## Interview Questions

### Q: What is the "Orphan Rule" in Rust?
**A:** The Orphan Rule (or Coherence) states that you can only implement a trait for a type if either the trait or the type is defined in your current crate. This prevents "action at a distance" where two unrelated crates could provide conflicting implementations for the same type, breaking compilation.

### Q: What is the difference between `impl Trait` and `dyn Trait`?
**A:** `impl Trait` uses **static dispatch** (monomorphization), creating a separate copy of the function for each concrete type at compile time. It is faster but increases binary size. `dyn Trait` uses **dynamic dispatch** via a vtable (pointer to code), allowing type erasure at runtime at the cost of a slight performance hit and preventing inlining.

### Q: Can a trait contain fields?
**A:** No, Rust traits can only contain methods, associated types, and constants. They cannot contain data fields. State must be held in the concrete `struct` or `enum` that implements the trait.
