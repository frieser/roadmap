#Rust
---
---

## Summary
`Arc<T>` stands for **Atomic Reference Counted**. It is the thread-safe version of `Rc<T>`. It allows multiple threads to own the same data by using atomic operations to manage the reference count. It is the go-to primitive for sharing immutable data across threads.

## Detailed Explanation

### 1. Thread-Safe Shared Ownership
Like `Rc`, `Arc` allows multiple owners.
- **Atomic Counting**: Uses atomic operations (like `fetch_add`) for reference counting. This incurs a small performance penalty compared to `Rc` but guarantees safety across threads.
- **Send and Sync**: `Arc<T>` implements `Send` and `Sync` if `T` implements them. This allows it to be moved into closures spawned by `std::thread`.

### 2. Immutability
`Arc` only provides shared (immutable) access to the underlying data.
- To mutate data shared via `Arc`, you must wrap the data in a thread-safe locking primitive (Mutex or RwLock).
- **The Pattern**: `Arc<Mutex<T>>` is the standard way to share mutable state between threads.

## Rust Application

### Sharing Immutable Config
Sharing a large read-only structure between threads.

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let big_data = Arc::new(vec![1, 2, 3, 4, 5]);
    let mut handles = vec![];

    for i in 0..3 {
        let data_ref = Arc::clone(&big_data);
        handles.push(thread::spawn(move || {
            println!("Thread {} sees data: {:?}", i, data_ref);
        }));
    }

    for h in handles {
        h.join().unwrap();
    }
}
```

### Shared Mutable State (Arc + Mutex)
The most common concurrency pattern in Rust.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        }));
    }

    for h in handles {
        h.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

## Interview Questions

### Q: Why not use `Arc` for everything instead of `Rc`?
**A:** Performance. `Arc` uses atomic operations, which are more expensive for the CPU than simple integer arithmetic. If you are in a single-threaded context (like a UI tree or linked list), `Rc` is faster and semantically sufficient.

### Q: Does `Arc` implement the `Copy` trait?
**A:** No. `Copy` implies a bitwise copy of the memory. `Arc` requires executing code (incrementing the atomic counter) when duplicated, so it implements `Clone`, not `Copy`. You must explicitly call `.clone()`.

### Q: What happens if you clone an `Arc`?
**A:** You get a new `Arc` instance pointing to the *same* allocation on the heap, and the strong reference count is atomically incremented by 1. The underlying data `T` is **not** copied.
