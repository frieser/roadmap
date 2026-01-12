#Rust
---
---

## Summary
Rust lacks exceptions. Instead, it uses two enums: `Option<T>` for values that may be absent, and `Result<T, E>` for operations that may fail. This forces developers to handle errors explicitly, resulting in more robust software.

## Detailed Explanation

### The `Option<T>` Enum
Used when a value might be missing (null replacement).
- Variants: `Some(T)`, `None`.
- Common methods: `unwrap()`, `expect()`, `is_some()`, `map()`.

```rust
fn find_user(id: i32) -> Option<String> {
    if id == 1 { Some("Alice".to_string()) } else { None }
}
```

### The `Result<T, E>` Enum
Used for recoverable errors.
- Variants: `Ok(T)` (Success), `Err(E)` (Failure).
- Must be used (compiler warning if ignored).

```rust
use std::fs::File;

let f = File::open("hello.txt");
let f = match f {
    Ok(file) => file,
    Err(error) => panic!("Problem opening the file: {:?}", error),
};
```

### Unwrapping
- **`unwrap()`**: Returns the value inside `Ok`/`Some` or panics (crashes) if it's `Err`/`None`. Used in prototypes.
- **`expect("msg")`**: Same as `unwrap`, but lets you provide a custom panic message. Better for debugging.

```rust
let f = File::open("hello.txt").unwrap(); // Crash if missing
```

## Interview Questions

1. **Why doesn't Rust use exceptions?**
   - Exceptions break control flow invisibly and make it hard to reason about where errors can occur. `Result` makes error paths explicit in the type signature, ensuring callers handle failure cases.

2. **When is it acceptable to use `unwrap()`?**
   - In quick prototypes, test code, or when you are mathematically certain the operation cannot fail (though `expect` is still better for documentation). In production code, explicit error handling (match/?) is preferred.

3. **What is the difference between `Option` and `Result`?**
   - `Option` represents the **possibility of absence** (something is there or it isn't). `Result` represents the **possibility of failure** (operation succeeded with value, or failed with an error explanation).
