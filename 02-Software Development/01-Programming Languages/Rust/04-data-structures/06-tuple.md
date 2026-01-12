#Rust
---
---

## Summary
A Tuple is a general-purpose compound type that groups together a variety of values with different types into one type. Tuples have a fixed length: once declared, they cannot grow or shrink. They are commonly used to return multiple values from a function.

## Detailed Explanation

### Definition & Access
Tuples are defined with parentheses `()`.
- Access elements using dot notation with the index (`.0`, `.1`).
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = tup.0;
    let six_point_four = tup.1;
    let one = tup.2;
}
```

### Destructuring
You can break a tuple into individual variables using pattern matching.
```rust
let tup = (500, 6.4, 1);
let (x, y, z) = tup;
println!("y is: {}", y);
```

### The Unit Type `()`
The tuple with zero elements `()` is called the **Unit Type**.
- It represents an empty value or "no value returned".
- Expressions that don't return a value implicitly return `()`.
- Functions without a `->` return type implicitly return `()`.

### Usage
Commonly used to return multiple values from a function without defining a specific struct.
```rust
fn calculate_stats(numbers: &[i32]) -> (i32, i32) {
    (min, max)
}
```

## Interview Questions

1. **What is the difference between a Tuple and an Array in Rust?**
   - A Tuple can contain elements of **different types** (e.g., `(i32, f64)`). An Array must contain elements of the **same type** (e.g., `[i32; 5]`). Both have fixed sizes.

2. **What is the "Unit Type" and when is it used?**
   - The unit type is `()`. It is used to indicate the absence of a meaningful value. It is the default return type for functions that don't explicitly return anything (similar to `void` in C/Java, but it is an actual value in Rust).

3. **Can you modify an element of a tuple?**
   - Yes, if the tuple variable is declared as mutable (`mut`).
   ```rust
   let mut x = (1, 2);
   x.0 = 5;
   ```
