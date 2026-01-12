# Async/Await Resurrection

## Summary
The "Async/Await Resurrection" refers to the re-introduction of first-class asynchronous programming in Zig after its removal in 0.10.0. The new model, targeted for the 1.0 release (around 2026), solves the "colorless" async function problem by using **Dependency Injection** for I/O operations, allowing code to function both synchronously and asynchronously depending on the context.

## Detailed Explanation

### History & Removal
*   **Early Success**: Zig 0.6.0 featured a unique "suspend/resume" model that allowed for highly efficient, stackless coroutines.
*   **The Gap**: In Zig 0.10.0, async was disabled. The primary reasons were technical debt in the C++ compiler implementation and the realization that the old model still suffered from "function coloring" (where async functions could only be called by other async functions).
*   **The Return (2025/2026)**: The "Async Resurrection" brings back `suspend`/`resume` primitives but integrates them into a new I/O interface pattern.

### The "Colorless" Concept
Zig's 1.0 async model aims to solve the "Function Coloring" problem through **Dependency Injection of I/O**.
*   **I/O as an Interface**: Functions no longer assume a global, blocking I/O state. Instead, they accept an `Io` interface (similar to how they accept an `Allocator`).
*   **Polymorphic Asynchrony**: If you pass a *synchronous* `Io` implementation, the code runs linearly. If you pass an *asynchronous* `Io` implementation (e.g., backed by `io_uring` or `epoll`), the same code becomes asynchronous, suspending execution when I/O blocks.
*   **Mechanism**: The `suspend` and `resume` primitives remain, but they are often hidden behind the `Io` interface methods. This allows the ecosystem to remain unified rather than splitting into `sync_pkg` and `async_pkg`.

### Current Status (2026)
Core `std` libraries (DNS, File, Net) have been refactored to use this model. The `Writer` and `Reader` interfaces were famously broken in 2025 ("Writergate") to facilitate this shift.

## Interview Questions

**Q: How does Zig avoid the "function coloring" problem in its 1.0 roadmap?**
**A:** By making I/O an interface provided by the caller (Dependency Injection). Functions take an `Io` object (or similar abstraction), allowing the same business logic to behave synchronously or asynchronously based on the implementation passed in, without changing the function signature keywords.

**Q: Why was async removed in Zig 0.10.0?**
**A:** It was removed due to technical debt in the legacy C++ compiler and to allow the team to redesign the feature from scratch for the self-hosted compiler, focusing on a more robust, "colorless" design.

**Q: What is the role of `io_uring` in Zig's async model?**
**A:** `io_uring` is a Linux kernel interface for high-performance asynchronous I/O. Zig's async runtime (on Linux) is expected to leverage `io_uring` heavily to implement the non-blocking version of the standard I/O interfaces, providing extreme performance.
