#Golang
---
---

## Summary

Go provides two floating-point types: `float32` (single precision) and `float64` (double precision), both following the IEEE 754 standard. `float64` is the default and recommended for most use cases due to higher precision. Floating-point numbers can represent very large and very small values but have precision limitations that can cause subtle bugs in financial or comparison operations.

## Detailed Explanation

### **Floating-Point Types**

| Type | Size | Precision | Range |
| --- | --- | --- | --- |
| `float32` | 32 bits | ~7 decimal digits | ±1.18×10⁻³⁸ to ±3.4×10³⁸ |
| `float64` | 64 bits | ~15-16 decimal digits | ±2.23×10⁻³⁰⁸ to ±1.8×10³⁰⁸ |

### **Declaration and Initialization**

```go
package main

import "fmt"

func main() {
    // Explicit type
    var f32 float32 = 3.14
    var f64 float64 = 3.141592653589793
    
    // Short declaration (infers float64)
    pi := 3.14159  // Type: float64
    
    // Force float32
    pi32 := float32(3.14159)
    
    // Scientific notation
    avogadro := 6.022e23    // 6.022 × 10²³
    planck := 6.626e-34     // 6.626 × 10⁻³⁴
    
    // Hexadecimal float literals (Go 1.13+)
    hexFloat := 0x1.fp+2    // 7.75 (1.9375 × 2²)
    
    fmt.Printf("float32: %f\n", f32)
    fmt.Printf("float64: %.15f\n", f64)
    fmt.Printf("avogadro: %e\n", avogadro)
}
```

### **Precision Differences**

```go
func main() {
    // float32 has ~7 digits of precision
    var f32 float32 = 1.123456789
    fmt.Printf("float32: %.10f\n", f32)  // 1.1234568357 (inaccurate after 7 digits)
    
    // float64 has ~15-16 digits of precision
    var f64 float64 = 1.123456789012345678
    fmt.Printf("float64: %.18f\n", f64)  // 1.123456789012345600 (accurate to ~15 digits)
    
    // Why prefer float64?
    // - Default type for literals
    // - math package uses float64
    // - More precision, minimal overhead on modern CPUs
}
```

### **Special Values**

```go
import "math"

func main() {
    // Positive and negative infinity
    posInf := math.Inf(1)
    negInf := math.Inf(-1)
    
    fmt.Println(posInf)         // +Inf
    fmt.Println(negInf)         // -Inf
    fmt.Println(1.0 / 0.0)      // +Inf (division by zero)
    
    // NaN (Not a Number)
    nan := math.NaN()
    fmt.Println(nan)            // NaN
    fmt.Println(0.0 / 0.0)      // NaN
    fmt.Println(math.Sqrt(-1))  // NaN
    
    // Checking special values
    fmt.Println(math.IsInf(posInf, 1))  // true
    fmt.Println(math.IsInf(negInf, -1)) // true
    fmt.Println(math.IsNaN(nan))        // true
    
    // NaN is not equal to anything, including itself!
    fmt.Println(nan == nan)  // false
}
```

### **Floating-Point Comparison Pitfalls**

```go
func main() {
    // Classic precision problem
    a := 0.1
    b := 0.2
    c := 0.3
    
    fmt.Println(a + b == c)  // false! (surprise)
    fmt.Printf("%.20f\n", a+b)  // 0.30000000000000004441
    fmt.Printf("%.20f\n", c)    // 0.29999999999999998890
    
    // Solution: Compare with epsilon (tolerance)
    const epsilon = 1e-9
    if math.Abs((a+b) - c) < epsilon {
        fmt.Println("Approximately equal")
    }
}

// Helper function for float comparison
func almostEqual(a, b, epsilon float64) bool {
    return math.Abs(a-b) < epsilon
}
```

### **Arithmetic Operations**

```go
import "math"

func main() {
    a, b := 10.5, 3.2
    
    // Basic arithmetic
    fmt.Println(a + b)  // 13.7
    fmt.Println(a - b)  // 7.3
    fmt.Println(a * b)  // 33.6
    fmt.Println(a / b)  // 3.28125
    
    // No modulo for floats, use math.Mod
    fmt.Println(math.Mod(a, b))  // 0.9
    
    // math package functions
    fmt.Println(math.Sqrt(16))       // 4
    fmt.Println(math.Pow(2, 10))     // 1024
    fmt.Println(math.Abs(-5.5))      // 5.5
    fmt.Println(math.Floor(3.7))     // 3
    fmt.Println(math.Ceil(3.2))      // 4
    fmt.Println(math.Round(3.5))     // 4
    fmt.Println(math.Trunc(3.9))     // 3
    
    // Trigonometric
    fmt.Println(math.Sin(math.Pi/2)) // 1
    fmt.Println(math.Cos(0))         // 1
    
    // Logarithms
    fmt.Println(math.Log(math.E))    // 1 (natural log)
    fmt.Println(math.Log10(100))     // 2
    fmt.Println(math.Log2(8))        // 3
}
```

