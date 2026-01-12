# Input/Output (IO)

## Summary
Zig's IO system is built around the `Reader` and `Writer` generic interfaces, which provide a consistent API for any backing source (files, sockets, memory). Buffered IO (`std.io.BufferedReader`, `std.io.BufferedWriter`) is critical for performance to minimize syscalls.

## Detailed Explanation

### Core Concepts
*   **`std.io.Reader` / `std.io.Writer`**: Wrappers that provide helper methods like `readInt`, `readUntilDelimiter`, `print`, and `writeInt`.
*   **Buffered IO**: Use `std.io.bufferedWriter(stream)` to buffer writes.
*   **File IO**: Handled via `std.fs`. `std.fs.cwd()` gives access to the current working directory.

### Code Example
```zig
const std = @import("std");

pub fn main() !void {
    const file = try std.fs.cwd().createFile("hello.txt", .{});
    defer file.close();

    var buf_writer = std.io.bufferedWriter(file.writer());
    const writer = buf_writer.writer();

    try writer.print("Hello, {s}!\n", .{"Zig"});
    try buf_writer.flush(); // Crucial!
}
```

### Go Comparison
*   **Go**: `io.Reader` and `io.Writer` are interfaces. `bufio` provides buffering.
*   **Zig**: Readers/Writers are generic structs using duck typing.

## Interview Questions

**Q: How do Readers and Writers work in Zig without traditional interfaces?**
**A:** They use "duck typing" via generic wrapper structs. A Reader is a struct that contains a `context` and a `readFn`. Any type that implements a compliant `read` function can be wrapped.

**Q: Why is `flush()` necessary for buffered writers?**
**A:** Buffered writers store data in an internal memory buffer to reduce syscalls. `flush()` forces the write of the remaining buffered data to the underlying OS file descriptor. Forgetting to flush may result in truncated output.

**Q: What is the difference between `std.debug.print` and `std.io.getStdOut`?**
**A:** `std.debug.print` writes to stderr, may be protected by a mutex, and is intended for debugging. `std.io.getStdOut` is for actual program output and should be used in production code.
