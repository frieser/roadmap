# Debugging with GDB and LLDB

## Summary
Zig provides first-class support for debugging using standard tools like **GDB** and **LLDB**. By emitting standard **DWARF** debug information, Zig ensures that variables, types, and stack traces are visible. It particularly shines in **mixed C/Zig projects**, where the same debugger can seamlessly step between both languages due to Zig's compatible calling conventions and debug info format.

## Detailed Explanation

### Debug Information (DWARF)
When you compile Zig code in `Debug` or `ReleaseSafe` modes, or by passing the `-g` flag, the compiler generates DWARF info.
*   **Location**: Debug info is usually embedded in the binary or stored in separate `.dSYM` files (on macOS).
*   **Scope**: Zig maps its complex types (slices, optionals, error unions) to DWARF structures that standard debuggers can interpret, although some high-level abstractions may require custom visualizers.

### LLDB Support
LLDB is the preferred debugger for many Zig developers due to its integration with the LLVM toolchain.
*   **Command**: `lldb ./your-binary`
*   **Visualizers**: To make Zig slices (which are `struct { ptr: [*]T, len: usize }`) and optionals readable, developers often use Python-based type formatters. As of 2025, LLDB has improved its native handling of Zig-style structures.
*   **Example**: Stepping into a Zig function from C code works out of the box because Zig uses the C ABI by default for exported functions.

### Mixed C/Zig Debugging
One of Zig's "killer features" is its role as a C compiler (`zig cc`).
*   **Unified Context**: If you use `zig cc` to compile C and `zig build-exe` for Zig, the resulting DWARF information is unified.
*   **Seamless Transitions**: You can set a breakpoint in a C file, hit it, and then step into a Zig function that was called from C. The debugger maintains the full call stack across language boundaries.

### Debugging Features in Zig
*   **Built-in Trace**: When a program crashes in Debug mode, Zig provides a formatted stack trace with file names and line numbers automatically.
*   **@breakpoint()**: A built-in function that triggers a debugger break programmatically.

## Zig Code Examples

### Triggering a Breakpoint
```zig
const std = @import("std");

pub fn main() !void {
    const x: i32 = 42;
    if (x == 42) {
        // Triggers the debugger if attached
        @breakpoint();
    }
    std.debug.print("Value: {d}\n", .{x});
}
```

### Mixed C/Zig (C side)
```c
// main.c
extern int add_in_zig(int a, int b);

int main() {
    int result = add_in_zig(10, 20);
    return 0;
}
```

### Mixed C/Zig (Zig side)
```zig
// math.zig
export fn add_in_zig(a: i32, b: i32) i32 {
    return a + b; // Set breakpoint here in GDB/LLDB
}
```

## Interview Questions

**Q: How do you enable debug information in a Zig build?**
**A:** Use the `Debug` build mode (default) or pass the `-g` flag to `zig build-exe`, `zig build-lib`, or `zig run`.

**Q: What is the significance of DWARF in the context of Zig?**
**A:** DWARF is the standard debugging data format used by Zig. It allows external debuggers like GDB and LLDB to map machine code back to Zig source code, inspect variables, and reconstruct the call stack.

**Q: Can you debug a Zig program that calls C code using a single debugger session?**
**A:** Yes. Since Zig and C both use standard DWARF info and compatible ABIs, tools like GDB/LLDB can transition between Zig and C frames seamlessly in the same execution context.

**Q: What does `@breakpoint()` do?**
**A:** It is a compiler built-in that inserts a hardware breakpoint instruction into the binary, causing a debugger (if attached) to pause execution at that exact line.
