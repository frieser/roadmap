#Golang
---
---

## Summary

Go requires explicit type conversion between different types—there are no implicit conversions. The syntax `T(v)` converts value `v` to type `T`. For converting between strings and other types, Go provides the `strconv` package with functions like `Atoi`, `Itoa`, `ParseInt`, `ParseFloat`, `FormatInt`, and `FormatFloat`. Understanding these conversions is essential for working with user input, JSON, and data serialization.

## Detailed Explanation

### **Basic Type Conversion Syntax**

```go
package main

import "fmt"

func main() {
    // Syntax: Type(value)
    
    // Integer conversions
    var i int = 42
    var i64 int64 = int64(i)
    var i32 int32 = int32(i)
    var u uint = uint(i)
    
    // Float conversions
    var f float64 = float64(i)
    var f32 float32 = float32(f)
    
    // Float to int (truncates toward zero)
    f = 3.9
    i = int(f)
    fmt.Println(i)  // 3 (not rounded!)
    
    f = -3.9
    i = int(f)
    fmt.Println(i)  // -3 (truncates toward zero)
}
```

### **Numeric Conversions**

```go
func main() {
    // int to int (different sizes)
    var small int8 = 100
    var large int64 = int64(small)
    
    // Potential data loss (value too large)
    large = 1000
    small = int8(large)  // Truncated! small = -24
    
    // Safe conversion with bounds checking
    import "math"
    
    if large >= math.MinInt8 && large <= math.MaxInt8 {
        small = int8(large)
    } else {
        fmt.Println("Value out of range for int8")
    }
    
    // Signed to unsigned
    var signed int = -1
    var unsigned uint = uint(signed)
    fmt.Println(unsigned)  // 18446744073709551615 (wraps around!)
}
```

### **String Conversions with strconv**

#### String ↔ Integer

```go
import "strconv"

func main() {
    // String to int
    s := "42"
    i, err := strconv.Atoi(s)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println(i)  // 42
    
    // Int to string
    n := 123
    s = strconv.Itoa(n)
    fmt.Println(s)  // "123"
    
    // ParseInt: more control (base, bit size)
    s = "1010"
    i64, err := strconv.ParseInt(s, 2, 64)  // binary
    fmt.Println(i64)  // 10
    
    s = "FF"
    i64, err = strconv.ParseInt(s, 16, 64)  // hex
    fmt.Println(i64)  // 255
    
    // ParseUint for unsigned
    u64, err := strconv.ParseUint("255", 10, 8)
    fmt.Println(u64)  // 255
    
    // FormatInt: int to string with base
    s = strconv.FormatInt(255, 16)
    fmt.Println(s)  // "ff"
    
    s = strconv.FormatInt(10, 2)
    fmt.Println(s)  // "1010"
}
```

#### String ↔ Float

```go
import "strconv"

func main() {
    // String to float
    s := "3.14159"
    f, err := strconv.ParseFloat(s, 64)  // 64-bit precision
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println(f)  // 3.14159
    
    // Float to string
    f = 3.14159
    s = strconv.FormatFloat(f, 'f', 2, 64)  // 2 decimal places
    fmt.Println(s)  // "3.14"
    
    // Format options
    s = strconv.FormatFloat(f, 'e', 4, 64)  // scientific
    fmt.Println(s)  // "3.1416e+00"
    
    s = strconv.FormatFloat(f, 'g', -1, 64)  // compact
    fmt.Println(s)  // "3.14159"
}
```

#### String ↔ Bool

```go
import "strconv"

func main() {
    // String to bool
    b, err := strconv.ParseBool("true")
    fmt.Println(b)  // true
    
    // Accepts: "1", "t", "T", "true", "TRUE", "True"
    //          "0", "f", "F", "false", "FALSE", "False"
    b, err = strconv.ParseBool("1")
    fmt.Println(b)  // true
    
    b, err = strconv.ParseBool("FALSE")
    fmt.Println(b)  // false
    
    // Bool to string
    s := strconv.FormatBool(true)
    fmt.Println(s)  // "true"
}
```

### **String ↔ Byte Slice**

```go
func main() {
    // String to []byte
    s := "Hello"
    bytes := []byte(s)
    fmt.Println(bytes)  // [72 101 108 108 111]
    
    // []byte to string
    s = string(bytes)
    fmt.Println(s)  // "Hello"
    
    // String to []rune (for Unicode)
    s = "Hello, 世界"
    runes := []rune(s)
    fmt.Println(len(runes))  // 9 (characters, not bytes)
    
    // []rune to string
    s = string(runes)
}
```

### **Number to String (Simple)**

```go
import "fmt"

func main() {
    // Using fmt.Sprintf (convenient but slower)
    i := 42
    s := fmt.Sprintf("%d", i)
    fmt.Println(s)  // "42"
    
    f := 3.14159
    s = fmt.Sprintf("%.2f", f)
    fmt.Println(s)  // "3.14"
    
    // With formatting
    s = fmt.Sprintf("%08d", i)
    fmt.Println(s)  // "00000042"
    
    s = fmt.Sprintf("%x", 255)
    fmt.Println(s)  // "ff"
}
```

