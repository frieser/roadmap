#Zig
---

## Summary
Zig provides a rich set of primitive types with explicit control over bit-width and memory layout. A standout feature is the ability to define integers of any bit-width (e.g., `u3`, `i47`). Alignment is treated as a first-class property of types, ensuring that memory access is both safe and optimized for specific hardware architectures.

## Detailed Explanation

### Integers and Arbitrary Bit-Width
Zig supports standard integer types (`u8`, `i32`, `u128`, etc.) but also allows for non-standard bit-widths.
*   **Standard Types**: `i8`, `u8`, `i16`, `u16`, `i32`, `u32`, `i64`, `u64`, `i128`, `u128`.
*   **Arch-Specific**: `isize`, `usize` (pointer-sized integers).
*   **Arbitrary Bit-Width**: `u3`, `i7`, `u21`, etc. These are useful for bit-packing and matching hardware registers exactly.

```zig
const std = @import("std");

test "arbitrary bit-width integers" {
    const a: u3 = 5;
    const b: u3 = 2;
    const c: u4 = a + b; // Widening works implicitly
    try std.testing.expect(c == 7);
}
```

### Floats and Other Primitives
*   **Floats**: `f16` (half), `f32` (float), `f64` (double), `f80` (extended), `f128` (quad).
*   **Boolean**: `bool` (`true` or `false`).
*   **Void**: `void`. Always occupies 0 bytes of memory.
*   **NoReturn**: `noreturn`. Used for functions that never return (like `exit` or infinite loops).
*   **Type**: `type`. In Zig, types themselves are values at compile time (`comptime`).

### Undefined and Null
*   `undefined`: Represents uninitialized memory. Using it helps catch bugs where data is used before being set, and in debug modes, Zig often fills this memory with garbage (like `0xAA`).
*   `null`: Only used with **Optional Types** (e.g., `?i32`).

### Alignment Rules
Every type has an alignment requirement. For example, a `u32` typically requires 4-byte alignment.
*   `@alignOf(T)`: Returns the alignment of type `T`.
*   `align(N)`: Specifies a custom alignment for a variable or pointer.
*   `@alignCast(ptr)`: Safely casts a pointer to a higher alignment, with a runtime safety check in debug modes.

```zig
test "alignment basics" {
    var x: i32 = 123;
    const ptr: *align(4) i32 = &x;
    
    // Explicitly set alignment
    var y: i32 align(8) = 456;
    const ptr2: *align(8) i32 = &y;
    
    _ = ptr;
    _ = ptr2;
}
```

## Interview Questions
*   **Q: What is the difference between `undefined` and `null` in Zig?**
    *   **A:** `undefined` means the memory is uninitialized and contains garbage; it is a value that can be assigned to any type. `null` is a specific value used only with optional types to represent the absence of a value.
*   **Q: How does Zig handle integers of arbitrary bit-width?**
    *   **A:** Zig allows defining integers like `u3` or `i7`. These behave like regular integers but the compiler ensures they fit within the specified bits. This is particularly useful for low-level programming and bitfield-like structures without the ambiguity of C bitfields.
*   **Q: What is the purpose of `@alignOf` and when would you use it?**
    *   **A:** `@alignOf` returns the byte alignment required for a type on the target architecture. It's used when performing manual memory allocation, implementing custom allocators, or interfacing with hardware/C-code that has strict alignment requirements.
