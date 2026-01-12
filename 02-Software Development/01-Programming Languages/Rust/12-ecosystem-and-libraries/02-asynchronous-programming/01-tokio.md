# Tokio
---
---

## Summary
Tokio is the de-facto standard asynchronous runtime for Rust. It provides the building blocks needed to write network applications: an event loop (reactor), a task scheduler, and asynchronous I/O primitives for files and sockets. It is the foundation for most of the Rust web ecosystem (Axum, Hyper, Reqwest).

## Detailed Explanation

### Core Philosophy
Tokio is designed for **reliability** and **speed**. It uses a work-stealing scheduler to distribute tasks efficiently across CPU cores. It is "opinionated" in the sense that it provides a comprehensive ecosystem (timers, synchronization, I/O) rather than just a bare-bones executor.

### Key Features
*   **Work-Stealing Scheduler**: Multi-threaded runtime that automatically balances load between cores.
*   **Non-blocking I/O**: Efficient async implementations of `TcpStream`, `UdpSocket`, `File`, etc.
*   **Timers**: Efficient handling of timeouts and intervals.
*   **Synchronization**: Async-aware Mutexes, RwLocks, Channels (MPSC, Broadcast, Watch).
*   **Tracing**: Deep integration with the `tracing` crate for observability.

### Use Cases
*   **Network Servers**: HTTP, gRPC, WebSocket servers.
*   **Database Drivers**: Most async DB drivers (SQLx, mongodb) build on Tokio.
*   **CLI Tools**: Tools that perform parallel I/O operations (like `ripgrep`'s async cousins).

### Code Example
*Dependencies: `tokio`*

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;

    loop {
        // Accept a new socket
        let (mut socket, _) = listener.accept().await?;

        // Spawn a new task for each connection so we don't block
        tokio::spawn(async move {
            let mut buf = [0; 1024];

            // In a loop, read data from the socket and write the data back.
            loop {
                let n = match socket.read(&mut buf).await {
                    // socket closed
                    Ok(n) if n == 0 => return,
                    Ok(n) => n,
                    Err(e) => {
                        eprintln!("failed to read from socket; err = {:?}", e);
                        return;
                    }
                };

                // Write the data back
                if let Err(e) = socket.write_all(&buf[0..n]).await {
                    eprintln!("failed to write to socket; err = {:?}", e);
                    return;
                }
            }
        });
    }
}
```

## Interview Questions

1.  **Q: What is the difference between `tokio::spawn` and `std::thread::spawn`?**
    *   **A:** `std::thread::spawn` creates an OS-level thread, which has significant memory and context-switching overhead. `tokio::spawn` creates a "Green Thread" or Task. Thousands of Tokio tasks can run multiplexed on a single OS thread (or a small pool of them), making them extremely lightweight and efficient for I/O-bound work.

2.  **Q: Why do we need the `#[tokio::main]` attribute?**
    *   **A:** Rust's `main` function cannot be `async` by default because the standard library doesn't include a built-in runtime. `#[tokio::main]` is a macro that transforms your async `main` into a synchronous function that initializes the Tokio runtime and blocks on the execution of your code.

3.  **Q: What happens if you run blocking code (like `std::thread::sleep`) inside a Tokio task?**
    *   **A:** You will "block the thread," preventing the Tokio scheduler from running other tasks on that thread. Since Tokio uses a small pool of threads, blocking one can severely degrade performance or cause deadlocks. You should use `tokio::time::sleep` (which yields control) or `tokio::task::spawn_blocking` for CPU-intensive operations.

4.  **Q: Explain "Work Stealing" in the context of Tokio.**
    *   **A:** Tokio maintains a queue of tasks for each processor core. If one core finishes its tasks while others are still busy, it will "steal" tasks from the other cores' queues. This ensures efficiently balanced CPU utilization without a central bottleneck.
