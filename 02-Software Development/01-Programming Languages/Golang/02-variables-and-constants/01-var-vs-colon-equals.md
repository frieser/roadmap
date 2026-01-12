#Golang
---
---

## Summary

Go provides two ways to declare variables: the `var` keyword and the `:=` short declaration operator. The `var` keyword works everywhere and allows explicit type declaration, while `:=` is shorter but only works inside functions. Understanding when to use each is essential for writing idiomatic Go code.

## Detailed Explanation

### **The var Keyword**

The `var` keyword is the traditional way to declare variables:

```go
// With explicit type
var name string
var age int
var isActive bool

// With initialization
var name string = "Alice"
var age int = 30

// Type inference (type derived from value)
var name = "Alice"  // string inferred
var age = 30        // int inferred

// Multiple variables
var x, y, z int
var a, b, c = 1, 2, "three"

// Grouped declaration (block)
var (
    name    string = "Alice"
    age     int    = 30
    isAdmin bool   = false
)
```

### **The := Short Declaration**

The `:=` operator declares AND initializes a variable with type inference:

```go
func main() {
    name := "Alice"       // var name string = "Alice"
    age := 30             // var age int = 30
    x, y := 10, 20        // Multiple variables
    
    // Type is inferred from the right side
    pi := 3.14            // float64
    count := 100          // int
    active := true        // bool
}
```

### **Key Differences**

| Aspect | `var` | `:=` |
| --- | --- | --- |
| **Scope** | Package or function level | Function level only |
| **Type declaration** | Can be explicit | Always inferred |
| **Without initial value** | Allowed (zero value) | Not allowed |
| **Reassignment** | Use `=` after declaration | Use `=` after declaration |
| **Multiple declarations** | Block syntax supported | Inline only |

### **When to Use var**

#### 1. Package-Level Variables

```go
package main

// ✓ var works at package level
var globalConfig = "default"
var MaxRetries = 3

// ✗ := doesn't work at package level
// name := "Alice"  // Compilation error!

func main() {
    fmt.Println(globalConfig)
}
```

#### 2. Declaring Without Initialization

```go
func process() {
    // Declare now, assign later
    var result string
    var count int
    
    if condition {
        result = "success"
        count = 10
    } else {
        result = "failure"
        count = 0
    }
    
    // ✗ := requires initialization
    // result :=  // Error: expected expression
}
```

#### 3. Explicit Type Needed

```go
func example() {
    // When you need a specific type
    var num int64 = 100       // Explicit int64
    var price float32 = 9.99  // Explicit float32
    
    // := would infer int and float64
    n := 100      // int (not int64)
    p := 9.99     // float64 (not float32)
    
    // Interface types
    var w io.Writer = os.Stdout
    
    // Empty interface for any type
    var data interface{}
    data = "string"
    data = 123
}
```

#### 4. Grouped Declarations

```go
// Clean grouping of related variables
var (
    ErrNotFound   = errors.New("not found")
    ErrTimeout    = errors.New("timeout")
    ErrPermission = errors.New("permission denied")
)

// Configuration block
var (
    host     = "localhost"
    port     = 8080
    debug    = false
    maxConns = 100
)
```

### **When to Use :=**

#### 1. Inside Functions (Most Common)

```go
func calculateTotal(items []Item) float64 {
    total := 0.0  // Short and clean
    
    for _, item := range items {
        price := item.Price * float64(item.Quantity)
        total += price
    }
    
    return total
}
```

#### 2. Error Handling Pattern

```go
func readFile(path string) ([]byte, error) {
    file, err := os.Open(path)  // := for both variables
    if err != nil {
        return nil, err
    }
    defer file.Close()
    
    data, err := io.ReadAll(file)  // := reuses err (see below)
    if err != nil {
        return nil, err
    }
    
    return data, nil
}
```

### **The Redeclaration Rule**

`:=` has a special rule: it can redeclare variables if at least one variable is new:

```go
func example() {
    x := 10
    
    // ✗ Error: no new variables
    // x := 20
    
    // ✓ OK: y is new, x is reassigned
    x, y := 20, 30
    
    // ✓ OK: common pattern with err
    file, err := os.Open("a.txt")
    data, err := io.ReadAll(file)  // err is reassigned, data is new
    
    fmt.Println(x, y, file, data, err)
}
```

### **Common Patterns**

#### Type Conversion

```go
func convert() {
    // Explicit type with var
    var i int64 = 100
    var f float64 = float64(i)
    
    // With :=, use conversion
    n := int64(100)
    price := float32(9.99)
}
```

#### Multiple Return Values

```go
func multiReturn() {
    // := is perfect for multiple returns
    value, ok := someMap["key"]
    result, err := doSomething()
    
    // Discard with blank identifier
    value, _ := someMap["key"]
    _, err := doSomething()
}
```

#### Loop Variables

```go
func loops() {
    // := in for loops
    for i := 0; i < 10; i++ {
        fmt.Println(i)
    }
    
    // Range with :=
    for index, value := range slice {
        fmt.Println(index, value)
    }
}
```

### **Best Practices**

```go
// ✓ Good: := inside functions
func good() {
    name := "Alice"
    count := 0
    result, err := doWork()
}

// ✓ Good: var for package-level
var DefaultTimeout = 30 * time.Second

// ✓ Good: var when type matters
var num int64 = 100

// ✓ Good: var when declaring without value
var result string

// ✗ Avoid: var inside functions when := works
func verbose() {
    var name string = "Alice"  // Use name := "Alice"
}
```

## Interview Questions

**Q: What is the difference between `var` and `:=` in Go?**
**A:** `var` is the full variable declaration keyword that works at both package and function level, allows explicit type declaration, and permits declaration without initialization. `:=` is the short declaration operator that only works inside functions, always requires initialization, and uses type inference. Use `var` for package-level variables or when you need explicit types; use `:=` inside functions for concise code.

**Q: Can you use `:=` at the package level?**
**A:** No. The `:=` operator only works inside functions. At the package level, you must use `var`. This is a compile-time error: attempting `:=` outside a function produces "non-declaration statement outside function body".

**Q: What is the redeclaration rule for `:=`?**
**A:** The `:=` operator can be used when at least one variable on the left side is new. Existing variables are reassigned rather than redeclared. This enables the common error-handling pattern where `err` is reused across multiple calls: `file, err := Open(); data, err := Read(file)`.

**Q: When should you prefer `var` over `:=`?**
**A:** Use `var` when: (1) declaring package-level variables, (2) you need an explicit type different from what would be inferred, (3) declaring a variable without initial value to assign later, or (4) grouping related declarations in a `var ()` block for readability.
