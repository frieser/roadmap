# Go Build

## Summary
`go build` is the command used to compile Go source code into a standalone binary executable. It compiles the specified package and all its dependencies. By default, it names the binary after the directory or the main file and places it in the current working directory.

## Detailed Explanation

### Basic Usage
```bash
# Build current directory
go build .

# Build specific package
go build ./cmd/server

# Output to specific name/location
go build -o bin/app ./cmd/main.go
```

### Cross-Compilation
Go makes cross-compilation trivial. You control the target OS and Architecture using environment variables.
```bash
# Build for Linux AMD64 on a Mac
GOOS=linux GOARCH=amd64 go build -o app-linux .

# Build for Windows
GOOS=windows GOARCH=amd64 go build -o app.exe .
```

### Build Flags
*   `-o name`: Output filename.
*   `-v`: Print the names of packages as they are compiled.
*   `-race`: Enable the Data Race Detector (adds overhead, use for testing/debugging).
*   `-ldflags`: Pass flags to the linker (often used to inject version numbers).
    ```bash
    go build -ldflags "-X main.version=1.0.0"
    ```

## Interview Questions

**Q: What is the difference between `go build` and `go install`?**
**A:** `go build` compiles the program and leaves the binary in the **current directory** (or wherever `-o` points). `go install` compiles the program and moves the binary to `$GOPATH/bin` (or `$GOBIN`), making it available system-wide if that path is in your `$PATH`.

**Q: How do you reduce the size of a Go binary?**
**A:** You can use linker flags to strip debug information and symbol tables: `go build -ldflags "-s -w"`. This can reduce binary size by 20-30% without affecting functionality (though debugging becomes harder).

**Q: What does the `-race` flag do?**
**A:** It instruments the binary with the Go Race Detector. This adds runtime checks to detect race conditions (concurrent access to shared memory). It significantly increases memory usage and CPU overhead (approx 10x), so it should NOT be used in production builds, but is essential for testing and CI/CD pipelines.
