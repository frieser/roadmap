#Rust
---
---

## Summary
The `char` type in Rust represents a **Unicode Scalar Value**. Unlike C/C++ where `char` is 1 byte, in Rust `char` is **4 bytes**. This allows it to represent accented letters, Chinese/Japanese/Korean characters, emojis, and zero-width spaces natively.

## Detailed Explanation

### Definition
Specified with single quotes `'`.
```rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

### Unicode Scalar Value
A `char` represents a value in the range `U+0000` to `U+D7FF` and `U+E000` to `U+10FFFF`.
- It is *not* always equal to a "character" in the human sense (grapheme cluster).
- e.g., A specific accented character might be one `char` or composed of two `char`s (base letter + accent modifier).

### Methods
The standard library provides useful methods on `char`:
- `is_alphabetic()`
- `is_numeric()`
- `is_whitespace()`
- `to_uppercase()` / `to_lowercase()`

```rust
let c = '7';
if c.is_numeric() {
    println!("It's a number!");
}
```

## Interview Questions

1. **What is the size of a `char` in Rust?**
   - 4 bytes. This allows it to store any Unicode Scalar Value.

2. **Why is `char` not 1 byte like in C?**
   - Because Rust is designed to be Unicode-correct by default. 1 byte is only sufficient for ASCII. Rust uses `u8` for bytes and `char` for Unicode characters.

3. **Can you index a String to get a `char`?**
   - No. `my_string[0]` is invalid in Rust because Strings are UTF-8 encoded, and a single character might take 1 to 4 bytes. Indexing by byte position might land in the middle of a character, yielding an invalid sequence. You must iterate using `.chars()`.
