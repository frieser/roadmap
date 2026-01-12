# Memory Safety without Garbage Collection

## Summary
Zig achieves a high degree of memory safety without a garbage collector or strict borrow checker by using **Runtime Safety Checks**, **Optional Types**, and **Explicit Alignment**. While not "safe" in the formal Rust sense (compile-time guarantees), Zig eliminates entire classes of bugs (null pointers) and catches others (overflow, bounds) in Debug mode.

## Detailed Explanation

### 1. No Null Pointers
Pointers `*T` cannot be null. To have a nullable value, you must use an Optional `?*T`. You are forced to unwrap it before use.
```zig
var ptr: ?*i32 = null;
// ptr.* = 5; // Compile Error
if (ptr) |p| p.* = 5; // Safe
```

### 2. Runtime Safety (Bounds Checking)
In `Debug` and `ReleaseSafe` modes, Zig inserts checks for:
*   Array/Slice out-of-bounds access.
*   Integer overflow.
*   Cast truncation.
*   Unreachable code execution.

If triggered, the program panics safely (crashes) rather than continuing with corrupted state.

### 3. Undefined Behavior Protection
Using `undefined` memory is UB, but debug builds often fill `undefined` with garbage patterns (`0xAA`) to force crashes early if read. The GPA allocator also quarantines freed memory to catch Use-After-Free bugs.

### Go Comparison
*   **Go**: Safe because of GC (no use-after-free) and runtime bounds checks.
*   **Zig**: Safe because of Optionals (no nil dereference) and explicit allocator discipline. Use-after-free is possible in ReleaseFast but detectable in Debug.

## Interview Questions

**Q: How does Zig prevent null pointer dereferences?**
**A:** Standard pointers (`*T`) are strictly non-nullable. Nullability must be opted into via Optional types (`?*T`). The compiler forces the programmer to unwrap/check the optional before accessing the value.

**Q: What is "ReleaseSafe" mode?**
**A:** It is a build mode that applies optimizations (like ReleaseFast) but keeps runtime safety checks enabled (bounds checks, overflow checks). It offers a balance of speed and safety for production environments where safety is critical.

**Q: Does Zig guarantee memory safety like Rust?**
**A:** No. Zig allows manual memory management, which means Use-After-Free and Double-Free bugs are possible (though mitigated by the GPA). Rust prevents these at compile time. Zig prioritizes simplicity and control, relying on tooling (Debug builds, tests) to catch these issues.
