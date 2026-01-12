# Compiler and Linker Flags

## Summary
Go allows extensive control over the build process via flags. You can inject variables at build time, reduce binary size, enable race detection, or enforce reproducible builds without changing the source code.

## Detailed Explanation

### Common Flags
*   **`-race`**: Enables the data race detector. Adds runtime overhead but is essential for debugging concurrency.
*   **`-trimpath`**: Removes absolute file system paths from the binary (useful for security and reproducibility).

### Linker Flags (`-ldflags`)
Passed to the linker to modify the final binary.
*   **`-s`**: Omit the symbol table and debug information.
*   **`-w`**: Omit the DWARF symbol table.
*   **Result**: Combining `-s -w` can reduce binary size by 20-30%.

### Variable Injection (`-X`)
You can set the value of a string variable in the main package (or any package) at build time. This is standard for versioning.

**Pattern**: `-ldflags "-X package.variable=value"`

### Code Example: Version Injection

**File: `main.go`**
```go
package main

import "fmt"

var (
    Version = "dev"
    Commit  = "none"
)

func main() {
    fmt.Printf("Version: %s, Commit: %s\n", Version, Commit)
}
```

**Build Command**:
```bash
go build -ldflags "-X main.Version=1.0.0 -X main.Commit=$(git rev-parse --short HEAD) -s -w" -o myapp
```

**Output**:
```text
Version: 1.0.0, Commit: a1b2c3d
```

## Interview Questions

**Q: How do you reduce the size of a Go binary?**
**A:** Use `-ldflags "-s -w"` to strip debug information and symbol tables. You can also use `upx` to compress it further.

**Q: How do you inject a git commit hash into the binary?**
**A:** Use the `-X` linker flag: `-ldflags "-X main.Commit=$(git rev-parse HEAD)"`.

**Q: What does `-race` do?**
**A:** It instruments the code to detect data races at runtime. It increases CPU and memory usage, so it's typically used in testing or staging, not production.
