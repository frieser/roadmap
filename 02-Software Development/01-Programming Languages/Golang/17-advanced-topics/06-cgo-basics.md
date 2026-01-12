# CGO Basics

## Summary
CGO allows Go packages to call C code. It enables interoperability with existing C libraries but comes with costs: increased build complexity, loss of cross-compilation ease, and runtime performance overhead due to stack switching.

## Detailed Explanation

### How It Works
CGO uses a special pseudo-package `import "C"`. The comment immediately preceding this import is treated as a C header or implementation.

### Key Concepts
1.  **Preamble**: The C code in the comment block.
2.  **Types**: Go provides access to C types via the `C` namespace (e.g., `C.int`, `C.float`).
3.  **Memory**:
    *   Go pointers passed to C must be pinned (handled automatically in simple cases).
    *   C strings (`char*`) must be manually converted to Go strings and vice-versa.
    *   **Memory Leaks**: Memory allocated in C (e.g., `C.CString`) is NOT managed by Go's GC. You must free it.

### Performance Overhead
Calling C from Go is not a simple function call.
1.  **Stack Switching**: Go uses small, dynamic stacks. C uses large, fixed stacks. The runtime must switch stacks.
2.  **Scheduling**: The Go scheduler might hand off the thread to the OS.

### Code Example: Hello from C

```go
package main

/*
#include <stdio.h>
#include <stdlib.h>

void myPrint(char* s) {
    printf("C says: %s\n", s);
}
*/
import "C"
import (
    "unsafe"
)

func main() {
    // Convert Go string to C string (Allocates!)
    cs := C.CString("Hello from Go")
    
    // MUST Free the C string to avoid leak
    defer C.free(unsafe.Pointer(cs))
    
    // Call C function
    C.myPrint(cs)
}
```

## Interview Questions

**Q: What is the main downside of using CGO?**
**A:** It disables cross-compilation by default (requires a C toolchain for the target), increases build time, and introduces runtime overhead for every call. It also loses Go's memory safety for the C portions.

**Q: How do you handle strings in CGO?**
**A:** Use `C.CString(goStr)` to convert to C, and `C.GoString(cPtr)` to convert to Go. Crucially, `C.CString` allocates memory using C's malloc, so you **must** call `C.free` (via `unsafe`) to release it.

**Q: Does the Go Garbage Collector manage C memory?**
**A:** No. Any memory allocated by C (malloc) or via CGO helpers like `C.CString` is invisible to the Go GC and must be managed manually.
