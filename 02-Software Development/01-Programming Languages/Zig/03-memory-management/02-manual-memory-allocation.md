# Manual Memory Allocation

## Summary
Zig requires manual calls to allocate and free memory using the `Allocator` API. The primary methods are `create`/`destroy` for single items and `alloc`/`free` for slices (arrays). Developers handle ownership manually, usually following a "creator frees" convention.

## Detailed Explanation

### Single Item: `create` / `destroy`
Used for allocating a single struct or primitive on the heap.
*   `create(T)` returns `!*T` (Error Union of Pointer).
*   `destroy(ptr)` frees the memory.

```zig
const ptr = try allocator.create(i32);
ptr.* = 100;
allocator.destroy(ptr);
```

### Multiple Items: `alloc` / `free`
Used for allocating dynamic arrays.
*   `alloc(T, count)` returns `![]T` (Error Union of Slice).
*   `free(slice)` frees the memory.

```zig
const slice = try allocator.alloc(u8, 1024);
defer allocator.free(slice);
```

### Ownership Semantics
Zig has no "Borrow Checker" (Rust) or GC (Go).
*   **Ownership**: If you create it, you own it.
*   **Transfer**: If you pass a pointer to a struct, document who is responsible for freeing it.
*   **Double Free**: Calling `free` twice is Undefined Behavior (though GPA detects it in debug).

### Go Comparison
*   **Go**: `x := new(int)` allocates. GC frees it later.
*   **Zig**: `allocator.create(i32)` allocates. You MUST call `allocator.destroy()`. Forgetting to do so results in a memory leak.

## Interview Questions

**Q: What is the difference between `allocator.create` and `allocator.alloc`?**
**A:** `create` allocates memory for a **single** instance of a type and returns a pointer (`*T`). `alloc` allocates memory for **multiple** instances (an array) and returns a slice (`[]T`).

**Q: What happens if you forget to free memory in Zig?**
**A:** A **Memory Leak** occurs. The memory remains occupied until the program terminates. Using `GeneralPurposeAllocator` in debug mode will print a leak report to stderr on exit.

**Q: Is it safe to pass a pointer to a stack variable to another function?**
**A:** Yes, as long as the receiving function does not store that pointer and use it after the stack frame returns. Unlike Go, Zig does not perform "escape analysis" to automatically move stack variables to the heap. Returning a pointer to a stack variable is a classic bug (Use-After-Return).
