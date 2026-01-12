# Intro to Comptime

## Summary
`comptime` is Zig's "killer feature." It allows you to execute arbitrary Zig code during the compilation process. This unifies macros, metaprogramming, and generics into a single concept: running code at compile time to generate values or types.

## Detailed Explanation

### The `comptime` Keyword
Marks an expression or block to be evaluated by the compiler.
```zig
const x = comptime {
    var sum = 0;
    for (0..5) |i| sum += i;
    return sum; // 10, calculated at compile time
};
```

### Inline Loops
`inline for` and `inline while` tell the compiler to unroll the loop. This is essential when iterating over types (which don't exist at runtime).

```zig
const types = .{ i32, f64, bool };
inline for (types) |T| {
    // Generates code for each type T
    std.debug.print("{s}\n", .{@typeName(T)});
}
```

### Compile-Time Variables
Variables declared in a `comptime` block or marked `comptime` are constants at runtime.

### Go Comparison
*   **Go**: Has no macro system or compile-time execution. `go generate` is an external tool.
*   **Zig**: Metaprogramming is built-in. You use the same syntax for runtime and compile-time logic.

## Interview Questions

**Q: What is the difference between `var` and `comptime var`?**
**A:** `var` is a runtime variable (unless inside a comptime block). `comptime var` exists only during compilation; it can be modified by compile-time logic but is baked into a constant (or vanishes) in the final binary.

**Q: Why do we need `inline for`?**
**A:** Standard `for` loops work on runtime values. `inline for` unrolls the loop at compile time, which is required when iterating over things that only exist at compile time, like **Types** or **Struct Fields**.

**Q: Can `comptime` code allocate memory?**
**A:** Yes, the compiler has an interpreter that can handle memory allocation, but this memory must be resolved or discarded before the final binary is produced. You can't pass a pointer allocated at compile-time to runtime code directly (unless it's promoted to static data).
