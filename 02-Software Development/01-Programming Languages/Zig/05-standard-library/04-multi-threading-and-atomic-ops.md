# Multi-threading and Atomic Ops

## Summary
Zig provides low-level concurrency primitives in `std.Thread` (threads, mutexes, condition variables) and built-in atomic operations. It maps closely to OS threads and hardware instructions.

## Detailed Explanation

### Threads
`std.Thread.spawn` creates an OS thread.
```zig
const thread = try std.Thread.spawn(.{}, myFunction, .{arg1});
thread.join();
```

### Synchronization
*   `std.Thread.Mutex`: `lock()`, `unlock()`.
*   `std.Thread.Condition`: `wait()`, `signal()`.

### Atomics
Zig uses builtins for atomic operations, ensuring correct memory ordering.
*   `@atomicLoad`, `@atomicStore`
*   `@atomicRmw` (Read-Modify-Write)
*   `@cmpxchgStrong`

```zig
var counter: i32 = 0;
_ = @atomicRmw(i32, &counter, .Add, 1, .seq_cst);
```

### Go Comparison
*   **Go**: Goroutines (M:N scheduling), Channels. High-level.
*   **Zig**: OS Threads (1:1), Mutexes/Atomics. Low-level. Zig does not have built-in "async/await" in the current stable release (it was removed for rework).

## Interview Questions

**Q: How do you pass arguments to a thread in Zig?**
**A:** `std.Thread.spawn` takes a tuple of arguments as its third parameter. These are passed to the entry function. You must ensure that any pointers passed remain valid for the thread's lifetime (e.g., not pointing to a stack variable that is about to return).

**Q: What is the difference between `@cmpxchgStrong` and `@cmpxchgWeak`?**
**A:** `Strong` is guaranteed to succeed if the value matches. `Weak` is allowed to fail spuriously (even if the value matches), which is a property of some hardware architectures (like ARM). Weak is often used in loops where a retry is cheap.

**Q: Does Zig have Green Threads or Coroutines like Go?**
**A:** Currently, no (as of 0.11+). Zig's `async` functionality was removed from the compiler for a major rework. Standard threading uses OS threads (1:1 mapping).
