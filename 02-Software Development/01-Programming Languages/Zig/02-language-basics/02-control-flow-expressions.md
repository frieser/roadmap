#Zig
---

## Summary
Control flow in Zig is expression-based, meaning structures like `if` and `switch` return values. Zig avoids "hidden" control flow (like exceptions) and instead uses explicit patterns for error handling and optional values through "payload capture".

## Detailed Explanation

### If Expressions
The `if` statement can be used as an expression. The `else` clause is mandatory when using `if` as an expression.

```zig
const std = @import("std");

test "if expression" {
    const score = 85;
    const result = if (score >= 90) "A" else "B";
    try std.testing.expect(std.mem.eql(u8, result, "B"));
}
```

### While Loops
`while` loops in Zig are flexible. They can have a "continue expression" which is executed after the loop body but before the next condition check.

```zig
test "while with continue expression" {
    var i: u32 = 0;
    var sum: u32 = 0;
    while (i < 3) : (i += 1) { // i += 1 is the continue expression
        sum += i;
    }
    try std.testing.expect(sum == 3); // 0 + 1 + 2
}
```

### For Loops
`for` loops iterate over arrays or slices. They can capture multiple items at once and also support the `0..` syntax to capture the current index.

```zig
test "multi-item for loop" {
    const a = [_]u8{ 1, 2, 3 };
    const b = [_]u8{ 4, 5, 6 };
    var sum: u8 = 0;

    for (a, b, 0..) |item_a, item_b, index| {
        sum += item_a + item_b + @as(u8, @intCast(index));
    }
    // (1+4+0) + (2+5+1) + (3+6+2) = 5 + 8 + 11 = 24
    try std.testing.expect(sum == 24);
}
```

### Switch Expressions
`switch` is exhaustive in Zig; all possible cases must be covered (or an `else` provided). It can also capture payloads from tagged unions.

```zig
test "switch expression" {
    const x: u8 = 10;
    const result = switch (x) {
        0...9 => "single digit",
        10...99 => "double digit",
        else => "many digits",
    };
    try std.testing.expect(std.mem.eql(u8, result, "double digit"));
}
```

### Payload Capture
Zig uses the `|payload|` syntax to capture values from optionals, error unions, and loops.

```zig
test "payload capture" {
    const maybe_val: ?u32 = 42;
    if (maybe_val) |val| {
        try std.testing.expect(val == 42);
    } else {
        // handle null
    }
}
```

### Labels and Loop Break
Loops and blocks can be labeled, allowing `break` and `continue` to target specific contexts. Blocks can also return values using labels.

```zig
test "labeled block expression" {
    const value = blk: {
        const x = 5;
        const y = 10;
        break :blk x + y;
    };
    try std.testing.expect(value == 15);
}
```

## Interview Questions
*   **Q: What does it mean that `if` is an expression in Zig?**
    *   **A:** It means `if` can return a value, similar to the ternary operator (`?:`) in C/C++, but more readable and powerful. Example: `const x = if (cond) a else b;`.
*   **Q: How do you capture the index in a `for` loop?**
    *   **A:** By using the `0..` range syntax alongside the collection. Example: `for (items, 0..) |item, index| { ... }`.
*   **Q: What is a "continue expression" in a `while` loop?**
    *   **A:** It is a piece of code (following the colon) that executes at the end of every loop iteration, right before the condition is re-evaluated. It is commonly used for incrementing counters.
*   **Q: Why is `switch` exhaustive in Zig?**
    *   **A:** Exhaustiveness ensures that you've considered every possible state (especially for enums and unions), preventing bugs where a new state is added but not handled in all logic paths.
