# Hello World Anatomy

## Summary
A basic Zig program consists of importing the standard library, defining a public `main` function, and using a writer to print output. It demonstrates Zig's explicit error handling and format string syntax.

## Detailed Explanation
Let's break down the structure of a minimal Zig program.

### Code Example

```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, {s}!\n", .{"World"});
}
```

### Breakdown
1.  **`const std = @import("std");`**:
    *   Imports the Standard Library.
    *   `@` indicates a compiler builtin function.
    *   Assigns the library namespace to a constant struct `std`.

2.  **`pub fn main() !void { ... }`**:
    *   `pub`: Makes the function public/visible to the build system/linker.
    *   `fn`: Function declaration.
    *   `!void`: The return type. This is an **Error Union**. It means the function returns either `void` (success) or an `error`. This is required because `stdout.print` can fail.

3.  **`const stdout = std.io.getStdOut().writer();`**:
    *   Gets a handle to standard output and wraps it in a generic `Writer` interface.

4.  **`try stdout.print(...)`**:
    *   `stdout.print` performs formatted I/O.
    *   `try`: This keyword checks the return value. If it's an error, it returns the error from `main` immediately. If it's success, it unwraps the value (if any) and continues.

5.  **Format String `Hello, {s}!\n`**:
    *   `{s}`: Format specifier for a string slice. Zig is strict; you must match the specifier to the type.
    *   `.{"World"}`: The arguments are passed as an anonymous struct tuple.

### Go Comparison
In Go:
```go
package main
import "fmt"
func main() {
    fmt.Printf("Hello, %s!\n", "World")
}
```
*   Go's `fmt.Printf` ignores errors by default (returns them, but you can ignore variables).
*   Zig requires you to handle the error with `try`, or the compiler will complain that the return value is unused.

## Interview Questions

**Q: What does the `!void` return type signify in the `main` function?**
**A:** It indicates that the function returns an **Error Union**. It can either return `void` (meaning it completed successfully) or an `error`. This allows the `main` function to propagate errors (like I/O failures) back to the runtime system.

**Q: Why do arguments to `print` need to be wrapped in `.{}`, like `.{"World"}`?**
**A:** The `print` function expects a tuple (an anonymous struct) containing the arguments. This allows Zig to validate types against the format string at compile time (mostly) or runtime without variadic functions (which Zig does not support in the traditional C sense).

**Q: What is the purpose of the `try` keyword?**
**A:** `try x` is syntax sugar for `x catch |err| return err`. It evaluates an expression that returns an error union. If it's an error, it immediately returns that error from the current function. If it's a success value, that value is unwrapped and returned to the expression.
