#Rust
---
---

## Summary
Arrays, Vectors, and HashMaps are the workhorse collections in Rust.
- **Array**: Fixed size, stack allocated, same type.
- **Vector**: Resizable, heap allocated, same type.
- **HashMap**: Key-Value store, heap allocated, fast lookups.

## Detailed Explanation

### 1. Array (`[T; N]`)
- **Location**: Stack.
- **Size**: Fixed at compile time.
- **Use case**: When you know the exact number of elements (e.g., RGB color, fixed buffer).
```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
let buffer = [0; 1024]; // Initialize 1024 zeros
```

### 2. Vector (`Vec<T>`)
- **Location**: Heap (pointer, capacity, length stored on stack).
- **Size**: Dynamic (growable).
- **Use case**: The default list type when size isn't known.
```rust
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);

// Macro initialization
let v2 = vec![1, 2, 3];
```
- **Access**: `v[index]` (panics if out of bounds) or `v.get(index)` (returns `Option<&T>`).

### 3. HashMap (`HashMap<K, V>`)
- **Location**: Heap.
- **Characteristics**: Uses a hashing function (SipHash by default) to map keys to values.
- **Keys**: Must implement `Eq` and `Hash` traits.
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// Access
let score = scores.get("Blue"); // Returns Option<&i32>
```
- **Updating**: `entry` API is idiomatic.
```rust
scores.entry(String::from("Blue")).or_insert(50);
```

## Interview Questions

1. **What is the difference between an Array and a Vector?**
   - An Array (`[T; N]`) has a fixed size known at compile time and is stored on the stack. A Vector (`Vec<T>`) is dynamic, growable, and its data is stored on the heap. Use `Vec` when the size is unknown or changes.

2. **How does Rust handle out-of-bounds access?**
   - If you use square brackets `v[100]`, Rust **panics** at runtime to prevent memory corruption (buffer overflow). If you use `v.get(100)`, it returns `None`, allowing safe handling.

3. **What traits must a type implement to be used as a key in a HashMap?**
   - It must implement `Eq` (equality) and `Hash` (hashing strategy). Integers, Strings, and Bools implement these by default. Floating point numbers (`f32`, `f64`) do NOT implement `Eq`, so they cannot be keys.
