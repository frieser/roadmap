#Rust
---
---

## Summary
Integers are whole numbers without a fractional component. Rust provides signed (`i`) and unsigned (`u`) integers of various fixed bit widths (8, 16, 32, 64, 128) and architecture-dependent sizes (`isize`, `usize`). The default integer type in Rust is `i32`.

## Detailed Explanation

### Integer Types

| Size | Signed | Unsigned |
|------|--------|----------|
| 8-bit | `i8` | `u8` |
| 16-bit | `i16` | `u16` |
| 32-bit | `i32` | `u32` |
| 64-bit | `i64` | `u64` |
| 128-bit | `i128` | `u128` |
| Arch | `isize` | `usize` |

- **Signed**: Can be positive or negative (Two's complement). Range: $-(2^{n-1})$ to $2^{n-1} - 1$.
- **Unsigned**: Always positive. Range: $0$ to $2^n - 1$.
- **Arch-dependent**: `usize` and `isize` depend on the machine architecture (32-bit on x86, 64-bit on x86_64). `usize` is primarily used for indexing collections.

### Literal Forms

| Form | Example |
|------|---------|
| Decimal | `98_222` |
| Hex | `0xff` |
| Octal | `0o77` |
| Binary | `0b1111_0000` |
| Byte (`u8` only) | `b'A'` |

*Note: You can use `_` as a visual separator (e.g., `1_000_000`).*

### Overflow Behavior
Rust handles integer overflow differently based on compilation mode:
- **Debug Mode**: Checks for overflow and panics at runtime.
- **Release Mode**: Does NOT check for overflow. It performs two's complement wrapping (e.g., `u8::MAX + 1` becomes `0`).

To handle overflow explicitly, use methods like:
- `wrapping_add`
- `checked_add` (returns `Option`)
- `overflowing_add`
- `saturating_add`

```rust
let a: u8 = 255;
let b = a.wrapping_add(20); // 19
let c = a.saturating_add(20); // 255
```

## Interview Questions

1. **What is the difference between `i32` and `isize`?**
   - `i32` is a fixed 32-bit signed integer. `isize` is an architecture-dependent signed integer (32 bits on 32-bit CPUs, 64 bits on 64-bit CPUs). `isize` (and `usize`) is typically used for memory addressing and collection indexing.

2. **How does Rust handle integer overflow in release mode?**
   - In release mode, Rust does not panic on integer overflow. Instead, it performs two's complement wrapping (e.g., 256 wraps to 0 for a `u8`). This behavior is considered defined but often undesirable, so explicit methods like `checked_add` should be used when overflow is a concern.

3. **Why is `i32` the default integer type?**
   - It is generally the fastest type on most modern CPUs, even 64-bit ones, and provides a large enough range for most use cases (approx +/- 2 billion).
