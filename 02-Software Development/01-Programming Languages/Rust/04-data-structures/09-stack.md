#Rust
---
---

## Summary
Rust does not have a dedicated `Stack` type in the standard library. Instead, the standard **`Vec<T>`** (Vector) functions as a stack. It provides `push` and `pop` operations which run in `O(1)` amortized time and add/remove from the end of the vector.

## Detailed Explanation

### Implementation
A Stack is LIFO (Last In, First Out).
- **Push**: `Vec::push` adds to the end.
- **Pop**: `Vec::pop` removes from the end.
- **Peek**: `Vec::last` looks at the end without removing.

```rust
fn main() {
    let mut stack = Vec::new();

    // Push (LIFO)
    stack.push(1);
    stack.push(2);
    stack.push(3);

    // Pop
    while let Some(top) = stack.pop() {
        println!("{}", top); // Prints 3, then 2, then 1
    }
}
```

### Performance
- **Time Complexity**: `push` and `pop` are `O(1)` amortized. Occasionally `push` triggers a resize (allocation + copy), making it `O(N)`, but this is rare.
- **Space**: Contiguous memory, very cache-efficient.

## Interview Questions

1. **Does Rust have a `Stack` collection?**
   - No, it does not have a distinct type named `Stack`. The standard `Vec<T>` is used as a stack because adding/removing from the end is efficient and semantically equivalent to stack operations.

2. **What happens if you pop from an empty stack?**
   - `Vec::pop()` returns an `Option<T>`. If the stack is empty, it returns `None`. This prevents the "stack underflow" crashes common in other languages; you must handle the empty case safely.

3. **Why is `Vec` preferred over `LinkedList` for a stack?**
   - `Vec` stores data contiguously in memory, offering superior CPU cache locality and lower memory overhead (no pointers between nodes) compared to `LinkedList`.
