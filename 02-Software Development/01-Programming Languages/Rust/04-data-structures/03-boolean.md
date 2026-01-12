#Rust
---
---

## Summary
The Boolean type in Rust is specified as `bool`. It has two possible values: `true` and `false`. It is one byte in size. Booleans are primarily used in control flow (`if`, `while`) and logical operations.

## Detailed Explanation

### Usage
Booleans are the result of comparison operators (`==`, `>`, `<`, etc.).
```rust
fn main() {
    let t = true;
    let f: bool = false; // Explicit type annotation

    if t {
        println!("It's true!");
    }
}
```

### Casting
You can cast a `bool` to an integer using `as`.
- `true` becomes `1`
- `false` becomes `0`
- **Note**: You cannot cast an integer to a `bool` using `as` (safety feature). You must use a comparison (e.g., `x != 0`).

```rust
let b = true;
let n = b as i32; // n is 1
```

### Logical Operators
- `&&` : Logical AND (Short-circuiting)
- `||` : Logical OR (Short-circuiting)
- `!` : Logical NOT

```rust
let a = true;
let b = false;
let c = a && b; // false
```

## Interview Questions

1. **What is the size of a `bool` in memory?**
   - A `bool` is 1 byte (8 bits) in size, even though it only needs 1 bit of information. This is for memory addressing efficiency.

2. **Can you evaluate an integer as a boolean (e.g., `if 1 { ... }`)?**
   - No. Unlike C or JavaScript, Rust does not perform implicit conversion ("truthy/falsy"). An `if` condition *must* be an expression that evaluates explicitly to a `bool`. You must write `if x != 0 { ... }`.

3. **Does Rust support short-circuit evaluation for booleans?**
   - Yes. In an expression like `a && b`, if `a` is false, `b` is never evaluated (and its side effects, if any, will not happen).
