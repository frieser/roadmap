# Installation and Toolchain

## Summary
Zig is distributed as a single, static binary that includes the compiler, build system, and C/C++ cross-compiler. Installing it involves downloading this binary and adding it to your PATH. The toolchain is self-contained, requiring no external system dependencies.

## Detailed Explanation
The Zig toolchain is famous for its "zero-dependency" distribution. The entire language standard library, compiler, and build system are packaged into one executable.

### Installation Methods
1.  **Direct Download (Recommended)**:
    *   Download the tarball/zip from [ziglang.org/download](https://ziglang.org/download/).
    *   Extract it.
    *   Add the folder to your system `PATH`.
2.  **Package Managers**:
    *   **macOS**: `brew install zig`
    *   **Windows**: `winget install zig.zig`
    *   **Linux**: `snap install zig --classic` (though downloading the binary is often preferred for version control).

### The `zig` Command
*   `zig version`: Check installed version.
*   `zig run main.zig`: Compiles and runs a single file immediately.
*   `zig build-exe main.zig`: Compiles to a binary.
*   `zig build`: Runs the build steps defined in `build.zig`.
*   `zig test main.zig`: Runs the test block within the file.
*   `zig cc`: Acts as a C compiler (drop-in replacement for clang/gcc).
*   `zig c++`: Acts as a C++ compiler.

### Zig Language Server (ZLS)
ZLS is the official Language Server Protocol (LSP) implementation for Zig. It provides:
*   Autocompletion
*   Go-to-definition
*   Hover information
*   Formatting (integrated with `zig fmt`)

It is a separate binary that should be installed and configured in your editor (VS Code, Neovim, etc.) for the best development experience.

### Go Comparison
Like Go (`go run`, `go build`, `go test`), Zig provides a unified CLI tool. However, Zig's CLI also includes a full C/C++ cross-compiler toolchain, which Go does not.

```bash
# Compile a C file using Zig!
zig cc main.c -o myapp
```

## Interview Questions

**Q: What makes the Zig compiler toolchain unique compared to GCC or Clang?**
**A:** The Zig compiler (`zig`) is a self-contained binary that includes the Zig compiler, the Zig standard library, *and* a full Clang/LLVM toolchain for compiling C/C++. It supports cross-compilation to any supported target out of the box without installing separate cross-compilers.

**Q: How do you run tests in a Zig project?**
**A:** You can use `zig test filename.zig` to run tests defined in a `test` block within that specific file. For larger projects, `zig build test` is typically configured in `build.zig` to run the entire test suite.

**Q: What is `zig cc`?**
**A:** `zig cc` is a command that invokes the internal Clang compiler bundled with Zig. It allows you to use Zig as a C compiler, leveraging Zig's seamless cross-compilation capabilities for C projects.
