#Zig
---

## Summary
Pointers and slices are the core of Zig's memory model. Zig distinguishes between pointers to a single item, pointers to many items, and slices (which bundle a pointer with a length). This granularity provides both performance and safety by making memory access patterns explicit.

## Detailed Explanation

### Single-Item Pointers (`*T`)
Points to exactly one value of type `T`. Pointer arithmetic is **not allowed** on single-item pointers.

```zig
const std = @import("std");

test "single item pointer" {
    var x: i32 = 123;
    const ptr = &x;
    ptr.* = 456; // Dereference and assign
    try std.testing.expect(x == 456);
}
```

### Many-Item Pointers (`[*]T`)
Points to an unknown number of items. Pointer arithmetic **is allowed**. These are similar to C pointers but lack bounds checking.

```zig
test "many item pointer" {
    const array = [_]i32{ 1, 2, 3 };
    const ptr: [*]const i32 = &array;
    try std.testing.expect(ptr[1] == 2);
}
```

### Slices (`[]T`)
A slice is a "fat pointer"—it contains a many-item pointer and a `usize` length. Slices are the preferred way to pass arrays around in Zig because they carry their own bounds.

```zig
test "slices" {
    var array = [_]i32{ 1, 2, 3, 4, 5 };
    const slice: []i32 = array[1..4]; // Items at indices 1, 2, 3
    try std.testing.expect(slice.len == 3);
    try std.testing.expect(slice[0] == 2);
}
```

### Sentinel Termination
Zig supports pointers and slices that are terminated by a specific value (sentinel). The most common is the null-terminated string: `[*:0]u8` (many-item pointer to `u8` terminated by `0`).

```zig
test "sentinel termination" {
    const c_string: [*:0]const u8 = "hello";
    try std.testing.expect(c_string[5] == 0);
}
```

### Constant Pointers
Zig enforces pointer constness. A `*const T` cannot be used to modify the underlying data.

### Pointers and Alignment
Pointers in Zig carry alignment information in their type (e.g., `*align(4) i32`). This allows the compiler to generate optimal load/store instructions for the target CPU.

## Interview Questions
*   **Q: What is the difference between `*T` and `[*]T`?**
    *   **A:** `*T` is a pointer to a single item and does not allow pointer arithmetic, making it safer. `[*]T` is a many-item pointer (like a C pointer) that allows arithmetic but provides no safety guarantees regarding bounds.
*   **Q: Why are slices preferred over many-item pointers?**
    *   **A:** Slices include a length (`len` field), which allows Zig to perform safety checks (bounds checking) in debug modes, preventing buffer overflows. Many-item pointers have no inherent length.
*   **Q: How do you perform pointer arithmetic in Zig?**
    *   **A:** You must first ensure you have a many-item pointer (`[*]T`). You can then use the `ptr + offset` syntax or index into it like an array (`ptr[i]`).
*   **Q: What is a "fat pointer" in the context of Zig?**
    *   **A:** A fat pointer (specifically a slice) is a structure that contains both a memory address (the pointer) and metadata (the length). In Zig, a slice is twice the size of a regular pointer on most architectures.
