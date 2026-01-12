#Rust
---
---

## Summary
Borrowing allows you to access data without taking ownership of it. You create **references** (`&T` or `&mut T`) which are pointers guaranteed to be valid. Slices are a special kind of reference that refer to a contiguous sequence of elements in a collection rather than the whole collection.

## Detailed Explanation

### References and Borrowing
Instead of moving ownership, we can pass a reference (borrow).
```rust
fn calculate_len(s: &String) -> usize {
    s.len()
} // s goes out of scope, but since it doesn't own the value, nothing is dropped.
```

### The Rules of Borrowing (The Borrow Checker)
At any given time, you can have **EITHER**:
1. **One mutable reference** (`&mut T`)
   **OR**
2. **Any number of immutable references** (`&T`)

This rule prevents **Data Races** at compile time. You cannot mutate data while others are reading it, and you cannot have two writers simultaneously.

### Mutable References
```rust
let mut s = String::from("hello");

let r1 = &mut s;
r1.push_str(", world");

// let r2 = &mut s; // Error! Cannot borrow `s` as mutable more than once at a time
```

### Dangling References
Rust guarantees that all references are valid. You cannot return a reference to a variable created inside a function (because that variable will be dropped).
```rust
// This will not compile
// fn dangle() -> &String {
//     let s = String::from("hello");
//     &s // s is dropped here, &s would point to freed memory
// }
```

### Slices
A slice is a dynamically sized view into a block of memory.
- **String Slice (`&str`)**: A view into a String.
- **Array Slice (`&[T]`)**: A view into an Array or Vector.

```rust
let s = String::from("hello world");
let hello: &str = &s[0..5];
let world: &str = &s[6..11];

let a = [1, 2, 3, 4, 5];
let slice: &[i32] = &a[1..3]; // contains [2, 3]
```

## Interview Questions

1. **What is the difference between `&String` and `String`?**
   - `String` is an owned type that manages heap memory. `&String` is a reference (borrow) to a `String` owned by someone else. Using `&String` does not transfer ownership.

2. **Why can you have only one mutable reference at a time?**
   - To prevent data races. If two pointers could modify the same data simultaneously without synchronization, it would lead to undefined behavior. Rust enforces this exclusivity at compile time.

3. **What is a "Slice"?**
   - A slice is a reference to a contiguous sequence of elements in a collection. It is a "fat pointer" containing a pointer to the start of the data and the length of the slice. It allows safe access to a subset of an array or string without copying.
