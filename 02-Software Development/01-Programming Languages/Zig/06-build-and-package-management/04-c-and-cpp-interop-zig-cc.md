# C and C++ Interop via zig cc

## Summary
`zig cc` and `zig c++` are drop-in replacements for `clang` and `clang++`. They allow Zig to compile C/C++ projects, enabling cross-compilation and easy integration of C libraries into Zig projects via `build.zig`.

## Detailed Explanation

### `zig cc`
A wrapper around Zig's internal Clang frontend.
*   Supports standard flags (`-O2`, `-Wall`, `-I`).
*   Supports `-target` for cross-compilation.

### Build System Integration
In `build.zig`, you can add C sources to a Zig executable.

```zig
exe.addCSourceFile(.{
    .file = b.path("src/c_code.c"),
    .flags = &.{"-std=c99"},
});
exe.linkLibC();
```

### Go Comparison
*   **Go**: CGO. Notorious for being slow (overhead) and complicating the build (requires external GCC).
*   **Zig**: Native integration. Zig *is* a C compiler. Mixing languages is seamless and often statically linked.

## Interview Questions

**Q: What is the advantage of using `zig cc` over `gcc`?**
**A:** `zig cc` supports cross-compilation out of the box for dozens of targets without needing to install separate toolchains. It also has caching enabled by default.

**Q: Can Zig link against C++ libraries?**
**A:** Yes. `zig c++` can compile C++ code, and `build.zig` supports adding C++ source files. Zig code can interact with C++ via a C-compatible ABI (using `extern "C"` in C++).

**Q: Does `zig cc` use system headers?**
**A:** It can, but for many targets (like Linux/musl or Windows/mingw), it uses its own bundled headers to ensure reproducibility and cross-compilation support.
