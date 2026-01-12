#Rust
---
---

## Summary
For a Queue (FIFO - First In, First Out), Rust provides **`VecDeque`** (Vector Double-Ended Queue) in `std::collections`. While a standard `Vec` is great for stacks, it is terrible for queues because removing from the front (`remove(0)`) is `O(N)`. `VecDeque` is implemented as a **ring buffer**, allowing `O(1)` insertion and removal at both ends.

## Detailed Explanation

### Implementation
- **Type**: `std::collections::VecDeque`
- **Push Back**: `push_back()`
- **Pop Front**: `pop_front()`

```rust
use std::collections::VecDeque;

fn main() {
    let mut queue = VecDeque::new();

    // Enqueue
    queue.push_back("Job 1");
    queue.push_back("Job 2");
    queue.push_back("Job 3");

    // Dequeue
    while let Some(job) = queue.pop_front() {
        println!("Processing: {}", job); // 1, 2, 3
    }
}
```

### Ring Buffer Internals
`VecDeque` manages a growable ring buffer. When the buffer is full, it reallocates. It is slightly slower than `Vec` for indexing but significantly faster for front-operations.

## Interview Questions

1. **Why shouldn't you use `Vec` as a Queue?**
   - Because removing the first element of a `Vec` (`.remove(0)`) requires shifting all subsequent elements to the left, which is an `O(N)` operation. A Queue requires `O(1)` dequeue, which `VecDeque` provides.

2. **What is the underlying implementation of `VecDeque`?**
   - It is implemented as a growable ring buffer (circular buffer). This allows adding and removing items from both the front and back in amortized constant time `O(1)`.

3. **Can `VecDeque` be used as a Stack too?**
   - Yes, since it supports push/pop at both ends (`push_back`/`pop_back`), it can function as a stack. However, `Vec` is slightly more efficient for pure stack use cases due to simpler implementation logic.
