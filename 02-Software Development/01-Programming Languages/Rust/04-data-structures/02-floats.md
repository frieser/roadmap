#Rust
---
---

## Summary
Floating-point numbers represent numbers with fractional parts. Rust has two primitive types for floating-point numbers: `f32` (single precision) and `f64` (double precision). The default is `f64` because on modern CPUs it is roughly the same speed as `f32` but with much higher precision.

## Detailed Explanation

### Types
- **`f32`**: 32-bit floating point. Implements IEEE-754 single precision.
- **`f64`**: 64-bit floating point. Implements IEEE-754 double precision. Default type.

```rust
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
}
```

### Characteristics
- **IEEE-754**: Rust strictly follows this standard.
- **Special Values**: Supports `INFINITY`, `NEG_INFINITY`, `NAN` (Not a Number), and signed zeros.
- **No Ord**: Floats do not implement the `Ord` trait (only `PartialOrd`). This means **you cannot use floats as keys in a `HashMap`** or sort a `Vec<f64>` directly without a wrapper or custom comparator, because `NaN != NaN`.

### Comparison Trap
Never compare floats directly with `==` due to precision issues.
```rust
let a = 0.1 + 0.2;
let b = 0.3;
// a == b might be false! 
// a is approx 0.30000000000000004
```
Use an epsilon (small margin of error) for comparison.
```rust
let epsilon = 1e-10;
if (a - b).abs() < epsilon {
    println!("Equal!");
}
```

## Interview Questions

1. **Why is `f64` the default floating-point type in Rust?**
   - Because on modern processors, `f64` performance is very similar to `f32`, but it offers significantly greater precision, making it the safer choice for general-purpose math.

2. **Why can't you use `f64` as a key in a `HashMap`?**
   - `HashMap` keys must implement the `Eq` and `Hash` traits. Floats only implement `PartialEq` (not `Eq`) because `NaN != NaN` according to IEEE-754. Since equality is not reflexive for all values, they cannot safely be used as hash keys.

3. **How do you handle `NaN` in Rust?**
   - You can check for it using `.is_nan()`. Since `NaN` propagates through calculations and causes comparisons to fail (return false), you must handle it explicitly if your data might contain invalid numbers.
