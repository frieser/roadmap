#Zig
---

## Summary
Functions in Zig are declared with the `fn` keyword. They are designed to be explicit and predictable, with no hidden parameters or implicit behavior. Zig supports recursion, and its powerful compile-time execution (`comptime`) allows functions to operate on types and generate code during compilation.

## Detailed Explanation

### Function Declaration
Functions have a fixed signature: `fn name(params) return_type`. Parameters are immutable by default within the function body.

```zig
const std = @import("std");

fn add(a: i32, b: i32) i32 {
    return a + b;
}

test "basic function" {
    try std.testing.expect(add(1, 2) == 3);
}
```

### Pure Functions and Side Effects
While Zig doesn't have a keyword for "pure" functions, the philosophy of Zig encourages explicit side effects. Functions that modify data typically take a pointer as an argument.

### Recursion
Zig supports standard recursion. However, because Zig doesn't have a managed stack or a runtime that handles stack overflows gracefully, developers must be mindful of deep recursion in systems with limited memory.

```zig
fn factorial(n: u32) u32 {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}

test "recursion" {
    try std.testing.expect(factorial(5) == 120);
}
```

### Inline Functions
The `inline` keyword hints (and usually forces) the compiler to expand the function body at the call site. This is useful for performance optimization and certain `comptime` tricks.

```zig
inline fn fastAdd(a: i32, b: i32) i32 {
    return a + b;
}
```

### Comptime Functions
One of Zig's most powerful features is the ability to run functions at compile time. If a function's arguments are `comptime`-known, the function can be evaluated by the compiler.

```zig
fn getArraySize(comptime T: type) usize {
    return @sizeOf(T) * 8;
}

test "comptime function" {
    const size = comptime getArraySize(u32);
    try std.testing.expect(size == 32);
}
```

### Reflection and Recursion
Zig can perform reflection on functions using `@typeInfo`. This allows generic code to inspect function parameters and return types at compile time.

## Interview Questions
*   **Q: Can function parameters be modified inside the function in Zig?**
    *   **A:** No, function parameters are immutable. If you need to modify a value, you must pass a pointer to it (e.g., `fn modify(x: *i32)`).
*   **Q: How does Zig handle recursion limits?**
    *   **A:** Zig uses the native stack. There is no built-in "recursion limit" in the language itself, but deep recursion will eventually cause a stack overflow. For compile-time recursion (`comptime`), the compiler has a configurable limit to prevent infinite loops during compilation.
*   **Q: What is the difference between a regular function and an `inline` function?**
    *   **A:** A regular function involves a call overhead (jumping to a memory address). An `inline` function has its body copied directly into the call site by the compiler, removing the call overhead but potentially increasing the binary size.
*   **Q: What is a `noreturn` type?**
    *   **A:** It is a special return type for functions that never return to the caller, such as `std.process.exit()` or a function containing an infinite loop.
