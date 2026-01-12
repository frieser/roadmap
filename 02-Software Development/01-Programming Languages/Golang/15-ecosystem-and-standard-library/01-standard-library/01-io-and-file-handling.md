# I/O and File Handling

## Summary
I/O in Go is built around the powerful `io.Reader` and `io.Writer` interfaces. These abstractions allow you to write code that doesn't care if it's reading from a file, a network connection, a buffer, or stdin. The `os` package provides the primitives for file system interaction (creating, opening, deleting), while `io` utilities (like `Copy`, `ReadAll`) glue everything together.

## Detailed Explanation

### The Core Interfaces (`io` package)
Everything in Go I/O revolves around these two interfaces. If your code accepts these, it becomes universally compatible.

1.  **`io.Reader`**:
    ```go
    type Reader interface {
        Read(p []byte) (n int, err error)
    }
    ```
    Reads up to `len(p)` bytes into `p`. Returns number of bytes read and any error (often `io.EOF` at the end).

2.  **`io.Writer`**:
    ```go
    type Writer interface {
        Write(p []byte) (n int, err error)
    }
    ```
    Writes bytes from `p` to the underlying stream.

### File Operations (`os` package)

#### Reading Files
**Method 1: Read entire file (Convenient but memory heavy)**
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // os.ReadFile (replaces obsolete ioutil.ReadFile)
    data, err := os.ReadFile("config.txt")
    if err != nil {
        panic(err)
    }
    fmt.Println(string(data))
}
```

**Method 2: Read incrementally (Memory efficient)**
```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    file, err := os.Open("large_log.txt")
    if err != nil {
        panic(err)
    }
    // CRITICAL: Always defer Close() immediately after checking error
    defer file.Close()

    buffer := make([]byte, 1024) // 1KB chunk
    for {
        n, err := file.Read(buffer)
        if err == io.EOF {
            break // End of file
        }
        if err != nil {
            panic(err)
        }
        fmt.Print(string(buffer[:n]))
    }
}
```

#### Writing Files
```go
package main

import "os"

func main() {
    // Create or Truncate file
    file, err := os.Create("output.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    data := []byte("Hello, World!\n")
    _, err = file.Write(data)
    if err != nil {
        panic(err)
    }
}
```

### Useful `io` Utilities
*   `io.Copy(dst, src)`: Streams data from a Reader to a Writer. Great for file copies or HTTP responses.
*   `io.ReadAll(r)`: Reads everything into a byte slice.
*   `io.LimitReader(r, n)`: Wraps a Reader to stop after `n` bytes.

### Deprecation Note: `io/ioutil`
As of Go 1.16, the `io/ioutil` package is deprecated.
*   `ioutil.ReadFile` -> `os.ReadFile`
*   `ioutil.WriteFile` -> `os.WriteFile`
*   `ioutil.ReadAll` -> `io.ReadAll`
*   `ioutil.Discard` -> `io.Discard`

## Interview Questions

**Q: Why is it important to `defer file.Close()`?**
**A:** `os.Open` asks the operating system for a file descriptor. These are limited resources. If you don't close them, your program will leak descriptors and eventually crash with "too many open files". `defer` ensures `Close()` runs even if the function panics or returns early.

**Q: What is the difference between `os.Open` and `os.Create`?**
**A:** `os.Open` opens a file for **reading only**. `os.Create` creates a file if it doesn't exist, or **truncates** (empties) it if it does, and opens it for reading and writing (mostly writing). To append, you must use `os.OpenFile` with specific flags (`os.O_APPEND|os.O_WRONLY`).

**Q: Explain the `io.Reader` interface contract regarding `io.EOF`.**
**A:** When `Read` reaches the end of the input, it returns `io.EOF` as the error. Crucially, it might return `n > 0` *and* `err == io.EOF` simultaneously (meaning "I read some data, and then hit the end"). Robust code must process the `n` bytes before checking `err`.
