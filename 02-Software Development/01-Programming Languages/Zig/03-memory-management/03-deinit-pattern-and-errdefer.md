# Deinit Pattern and Errdefer

## Summary
Zig uses the `defer` keyword to schedule cleanup code to run when a block exits. `errdefer` is a special variant that only runs if the block exits with an **error**. This pair is essential for robust resource management and preventing leaks during complex initialization sequences.

## Detailed Explanation

### The `deinit` Convention
Structs that manage resources (memory, file handles) usually have a public `deinit()` method.
```zig
const List = std.ArrayList(i32);
var list = List.init(allocator);
defer list.deinit(); // Ensure cleanup
```

### `defer`
Runs unconditionally when the scope ends (reverse order of declaration).
```zig
const file = try openFile();
defer closeFile(file); // Runs at end of function
```

### `errdefer`
Runs **only** if the function returns an error. This is critical for "transactional" initialization: if step 3 fails, undo steps 1 and 2, but if step 3 succeeds, keep 1 and 2 alive.

```zig
fn createThing() !*Thing {
    const t = try allocator.create(Thing);
    errdefer allocator.destroy(t); // Free 't' ONLY if subsequent steps fail

    t.buffer = try allocator.alloc(u8, 100);
    // If alloc fails, errdefer runs -> destroy(t) -> clean return
    
    return t; // Success! errdefer is skipped.
}
```

### Go Comparison
*   **Go**: Has `defer`, which runs at function exit.
*   **Zig**: Has `defer` (scope exit) and `errdefer` (error exit). Go lacks `errdefer`, forcing explicit `if err != nil { cleanup() }` blocks.

## Interview Questions

**Q: When does a `defer` statement execute in Zig?**
**A:** It executes when the current **block** (scope) exits, not necessarily the function. This allows for granular resource management inside loops or `if` blocks.

**Q: What problem does `errdefer` solve?**
**A:** It solves the "partial initialization" problem. When constructing a complex object, if a later step fails, you need to clean up the resources allocated in earlier steps. `errdefer` automates this "undo" logic specifically for error paths.

**Q: Can you have multiple `defer` statements in a single block?**
**A:** Yes. They execute in **Last-In, First-Out (LIFO)** order (reverse order of declaration).
