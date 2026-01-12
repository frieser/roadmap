#Golang
---
---

## Summary

The `bool` type in Go represents Boolean values with exactly two possible states: `true` and `false`. Booleans are fundamental for control flow (if, for, switch), logical operations, and conditional expressions. Go does not allow implicit conversion from other types to bool, making code more explicit and safer.

## Detailed Explanation

### **Boolean Basics**

```go
package main

import "fmt"

func main() {
    // Declaration
    var active bool          // Zero value: false
    var enabled bool = true
    
    // Short declaration
    isReady := true
    isComplete := false
    
    // Type information
    fmt.Printf("%v (type: %T)\n", active, active)
    // Output: false (type: bool)
}
```

### **Zero Value**

```go
func main() {
    var b bool
    fmt.Println(b)  // false (zero value)
    
    // Useful for flags that default to "off"
    type Config struct {
        Debug   bool  // false by default
        Verbose bool  // false by default
    }
    
    cfg := Config{}
    fmt.Println(cfg.Debug)  // false
}
```

### **Comparison Operators**

All comparison operators return `bool`:

```go
func main() {
    a, b := 10, 20
    
    // Equality
    fmt.Println(a == b)  // false
    fmt.Println(a != b)  // true
    
    // Relational
    fmt.Println(a < b)   // true
    fmt.Println(a <= b)  // true
    fmt.Println(a > b)   // false
    fmt.Println(a >= b)  // false
    
    // String comparison
    s1, s2 := "apple", "banana"
    fmt.Println(s1 < s2)   // true (lexicographic)
    fmt.Println(s1 == s2)  // false
}
```

### **Logical Operators**

```go
func main() {
    a, b := true, false
    
    // AND: both must be true
    fmt.Println(a && b)  // false
    fmt.Println(a && a)  // true
    
    // OR: at least one must be true
    fmt.Println(a || b)  // true
    fmt.Println(b || b)  // false
    
    // NOT: inverts the value
    fmt.Println(!a)      // false
    fmt.Println(!b)      // true
    
    // Complex expressions
    x, y, z := 1, 2, 3
    result := (x < y) && (y < z)  // true
    fmt.Println(result)
}
```

### **Short-Circuit Evaluation**

Go uses short-circuit evaluation for `&&` and `||`:

```go
func expensive() bool {
    fmt.Println("expensive() called")
    return true
}

func main() {
    // AND: if first is false, second is not evaluated
    result := false && expensive()  // expensive() NOT called
    fmt.Println(result)  // false
    
    // OR: if first is true, second is not evaluated
    result = true || expensive()    // expensive() NOT called
    fmt.Println(result)  // true
    
    // This is useful for guarding operations
    var ptr *int
    if ptr != nil && *ptr > 0 {  // Safe: *ptr not evaluated if nil
        fmt.Println("Valid pointer with positive value")
    }
}
```

### **Booleans in Control Flow**

```go
func main() {
    isValid := true
    count := 5
    
    // if statement
    if isValid {
        fmt.Println("Valid")
    }
    
    // Condition must be bool (no implicit conversion)
    // if count { }  // Error: non-bool count used as condition
    if count > 0 {   // Must use comparison
        fmt.Println("Has items")
    }
    
    // for loop
    running := true
    iterations := 0
    for running {
        iterations++
        if iterations >= 3 {
            running = false
        }
    }
    
    // switch with bool
    switch isValid {
    case true:
        fmt.Println("Is valid")
    case false:
        fmt.Println("Is invalid")
    }
}
```

### **No Implicit Type Conversion**

```go
func main() {
    n := 1
    
    // These are ERRORS in Go:
    // if n { }              // Error: non-bool n
    // var b bool = n        // Error: cannot use int as bool
    // var b bool = 1        // Error: cannot use 1 as bool
    
    // Must be explicit:
    if n != 0 {
        fmt.Println("n is non-zero")
    }
    
    b := n != 0  // Convert to bool explicitly
    fmt.Println(b)  // true
}
```

