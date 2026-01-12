# Collections: ArrayList and HashMap

## Summary
Zig provides manually-managed collections in `std`. `ArrayList` is a dynamic array, and `HashMap` is a key-value store. Both require an explicit allocator upon initialization and must be deinitialized to prevent leaks.

## Detailed Explanation

### ArrayList
Growable array.
*   `init(allocator)`
*   `append(item)`
*   `items` (slice access)
*   `toOwnedSlice()` (give ownership to caller)

### HashMap
*   **`std.AutoHashMap(K, V)`**: Default hashing.
*   **`std.StringHashMap(V)`**: Specialized for string keys.
*   **Unmanaged**: `ArrayListUnmanaged` stores state without the allocator (smaller size), requiring the allocator to be passed to every method.

### Code Example
```zig
var map = std.StringHashMap(i32).init(allocator);
defer map.deinit();

try map.put("apple", 5);
if (map.get("apple")) |v| {
    // ...
}
```

### Go Comparison
*   **Go**: `slice` (append) and `map` (make). Built-in and GC-managed.
*   **Zig**: Library types (`std.ArrayList`, `std.HashMap`). Manually managed.

## Interview Questions

**Q: What is the difference between `AutoHashMap` and `StringHashMap`?**
**A:** `AutoHashMap` hashes keys by value (or pointer address if the key is a pointer). For strings (`[]const u8`), `AutoHashMap` would hash the pointer and length, effectively checking for identity equality. `StringHashMap` explicitly hashes the *content* of the string.

**Q: When would you use `Unmanaged` collections (e.g., `ArrayListUnmanaged`)?**
**A:** When you need to minimize the memory footprint of the struct (it saves the size of the allocator pointer) or when the allocator is implicitly known/managed by the context (like inside a compiler pass using a global arena).

**Q: How do you iterate over a HashMap in Zig?**
**A:** You use `var it = map.iterator();` and `while (it.next()) |entry| { ... }`.
