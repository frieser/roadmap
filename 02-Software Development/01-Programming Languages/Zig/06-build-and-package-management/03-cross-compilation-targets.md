# Cross-compilation Targets

## Summary
Zig treats cross-compilation as a first-class feature. It can compile for almost any target without installing external toolchains. It uses "Target Triples" (arch-os-abi) and bundles necessary bits (libc headers, linker scripts) to make this seamless.

## Detailed Explanation

### Target Triples
`x86_64-linux-gnu`, `aarch64-macos-none`, `wasm32-freestanding`.

### Usage
*   CLI: `zig build -Dtarget=x86_64-windows`
*   Build Script: `b.standardTargetOptions` allows users to pass this flag.

### Features
*   **Bundled LibC**: Zig ships with source/headers for musl, mingw-w64, and glibc stubs.
*   **Dynamic Linking**: Can link against system libraries on other platforms if paths are provided.

### Go Comparison
*   **Go**: `GOOS=linux GOARCH=amd64`. Easy for pure Go, hard for CGO (requires cross-gcc).
*   **Zig**: Trivial for both Zig and C/C++. `zig cc` acts as a cross-compiler for C.

## Interview Questions

**Q: How does Zig compile for Windows from Linux without installing MinGW?**
**A:** Zig bundles the MinGW-w64 headers and libraries within its own binary. It uses its internal LLVM backend to generate the machine code and its own linker (`zld`) to produce the PE executable.

**Q: What is a "freestanding" target?**
**A:** A target without an operating system (OS=freestanding). This is used for kernel development, bootloaders, or bare-metal embedded programming (e.g., `thumbv7em-freestanding-eabihf`).

**Q: Why is `zig cc` useful for non-Zig projects?**
**A:** It allows C/C++ projects to leverage Zig's cross-compilation capabilities. You can set `CC="zig cc -target ..."` in a Makefile to easily cross-compile a C project.
