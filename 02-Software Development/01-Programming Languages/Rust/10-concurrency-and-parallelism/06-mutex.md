#Rust
---
---

## Summary
`Mutex<T>` (Mutual Exclusion) is a synchronization primitive that ensures only one thread can access the data `T` at any given time. It uses a locking mechanism to enforce exclusive access. In Rust, a Mutex "wraps" the data, making it impossible to access the data without holding the lock.

## Detailed Explanation

### 1. Lock and Guard
- **`.lock()`**: Blocks the current thread until the lock is acquired. Returns a `Result`.
- **`MutexGuard`**: The `.lock()` method returns a smart pointer called a `MutexGuard`.
  - It implements `Deref` to point to the inner data.
  - It implements `Drop`. When the guard goes out of scope, the lock is **automatically released**. This prevents "forgetting to unlock" bugs.

### 2. Poisoning
If a thread panics while holding a Mutex, the lock becomes "poisoned".
- Future calls to `.lock()` will return an `Err(PoisonError)`.
- This serves as a safety mechanism, warning other threads that the data might be in an inconsistent state.

### 3. Interior Mutability
`Mutex<T>` provides interior mutability. Even if the Mutex itself is immutable (e.g., inside an `Arc`), the data inside can be mutated once locked.

## Rust Application

### Basic Usage
Note how the lock is released automatically at the end of the scope.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let lock = Arc::new(Mutex::new(0));
    let lock_clone = Arc::clone(&lock);

    let handle = thread::spawn(move || {
        // Blocks until lock is acquired
        let mut num = lock_clone.lock().unwrap();
        *num += 1;
        // 'num' (the guard) is dropped here, unlocking the Mutex
    });

    handle.join().unwrap();
    println!("Result: {:?}", *lock.lock().unwrap());
}
```

### Handling Poisoning
```rust
let lock = Arc::new(Mutex::new(0));
let lock_clone = lock.clone();

thread::spawn(move || {
    let _guard = lock_clone.lock().unwrap();
    panic!("Oops!"); // Poisons the lock
});

// Main thread tries to lock
match lock.lock() {
    Ok(guard) => println!("Value: {}", *guard),
    Err(poisoned) => {
        println!("Lock was poisoned! Recovering...");
        let guard = poisoned.into_inner(); // Force access anyway
        println!("Recovered value: {}", *guard);
    }
}
```

## Interview Questions

### Q: How does Rust's Mutex differ from Mutexes in C/C++?
**A:** In Rust, the Mutex **owns** the data it protects (`Mutex<T>`). In C/C++, the mutex and the data are separate variables, requiring discipline to remember to lock the mutex before accessing the data. Rust enforces this at compile time; you *cannot* access the data without locking.

### Q: What is a deadlock and how do you avoid it with Mutexes?
**A:** A deadlock occurs when two threads wait for each other to release locks (e.g., A holds Lock1 waiting for Lock2, B holds Lock2 waiting for Lock1). 
- Avoidance: 
  1. Always acquire locks in the same global order.
  2. Minimize the scope of the lock (keep critical sections short).
  3. Use `try_lock()` to back off if a lock isn't available.

### Q: Does `Mutex<T>` allow concurrent reads?
**A:** No. `Mutex` enforces *exclusive* access. Even if multiple threads just want to read, they must wait their turn. Use `RwLock<T>` for concurrent readers.
