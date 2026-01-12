# Go Run

## Summary
The `go run` command is a convenience tool that compiles and executes a Go program in one step. It is primarily used for development, scripts, and quick testing. It compiles the code into a temporary directory, runs the binary, and then automatically cleans up the temporary files after execution finishes.

## Detailed Explanation

### How it works
1.  **Compile**: `go run` locates the `main` package in the specified file(s) or directory. It compiles them and their dependencies into a temporary executable in the OS's temp folder.
2.  **Execute**: It runs the resulting binary immediately.
3.  **Cleanup**: Once the program exits, the temporary binary is deleted.

### Usage
```bash
# Run a single file
go run main.go

# Run the package in the current directory
go run .

# Pass arguments to the program (use -- to separate go flags from app flags)
go run main.go --port 8080
```

## Interview Questions

**Q: Should you use `go run` in production?**
**A:** No. `go run` is for development. In production, you should compile the binary using `go build` and distribute the single executable. `go run` requires the Go toolchain to be installed on the server (which increases attack surface) and recompiles the code every time it starts, increasing startup time.

**Q: Where does `go run` store the executable?**
**A:** It stores it in a temporary directory (usually `/tmp/go-build...` on Linux/macOS). You can see the location by running `go run -work main.go`, which preserves the directory and prints its path.

**Q: Can `go run` execute multiple files?**
**A:** Yes. If your `main` package is split across multiple files (e.g., `main.go`, `helpers.go`), you must list them all: `go run main.go helpers.go`. Alternatively, using `go run .` automatically includes all files in the current package.
