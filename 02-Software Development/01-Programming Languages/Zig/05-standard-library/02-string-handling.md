# String Handling

## Summary
In Zig, strings are simply null-terminated arrays or slices of bytes (`[]const u8`). Zig assumes UTF-8 by convention but operates on bytes by default. `std.mem` provides manipulation functions, and `std.fmt` handles formatting.

## Detailed Explanation

### Slices and Arrays
*   **String Literal**: `"hello"` is a `*const [5:0]u8` (pointer to null-terminated array).
*   **String Slice**: `[]const u8` is the standard type for passing strings.

### Standard Library
*   **`std.mem`**: `eql`, `indexOf`, `split`, `trim`, `replace`.
*   **`std.fmt`**: `allocPrint` (allocate new string), `bufPrint` (write to buffer).
*   **`std.ArrayList(u8)`**: Used as a StringBuilder.

```zig
var builder = std.ArrayList(u8).init(allocator);
try builder.appendSlice("Hello");
const str = builder.items;
```

### Unicode
Use `std.unicode.Utf8View` to iterate over code points instead of bytes.

### Go Comparison
*   **Go**: `string` is a read-only slice of bytes. Range loop iterates runes (Unicode).
*   **Zig**: `[]const u8` is the string type. `for` loop iterates bytes. You must explicitly use `Utf8View` for Unicode iteration.

## Interview Questions

**Q: Does Zig have a dedicated `string` type?**
**A:** No. Zig uses `[]const u8` (slice of immutable bytes) for strings. This reflects Zig's low-level nature, treating strings as just memory.

**Q: How do you handle Unicode/UTF-8 in Zig?**
**A:** Zig strings are UTF-8 by convention. To iterate over characters (code points) instead of bytes, you must use `std.unicode.Utf8View`.

**Q: What is the difference between `++` and `**` operators?**
**A:** These are **comptime-only** operators. `++` concatenates two arrays/strings, and `**` repeats an array/string. They create new compile-time constants and cannot be used on runtime slices.
