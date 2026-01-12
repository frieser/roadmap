#Rust
---
---

## Summary
Testing in Rust is a first-class citizen, with a built-in test framework integrated directly into `cargo`. Rust distinguishes between **Unit Tests** (small, isolated tests usually placed in the same file as the code) and **Integration Tests** (tests that use the library as an external consumer, placed in the `tests/` directory). The compiler provides attributes like `#[test]`, `#[cfg(test)]`, and macros like `assert_eq!` to facilitate robust testing.

## Detailed Explanation

### 1. Unit Tests
Unit tests are designed to test private implementation details and small units of logic.
- **Location**: Defined within the same file as the code, usually in a `mod tests` module annotated with `#[cfg(test)]`.
- **Private Access**: Since they are in the same file (and a child module), they can access private functions and fields using `use super::*;`.
- **Compilation**: The `#[cfg(test)]` attribute ensures the test code is only compiled when running `cargo test`, not in the final binary.

### 2. Integration Tests
Integration tests check how different parts of your library work together and verify the public API.
- **Location**: Placed in the `tests/` directory at the project root (next to `src/`).
- **Behavior**: Each file in `tests/` is compiled as a separate crate. This means they can only call `pub` functions, simulating how a real user would use your crate.
- **Shared Code**: To share setup code between integration tests, use a `tests/common/mod.rs` file (module) rather than `tests/common.rs`, so `common` itself isn't treated as a test crate.

### 3. Documentation Tests (Doc Tests)
Rust runs code examples in your documentation comments (`///`) as tests.
- This ensures your documentation never goes out of date.
- Use ` ```rust ` blocks in comments.

### 4. Attributes and Macros
- `#[test]`: Marks a function as a test.
- `#[should_panic]`: Asserts that the test function should panic.
- `#[ignore]`: Skips the test during normal runs (run with `cargo test -- --ignored`).
- `assert!(expr)`: Panics if false.
- `assert_eq!(left, right)`: Panics if left != right.
- `assert_ne!(left, right)`: Panics if left == right.
- `Result<T, E>`: Tests can return `Result<(), E>`, allowing the use of the `?` operator.

## Rust Application

### Unit Testing Example

```rust
// src/lib.rs

pub fn add(a: i32, b: i32) -> i32 {
    internal_adder(a, b)
}

// Private function
fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    // Import everything from the parent module
    use super::*;

    #[test]
    fn test_internal_logic() {
        // We can test private functions here
        assert_eq!(internal_adder(2, 2), 4);
    }

    #[test]
    fn test_public_api() {
        assert_eq!(add(10, 5), 15);
    }

    #[test]
    #[should_panic(expected = "overflow")]
    fn test_panic() {
        // Simulating a panic scenario
        panic!("overflow");
    }
}
```

### Integration Testing Example

```rust
// tests/integration_test.rs
// Note: This file sits in tests/, not src/

// We must import the crate as an external dependency
use my_project; // assuming package name is my_project

#[test]
fn test_external_usage() {
    assert_eq!(my_project::add(3, 3), 6);
    // Cannot access my_project::internal_adder here!
}
```

## Interview Questions

### Q: What is the difference between Unit and Integration tests in Rust?
**A:** Unit tests are defined inside the source files (usually in a `tests` module) and can test private implementation details. Integration tests live in the `tests/` directory, are compiled as separate crates, and can only access the public API, ensuring the library works correctly for external users.

### Q: How do you separate test dependencies from production dependencies?
**A:** You can specify `[dev-dependencies]` in `Cargo.toml`. These crates (like `mockall` or `proptest`) are only linked for compiling tests, examples, and benchmarks, keeping the production binary lean.

### Q: What is the purpose of the `#[cfg(test)]` attribute?
**A:** It tells the Rust compiler to compile the annotated module (and its contents) only when running tests (`cargo test`). This prevents test code and helper functions from being included in the final production binary.

### Q: Can a test function return a Result?
**A:** Yes. A test signature can be `fn test_name() -> Result<(), E>`. This allows using the `?` operator inside tests. If the function returns `Err`, the test fails.
