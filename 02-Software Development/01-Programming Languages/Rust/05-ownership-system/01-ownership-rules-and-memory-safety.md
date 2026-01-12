#Rust
---
---

## Summary
Ownership is Rust's most unique feature. It enables memory safety without a garbage collector. The compiler checks ownership rules at compile time to ensure no data races, null dereferences, or memory leaks occur. If the code violates these rules, it simply won't compile.

## Detailed Explanation

### The Three Rules of Ownership
1. **Each value in Rust has a variable that’s called its owner.**
2. **There can only be one owner at a time.**
3. **When the owner goes out of scope, the value will be dropped.**

### Scope and Drop
When a variable goes out of scope (usually the end of a `{}` block), Rust automatically calls the `drop` function (destructor) to clean up the heap memory.
```rust
{
    let s = String::from("hello"); // s is valid from here
    // do stuff with s
}                                  // s is now invalid; memory is freed.
```

### Move Semantics (Move vs Copy)
- **Copy**: Types that are stored entirely on the stack (like integers, booleans, chars) implement the `Copy` trait. Assigning them copies the bits.
  ```rust
  let x = 5;
  let y = x; // x is still valid
  ```
- **Move**: Types that own resources on the heap (like `String`, `Vec`) do **not** implement `Copy`. Assigning them transfers ownership. The old variable becomes invalid.
  ```rust
  let s1 = String::from("hello");
  let s2 = s1; // Ownership moved to s2
  // println!("{}", s1); // Compile Error! s1 is invalid.
  ```

### Clone
If you want to deeply copy the heap data, you must call `.clone()` explicitly.
```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // Heap data copied
// Both s1 and s2 are valid
```

## Interview Questions

1. **What is "Ownership" in Rust?**
   - It is a set of rules checked at compile-time that governs how a program manages memory. It tracks which variable "owns" a piece of memory and ensures that memory is cleaned up exactly when the owner goes out of scope.

2. **What happens when you assign a `String` variable to another variable?**
   - A "Move" occurs. The pointer, length, and capacity are copied to the new variable, but the original variable is invalidated to prevent a "double free" error when they go out of scope.

3. **What is the `Copy` trait?**
   - It is a marker trait for types that can be duplicated simply by copying bits (bitwise copy) and do not manage external resources (like heap memory). Types like integers, floats, and bools are `Copy`; `String` and `Vec` are not.
