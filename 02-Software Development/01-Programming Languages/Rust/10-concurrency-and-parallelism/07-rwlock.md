#Rust
---
---

## Summary
`RwLock<T>` (Read-Write Lock) is a synchronization primitive designed for scenarios where data is read often but written rarely. It allows **multiple concurrent readers** OR **one exclusive writer** at a time. This provides better performance than `Mutex` in read-heavy workloads.

## Detailed Explanation

### 1. Reader-Writer Policy
- **Read Lock (`.read()`)**: Multiple threads can hold a read lock simultaneously. They get shared (immutable) access to the data. Blocks if a writer holds the lock.
- **Write Lock (`.write()`)**: Only one thread can hold a write lock. It gets exclusive (mutable) access. Blocks if *any* readers or writers hold the lock.

### 2. Guards
- `RwLockReadGuard`: Returned by `.read()`. Allows reading `T`.
- `RwLockWriteGuard`: Returned by `.write()`. Allows mutating `T`.
- Like Mutex, dropping the guard releases the lock.

### 3. Priority
Rust's standard `RwLock` implementation priorities (OS-dependent) usually favor writers to prevent writer starvation, but this behavior is not strictly guaranteed by the API.

## Rust Application

### Read-Heavy Cache Example
A common use case: A shared configuration or cache where many threads read values, but updates happen infrequently.

```rust
use std::sync::{Arc, RwLock};
use std::thread;
use std::time::Duration;

fn main() {
    let data = Arc::new(RwLock::new(vec![1, 2, 3]));
    let mut handles = vec![];

    // Spawn Reader Threads (can run concurrently)
    for i in 0..3 {
        let data = Arc::clone(&data);
        handles.push(thread::spawn(move || {
            let r = data.read().unwrap(); // Shared access
            println!("Reader {}: {:?}", i, *r);
            thread::sleep(Duration::from_millis(100)); // Simulating work
        }));
    }

    // Spawn Writer Thread (needs exclusive access)
    {
        let data = Arc::clone(&data);
        handles.push(thread::spawn(move || {
            thread::sleep(Duration::from_millis(50));
            let mut w = data.write().unwrap(); // Blocks until all readers are done
            w.push(4);
            println!("Writer updated data");
        }));
    }

    for h in handles {
        h.join().unwrap();
    }
}
```

## Interview Questions

### Q: When should you choose `RwLock` over `Mutex`?
**A:** When the workload is **read-heavy**. If reads are frequent and writes are rare, `RwLock` allows high concurrency. If writes are frequent or the critical section is extremely short, the overhead of managing reader counts in `RwLock` might make it slower than a simple `Mutex`.

### Q: Can `RwLock` deadlock?
**A:** Yes.
1. Standard cycle deadlocks (A waits for B, B waits for A).
2. **Upgrade Deadlock**: A thread holding a read lock tries to acquire a write lock on the same `RwLock`. It will wait for itself to release the read lock (which it can't do because it's blocked waiting for the write lock). Rust's `RwLock` does *not* support lock upgrading. You must drop the read lock before acquiring the write lock.

### Q: Is `RwLock<T>` Send and Sync?
**A:** 
- `RwLock<T>` is `Send` if `T` is `Send` and `Sync`.
- `RwLock<T>` is `Sync` if `T` is `Send` and `Sync`.
This ensures that if multiple threads can access `T` (via read locks), `T` itself must be safe to share.
