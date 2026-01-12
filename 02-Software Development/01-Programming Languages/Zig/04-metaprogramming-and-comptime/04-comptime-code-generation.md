# Comptime Code Generation

## Summary
Zig allows you to generate code logic and types programmatically. This can replace preprocessor macros, code generation tools (like `protoc`), and complex boilerplate.

## Detailed Explanation

### `@compileLog`
Prints values during compilation. Useful for debugging comptime logic.

### `@compileError`
Aborts compilation with a message. Used for enforcing type constraints.

### Generating Types
You can calculate alignment, bit-widths, or field counts based on input parameters.

```zig
fn MakeInt(comptime bits: usize) type {
    return @Type(.{ .Int = .{ .signedness = .unsigned, .bits = bits } });
}
const u24 = MakeInt(24);
```

### Go Comparison
*   **Go**: `go generate` runs external scripts to write source files.
*   **Zig**: Code generation is internal. The compiler executes Zig code to produce the final program structure.

## Interview Questions

**Q: What is `@compileError` used for?**
**A:** It allows library authors to provide custom, readable error messages when a user misuses a generic API (e.g., passing a float to a function that only supports integers).

**Q: Can you construct a Struct type programmatically from a list of strings?**
**A:** Yes, by populating the `std.builtin.Type` union and passing it to `@Type()`. This allows for extremely powerful DSLs and mapping constructs.

**Q: What is the primary advantage of internal code generation vs external tools (like in Go/Protobuf)?**
**A:** Integration. You don't need a separate build step or external dependencies. The code generation changes immediately when you change the source, and the compiler checks it all in one pass.
