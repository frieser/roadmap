#Rust
---
---

## Summary
Enums (Enumerations) in Rust are much more powerful than in C/C++. They are **Algebraic Data Types**, meaning each variant can hold different types and amounts of data. The `Option<T>` enum is Rust's solution to the "null" problem, encoding the presence or absence of a value in the type system.

## Detailed Explanation

### Defining Enums
Enums list all possible variants of a type.
```rust
enum IpAddrKind {
    V4,
    V6,
}
```

### Enums with Data
Each variant can define the type of data it holds. This avoids the need for a separate struct to hold the data and the type tag.
```rust
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },    // Named fields (struct-like)
    Write(String),              // Single String
    ChangeColor(i32, i32, i32), // Tuple-like
}
```

### The `Option` Enum
Rust has no `null`. Instead, it uses `Option<T>`, which is included in the prelude.
```rust
enum Option<T> {
    None,
    Some(T),
}
```
- To use a value that *might* be null, you must explicitly handle the `None` case (usually via `match` or `unwrap`).
- This prevents Null Pointer Exceptions entirely.

```rust
let some_number = Some(5);
let absent_number: Option<i32> = None;

// You cannot add Option<T> to T directly
// let sum = x + some_number; // Error! Must handle None case.
```

## Interview Questions

1. **How do Rust enums differ from C enums?**
   - C enums are just named integers. Rust enums are algebraic data types; each variant can hold arbitrary data (structs, tuples, strings), functioning effectively like a safe tagged union.

2. **Why doesn't Rust have `null`?**
   - `null` leads to billion-dollar mistakes (null pointer exceptions). Rust replaces it with `Option<T>`, forcing the programmer to explicitly handle the case where a value might be missing before using it.

3. **How do you access the data inside an Enum variant?**
   - You must use pattern matching (like `match` or `if let`) to extract the data. You cannot access it directly like a struct field because the compiler needs to check which variant is actually present.
