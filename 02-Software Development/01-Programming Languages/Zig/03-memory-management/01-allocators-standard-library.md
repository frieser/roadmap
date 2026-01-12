# Allocators and Standard Library

## Summary
Zig does not have a global allocator or garbage collector. Instead, memory management is explicit. The standard library (`std.mem`) provides an `Allocator` interface, and `std.heap` offers implementations like `GeneralPurposeAllocator` (GPA), `ArenaAllocator`, and `FixedBufferAllocator`.

## Detailed Explanation

### The Allocator Interface
Functions that need to allocate memory accept an `allocator: std.mem.Allocator` parameter. This pattern is pervasive in the standard library (e.g., `ArrayList`, `HashMap`).

### Common Allocators
1.  **GeneralPurposeAllocator (GPA)**: The default for general application code. It includes debug features to detect memory leaks and use-after-free errors.
2.  **ArenaAllocator**: Wraps another allocator. It allows you to allocate many objects and free them all at once by deinitializing the arena. Great for request lifecycles.
3.  **FixedBufferAllocator**: Allocates from a pre-allocated slice of memory (stack or heap). Very fast, no syscalls, but fails if it runs out of space.
4.  **page_allocator**: Asks the OS for whole pages of memory. Usually slow for small objects; often used as the backing allocator for an Arena or GPA.

### Code Example
```zig
const std = @import("std");

pub fn main() !void {
    // 1. Initialize GPA
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit(); // Detect leaks at end of scope
    const allocator = gpa.allocator();

    // 2. Use with ArrayList
    var list = std.ArrayList(i32).init(allocator);
    defer list.deinit();
    try list.append(42);
}
```

### Go Comparison
*   **Go**: `make([]int, 10)` or `new(Struct)` automatically uses the runtime's heap allocator and garbage collector. You don't choose the strategy.
*   **Zig**: You must choose the allocator. This allows optimization (e.g., using an Arena for a web handler) that Go's GC cannot easily match.

## Interview Questions

**Q: Why doesn't Zig use a global allocator by default?**
**A:** To avoid hidden global state and to allow libraries to be used in environments where a global heap might not exist (e.g., embedded, kernels, WASM). Explicit allocators make memory dependencies clear in the function signature.

**Q: What is the benefit of an `ArenaAllocator`?**
**A:** It improves performance by reducing the number of `free` calls. You can allocate thousands of small objects and free them all instantly by resetting the arena, which also improves cache locality.

**Q: How does `GeneralPurposeAllocator` help with debugging?**
**A:** In Debug/ReleaseSafe modes, it tracks allocations to report memory leaks upon deinitialization and can quarantine freed memory to detect use-after-free bugs.
