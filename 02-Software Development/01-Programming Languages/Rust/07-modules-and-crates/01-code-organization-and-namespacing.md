#Rust
---
---

## Summary
Rust's module system controls the **scope** and **privacy** of paths. It allows you to organize code into hierarchical groups (modules) and separate files. Key concepts include the `mod` keyword to declare modules, the `pub` keyword to make items public (since everything is private by default), and the `use` keyword to bring paths into scope.

## Detailed Explanation

### 1. The Module System
Modules let you manage the complexity of your program.
- **`mod` keyword**: defining a module.
- **Inline modules**: `mod network { ... }`
- **File modules**: `mod network;` tells Rust to look for `network.rs` or `network/mod.rs` (legacy style).

### 2. Privacy (Visibility)
By default, everything in Rust is **private** to its parent module.
- `pub`: Public to everyone.
- `pub(crate)`: Public within the current crate only.
- `pub(super)`: Public to the parent module only.
- `pub(in path)`: Public in a specific module path.

### 3. Paths and `use`
- **Absolute paths**: Start with the crate name or `crate::` (the root).
- **Relative paths**: Start with `self::`, `super::`, or an identifier in the current module.
- **`use`**: Creates a shortcut to a path to avoid typing full paths repeatedly.
- **Re-exporting**: `pub use internal::module` allows you to expose internal implementation details as a top-level public API.

## Rust Application

### Module Hierarchy Example

```rust
// lib.rs

// Declare a module named 'front_of_house'
// Rust looks for front_of_house.rs or front_of_house/mod.rs
pub mod front_of_house;

pub use crate::front_of_house::hosting; // Re-export

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
    
    // Using the re-export
    hosting::add_to_waitlist();
}
```

```rust
// front_of_house.rs
pub mod hosting {
    pub fn add_to_waitlist() {}
}

mod serving {
    fn take_order() {}
}
```

## Interview Questions

### Q: What is the difference between `mod.rs` and the new file system mapping?
**A:** In the 2015 edition, a submodule `foo` inside `bar` had to be at `bar/foo/mod.rs`. Since the 2018 edition, Rust supports `bar/foo.rs`. This avoids having many files named `mod.rs` open in your editor, which can be confusing.

### Q: What does `pub(crate)` do?
**A:** It makes an item visible to the entire crate (the current project), but not to external users who depend on the crate. It's useful for internal APIs that are shared across modules but shouldn't be part of the public interface.

### Q: How do you re-export a type to make it easier for users to access?
**A:** Use `pub use`. For example, if you have a deeply nested type `crate::utils::geometry::Point`, you can add `pub use utils::geometry::Point;` in `lib.rs` so users can import it as `use my_crate::Point;`.
