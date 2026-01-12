# Bufio Package

## Summary
The `bufio` (buffered I/O) package wraps `io.Reader` and `io.Writer` objects to create buffered readers and writers. This drastically improves performance by reducing the number of system calls (syscalls). Instead of reading/writing 1 byte at a time to the disk/network, `bufio` reads/writes big chunks (default 4KB) into a memory buffer and only accesses the underlying I/O when the buffer is full (write) or empty (read).

## Detailed Explanation

### 1. `bufio.Reader`
Allows "peeking" at data and reading line-by-line efficiently.

**Example: Reading a file line-by-line**
```go
file, _ := os.Open("access.log")
defer file.Close()

scanner := bufio.NewScanner(file)
for scanner.Scan() {
    line := scanner.Text() // string
    // Process line...
}

if err := scanner.Err(); err != nil {
    log.Fatal(err)
}
```
*   `Scanner` is convenient for text but has a max token size (buffer limit).
*   For arbitrary binary data, use `reader := bufio.NewReader(file)` and methods like `ReadBytes` or `ReadString`.

### 2. `bufio.Writer`
Accumulates writes in memory. You **MUST** call `Flush()` to push the final chunk to the underlying writer.

**Example: Efficient writing**
```go
file, _ := os.Create("output.txt")
defer file.Close()

writer := bufio.NewWriter(file)

for i := 0; i < 1000; i++ {
    // Writes to memory buffer, not disk
    writer.WriteString("Logging some data...\n") 
}

// Write any remaining data in buffer to disk
writer.Flush() 
```

### 3. Why it matters
Every `os.File.Write` triggers a syscall. Context switching to the kernel is expensive.
*   **Without bufio**: 1000 writes = 1000 syscalls.
*   **With bufio**: 1000 writes -> Buffer fills maybe twice -> 2 syscalls.

## Interview Questions

**Q: What is the most common bug when using `bufio.Writer`?**
**A:** Forgetting to call `writer.Flush()` at the end. Since `bufio.Writer` holds data in memory to minimize I/O, the last chunk of data will remain in the buffer and never be written to the file/socket if `Flush()` is not called explicitly before closing the resource.

**Q: When should you prefer `bufio.Scanner` over `bufio.Reader`?**
**A:** Use `Scanner` when processing simple streams of data divided by a delimiter (like newlines in a text file) and you don't expect extremely long tokens. Use `Reader` when you need more low-level control, need to read specific byte counts, or need to handle lines longer than the Scanner's default buffer limit (64KB).

**Q: Does `bufio` always improve performance?**
**A:** Not always. If you are already writing large chunks of data (e.g., `os.Write` with a 1MB slice), `bufio` just adds an unnecessary memory copy overhead. `bufio` shines when you are doing *many small* reads or writes.