### **Boolean Patterns**

#### Toggle

```go
func main() {
    flag := true
    flag = !flag  // Toggle
    fmt.Println(flag)  // false
}
```

#### Default True

```go
type Config struct {
    DisableFeature bool  // false = feature enabled (default)
}

// Alternative with pointer for explicit "unset"
type Option struct {
    Enabled *bool  // nil = use default, non-nil = explicit value
}

func isEnabled(opt *Option) bool {
    if opt.Enabled == nil {
        return true  // Default
    }
    return *opt.Enabled
}
```

#### Boolean Flag Helpers

```go
// Helper to get bool pointer (useful for optional fields)
func boolPtr(b bool) *bool {
    return &b
}

type Request struct {
    Force *bool `json:"force,omitempty"`
}

func main() {
    req := Request{Force: boolPtr(true)}
    fmt.Println(*req.Force)  // true
}
```

### **Formatting Booleans**

```go
func main() {
    b := true
    
    fmt.Printf("%v\n", b)   // true
    fmt.Printf("%t\n", b)   // true (verb for booleans)
    fmt.Printf("%T\n", b)   // bool
    
    // Custom string representation
    status := map[bool]string{
        true:  "enabled",
        false: "disabled",
    }
    fmt.Println(status[b])  // enabled
    
    // strconv
    import "strconv"
    s := strconv.FormatBool(b)
    fmt.Println(s)  // "true"
    
    parsed, _ := strconv.ParseBool("true")
    fmt.Println(parsed)  // true
}
```

### **strconv Boolean Parsing**

```go
import "strconv"

func main() {
    // ParseBool accepts: "1", "t", "T", "true", "TRUE", "True"
    //                    "0", "f", "F", "false", "FALSE", "False"
    
    b1, _ := strconv.ParseBool("true")   // true
    b2, _ := strconv.ParseBool("TRUE")   // true
    b3, _ := strconv.ParseBool("1")      // true
    b4, _ := strconv.ParseBool("T")      // true
    
    b5, _ := strconv.ParseBool("false")  // false
    b6, _ := strconv.ParseBool("0")      // false
    
    _, err := strconv.ParseBool("yes")   // Error: invalid syntax
    fmt.Println(err)
}
```

### **Common Mistakes**

```go
// ✗ Mistake: Comparing bool to true/false
if isValid == true {  // Redundant
    // ...
}

// ✓ Correct: Direct use
if isValid {
    // ...
}

// ✗ Mistake: Negating comparison
if isValid == false {  // Awkward
    // ...
}

// ✓ Correct: Use NOT operator
if !isValid {
    // ...
}

// ✗ Mistake: Returning comparison result
func isPositive(n int) bool {
    if n > 0 {
        return true
    }
    return false
}

// ✓ Correct: Return the expression directly
func isPositive(n int) bool {
    return n > 0
}
```

## Interview Questions

**Q: What is the zero value of a bool in Go?**
**A:** The zero value of `bool` is `false`. This is useful for struct fields where you want a feature to be "off" by default, or flags that should require explicit enabling.

**Q: Why doesn't Go allow `if n { }` where n is an integer?**
**A:** Go does not allow implicit type conversion to bool. This prevents bugs common in C/C++ where `if (n)` can have surprising behavior (e.g., null pointers, empty strings). In Go, you must write `if n != 0 { }` explicitly, making the intent clear.

**Q: What is short-circuit evaluation in Go?**
**A:** For `&&` (AND), if the first operand is `false`, the second is not evaluated. For `||` (OR), if the first operand is `true`, the second is not evaluated. This is used for performance and to guard operations: `if ptr != nil && *ptr > 0` safely avoids dereferencing nil.

**Q: How do you parse a string to bool in Go?**
**A:** Use `strconv.ParseBool(s)`. It accepts "1", "t", "T", "true", "TRUE", "True" as true values, and "0", "f", "F", "false", "FALSE", "False" as false values. It returns an error for any other input like "yes" or "no".
