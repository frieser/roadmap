#Rust
---
---

## Summary
Rust has two main string types: `String` (owned, heap-allocated, growable) and `&str` (string slice, borrowed, fixed view). Both are always valid **UTF-8**. Rust prevents common string errors (like missing null terminators or invalid encoding) at the cost of simpler indexing.

## Detailed Explanation

### 1. `String` (Owned)
- **What**: A growable, mutable, owned, UTF-8 encoded string.
- **Where**: Stored on the heap.
- **Usage**: When you need to modify string data or own it.
```rust
let mut s = String::from("Hello");
s.push_str(", world!"); // Appends to string
```

### 2. `&str` (String Slice)
- **What**: A reference to a sequence of UTF-8 bytes.
- **Where**: Can point to the heap (`String`), stack, or static memory (string literals).
- **Usage**: Function arguments (efficient, no allocation).
```rust
let s: &str = "Hello World"; // String literal (static lifetime)
```

### Internal Representation
A `String` is a wrapper over a `Vec<u8>`.
- **Length**: Number of *bytes*, not characters.
- **Indexing**: You cannot write `s[0]`.

### Iterating
- **By Bytes**: `.bytes()`
- **By Scalar Values**: `.chars()`
- **By Grapheme Clusters** (Human chars): Requires external crate (`unicode-segmentation`).

```rust
for c in "नमस्ते".chars() {
    println!("{}", c); // Prints न, म, स, ्, त, े
}
```

### Other String Types
- **`OsString` / `OsStr`**: Platform-native strings (not necessarily UTF-8). Used for paths/env vars.
- **`CString` / `CStr`**: Null-terminated strings for FFI (interfacing with C).

## Interview Questions

1. **What is the difference between `String` and `&str`?**
   - `String` is an owned, heap-allocated, growable buffer. `&str` is a view (slice) into that buffer (or into a string literal). Use `&str` for function parameters to allow passing both `String` and literals without allocation.

2. **Why can't you index strings like `s[0]` in Rust?**
   - Because Rust strings are UTF-8. A character can be 1 to 4 bytes. Accessing the 0th byte might return part of a multi-byte character, which is invalid. Random access is O(N) in UTF-8, so Rust forces you to be explicit (iterate chars or bytes) to avoid hiding the cost.

3. **When would you use `OsString` instead of `String`?**
   - When interacting with the operating system (e.g., file paths, environment variables). Windows and Linux allow byte sequences in paths that are not valid UTF-8, so standard `String` cannot represent them safely.
