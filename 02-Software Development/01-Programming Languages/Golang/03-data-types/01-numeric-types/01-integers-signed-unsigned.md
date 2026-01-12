#Golang
---
---

## Summary

Go provides a rich set of integer types with explicit sizes: signed integers (`int8`, `int16`, `int32`, `int64`) and unsigned integers (`uint8`, `uint16`, `uint32`, `uint64`). The `int` and `uint` types are platform-dependent (32 or 64 bits). Go also includes `byte` (alias for `uint8`) and `rune` (alias for `int32`). Understanding integer sizes, ranges, and overflow behavior is essential for efficient and safe Go programming.

## Detailed Explanation

### **Integer Types Overview**

| Type | Size | Range |
| --- | --- | --- |
| `int8` | 8 bits | -128 to 127 |
| `int16` | 16 bits | -32,768 to 32,767 |
| `int32` | 32 bits | -2,147,483,648 to 2,147,483,647 |
| `int64` | 64 bits | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |
| `uint8` | 8 bits | 0 to 255 |
| `uint16` | 16 bits | 0 to 65,535 |
| `uint32` | 32 bits | 0 to 4,294,967,295 |
| `uint64` | 64 bits | 0 to 18,446,744,073,709,551,615 |
| `int` | 32 or 64 bits | Platform-dependent |
| `uint` | 32 or 64 bits | Platform-dependent |
| `uintptr` | 32 or 64 bits | Large enough to hold a pointer |

### **Type Aliases**

```go
type byte = uint8  // Alias for uint8, used for raw data
type rune = int32  // Alias for int32, used for Unicode code points
```

### **Declaration and Initialization**

```go
package main

import "fmt"

func main() {
    // Explicit type declaration
    var a int8 = 127
    var b int16 = 32767
    var c int32 = 2147483647
    var d int64 = 9223372036854775807
    
    // Unsigned integers
    var ua uint8 = 255
    var ub uint16 = 65535
    var uc uint32 = 4294967295
    var ud uint64 = 18446744073709551615
    
    // Platform-dependent (use for general-purpose)
    var i int = 42
    var u uint = 42
    
    // Short declaration (infers int)
    x := 100  // Type: int
    
    // Force specific type
    y := int32(100)
    z := uint64(100)
    
    fmt.Printf("int8: %d, uint8: %d\n", a, ua)
}
```

### **Integer Literals**

```go
func main() {
    // Decimal (base 10)
    dec := 42
    
    // Binary (base 2) - prefix 0b or 0B
    bin := 0b101010  // 42
    
    // Octal (base 8) - prefix 0o or 0O (or just 0)
    oct := 0o52      // 42
    octOld := 052    // 42 (legacy syntax)
    
    // Hexadecimal (base 16) - prefix 0x or 0X
    hex := 0x2A      // 42
    
    // Underscores for readability (Go 1.13+)
    million := 1_000_000
    binary := 0b1010_1010
    hexAddr := 0xFF_FF_FF_FF
    
    fmt.Printf("dec: %d, bin: %d, oct: %d, hex: %d\n", dec, bin, oct, hex)
}
```

### **Signed vs Unsigned**

```go
func main() {
    // Signed: can represent negative numbers
    var signed int8 = -100
    fmt.Println(signed)  // -100
    
    // Unsigned: only non-negative numbers
    var unsigned uint8 = 200
    fmt.Println(unsigned)  // 200
    
    // Unsigned is useful for:
    // - Bitwise operations
    // - Array/slice indices
    // - Byte manipulation
    // - When negative values are impossible
}
```

### **Platform-Dependent Types**

```go
import (
    "fmt"
    "unsafe"
)

func main() {
    var i int
    var u uint
    var ptr uintptr
    
    // Size depends on architecture
    fmt.Println("Size of int:", unsafe.Sizeof(i))     // 4 or 8
    fmt.Println("Size of uint:", unsafe.Sizeof(u))    // 4 or 8
    fmt.Println("Size of uintptr:", unsafe.Sizeof(ptr)) // 4 or 8
    
    // Use int for general-purpose integers
    // Use int64/int32 when size matters (serialization, file formats)
}
```

### **Integer Overflow**

Go silently wraps on overflow (no runtime error):

```go
func main() {
    var max int8 = 127
    max++
    fmt.Println(max)  // -128 (wrapped around!)
    
    var umax uint8 = 255
    umax++
    fmt.Println(umax)  // 0 (wrapped around!)
    
    // Underflow
    var min uint8 = 0
    min--
    fmt.Println(min)  // 255 (wrapped around!)
}
```

### **Detecting Overflow**

