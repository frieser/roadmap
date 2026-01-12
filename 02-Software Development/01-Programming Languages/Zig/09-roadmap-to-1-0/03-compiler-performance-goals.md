# Compiler Performance Goals

## Summary
"The compiler is too damn slow, that's why we have bugs." — Andrew Kelley.
Zig 1.0 aims for best-in-class compiler performance. This is achieved through the self-hosted compiler ("Stage 3"), native backends for debug builds (bypassing LLVM), and a radical incremental compilation architecture that patches binaries in-place.

## Detailed Explanation

### Self-Hosted Compiler (Stage 3)
The transition from a C++ compiler to one written entirely in Zig (Stage 3) is complete. This allows the compiler to use Zig's own features (like efficient allocators and compile-time logic) to optimize itself.

### Incremental Compilation & "Yeeting" LLVM
*   **Native Backends**: For Debug builds, Zig avoids LLVM entirely. It uses its own native x86, ARM, and WASM backends. This eliminates the massive overhead of LLVM's optimization passes during development.
*   **Binary Patching**: Zig's incremental compiler doesn't just recompile files; it patches the existing binary in-place. If you change a single function, the compiler only updates that specific slice of the executable.
*   **Speed Goals**: The target is **near-instant** feedback (sub-100ms) for projects of any size.

### Memory Usage Goals
*   **Low Overhead**: Zig aims to keep compiler memory usage proportional to the *change* in code, not the *total size* of the code, thanks to its incremental data structures.

### Current Status (2026)
In version 0.15.1 and beyond, developers report that Zig is significantly faster than Rust and Go for cold builds, and "near-instant" for incremental debug builds.

## Interview Questions

**Q: Why does Zig use its own backends instead of LLVM for debug builds?**
**A:** LLVM is designed for high-quality code optimization, which makes it computationally expensive and slow. Zig's native backends prioritize compilation speed above all else, enabling a "save-and-run" feedback loop that feels instant, which is crucial for developer productivity.

**Q: How does Zig's incremental compilation differ from traditional "separate compilation"?**
**A:** Traditional separate compilation compiles `.c` files to `.o` files and then links them. Linking is often a bottleneck. Zig's incremental compiler skips the linking step by "patching" the final binary directly at the function level, treating the executable as a mutable database of code.

**Q: What is the "Stage 3" compiler?**
**A:** It refers to the Zig compiler written in Zig, compiled by itself. It represents the final architecture of the compiler, replacing the original C++ bootstrap compiler (Stage 1) and the transitional version (Stage 2).
