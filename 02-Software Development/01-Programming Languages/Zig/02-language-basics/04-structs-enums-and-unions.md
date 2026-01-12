#Zig
---

## Summary
Zig's composite types—Structs, Enums, and Unions—are the building blocks for data modeling. Unlike many languages, Zig allows these types to have methods (functions) associated with them. The language also provides "Tagged Unions" for safe variant handling and specialized structs for memory layout control.

## Detailed Explanation

### Structs
Structs are the primary way to group data. Fields are named, and Zig gives no guarantee about the in-memory order of fields unless you use `extern` or `packed`.

```zig
const std = @import("std");

const Point = struct {
    x: f32,
    y: f32,

    // Method (function associated with the struct)
    pub fn distance(self: Point) f32 {
        return @sqrt(self.x * self.x + self.y * self.y);
    }
};

test "struct with methods" {
    const p = Point{ .x = 3.0, .y = 4.0 };
    try std.testing.expect(p.distance() == 5.0);
}
```

*   **Extern Structs**: `extern struct` follows the C ABI layout, ensuring compatibility with C code.
*   **Packed Structs**: `packed struct` has a bit-perfect layout, allowing you to define exact bit positions for fields (useful for hardware registers).

### Enums
Enums define a set of named constants. They can have an underlying integer type and even include methods.

```zig
const Status = enum(u2) {
    ok = 0,
    loading = 1,
    error = 2,

    pub fn isFinished(self: Status) bool {
        return self != .loading;
    }
};
```

### Unions and Tagged Unions
A `union` allows multiple fields to share the same memory location. Only one field can be active at a time.
A **Tagged Union** is a union that stores an invisible "tag" (enum) to keep track of which field is currently active. This is Zig's version of an algebraic data type (like Rust's `enum`).

```zig
const IntOrFloat = union(enum) {
    int: i32,
    float: f32,
};

test "tagged union" {
    var val = IntOrFloat{ .int = 10 };
    
    switch (val) {
        .int => |i| try std.testing.expect(i == 10),
        .float => |f| try std.testing.expect(f == 10.0),
    }
}
```

### Methods and Namespacing
Structs, enums, and unions in Zig act as namespaces. You can put any declaration inside them, including other structs or constant values.

## Interview Questions
*   **Q: Does Zig guarantee the order of fields in a normal `struct`?**
    *   **A:** No. The compiler is free to reorder fields to minimize padding and optimize size. To guarantee order (e.g., for C interop), you must use an `extern struct`.
*   **Q: What is a Tagged Union and why is it useful?**
    *   **A:** A Tagged Union is a union combined with an enum tag that tracks which field is active. It is useful because it provides type safety: the compiler ensures you can only access the field that matches the current tag (usually via a `switch` statement).
*   **Q: How do methods work in Zig since there is no `class` keyword?**
    *   **A:** Methods are just regular functions declared inside a struct, enum, or union. By convention, the first parameter is named `self`. If the first parameter is of the type of the container, you can call it using the dot syntax (`object.method()`).
*   **Q: When would you use a `packed struct`?**
    *   **A:** You use a `packed struct` when you need exact control over the bit-level layout of your data, such as when matching a specific hardware register format or a network protocol header where fields are not byte-aligned.
