# Zig C Interoperability: Exported Zig Functions for C
---

## Summary
Zig allows its functions and variables to be called from C code by using the `export` keyword. This ensures that the symbols are visible to the linker with C linkage and adhere to the target's C calling convention. This is essential for creating shared libraries (`.so`, `.dll`) or when Zig code is being integrated into an existing C/C++ codebase.

## Detailed Explanation

### 1. The `export` Keyword
In Zig, symbols are private by default. The `export` keyword makes a function or variable visible to the linker as a global symbol.
*   It implies `extern` linkage.
*   It ensures the symbol name is not mangled.

```zig
export fn add(a: i32, b: i32) i32 {
    return a + b;
}
```

### 2. Calling Convention (`callconv`)
By default, exported functions in Zig use the C calling convention (`callconv(.C)`). However, it is good practice to be explicit if the function is specifically intended for C interop.
*   **C Calling Convention**: `callconv(.C)` ensures arguments are passed in registers and stack according to the platform's C ABI.
*   **Zig Calling Convention**: `callconv(.Inline)` or the default (unspecified) allows Zig to optimize calls, but these are not stable for external C code.

### 3. Extern Structs and Enums
When passing data structures between Zig and C, you must use `extern` to ensure memory layout compatibility.
*   **`extern struct`**: Guaranteed to have the same field order and padding as a C struct.
*   **`extern enum`**: Guaranteed to have a specific integer backing type (usually `c_int` by default) compatible with C enums.

### 4. Generating C Header Files
Zig does not automatically generate a `.h` file for your exported functions. However, the Zig build system can emit them.

In `build.zig`:
```zig
const lib = b.addSharedLibrary(.{
    .name = "my_zig_lib",
    .root_source_file = b.path("src/main.zig"),
    .target = target,
    .optimize = optimize,
});
// Emit the C header file
lib.emit_h = true; 
```
This will generate a header file in the build directory containing the C declarations for all `export`ed symbols.

## Code Examples

### Exporting a Function with C ABI
```zig
const std = @import("std");

/// This function is callable from C as:
/// int32_t zig_multiply(int32_t a, int32_t b);
export fn zig_multiply(a: i32, b: i32) callconv(.C) i32 {
    return a * b;
}
```

### Exporting a Struct
```zig
pub const Vec3 = extern struct {
    x: f32,
    y: f32,
    z: f32,
};

export fn get_magnitude(v: Vec3) f32 {
    return @sqrt(v.x * v.x + v.y * v.y + v.z * v.z);
}
```

### Using Zig as a C Library (C side)
**main.c**:
```c
#include <stdio.h>
#include <stdint.h>

// Manually declared or generated via emit_h
extern int32_t zig_multiply(int32_t a, int32_t b);

int main() {
    int32_t result = zig_multiply(10, 5);
    printf("Result from Zig: %d\n", result);
    return 0;
}
```

## Interview Questions

**Q: What does the `export` keyword do in Zig?**
**A:** It makes a function or variable visible to the linker as a global symbol with C linkage. It prevents symbol mangling and ensures that the function is available to be called by other languages or object files that follow the C ABI.

**Q: Why do we need to use `extern struct` when passing data to C?**
**A:** Zig's regular `struct` has no guaranteed memory layout; the compiler can reorder fields to optimize for size or performance. An `extern struct` follows the platform's C ABI layout rules, ensuring that C code sees the fields in the correct order and with the expected padding.

**Q: How do you change the name of an exported function in Zig?**
**A:** You can use the `@export` builtin if you need more control, such as dynamic names or exporting a function with a different name than its Zig identifier:
`@export(my_internal_fn, .{ .name = "c_visible_name", .linkage = .strong });`

**Q: Can Zig export functions that use Zig-specific types like slices or error unions?**
**A:** Not directly to C. C does not understand Zig slices (`[]u8`) or error unions (`!u8`). To export such functionality, you must "lower" the interface to C-compatible types, such as passing a pointer and a length separately instead of a slice, or using an integer status code instead of an error union.
