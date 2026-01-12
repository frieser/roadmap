# Building Executables

## Summary
Go compiles to a single, static binary by default, making deployment incredibly simple (just copy the file). However, you can optimize the build process to produce smaller binaries, inject metadata (versioning), or control the linking mode (static vs dynamic) using build flags.

## Detailed Explanation

### 1. Reducing Binary Size
Go binaries include debugging information (DWARF tables) by default, which can be large. You can strip this using linker flags (`-ldflags`).

```bash
# -s: Omit the symbol table and debug information
# -w: Omit the DWARF symbol table
go build -ldflags="-s -w" -o app
```
This typically reduces binary size by 25-30%.

### 2. Static vs Dynamic Linking
By default, Go uses **static linking** for Go code but might use **dynamic linking** for C libraries (libc) if using `net` or `os/user` packages on Linux.
*   **Pure Static**: To create a truly portable binary (e.g., for `FROM scratch` Docker images), disable CGO.
    ```bash
    CGO_ENABLED=0 go build -o app
    ```

### 3. Injecting Metadata
You can set the value of string variables at build time using `-X`.
```go
// main.go
package main
var Version = "dev"
```
```bash
go build -ldflags="-X main.Version=1.0.0"
```

## Interview Questions

**Q: Why might a Go binary fail to run in a `scratch` Docker container?**
**A:** If the binary was compiled with `CGO_ENABLED=1` (default on standard Go installs), it might depend on the system's `glibc` shared library. A `scratch` image is empty and has no `glibc`. To fix this, build with `CGO_ENABLED=0` to force a pure static binary that requires no external libraries.

**Q: What does `go build -trimpath` do?**
**A:** By default, the full file path of your source code is embedded in the binary for panic stack traces (e.g., `/Users/username/go/src/...`). `-trimpath` replaces these absolute paths with module paths (e.g., `github.com/user/project/...`). This makes builds **reproducible** (independent of the builder's directory structure) and improves security/privacy.

**Q: How does `upx` relate to Go binaries?**
**A:** `upx` is an external tool (packer) that can compress executables. You can run `upx --best my-go-binary` to shrink it drastically (e.g., from 10MB to 3MB). The binary decompresses itself in memory at runtime. It trades startup time (decompression) for disk usage.
