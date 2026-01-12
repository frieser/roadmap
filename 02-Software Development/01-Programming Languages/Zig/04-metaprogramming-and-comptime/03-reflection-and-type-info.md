# Reflection and Type Info

## Summary
Zig provides powerful compile-time reflection via `@typeInfo(T)`. This builtin returns a tagged union describing the type (struct fields, function args, alignment, etc.). You can inspect this info to generate code, serialization logic, or validation checks.

## Detailed Explanation

### `@typeInfo(T)`
Returns `std.builtin.Type`.
```zig
const Info = @typeInfo(MyStruct);
if (Info == .Struct) {
    // iterate fields
}
```

### `@field(obj, "name")`
Accesses a field by its string name. Combined with `inline for`, this allows iterating over struct fields.

```zig
fn printFields(obj: anytype) void {
    inline for (@typeInfo(@TypeOf(obj)).Struct.fields) |field| {
        const val = @field(obj, field.name);
        std.debug.print("{s}: {any}\n", .{field.name, val});
    }
}
```

### Go Comparison
*   **Go**: Uses the `reflect` package at **runtime**. It is slow and not type-safe.
*   **Zig**: Reflection happens at **compile time**. It incurs zero runtime cost (the compiler unrolls the logic) and is fully type-safe.

## Interview Questions

**Q: Does Zig have runtime reflection?**
**A:** No. Zig reflection (`@typeInfo`) is strictly a compile-time feature. Information about types is resolved during compilation. If you need runtime reflection, you must build it yourself (e.g., by generating a lookup table at compile time).

**Q: How is `@field` useful?**
**A:** It allows accessing struct members dynamically using strings known at compile time. This is essential for writing generic serialization libraries (like JSON parsers) that need to map string keys to struct fields without boilerplate.

**Q: What is `@TypeOf`?**
**A:** It is a builtin that returns the `type` of a given expression or variable. It is often used in generic functions (`anytype` params) to determine what was passed in.
