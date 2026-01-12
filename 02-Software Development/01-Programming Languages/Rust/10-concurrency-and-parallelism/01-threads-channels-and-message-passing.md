#Rust
---
---

## Summary
Rust provides low-level concurrency primitives through its standard library (`std::thread`) and safe cross-thread communication via `mpsc` (Multi-Producer, Single-Consumer) channels. The safety of these operations is guaranteed at compile-time by the `Send` and `Sync` marker traits, which ensure that data is only moved or shared between threads when it is safe to do so, effectively eliminating data races.

## Detailed Explanation

### 1. Threads (`std::thread`)
Rust uses **1:1 threading**, meaning each language thread maps to one operating system thread. 
- **Spawning**: `thread::spawn` creates a new thread. It takes a closure containing the code to execute.
- **Joining**: To ensure a thread finishes before the main thread exits, you must call `.join()` on the `JoinHandle` returned by `spawn`.
- **The `move` Keyword**: Used with closures to transfer ownership of captured variables from the environment into the thread. Without `move`, Rust's borrow checker would prevent the thread from using data that might not live long enough.

### 2. Channels (`std::sync::mpsc`)
Message passing is a popular concurrency pattern summarized by the mantra: *"Do not communicate by sharing memory; instead, share memory by communicating."*
- **`mpsc`**: Stands for **Multi-Producer, Single-Consumer**.
- **The Pair**: `mpsc::channel()` returns a `(Sender, Receiver)` tuple.
- **Multiple Producers**: You can call `.clone()` on the `Sender` to allow multiple threads to send messages to the same `Receiver`.
- **Single Consumer**: Only one `Receiver` exists. It cannot be cloned.
- **Blocking vs. Non-blocking**: `recv()` blocks the current thread until a message is available, while `try_recv()` returns immediately with a `Result`.

### 3. Send and Sync Traits
These are **marker traits** (they have no methods) that the compiler uses to track thread safety.
- **`Send`**: A type is `Send` if ownership of its values can be transferred between threads. Almost all Rust types are `Send` (e.g., `i32`, `String`, `Vec`). A notable exception is `Rc<T>`, which is not `Send` because its reference count is not updated atomically.
- **`Sync`**: A type is `Sync` if it is safe for multiple threads to have a reference (`&T`) to it. A type `T` is `Sync` if and only if `&T` is `Send`.
- **Common Combinations**: 
  - `Arc<T>` is `Send + Sync` if `T` is `Send + Sync`.
  - `Mutex<T>` makes a non-`Sync` type `T` safe to share, providing `T` is `Send`.

## Code Examples

### Basic Thread and Channel Usage
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    // Create a channel
    let (tx, rx) = mpsc::channel();

    // Spawn a thread and move the sender into it
    thread::spawn(move || {
        let val = String::from("hi");
        thread::sleep(Duration::from_secs(1));
        tx.send(val).unwrap(); // send takes ownership of 'val'
    });

    // Receive the message in the main thread
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### Advanced Pattern: Worker Pool (Multi-Producer)
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Create 3 worker threads
    for i in 0..3 {
        let tx_clone = tx.clone(); // Clone the sender for each thread
        thread::spawn(move || {
            tx_clone.send(format!("Message from thread {}", i)).unwrap();
        });
    }

    // Drop the original sender so the receiver knows no more messages are coming
    drop(tx);

    // Iterate over received messages until all senders are dropped
    for msg in rx {
        println!("Received: {}", msg);
    }
}
```

## Common Use Cases
1. **Background Tasks**: Offloading long-running computations (e.g., file processing, image encoding) to keep the main/UI thread responsive.
2. **Producer-Consumer Pipelines**: Decomposing a complex task into stages, each running in its own thread and communicating via channels.
3. **Actor-like Systems**: Using channels to send commands to a dedicated "manager" thread that owns a specific resource.

## Interview Questions

**Q: What is the difference between `Send` and `Sync`?**
**A:** `Send` indicates that a type can be moved to another thread (ownership transfer). `Sync` indicates that a type can be safely shared between threads via references. Formally, `T` is `Sync` if `&T` is `Send`.

**Q: Why is `Rc<T>` not `Send` or `Sync`?**
**A:** `Rc<T>` uses a non-atomic reference counter. If two threads cloned an `Rc` simultaneously, the counter could be corrupted, leading to memory leaks or use-after-free bugs. `Arc<T>` (Atomic Reference Counter) is the thread-safe alternative.

**Q: How does a receiver know when to stop waiting for messages?**
**A:** The `Receiver` will return an error (or terminate an iterator) when all associated `Sender` handles (including the original and all clones) have been dropped.

**Q: What happens if you try to use a non-`Send` type in `thread::spawn`?**
**A:** The code will fail to compile. Rust's compiler checks the bounds of the closure passed to `spawn`, which requires that all captured variables implement `Send`.
