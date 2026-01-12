#Golang
---
---

## Summary

Go natively supports complex numbers with two types: `complex64` (using two `float32` values) and `complex128` (using two `float64` values). Complex numbers have a real and imaginary part, created using the `complex()` built-in or literal syntax with the `i` suffix. Go provides functions to extract components (`real()`, `imag()`) and a `cmplx` package for complex mathematical operations.

## Detailed Explanation

### **Complex Number Types**

| Type | Components | Precision |
| --- | --- | --- |
| `complex64` | Two `float32` values | ~7 decimal digits per component |
| `complex128` | Two `float64` values | ~15 decimal digits per component |

### **Creating Complex Numbers**

```go
package main

import "fmt"

func main() {
    // Using complex() built-in
    c1 := complex(3, 4)  // 3 + 4i (complex128)
    
    // Using literal syntax with 'i'
    c2 := 3 + 4i         // 3 + 4i (complex128)
    
    // Explicit types
    var c64 complex64 = complex(float32(3), float32(4))
    var c128 complex128 = complex(3.0, 4.0)
    
    // Short declaration with explicit type
    c3 := complex64(3 + 4i)
    
    // Pure imaginary
    imag := 5i           // 0 + 5i
    
    // Pure real (still complex)
    real := complex(7, 0)  // 7 + 0i
    
    fmt.Println(c1)   // (3+4i)
    fmt.Println(c64)  // (3+4i)
    fmt.Println(c128) // (3+4i)
}
```

### **Extracting Components**

```go
func main() {
    c := 3 + 4i
    
    // Built-in functions
    r := real(c)  // 3 (float64)
    i := imag(c)  // 4 (float64)
    
    fmt.Printf("Complex: %v\n", c)       // (3+4i)
    fmt.Printf("Real: %f\n", r)          // 3.000000
    fmt.Printf("Imaginary: %f\n", i)     // 4.000000
    fmt.Printf("Type of real: %T\n", r)  // float64
    
    // For complex64, components are float32
    var c64 complex64 = 3 + 4i
    r32 := real(c64)  // float32
    fmt.Printf("Type: %T\n", r32)  // float32
}
```

### **Arithmetic Operations**

```go
func main() {
    a := 3 + 4i
    b := 1 + 2i
    
    // Addition
    fmt.Println(a + b)  // (4+6i)
    
    // Subtraction
    fmt.Println(a - b)  // (2+2i)
    
    // Multiplication
    // (3+4i)(1+2i) = 3 + 6i + 4i + 8i² = 3 + 10i - 8 = -5 + 10i
    fmt.Println(a * b)  // (-5+10i)
    
    // Division
    // (3+4i)/(1+2i) = (3+4i)(1-2i)/((1+2i)(1-2i)) = (11-2i)/5
    fmt.Println(a / b)  // (2.2-0.4i)
    
    // Equality
    fmt.Println(a == 3+4i)  // true
    fmt.Println(a == b)     // false
}
```

### **The cmplx Package**

```go
import (
    "fmt"
    "math/cmplx"
)

func main() {
    c := 3 + 4i
    
    // Absolute value (modulus): |z| = sqrt(real² + imag²)
    fmt.Println(cmplx.Abs(c))  // 5 (3-4-5 triangle)
    
    // Phase angle (argument) in radians
    fmt.Println(cmplx.Phase(c))  // 0.9272... (arctan(4/3))
    
    // Conjugate: a + bi → a - bi
    fmt.Println(cmplx.Conj(c))  // (3-4i)
    
    // Polar form (r, θ)
    r, theta := cmplx.Polar(c)
    fmt.Printf("r=%f, θ=%f\n", r, theta)
    
    // Create from polar
    c2 := cmplx.Rect(5, 0.927295)  // Approximately 3+4i
    fmt.Println(c2)
    
    // Square root
    fmt.Println(cmplx.Sqrt(-1))   // (0+1i) = i
    fmt.Println(cmplx.Sqrt(c))    // (2+1i)
    
    // Power
    fmt.Println(cmplx.Pow(c, 2))  // (-7+24i)
    
    // Exponential: e^(a+bi) = e^a * (cos(b) + i*sin(b))
    fmt.Println(cmplx.Exp(1i * 3.14159))  // ≈ (-1+0i) (Euler's formula)
    
    // Natural logarithm
    fmt.Println(cmplx.Log(c))
    
    // Trigonometric functions
    fmt.Println(cmplx.Sin(c))
    fmt.Println(cmplx.Cos(c))
    fmt.Println(cmplx.Tan(c))
}
```

### **Euler's Identity**