### **Interface Type Assertions**

```go
func main() {
    var i interface{} = "hello"
    
    // Type assertion
    s, ok := i.(string)
    if ok {
        fmt.Println(s)  // hello
    }
    
    // Without ok (panics if wrong type)
    s = i.(string)  // Works
    // n := i.(int)  // Panic!
    
    // Type switch
    switch v := i.(type) {
    case string:
        fmt.Println("String:", v)
    case int:
        fmt.Println("Int:", v)
    default:
        fmt.Println("Unknown type")
    }
}
```

### **Custom Type Conversions**

```go
// Type aliases require conversion
type UserID int
type ProductID int

func main() {
    var uid UserID = 100
    var pid ProductID = 200
    
    // Cannot mix types without conversion
    // pid = uid  // Error!
    
    // Must convert explicitly
    pid = ProductID(uid)
    
    // Convert to underlying type
    var i int = int(uid)
    fmt.Println(i)
}
```

### **Conversion Functions Table**

| Conversion | Function | Example |
| --- | --- | --- |
| string → int | `strconv.Atoi` | `i, err := strconv.Atoi("42")` |
| int → string | `strconv.Itoa` | `s := strconv.Itoa(42)` |
| string → int64 | `strconv.ParseInt` | `i, err := strconv.ParseInt("42", 10, 64)` |
| int64 → string | `strconv.FormatInt` | `s := strconv.FormatInt(42, 10)` |
| string → float64 | `strconv.ParseFloat` | `f, err := strconv.ParseFloat("3.14", 64)` |
| float64 → string | `strconv.FormatFloat` | `s := strconv.FormatFloat(3.14, 'f', 2, 64)` |
| string → bool | `strconv.ParseBool` | `b, err := strconv.ParseBool("true")` |
| bool → string | `strconv.FormatBool` | `s := strconv.FormatBool(true)` |
| string → []byte | `[]byte(s)` | `bytes := []byte("hello")` |
| []byte → string | `string(b)` | `s := string(bytes)` |
| string → []rune | `[]rune(s)` | `runes := []rune("hello")` |
| int → float | `float64(i)` | `f := float64(42)` |
| float → int | `int(f)` | `i := int(3.14)` (truncates) |

### **Error Handling Patterns**

```go
import (
    "fmt"
    "strconv"
)

func parseUserInput(input string) (int, error) {
    n, err := strconv.Atoi(input)
    if err != nil {
        return 0, fmt.Errorf("invalid number %q: %w", input, err)
    }
    return n, nil
}

func main() {
    inputs := []string{"42", "abc", "3.14", ""}
    
    for _, input := range inputs {
        n, err := parseUserInput(input)
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Printf("Parsed: %d\n", n)
        }
    }
}
```

### **Best Practices**

```go
// ✓ Good: Use strconv for string/number conversion
i, err := strconv.Atoi(input)
if err != nil {
    return err
}

// ✗ Avoid: Ignoring errors
i, _ := strconv.Atoi(input)  // Dangerous!

// ✓ Good: Check bounds before narrowing conversion
if value >= math.MinInt8 && value <= math.MaxInt8 {
    small := int8(value)
}

// ✓ Good: Use fmt.Sprintf for complex formatting
s := fmt.Sprintf("User %d: %s", id, name)

// ✓ Good: strconv is faster for simple conversions
s := strconv.Itoa(n)  // Faster than fmt.Sprintf("%d", n)
```

## Interview Questions

**Q: Why does Go require explicit type conversion?**
**A:** Go prioritizes type safety over convenience. Implicit conversions can silently truncate data, change sign, or lose precision, causing subtle bugs. Explicit conversion forces developers to acknowledge these operations, making code safer and intentions clearer.

**Q: What is the difference between `strconv.Atoi` and `strconv.ParseInt`?**
**A:** `Atoi` is shorthand for `ParseInt(s, 10, 0)` and returns an `int`. `ParseInt` provides control over the base (2-36) and bit size (0, 8, 16, 32, 64), returning `int64`. Use `Atoi` for simple decimal strings, `ParseInt` for different bases or specific sizes.

**Q: How do you safely convert a larger integer type to a smaller one?**
**A:** Check that the value is within the target type's range using constants from the `math` package before converting. For example: `if value >= math.MinInt8 && value <= math.MaxInt8 { small := int8(value) }`.

**Q: What happens when you convert a float to an int in Go?**
**A:** The decimal portion is truncated toward zero (not rounded). `int(3.9)` gives 3, and `int(-3.9)` gives -3. For rounding, use `int(math.Round(f))`. For floor/ceiling, use `int(math.Floor(f))` or `int(math.Ceil(f))`.
