#Rust
---
---

## Summary
Rust is a statically typed language with a focus on safety. Variables are immutable by default (`let`), encouraging a functional style where state changes are explicit (`mut`). It supports strong type inference, meaning you rarely need to annotate types manually.

## Detailed Explanation

### Variables and Mutability
By default, variables are immutable. This is a safety feature to prevent accidental state modification.
- **`let`**: Binds a value to a variable (immutable).
- **`mut`**: Explicitly marks a variable as mutable.
- **Shadowing**: You can declare a new variable with the same name as a previous one, effectively "shadowing" it. This is useful for type transformations (e.g., parsing a string to a number).

```rust
fn main() {
    let x = 5; // Immutable
    // x = 6; // Error!

    let mut y = 5; // Mutable
    y = 6; // OK

    let spaces = "   ";
    let spaces = spaces.len(); // Shadowing: changes type from &str to usize
}
```

### Data Types
Rust has two subsets of data types: Scalar and Compound.

#### Scalar Types
- **Integers**: `i8`, `u8`, `i32` (default), `u64`, `isize`, `usize` (arch-dependent).
- **Floating-point**: `f32`, `f64` (default).
- **Boolean**: `bool` (`true`, `false`).
- **Character**: `char` (4 bytes, represents a Unicode Scalar Value).

#### Compound Types
- **Tuple**: Fixed-size collection of potentially different types.
  ```rust
  let tup: (i32, f64, u8) = (500, 6.4, 1);
  let (x, y, z) = tup; // Destructuring
  let five_hundred = tup.0; // Access by index
  ```
- **Array**: Fixed-size collection of the *same* type. Stack allocated.
  ```rust
  let a = [1, 2, 3, 4, 5];
  let first = a[0];
  ```

### Constants vs Static
- **`const`**: Always inlined, no fixed memory address. Must have explicit type. Used for magic numbers/strings.
- **`static`**: Has a fixed memory address. Mutable statics are `unsafe`.

```rust
const MAX_POINTS: u32 = 100_000;
static HELLO: &str = "Hello, world!";
```

## Interview Questions

1. **What is the difference between `const` and `let`?**
   - `const` requires a type annotation, can be set to a constant expression (computed at compile time), and is valid for the entire scope of the program. `let` is for runtime bindings and can use type inference.

2. **Why does Rust make variables immutable by default?**
   - To promote safety and concurrency. If data cannot change, it cannot have race conditions. It forces developers to be explicit about where state changes happen, making code easier to reason about.

3. **What is variable shadowing and when is it useful?**
   - Shadowing allows declaring a new variable with the same name as a previous one. It is useful for transforming values (e.g., `let x = x.trim().parse().unwrap()`) without having to come up with unique names like `x_str` and `x_int`.