```go
import (
    "fmt"
    "math"
    "math/cmplx"
)

func main() {
    // Euler's formula: e^(iπ) + 1 = 0
    eulerResult := cmplx.Exp(1i*math.Pi) + 1
    
    // Due to floating-point precision, result is very close to 0
    fmt.Printf("e^(iπ) + 1 = %v\n", eulerResult)
    // Output: (0+1.2246467991473515e-16i) ≈ 0
    
    // Euler's formula: e^(iθ) = cos(θ) + i*sin(θ)
    theta := math.Pi / 4  // 45 degrees
    euler := cmplx.Exp(complex(0, theta))
    manual := complex(math.Cos(theta), math.Sin(theta))
    
    fmt.Printf("e^(iπ/4) = %v\n", euler)
    fmt.Printf("cos + i*sin = %v\n", manual)
}
```

### **Practical Applications**

#### Signal Processing (FFT components)

```go
func dftPoint(signal []float64, k int) complex128 {
    N := len(signal)
    var sum complex128
    for n := 0; n < N; n++ {
        angle := -2 * math.Pi * float64(k*n) / float64(N)
        sum += complex(signal[n], 0) * cmplx.Exp(complex(0, angle))
    }
    return sum
}
```

#### Mandelbrot Set

```go
func mandelbrot(c complex128, maxIter int) int {
    z := complex(0, 0)
    for i := 0; i < maxIter; i++ {
        z = z*z + c
        if cmplx.Abs(z) > 2 {
            return i
        }
    }
    return maxIter
}

func main() {
    // Check if point is in Mandelbrot set
    c := complex(-0.5, 0.5)
    iterations := mandelbrot(c, 100)
    if iterations == 100 {
        fmt.Println("In Mandelbrot set")
    }
}
```

#### Electrical Engineering (Impedance)

```go
func main() {
    // Complex impedance: Z = R + jX
    R := 100.0   // Resistance (Ohms)
    X := 50.0    // Reactance (Ohms)
    Z := complex(R, X)
    
    // Magnitude and phase
    magnitude := cmplx.Abs(Z)
    phase := cmplx.Phase(Z) * 180 / math.Pi  // Convert to degrees
    
    fmt.Printf("|Z| = %.2f Ω\n", magnitude)
    fmt.Printf("∠Z = %.2f°\n", phase)
}
```

### **Type Conversion**

```go
func main() {
    // complex64 to complex128
    var c64 complex64 = 3 + 4i
    c128 := complex128(c64)
    
    // complex128 to complex64 (may lose precision)
    var bigC complex128 = 3.141592653589793 + 2.718281828459045i
    smallC := complex64(bigC)
    fmt.Println(smallC)  // (3.1415927+2.7182818i)
    
    // Cannot directly convert to/from non-complex
    // real := float64(c128)  // Error!
    // Use real() and imag() instead
    realPart := real(c128)
    imagPart := imag(c128)
}
```

### **Formatting Output**

```go
func main() {
    c := 3.14159 + 2.71828i
    
    fmt.Printf("%v\n", c)     // (3.14159+2.71828i)
    fmt.Printf("%+v\n", c)    // (3.14159+2.71828i)
    fmt.Printf("%f\n", c)     // (3.141590+2.718280i)
    fmt.Printf("%.2f\n", c)   // (3.14+2.72i)
    fmt.Printf("%e\n", c)     // (3.141590e+00+2.718280e+00i)
    fmt.Printf("%T\n", c)     // complex128
}
```

## Interview Questions

**Q: What are the two complex number types in Go?**
**A:** `complex64` (composed of two `float32` values) and `complex128` (composed of two `float64` values). The default type for complex literals like `3+4i` is `complex128`. Use `complex128` for most purposes due to higher precision.

**Q: How do you create a complex number and extract its parts?**
**A:** Create using `complex(real, imag)` or literal syntax `3+4i`. Extract parts using built-in functions: `real(c)` returns the real part and `imag(c)` returns the imaginary part. For `complex128`, these return `float64`; for `complex64`, they return `float32`.

**Q: What is the purpose of the cmplx package?**
**A:** The `math/cmplx` package provides mathematical functions for complex numbers including `Abs` (modulus), `Phase` (argument), `Conj` (conjugate), `Sqrt`, `Pow`, `Exp`, `Log`, and trigonometric functions. It also provides `Polar` and `Rect` for conversion between Cartesian and polar forms.

**Q: What practical applications use complex numbers in Go?**
**A:** Signal processing (FFT, filters), electrical engineering (impedance, AC circuits), fractal generation (Mandelbrot/Julia sets), control systems, and physics simulations. Go's native complex support makes these calculations straightforward without external libraries.
