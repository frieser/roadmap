# Unsafe Package

## Summary
The `unsafe` package allows programmers to bypass Go's type safety and memory security protections. It provides access to low-level memory operations, such as pointer arithmetic and arbitrary type casting. It should be used with extreme caution as it can lead to memory corruption and crashes.

## Detailed Explanation

### Key Types
1.  **`unsafe.Pointer`**: A special pointer type that can bridge the gap between any two types.
    *   `*T` <-> `unsafe.Pointer` <-> `*U`
2.  **`uintptr`**: An integer type large enough to hold a memory address. Used for pointer arithmetic.
    *   `unsafe.Pointer` <-> `uintptr`

### The Rules of Unsafe
1.  **Arbitrary Casting**: You can convert a `*float64` to `*uint64` to inspect bits.
2.  **Pointer Arithmetic**: You can convert to `uintptr`, add an offset, and convert back.
3.  **GC Danger**: A `uintptr` is just a number. The Garbage Collector does not track it. If you store an address in a `uintptr` and the GC runs, it might collect the object. You must convert back to `unsafe.Pointer` immediately.

### Code Example: Pointer Arithmetic

```go
package main

import (
    "fmt"
    "unsafe"
)

type Data struct {
    a int64
    b int64
}

func main() {
    d := Data{a: 10, b: 20}
    
    // Get pointer to start of struct
    p := unsafe.Pointer(&d)
    
    // Cast to *int64 to read first field 'a'
    pa := (*int64)(p)
    fmt.Println("a:", *pa)
    
    // Calculate offset for 'b'
    // uintptr(p) + offset
    pbOffset := uintptr(p) + unsafe.Offsetof(d.b)
    pb := (*int64)(unsafe.Pointer(pbOffset))
    
    fmt.Println("b:", *pb)
    
    // Modify via unsafe
    *pb = 99
    fmt.Println("Modified d.b:", d.b)
}
```

## Interview Questions

**Q: What is the difference between `unsafe.Pointer` and `uintptr`?**
**A:** `unsafe.Pointer` is a pointer managed by the GC (the object pointed to won't be collected). `uintptr` is just an integer; the GC ignores it. You use `uintptr` only for arithmetic and must convert back to `unsafe.Pointer` immediately.

**Q: Why is it called "unsafe"?**
**A:** Because it breaks the type system and memory safety guarantees. Misuse can corrupt memory, cause segfaults, or create race conditions that Go's runtime cannot prevent.

**Q: When would you use the unsafe package?**
**A:** Rarely. Use cases include interaction with C code (CGO), high-performance serialization (like `reflect` internals), or system calls where precise memory layout is required.
