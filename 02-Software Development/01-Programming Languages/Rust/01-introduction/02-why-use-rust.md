#Rust
---
---

## Summary
Rust is a modern systems programming language that focuses on safety, speed, and concurrency. It achieves memory safety without a garbage collector (GC) by using a unique ownership system, making it ideal for performance-critical applications where reliability is paramount.

## Detailed Explanation

### Memory safety without GC
Unlike many modern languages (Java, Python, Go) that use a Garbage Collector to manage memory at runtime, Rust manages memory through its **ownership system** at compile time. This results in:
- **No GC Pauses**: Predictable performance without "stop-the-world" latency.
- **Efficient Resource Management**: Memory is freed the moment it is no longer needed.
- **Safety**: Prevents common bugs like null pointer dereferencing, buffer overflows, and use-after-free.

### Ownership & Borrow Checker
The core of Rust's safety is built on three main rules:
1. Each value in Rust has a variable that’s called its **owner**.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value will be dropped.

The **Borrow Checker** enforces these rules by tracking references. You can have:
- Any number of immutable references (`&T`).
- **OR** exactly one mutable reference (`&mut T`).
This prevents **Data Races** at compile time.

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // Ownership moves to s2. s1 is no longer valid.
    
    // println!("{}", s1); // This would cause a compile error!
    
    let s3 = &s2; // Immutable borrow
    println!("Borrowed: {}", s3);
}
```

### Fearless Concurrency
Concurrent programming is notoriously difficult due to data races. Rust makes it "fearless" because the compiler ensures that:
- Data is either shared immutably or owned by a single thread for mutation.
- Traits like `Send` and `Sync` define which types can be safely transferred or shared across threads.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v); // 'move' transfers ownership to the thread
    });

    handle.join().unwrap();
}
```

### Zero-cost abstractions
Rust follows the C++ philosophy: "What you don't use, you don't pay for." High-level abstractions like iterators, closures, and pattern matching are compiled down to machine code as efficient as hand-written assembly or C.

```rust
let numbers = vec![1, 2, 3];
// This iterator chain is optimized away to a simple loop.
let sum: i32 = numbers.iter().map(|x| x * 2).sum();
```

### Cargo ecosystem
Rust comes with **Cargo**, a world-class build tool and package manager. It handles:
- **Dependency Management**: Easily add libraries (crates) from crates.io.
- **Building & Testing**: `cargo build` and `cargo test`.
- **Documentation**: `cargo doc` automatically generates documentation for your project and dependencies.

## Interview Questions

1. **How does Rust achieve memory safety without a garbage collector?**
   - Through its ownership system and borrow checker, which track the lifecycle of every value at compile time and insert cleanup code (drop) automatically.

2. **What are the three rules of ownership?**
   - Each value has an owner; there is only one owner at a time; when the owner goes out of scope, the value is dropped.

3. **What is a Data Race, and how does Rust prevent it?**
   - A data race occurs when two threads access the same memory concurrently, at least one is writing, and there's no synchronization. Rust prevents this by disallowing multiple mutable references or a mix of mutable and immutable references to the same data.

4. **What is the difference between `String` and `&str`?**
   - `String` is an owned, heap-allocated, growable string. `&str` is a string slice, an immutable reference to a string sequence (usually a view into a `String` or a string literal).

5. **Explain "Zero-Cost Abstractions".**
   - It means that high-level features do not impose any runtime overhead compared to a lower-level implementation. The compiler optimizes them into the same efficient machine code.
