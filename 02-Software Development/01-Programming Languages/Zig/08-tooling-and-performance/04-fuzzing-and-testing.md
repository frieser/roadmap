# Fuzzing and Testing

## Summary
Testing is a first-class citizen in Zig. The language provides `test` blocks that live directly alongside code, a built-in test runner (`zig test`), and the `std.testing` namespace for assertions. Beyond unit testing, Zig is increasingly integrating **fuzzing** capabilities to find edge-case bugs by providing random inputs to functions.

## Detailed Explanation

### The `test` Block
In Zig, you don't need a separate directory for tests. You can write them in the same file as your logic.
*   **Visibility**: Tests have access to private members of the module they are in.
*   **Execution**: Only code inside `test` blocks is compiled when running `zig test`.

### `std.testing` Assertions
The standard library provides various helpers:
*   `expect(bool)`: Basic boolean check.
*   `expectEqual(expected, actual)`: Checks for equality.
*   `expectError(err, actual)`: Verifies that a function returns a specific error.

### Memory Leak Detection
One of Zig's most powerful testing features is the `std.testing.allocator`.
*   **Automatic Checks**: If you use this allocator in your tests, it will automatically fail the test if any memory is not properly freed (deallocated).
*   **Usage**: Pass it to any function that requires an `Allocator`.

### Fuzzing in Zig
Fuzzing involves passing random data to a function to see if it crashes or returns an error.
*   **Native Fuzzing**: Zig is working on integrating fuzzing directly into the toolchain.
*   **LibFuzzer**: Zig can easily interface with `libFuzzer` via its C interoperability.
*   **Strategy**: Create a test that takes a slice of bytes and passes it to your parsing logic.

## Zig Code Examples

### Basic Test Case
```zig
const std = @import("std");
const expect = std.testing.expect;

fn add(a: i32, b: i32) i32 {
    return a + b;
}

test "basic addition" {
    try expect(add(2, 2) == 4);
}
```

### Testing with Memory Leak Detection
```zig
const std = @import("std");

test "detect memory leak" {
    const allocator = std.testing.allocator;
    
    const list = try allocator.create(i32);
    // Oops! forgot to deinit/destroy
    // _ = list; 
    
    // Correct way:
    defer allocator.destroy(list);
}
```

### Fuzz-like Test Pattern
```zig
const std = @import("std");

fn parseData(data: []const u8) !void {
    if (data.len > 0 and data[0] == 'Z') {
        // ... logic ...
    }
}

test "fuzz parseData" {
    var prng = std.rand.DefaultPrng.init(0);
    const random = prng.random();
    
    var buffer: [100]u8 = undefined;
    for (0..10) |_| {
        random.bytes(&buffer);
        _ = parseData(&buffer) catch {};
    }
}
```

## Interview Questions

**Q: Where do you typically write tests in a Zig project?**
**A:** Tests are usually written directly in the `.zig` source files using `test` blocks, allowing them to live close to the code they verify and access private members.

**Q: How does the `std.testing.allocator` help in writing robust software?**
**A:** It tracks all allocations and deallocations. If a test finishes and there is still allocated memory, the allocator triggers a failure and prints a leak report, ensuring memory management is correct.

**Q: What is the command to run tests in a Zig file?**
**A:** `zig test filename.zig`.

**Q: How would you implement fuzz testing for a Zig function?**
**A:** You can create a test block that uses a random number generator (like `std.rand.DefaultPrng`) to generate arbitrary byte sequences and pass them to the target function, ensuring it handles all inputs without crashing (panicking).