```go
import "math"

func addInt64Safe(a, b int64) (int64, bool) {
    if b > 0 && a > math.MaxInt64-b {
        return 0, false  // Overflow
    }
    if b < 0 && a < math.MinInt64-b {
        return 0, false  // Underflow
    }
    return a + b, true
}

func main() {
    result, ok := addInt64Safe(math.MaxInt64, 1)
    if !ok {
        fmt.Println("Overflow detected!")
    }
}
```

### **Arithmetic Operations**

```go
func main() {
    a, b := 17, 5
    
    // Basic arithmetic
    fmt.Println(a + b)   // 22 (addition)
    fmt.Println(a - b)   // 12 (subtraction)
    fmt.Println(a * b)   // 85 (multiplication)
    fmt.Println(a / b)   // 3  (integer division, truncates)
    fmt.Println(a % b)   // 2  (modulo/remainder)
    
    // Increment/decrement
    a++  // a = 18
    b--  // b = 4
    
    // Compound assignment
    a += 10  // a = a + 10
    b *= 2   // b = b * 2
}
```

### **Bitwise Operations**

```go
func main() {
    a, b := uint8(0b1100), uint8(0b1010)
    
    fmt.Printf("AND:  %08b\n", a&b)   // 00001000
    fmt.Printf("OR:   %08b\n", a|b)   // 00001110
    fmt.Printf("XOR:  %08b\n", a^b)   // 00000110
    fmt.Printf("NOT:  %08b\n", ^a)    // 11110011
    
    // Bit shifts
    x := uint8(1)
    fmt.Printf("<<3:  %08b\n", x<<3)  // 00001000 (multiply by 8)
    
    y := uint8(16)
    fmt.Printf(">>2:  %08b\n", y>>2)  // 00000100 (divide by 4)
    
    // Bit clear (AND NOT)
    fmt.Printf("&^:   %08b\n", a&^b)  // 00000100
}
```

### **Type Conversion**

Go requires explicit conversion between numeric types:

```go
func main() {
    var i32 int32 = 100
    var i64 int64
    
    // i64 = i32  // Error: cannot use int32 as int64
    i64 = int64(i32)  // Explicit conversion required
    
    // Potential data loss
    var big int64 = 1000
    var small int8 = int8(big)  // Truncated! small = -24
    
    // Safe conversion check
    if big >= math.MinInt8 && big <= math.MaxInt8 {
        small = int8(big)
    } else {
        fmt.Println("Value out of range for int8")
    }
}
```

### **Common Use Cases**

```go
// Use int for general-purpose
func count(items []string) int {
    return len(items)  // len returns int
}

// Use int64 for timestamps
func timestamp() int64 {
    return time.Now().Unix()
}

// Use uint8 (byte) for raw data
func readByte(r io.Reader) (byte, error) {
    buf := make([]byte, 1)
    _, err := r.Read(buf)
    return buf[0], err
}

// Use specific sizes for protocols/file formats
type Header struct {
    Version uint16
    Length  uint32
    Flags   uint8
}
```

### **Constants with math Package**

```go
import "math"

func main() {
    fmt.Println("int8  range:", math.MinInt8, "to", math.MaxInt8)
    fmt.Println("int16 range:", math.MinInt16, "to", math.MaxInt16)
    fmt.Println("int32 range:", math.MinInt32, "to", math.MaxInt32)
    fmt.Println("int64 range:", math.MinInt64, "to", math.MaxInt64)
    
    fmt.Println("uint8  max:", math.MaxUint8)
    fmt.Println("uint16 max:", math.MaxUint16)
    fmt.Println("uint32 max:", math.MaxUint32)
    fmt.Println("uint64 max:", uint64(math.MaxUint64))
}
```

## Interview Questions

**Q: What is the difference between `int` and `int64` in Go?**
**A:** `int64` is always 64 bits on all platforms. `int` is platform-dependent: 32 bits on 32-bit systems and 64 bits on 64-bit systems. Use `int` for general-purpose integers and `int64` when you need a guaranteed size (e.g., for serialization, protocols, or when values might exceed 32-bit range).

**Q: What happens when an integer overflows in Go?**
**A:** Go silently wraps around without any runtime error. For `int8`, incrementing 127 gives -128. For `uint8`, incrementing 255 gives 0. This is by design for performance, but you must check for overflow manually when it matters.

**Q: When should you use unsigned integers?**
**A:** Use unsigned integers when values cannot be negative: array indices, byte manipulation, bitwise operations, sizes/lengths, and when you need the full positive range (e.g., `uint8` gives 0-255 vs `int8`'s -128 to 127). The Go standard library uses `int` for lengths/indices, so prefer `int` for compatibility.

**Q: Why does Go require explicit type conversion between integers?**
**A:** Go prioritizes type safety over convenience. Implicit conversions can silently truncate data or change sign, causing subtle bugs. Explicit conversion forces developers to acknowledge potential data loss, making code safer and intentions clearer.