### **Type Conversion**

```go
func main() {
    // Float to int (truncates toward zero)
    f := 3.9
    i := int(f)
    fmt.Println(i)  // 3 (not rounded!)
    
    // For rounding, use math functions
    fmt.Println(int(math.Round(f)))  // 4
    fmt.Println(int(math.Floor(f)))  // 3
    fmt.Println(int(math.Ceil(f)))   // 4
    
    // Int to float
    n := 42
    f64 := float64(n)
    
    // float32 to float64 (safe)
    var f32 float32 = 3.14
    f64 = float64(f32)
    
    // float64 to float32 (may lose precision)
    var bigFloat float64 = 1.123456789012345
    smallFloat := float32(bigFloat)
    fmt.Printf("%.15f -> %.7f\n", bigFloat, smallFloat)
}
```

### **Formatting Output**

```go
func main() {
    f := 123456.789
    
    // Default
    fmt.Printf("%v\n", f)     // 123456.789
    
    // Decimal notation
    fmt.Printf("%f\n", f)     // 123456.789000
    fmt.Printf("%.2f\n", f)   // 123456.79
    fmt.Printf("%10.2f\n", f) //  123456.79 (width 10)
    
    // Scientific notation
    fmt.Printf("%e\n", f)     // 1.234568e+05
    fmt.Printf("%E\n", f)     // 1.234568E+05
    fmt.Printf("%.2e\n", f)   // 1.23e+05
    
    // Compact notation (shortest representation)
    fmt.Printf("%g\n", f)     // 123456.789
    
    // Type information
    fmt.Printf("%T\n", f)     // float64
}
```

### **Constants from math Package**

```go
import "math"

func main() {
    fmt.Println(math.MaxFloat32)   // 3.4028235e+38
    fmt.Println(math.MaxFloat64)   // 1.7976931348623157e+308
    fmt.Println(math.SmallestNonzeroFloat32)  // 1e-45
    fmt.Println(math.SmallestNonzeroFloat64)  // 5e-324
    
    fmt.Println(math.Pi)   // 3.141592653589793
    fmt.Println(math.E)    // 2.718281828459045
    fmt.Println(math.Phi)  // 1.618033988749895 (golden ratio)
}
```

### **When NOT to Use Floats**

```go
// ✗ BAD: Using floats for money
var price float64 = 19.99
var quantity float64 = 3
total := price * quantity
fmt.Printf("%.2f\n", total)  // May show 59.97 or 59.970000000000006

// ✓ GOOD: Use integers (cents) for money
priceInCents := 1999
quantityInt := 3
totalCents := priceInCents * quantityInt
fmt.Printf("$%.2f\n", float64(totalCents)/100)  // $59.97

// ✓ ALTERNATIVE: Use a decimal library
// e.g., github.com/shopspring/decimal
```

## Interview Questions

**Q: What is the difference between `float32` and `float64` in Go?**
**A:** `float32` uses 32 bits with ~7 digits of precision, while `float64` uses 64 bits with ~15-16 digits of precision. `float64` is the default type for floating-point literals and is preferred because the math package uses it, it has higher precision, and modern CPUs handle it efficiently.

**Q: Why does `0.1 + 0.2 != 0.3` in Go?**
**A:** Floating-point numbers use binary representation (IEEE 754), and some decimal fractions like 0.1 cannot be represented exactly. This causes tiny rounding errors. To compare floats, use an epsilon tolerance: `math.Abs(a-b) < epsilon` instead of direct equality.

**Q: How do you convert a float to an int in Go?**
**A:** Use `int(floatValue)`, which truncates toward zero (3.9 → 3, -3.9 → -3). For proper rounding, use `int(math.Round(f))`. For floor/ceiling, use `int(math.Floor(f))` or `int(math.Ceil(f))`.

**Q: Why shouldn't you use floats for monetary calculations?**
**A:** Floating-point precision errors can accumulate in financial calculations, causing incorrect totals. Instead, use integers representing the smallest unit (cents), or use a decimal library like `github.com/shopspring/decimal` that provides exact decimal arithmetic.
