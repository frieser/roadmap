#Rust
---
---

## Summary
A **BinaryHeap** is a priority queue implemented with a binary heap. By default, it is a **Max-Heap** in Rust (the greatest element is always popped first). It is useful for task scheduling, pathfinding algorithms (like A* or Dijkstra), or any scenario where you need quick access to the "most important" item.

## Detailed Explanation

### Key Features
- **Structure**: Max-Heap (parents >= children).
- **Peek/Pop**: Accessing max is `O(1)`, removing max is `O(log N)`.
- **Push**: `O(log N)`.

```rust
use std::collections::BinaryHeap;

fn main() {
    let mut heap = BinaryHeap::new();

    // Push (Order doesn't matter)
    heap.push(1);
    heap.push(5);
    heap.push(2);

    // Pop (Always returns Max)
    assert_eq!(heap.pop(), Some(5));
    assert_eq!(heap.pop(), Some(2));
    assert_eq!(heap.pop(), Some(1));
}
```

### Min-Heap
To create a Min-Heap (smallest first), you can wrap the contained type in `std::cmp::Reverse`.

```rust
use std::cmp::Reverse;
let mut min_heap = BinaryHeap::new();
min_heap.push(Reverse(1));
min_heap.push(Reverse(5));

let Reverse(val) = min_heap.pop().unwrap();
// val is 1
```

## Interview Questions

1. **Is `BinaryHeap` in Rust a Max-Heap or Min-Heap?**
   - It is a Max-Heap by default. The item with the largest value (according to the `Ord` trait) is returned first.

2. **How do you implement a Min-Heap in Rust?**
   - By using the `std::cmp::Reverse` wrapper on the types stored in the heap. `Reverse` flips the ordering comparison, making the "largest" item (which the heap pops) actually the smallest value.

3. **What is the complexity of pushing an item to a BinaryHeap?**
   - `O(log N)` amortized time. It needs to bubble up the new element to restore the heap property.
