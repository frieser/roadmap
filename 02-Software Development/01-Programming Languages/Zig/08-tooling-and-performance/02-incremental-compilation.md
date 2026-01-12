# Incremental Compilation

## Summary
Incremental compilation is a core pillar of the Zig compiler's architecture, aimed at providing near-instant feedback loops. By using a sophisticated caching system (`ZIG_CACHE_DIR`) and a "binary patching" approach, Zig avoids re-compiling entire modules, instead updating only the parts of the executable that have changed.

## Detailed Explanation

### The ZIG_CACHE_DIR
Zig relies heavily on its global and local cache to avoid redundant work.
*   **Global Cache**: Usually located in `~/.cache/zig` (Linux) or `%LOCALAPPDATA%/zig` (Windows). It stores compiled dependencies and shared artifacts.
*   **Local Cache**: Found in `.zig-cache` within your project. It stores artifacts specific to the current build.
*   **Invalidation**: Zig uses content hashing of source files and compiler flags to determine if a cache entry is valid.

### Compiler Architecture for Speed
The Zig compiler is being rewritten to be "self-hosted" (written in Zig). This architecture supports incrementalism from the ground up:
*   **AST-based Incrementalism**: The compiler tracks dependencies at a granular level. If you change a function body, the compiler knows exactly which parts of the binary need to be updated.
*   **In-Place Binary Patching**: Instead of relinking the entire binary (which is slow), the Zig linker can "patch" the existing executable file on disk. This is significantly faster than traditional `LLVM -> Linker` pipelines.

### Incremental Compilation Flag
As of Zig 0.13+, incremental compilation is becoming the default for debug builds but can be explicitly controlled.
*   **Command**: `zig build -fincremental`
*   **Performance**: In 2025, Zig's x86 backend coupled with incremental compilation allows for sub-100ms rebuilds on medium-sized projects, a massive improvement over LLVM-based builds.

## Zig Code Examples

### Controlling Cache via CLI
```bash
# Force a clean build by ignoring the cache
zig build --clear-cache

# Specify a custom cache directory
zig build --cache-dir ./custom-cache
```

### Incremental-friendly code
Zig's `comptime` can sometimes impact incremental speeds if used excessively for global state, but for the most part, standard Zig code benefits automatically.
```zig
const std = @import("std");

// Changing this function body only requires patching 
// a small section of the binary in incremental mode.
pub fn fastUpdate(x: i32) i32 {
    return x * 2; 
}

pub fn main() void {
    std.debug.print("Result: {d}\n", .{fastUpdate(10)});
}
```

## Interview Questions

**Q: What is the purpose of `ZIG_CACHE_DIR`?**
**A:** It stores previously compiled artifacts (object files, dependencies) to speed up subsequent builds by reusing results for unchanged code.

**Q: How does Zig's incremental compilation differ from traditional "incremental linking"?**
**A:** While traditional linkers often re-examine all object files to produce a new binary, Zig's incremental linker aims to "patch" the existing binary in-place, updating only the modified machine code and data.

**Q: Why is the self-hosted backend important for incremental compilation?**
**A:** LLVM is a "batch" compiler that is difficult to use incrementally at a granular level. By writing its own backends (x86, ARM), the Zig team can implement fine-grained binary patching that LLVM doesn't support.

**Q: When would you want to disable incremental compilation?**
**A:** For final release builds (`ReleaseFast`, `ReleaseSmall`), you typically want a full optimization pass by LLVM, where incremental patching is not applicable as it might prevent global optimizations like LTO (Link Time Optimization).
