# Generic Data Structures

## Summary
Zig implements generics using functions that accept `type` parameters (known at `comptime`) and return a new `type`. There is no special `<T>` syntax; types are just first-class values during compilation.

## Detailed Explanation

### Generic Struct Pattern
To create a generic `List<T>`, you write a function that takes `T` and returns a struct.

```zig
fn List(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,

        pub fn init(allocator: std.mem.Allocator) !@This() {
             // ...
        }
    };
}

// Usage
const IntList = List(i32);
var list = IntList.init(allocator);
```

### `@This()`
A builtin that returns the type of the struct currently being defined. Useful inside generic templates.

### Go Comparison
*   **Go**: Uses `[T any]` syntax (since 1.18).
*   **Zig**: Uses standard function syntax. `fn List(comptime T: type) type`. This is more flexible, as you can pass values (e.g., `Matrix(4, 4)`) just as easily as types.

## Interview Questions

**Q: How does Zig implement generics without a generic syntax like `<T>`?**
**A:** Zig treats types as values that can be passed to functions at compile time. A "generic" is simply a function that takes a `type` as an argument and returns a new `type` (usually a struct defined inside the function).

**Q: What is implicit about Zig generics vs C++ templates?**
**A:** C++ templates are a separate sub-language. Zig generics use standard Zig imperative code. You can use `if`, `switch`, and loops inside the function generating the type to customize the result.

**Q: Can you constrain a generic type in Zig (like Interfaces/Traits)?**
**A:** Not with formal syntax. Instead, you check properties of the type using `@typeInfo(T)` inside the function and issue `@compileError` if the type doesn't satisfy requirements (Duck Typing at compile time).
